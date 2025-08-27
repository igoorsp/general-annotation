# Guia de Comunicação entre Lambdas (API Gateway vs Invoke)

> **Objetivo:** fornecer um material de arquitetura, com argumentos técnicos e de negócio, para decidir quando expor Lambdas via **API Gateway (HTTP)** e quando utilizar **invocação direta (`InvokeFunction`)** — cobrindo segurança, performance, custo, resiliência, rede (VPC Endpoints), observabilidade e padrões **síncronos** e **assíncronos**.

---

## 1) Sumário Executivo (TL;DR)
- **Lambda→Lambda interno:** prefira **`InvokeFunction` com IAM** e **VPC Endpoint (PrivateLink)** do serviço **Lambda**. Evite HTTP interno.
- **API Gateway:** use quando B **precisa ser um serviço HTTP** para **múltiplos clientes** (web/mobile/terceiros) e você precisa de **features de gateway** (autenticação pública/JWT, rate‑limit, WAF, mTLS, domínios). Para consumo apenas de A, **é overhead**.
- **Síncrono vs Assíncrono:** se A **não precisa da resposta imediata**, prefira **assíncrono** (Invoke `Event`/SQS/EventBridge) — menor latência percebida, custo e acoplamento. Se precisar resposta, mantenha **síncrono** com timeouts/backoff/idem.
- **Orquestração:** cadeias de 3+ etapas ou com regras/compensações → **Step Functions (Standard)**.

---

## 2) Contexto & Premissas
- Lambdas em **subnets privadas**; acesso a serviços AWS via **VPC Endpoints (Interface/Gateway)**.
- Possibilidade de **cross‑account** para B/C.
- Alguns fluxos exigem resposta imediata; outros podem ser **eventuais**.
- Métrica de latência: **p95/p99** (planejar timeouts por percentis, não pela média).

---

## 3) Por que **não** usar API Gateway para Lambda→Lambda interno (HTTP síncrono)
### 3.1 Overhead de latência & limites
- Hop extra (**HTTP**) adiciona serialização/headers/camadas de rede.
- **Timeout do API Gateway** é limitado (ordem de dezenas de segundos) — operações mais longas falham na borda mesmo que a Lambda conclua internamente.

### 3.2 Custo adicional
- Você paga **requests do API Gateway** **+** execução da Lambda. Em Invoke direto, não há tarifa de gateway.

### 3.3 Falta de back‑pressure
- HTTP não provê buffer nativo. Sob pico, você tem erro em cascata (timeout/throttle). Com assíncrono (SQS), há **fila/DLQ** e **controle de vazão**.

### 3.4 Complexidade operacional
- Autenticação/Autorização duplicada: token na borda **e** internamente.
-
- Gestão de **domínio/ACM**, **WAF**, **usage plans**, **resource policy** — úteis para público externo, mas desnecessários para s2s interno.

### 3.5 Superfície de ataque
- Mesmo com API privada, há mais **objetos** para proteger (API, rotas, autorizer, VPC Endpoint `execute-api`).

> **Conclusão:** HTTP interno entre Lambdas é **anti‑pattern** na maioria dos casos. Eleva latência, custo e superfície de ataque sem entregar valor quando o único cliente é a Lambda A.

---

## 4) Quando **faz sentido** usar API Gateway
- B é um **serviço HTTP** consumido por **múltiplos clientes** (apps, terceiros, parceiros).
- Precisa de **features de gateway**: autenticação pública (Cognito/JWT), rate limiting/quotas, WAF, mTLS, validação de schema/headers, **domínio customizado**.
- **Privado** de verdade: **HTTP API Private** + **VPC Endpoint `execute-api`** + **resource policy** restringindo `aws:SourceVpce`/conta.

> A mensagem: **API GW serve a borda**. Para tráfego interno restrito, **Invoke direto** é mais simples/seguro/barato.

---

## 5) Por que **`InvokeFunction`** é o padrão recomendado
### 5.1 Segurança
- **IAM** com *least privilege* no **caller** (Lambda A) + **resource‑based policy** no **callee** (Lambda B/C).
- **VPC Endpoint (Interface) `lambda`** com **Private DNS** → tráfego **privado** (sem NAT/Internet), controlado por **SG**.
- Identidade de usuário: termine o **JWT** no API GW/authorizer e propague apenas um **envelope de identidade** assinado (HMAC/KMS) com claims mínimas (Zero Trust), evitando HTTP interno para “repassar token”.

