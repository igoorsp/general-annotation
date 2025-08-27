Claro! Aqui está um documento estruturado em nível arquitetônico, perfeito para compartilhar com times de arquitetura. Ele resume os principais **anti‑patterns** mencionados na documentação da AWS e artigos confiáveis, incluindo explicações, riscos e alternativas recomendadas.

---

## Documentação de Arquitetura: Anti-Patterns em Arquiteturas Event-Driven com AWS Lambda

### 1. Contexto

Nesta documentação, analisamos anti‑patterns comuns em arquiteturas serverless event‑driven usando AWS Lambda — conforme descrito na documentação oficial da AWS e corroborado por artigos especialistas. O objetivo é oferecer argumentos sólidos para decisões arquiteturais e orientar equipes de desenvolvimento.

---

### 2. Anti-Patterns Identificados (com base na AWS)

Extraído da seção *Anti‑patterns in Lambda‑based event‑driven applications* da AWS Developer Guide:

* \*\*Monólito de Lambda ("Lambda Monolith")\*\*: função única que implementa toda a lógica da aplicação (várias rotas + integrações).
  **Riscos**: pacote grande, difícil manutenção, deploy arriscado, teste complicado, IAM excessivamente permissivo.
  **Recomendação**: decompor em micro-Lambdas, cada uma com lógica clara. ([Amazon Web Services, Inc.][1], [Amazon Web Services, Inc.][2], [Documentação AWS][3])

* **Padrões recursivos infinitos (“run‑away”)**: função que dispara evento via S3/SNS/EventBridge, que re-invoca a mesma função, gerando loop.
  **Riscos**: escalabilidade infinita, consumo ilimitado de recursos, alto custo, dificuldade de interrupção.
  **Recomendação**: usar SQS como buffer intermediário ou lógica para evitar recursão “auto‑trigger”. ([Documentação AWS][3])

* **Esperas síncronas dentro de uma Lambda**: executar tarefas sequenciais que poderiam ser paralelas (e.g. S3 → DynamoDB).
  **Riscos**: latência somada, custo elevado por função mais longa.
  **Recomendação**: separar em funções ativadas por eventos (ex. após o upload S3, outra Lambda processa e grava no DynamoDB). ([Amazon Web Services, Inc.][2])

---

### 3. Anti-Patterns Complementares (fontes externas)

Apesar de não estarem diretamente na página da AWS, esses padrões são amplamente reconhecidos como anti‑patterns em arquiteturas serverless:

* **Lambda chamando Lambda diretamente**

  * **Descobertas**: acoplamento forte, aumento de latência, dificuldade de observabilidade, custo duplicado.
  * **Alternativas**: SQS/SNS (eventos), Step Functions (orquestração). ([Orchestra][4])

* **“Lambda Pinball” ou “Grain of Sand”**

  * Chamadas encadeadas entre muitas Lambdas minúsculas, tornando o fluxo fragmentado e difícil de rastrear.
  * **Riscos**: alta latência, operação complexa, baixa observabilidade.
  * **Recomendação**: definir domínios estáveis com interfaces bem versionadas. ([Wikipedia][5])

* **Esperas síncronas prolongadas/futuras tarefas longas na mesma Lambda**

  * Exemplo: polling repetitivo de API externa dentro da Lambda.
  * **Riscos**: sobrecarga, custo, escalabilidade e timeout de 15 min.
  * **Recomendação**: adotar eventos (SNS/SQS) para orquestração. ([Stack Overflow][6], [Orchestra][7], [Medium][8])

---

### 4. Resumo em Tabela

