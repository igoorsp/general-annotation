Claro! Aqui está a versão das **perguntas com gabarito de respostas** para servir como base na sua entrevista com o candidato Júnior para backend com Java, Quarkus, Spring Boot e JPA.

---

## 🎯 **Fundamentos de Java**

**1. Qual a diferença entre classe e objeto?**

> Classe é um molde, uma definição de estrutura. Objeto é uma instância da classe, ou seja, algo concreto criado a partir da classe.

**2. O que são modificadores `final`, `static` e `abstract`?**

> * `final`: impede alteração (ex: classe não pode ser estendida, método não pode ser sobrescrito, variável não pode ser alterada).
> * `static`: pertence à classe, e não à instância (objeto).
> * `abstract`: define algo incompleto, que deve ser implementado por subclasses.

**3. O que é polimorfismo e herança?**

> * *Herança* permite que uma classe herde atributos e comportamentos de outra.
> * *Polimorfismo* permite que o mesmo método se comporte de forma diferente dependendo do objeto.

**4. Diferença entre `==` e `.equals()`?**

> `==` compara referências (endereços na memória), enquanto `.equals()` compara os valores dos objetos (quando sobrescrito corretamente).

---

## 🌱 **Spring Boot e Quarkus**

**5. O que é Injeção de Dependência? Como isso acontece no Spring ou Quarkus?**

> É o fornecimento automático de dependências por um framework. No Spring (`@Autowired`) e Quarkus (`@Inject`), o framework cria e injeta os objetos automaticamente.

**6. O que é uma `@RestController` e como ela funciona?**

> É uma anotação do Spring que marca a classe como um controlador REST. Ela trata requisições HTTP e retorna JSON diretamente.

**7. O que significa anotação `@Transactional`?**

> Marca que um método deve ser executado dentro de uma transação. Se houver erro, tudo é revertido.

**8. Para que serve `application.properties` ou `application.yml`?**

> São arquivos de configuração da aplicação: portas, acesso a banco, variáveis de ambiente, etc.

**9. Como você configura uma rota REST no Quarkus?**

> Utiliza a anotação `@Path("/exemplo")` na classe e `@GET`, `@POST` nos métodos, semelhante ao JAX-RS.

**10. Como injeta uma dependência no Quarkus?**

> Utilizando `@Inject` diretamente sobre a variável ou construtor.

---

## 💾 **JPA e Banco de Dados**

**11. O que é uma entidade (`@Entity`)?**

> Uma classe anotada com `@Entity` representa uma tabela no banco de dados.

**12. Como funcionam os relacionamentos (`@OneToMany`, `@ManyToOne`, etc)?**

> Definem como entidades se relacionam:
>
> * `@OneToMany`: um para muitos
> * `@ManyToOne`: muitos para um
> * `@OneToOne`, `@ManyToMany` também existem.

**13. Você já fez um `JOIN` ou usou `JPQL`?**

> (espera-se que o candidato saiba o básico)
> Sim, é possível fazer consultas com relacionamentos usando `JOIN FETCH` em JPQL ou `@Query`.

**14. O que significa `lazy` vs `eager` fetch?**

> * `LAZY`: carrega os dados sob demanda (somente quando acessados).
> * `EAGER`: carrega os dados imediatamente com a entidade principal.

---

## 🔧 **Boas práticas**

**15. Como estruturaria um projeto em camadas (controller, service, repository)?**

> * `Controller`: expõe a API REST.
> * `Service`: contém a lógica de negócio.
> * `Repository`: comunicação com o banco via JPA.

**16. Como trataria exceções em uma API?**

> Usaria `@ControllerAdvice` no Spring para tratar globalmente, ou blocos `try-catch` com retornos de `ResponseEntity`.

**17. O que são status HTTP e quais os mais comuns em APIs REST?**

> Códigos que indicam o resultado da requisição:
>
> * `200 OK`: sucesso
> * `201 Created`: criado com sucesso
> * `400 Bad Request`: erro do cliente
> * `404 Not Found`: recurso não encontrado
> * `500 Internal Server Error`: erro no servidor

---

## 🧩 **Design Patterns**

**18. Já ouviu falar em Design Patterns?**

> (espera-se um "sim" ou curiosidade)
> São soluções reutilizáveis para problemas recorrentes no desenvolvimento de software.

**19. O que é o padrão Singleton?**

> Garante que uma classe tenha apenas uma instância e fornece um ponto global de acesso a ela.

**20. Já usou padrão Repository ou Service?**

> Sim.
>
> * *Repository*: separa a lógica de persistência.
> * *Service*: concentra a regra de negócio.

**21. O que é DTO e por que é usado?**

> DTO (Data Transfer Object) é uma classe usada para transportar dados entre camadas, evitando exposição direta da entidade.

---

## 🧪 **Cenários práticos**

**22. Como criaria uma API para salvar um pedido no banco?**

> Criaria uma entidade `Order`, DTOs de entrada/saída, controller para receber os dados, service para regra de negócio e repository para salvar.

**23. Como trataria um erro de "pedido não encontrado"?**

> Lançaria uma exceção customizada (`OrderNotFoundException`) e trataria com `@ControllerAdvice` para retornar 404.

**24. Como garantiria que dois produtos não tenham o mesmo código?**

> Adicionaria uma restrição `@Column(unique=true)` e validaria antes de salvar.

---

## 💬 **Comportamentais e de aprendizado**

**25. O que faz quando encontra um erro desconhecido?**

> Investigo logs, mensagens de erro, pesquiso no Google/StackOverflow, e se necessário, peço ajuda.

**26. Como se atualiza sobre tecnologias?**

> Leio blogs, vídeos no YouTube, cursos online, documentação oficial.

**27. Está estudando algo novo atualmente?**

> (espera-se uma resposta sincera, como “Spring Boot”, “AWS”, “Clean Architecture” etc.)

**28. Prefere aprender sozinho, em grupo ou com mentorias?**

> (não há resposta certa, mas a resposta mostra o estilo de aprendizado do candidato)

---

Se quiser, posso gerar um **template para entrevista com espaço para anotações**, ou uma **planilha pronta para avaliação por critérios**. Deseja isso?
