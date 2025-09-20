# Documentação Spring Data JPA

Relembrando... As anotações (`@Entity`, `@Id`, `@GeneratedValue`, `@Column`) **não são do Spring em si**, mas da **JPA (Jakarta Persistence API)**. O Hibernate (implementação padrão no Spring Boot Starter JPA) é quem executa essas regras de fato, mas a **especificação** está em Jakarta.

### Onde olhar a documentação oficial:

* **Jakarta Persistence (JPA) Specification**
  [https://jakarta.ee/specifications/persistence/](https://jakarta.ee/specifications/persistence/)
  → É o documento oficial da especificação (antiga JPA do Java EE, agora Jakarta).

* **Javadoc do Jakarta Persistence**
  [https://jakarta.ee/specifications/persistence/3.1/apidocs/](https://jakarta.ee/specifications/persistence/3.1/apidocs/)
  → Aqui você encontra a documentação de cada anotação: `@Entity`, `@Id`, `@GeneratedValue`, `@Column` etc.

* **Hibernate ORM Documentation** (implementação mais usada)
  [https://hibernate.org/orm/documentation/](https://hibernate.org/orm/documentation/)
  → Explica detalhes além da especificação (estratégias de geração de ID, mapeamento avançado, otimizações).

---

* Para a **definição oficial e neutra**, consulte **Jakarta Persistence**.
* Para detalhes práticos (como o Hibernate implementa), use a **documentação do Hibernate**.

---

Para visualizar as operações realizadas no banco de dados. Você pode copiar essas propriedades abaixo no `application.properties`:

```properties
spring.jpa.show-sql=true
spring.jpa.format-sql=true
```