### 5.2 Performance
- Menos hops, menos serialização; **latência menor**.
- Evita limite de timeout do API GW.
- A única latência é a da **B** (e de C, se B→C), sem overhead de gateway.

### 5.3 Custo
- Sem custo por requisição de gateway.
- Em cadeias, **Invoke assíncrono** reduz o tempo que A “paga esperando B”.

### 5.4 Operação & Resiliência
- Menos componentes = menos pontos de falha.
- **Aliases/versions** (A chama `LambdaB:prod`) simplificam canary/rollback.
- **Reserved Concurrency** para isolar capacidade; **Provisioned Concurrency/SnapStart** para p95 sensível a cold start.

---

## 6) Síncrono vs Assíncrono com `InvokeFunction`
### 6.1 Conceitos
- **Síncrono** (`InvocationType: RequestResponse`): A **espera** B responder.
- **Assíncrono** (`InvocationType: Event`): A **não espera**; B processa depois (configure DLQ/retry autom.).

### 6.2 Efeitos práticos (exemplo de 500 ms por etapa)
- **2 Lambdas (A→B)**
  - **Síncrono**: latência percebida ≈ **1,0 s**; tempo cobrado somado ≈ **1,5 s** (A espera B)
  - **Assíncrono**: latência ≈ **0,5 s** (A responde já); cobrado ≈ **1,0 s**
- **3 Lambdas (A→B→C)**
  - **Síncrono**: latência ≈ **1,5 s**; cobrado ≈ **3,0 s**
  - **Assíncrono**: latência ≈ **0,5 s**; cobrado ≈ **1,5 s**

> **Resumo:** assíncrono **diminui latência do chamador** e **reduz GB‑s pagos** (A não fica ociosa aguardando B/C).

### 6.3 Quando usar cada um
- **Síncrono:** A **precisa** da resposta de B para completar a operação do cliente, e o tempo é **curto/estável**.
- **Assíncrono:** processamento em background; **picos** e **falhas transitórias** precisam de **buffer/DLQ**; integrações de longa duração.
- **Orquestração:** 3+ etapas, paralelos/compensações → **Step Functions (Standard)**.

---

## 7) Rede (VPC) & Endpoints
- Crie **Interface VPC Endpoint** para `lambda` (obrigatório para invocar sem Internet) e habilite **Private DNS**.
- Endpoints auxiliares (conforme uso): `logs`, `xray`, `kms`, `secretsmanager`, `ecr.api`, `ecr.dkr`, `sts`.
- **Security Groups**: permitir **443** da ENI da Lambda A → SG do endpoint `lambda`.
- **Cross‑account**: o caminho de rede não muda; garanta **IAM + resource policy** em B/C.

---

## 8) Segurança (detalhes práticos)
- **Caller (Role da A):** permitir apenas `lambda:InvokeFunction` no **ARN do alias** da B/C.
- **Callee (B/C):** resource‑based policy permitindo o principal (role de A) — **obrigatório** em cross‑account.
- **Identidade do usuário:** envelope assinado (HMAC/KMS) com claims mínimas e TTL curto (evita repasse de JWT bruto e dependência de JWKS em cada salto).
- **Zero Trust interno:** checagens de autorização no B/C usando atributos (tenant, scopes, roles) provenientes do envelope.

---

## 9) Observabilidade & Resiliência
- **X‑Ray** em A/B/C; propague **correlationId**.
- **Timeouts**: `timeout(B) < timeout(A)`; em cadeia, cada estágio com folga.
- **Retries** com **backoff exponencial**; evite retries duplicados (escolha um ponto da cadeia para retentar).
- **Idempotência** (chave de negócio/idempotencyKey) para reprocessos seguros.
- **DLQ** para assíncrono (SQS/EventBridge/Lambda async DLQ).

---