| Anti-Pattern                                | Descrição                                               | Riscos Principais                                       | Alternativas Recomendadas                                      |
| ------------------------------------------- | ------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------------------- |
| Lambda Monolith                             | Função com lógica consolidada, várias responsabilidades | Deploy arriscado, pacotes grandes, manutenção difícil   | Decompor em micro-Lambdas, cada com responsabilidade única     |
| Função recursiva via trigger (S3 loop)      | AWS evento desencadeia a própria função                 | Loop infinito, consumo de recursos, custo descontrolado | Usar SQS para desacoplar, evitar triggers recursivos           |
| Espera síncrona interna                     | Tarefas independentes executadas em sequência           | Latência e custo somados                                | Paralelizar com Lambdas independentes (S3 → Lambda → DynamoDB) |
| Lambda invocando Lambda sequencialmente     | Funções encadeadas por InvokeFunction síncrono          | Acoplamento, latência, custo duplicado                  | Usar eventos (SNS/SQS) ou Step Functions                       |
| Micro-chaining excessivo (“lambda pinball”) | Muitas Lambdas encadeadas em mini-funções               | Alta complexidade, rastreabilidade ruim                 | Agrupar lógica, manter contratos claros                        |
| Polling síncrono interno                    | Lambda aguardando repetidamente resultado externo       | Tempo de execução longo, custo elevado, menos escalável | Eventos assincrônicos, orquestração via Step Functions         |

---

### 5. Recomendações Arquiteturais

1. **Evite Lambdas monolíticas**

   * Divida responsabilidades, utilize micro-Lambdas desacopladas.

2. **Evite auto-invocação recursiva**

   * Utilize filas ou lógica condicional para prevenir loops.

3. **Paralelize tarefas independentes**

   * Transforme chamadas sequenciais em eventos acionadores.

4. **Não encadeie Lambdas diretamente**

   * Prefira fila (SQS/SNS) ou orchestration (Step Functions) para flexibilidade, observabilidade e retries.

5. **Simplifique pipelines com menos funções ou com Step Functions**

   * Menor complexidade e melhor monotoração.

6. **Evite polling síncrono**

   * Desacople via eventos para permitir escalabilidade e reduzir latência.

---

### 6. Conclusão

Adotar arquiteturas event-driven com Lambdas é poderoso, mas exige consciência de anti-patterns. Documentar e evitar os casos listados é essencial para garantir:

* **Manutenabilidade**
* **Escalabilidade**
* **Segurança e Resiliência**
* **Custo otimizado**

Se quiser, posso transformar este conteúdo no formato do documento que criamos anteriormente (com seções visuais, checklist, exemplos de arquitetura), ou preparar um PDF estruturado com diagramas comparativos.

[1]: https://aws.amazon.com/blogs/compute/operating-lambda-anti-patterns-in-event-driven-architectures-part-3/?utm_source=chatgpt.com "Operating Lambda: Anti-patterns in event-driven ..."
[2]: https://aws.amazon.com/blogs/compute/implementing-aws-lambda-error-handling-patterns/?utm_source=chatgpt.com "Implementing AWS Lambda error handling patterns"
[3]: https://docs.aws.amazon.com/lambda/latest/dg/concepts-event-driven-architectures.html?utm_source=chatgpt.com "Creating event-driven architectures with Lambda"
[4]: https://www.getorchestra.io/guides/lambda-anti-pattern-functions-calling-functions?utm_source=chatgpt.com "Lambda Anti-Pattern: Functions Calling Functions"
[5]: https://en.wikipedia.org/wiki/Serverless_computing?utm_source=chatgpt.com "Serverless computing"
[6]: https://stackoverflow.com/questions/68234638/how-to-use-event-driven-architecture-to-remove-api-based-lambda-calling-another?utm_source=chatgpt.com "How to use Event-Driven architecture to remove \"api-based ..."
[7]: https://www.getorchestra.io/guides/anti-pattern-in-aws-lambda-synchronous-waiting-within-a-function?utm_source=chatgpt.com "Anti-Pattern in AWS Lambda: Synchronous Waiting within ..."
[8]: https://medium.com/%40gedeon.dominguez/the-serverless-trap-anti-patterns-in-aws-lambda-and-azure-functions-540c903c4513?utm_source=chatgpt.com "The Serverless Trap: Anti-Patterns in AWS Lambda and ..."