## 10) Custos (visão relativa)
- **API Gateway** agrega custo por requisição e, em HTTP síncrono, mantém A **aguardando** B (mais GB‑s).
- **Invoke direto** elimina a tarifa do gateway e reduz latência; **assíncrono** elimina a espera paga de A.
- **Memória**: 512 MB dobra GB‑s vs 256 MB, mas reduz duração (mais CPU). Perfilar é essencial: se a duração cai ~metade, custo fica próximo com **latência menor**.

---

## 11) Matriz de decisão (resumo)

| Critério | API Gateway (HTTP) | Invoke Síncrono | Invoke Assíncrono | Step Functions |
|---|---|---|---|---|
| Clientes múltiplos/externos | **Forte** | Fraco | Fraco | Médio |
| Features de gateway (WAF, JWT, mTLS, domínio) | **Forte** | N/A | N/A | N/A |
| Latência baixa interna | Médio (overhead HTTP) | **Forte** | **Forte** (para A) | Médio |
| Custo | Maior (gateway + Lambda) | Menor | **Menor** | Depende (transições) |
| Resiliência a picos | Baixa (sem buffer) | Média | **Alta** (com SQS/EB) | Alta |
| Orquestração, retries por etapa | Fraco | Fraco | Médio | **Forte** |
| Segurança s2s privada | Médio (Private API + VPCe) | **Forte** | **Forte** | **Forte** |

---

## 12) Playbooks & Checklists
**Invoke (privado) — obrigatório:**
- [ ] VPC Endpoint **`lambda`** com **Private DNS**
- [ ] SGs (A → endpoint 443)
- [ ] Role de A com `lambda:InvokeFunction` no **alias** de B/C
- [ ] Resource policy em B/C (principal = role de A; cross‑account)
- [ ] X‑Ray + correlationId; timeouts coerentes; retries/backoff
- [ ] Idempotência; DLQ para assíncrono; Reserved/Provisioned Concurrency conforme SLA

**API Gateway — quando necessário:**
- [ ] HTTP API (preferível) público/privado conforme caso
- [ ] Autenticação (IAM/JWT) e **resource policy** (se privado, `aws:SourceVpce`)
- [ ] Domínio/ACM/CloudFront (se público)
- [ ] Logs/metrics; alarmes de 4xx/5xx/latência; proteção WAF

---

## 13) Exemplos (esqueleto)
**IAM (Role da A):**
```json
{
  "Effect": "Allow",
  "Action": "lambda:InvokeFunction",
  "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:LambdaB:*"
}
```

**Resource‑based policy (B):**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "AllowInvokeFromA",
    "Effect": "Allow",
    "Principal": { "AWS": "arn:aws:iam::ACCOUNT:role/RoleDaLambdaA" },
    "Action": "lambda:InvokeFunction",
    "Resource": "arn:aws:lambda:REGION:ACCOUNT:function:LambdaB:*"
  }]
}
```

**Envelope de identidade (conceito):**
- `identity`: `{ sub, scopes[], tenantId, iat, exp }` (claims mínimas)
- `signature`: HMAC/KMS sobre `identity`
- B valida assinatura/expiração e aplica autorização (ABAC/RBAC)

---

## 14) Riscos & Mitigações
- **Cadeias síncronas longas:** risco de latência e timeouts → usar **assíncrono** ou **Step Functions**.
- **Cold starts sensíveis:** **Provisioned Concurrency** (ou **SnapStart** para Java) nos caminhos críticos.
- **Throttling/downstream lento:** **Reserved Concurrency** e filas (SQS) para controle de vazão.
- **JWT em cada salto:** prefira envelope assinado com TTL curto; evite baixar JWKS em todo serviço.

---

## 15) Recomendação
1) **Padrão**: **API Gateway → Lambda A** (borda) **+** **A → B/C via `InvokeFunction`** usando **VPC Endpoint** e IAM.
2) **Assíncrono** sempre que possível (menos latência percebida e custo).
3) **Step Functions** para fluxos com 3+ etapas, retries/compensações e rastreabilidade.
4) **API Gateway** para B/C **apenas** quando eles precisarem ser APIs HTTP para múltiplos clientes ou exigir features específicas de gateway.

> Este desenho minimiza custo/latência/superfície de ataque, maximiza resiliência e mantém flexibilidade para crescer (orquestração/assíncrono) sem refatorações profundas.

