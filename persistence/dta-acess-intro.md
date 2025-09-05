# Acesso a Dados com Spring

Toda aplicação corporativa precisa acessar e escrever dados em alguma fonte de dados. De cara, o Spring nos oferece suporte às principais tecnologias de persistência adotadas por desenvolvedores Java.

Há suporte para JDBC, ORMs como Hibernate, iBatis, JPA, JDO e OXM, e esse suporte se da através de *templates*.

---

## JDBC (Java Database Connectivity)

É a API nativa do Java para se conectar a bancos relacionais.

* Você trabalha diretamente com SQL puro

Fluxo típico:

1. Abrir conexão com o banco (Connection).

2. Criar instrução SQL (PreparedStatement).

3. Executar e iterar resultados (ResultSet).

4. Fechar conexão e recursos manualmente.

Problemas do JDBC puro:

* Muito boilerplate

* Baixo nível (você cuida de conexões, commits, rollbacks)

* Difícil de manter quando a aplicação cresce

```java
Connection conn = DriverManager.getConnection(url, user, password);
String sql = "SELECT * FROM aluno WHERE id = ?";
PreparedStatement stmt = conn.prepareStatement(sql);
stmt.setInt(1, 1);
ResultSet rs = stmt.executeQuery();

while (rs.next()) {
    System.out.println(rs.getString("nome"));
}

rs.close();
stmt.close();
conn.close();
```

## ORM (Object-Relational-Mapping) - ex.: Hibernate, JPA

ORM é uma abstração em cima do JDBC.

Ele faz o mapeamento objeto-relacional, ou seja:

* Uma classe Java ↔ Uma tabela no banco.

* Um objeto ↔ Uma linha da tabela.

Você escreve código orientado a objetos, e o ORM cuida da tradução para SQL.

**Vantagens**:

* Muito menos código repetitivo.

* Não precisa escrever a maior parte do SQL.

* Gerencia conexões, cache, transações, lazy loading.

* Mais próximo da lógica de negócio → menos fricção entre mundo OO e relacional.

ex) Hibernate (via JPA)

```java
@Entity
public class Aluno {
    @Id
    private int id;
    private String nome;
    // getters e setters
}

// Uso
Aluno aluno = entityManager.find(Aluno.class, 1);
System.out.println(aluno.getNome());
```

## JDBC x ORMs

| **JDBC**                                           | **ORM (Hibernate/JPA)**                                                   |
| -------------------------------------------------- | ------------------------------------------------------------------------- |
| API nativa, baixo nível                            | Framework de abstração, alto nível                                        |
| Precisa escrever SQL manualmente                   | SQL é gerado automaticamente a partir de objetos                          |
| Código verboso, repetitivo                         | Código mais limpo, focado em lógica de negócio                            |
| Controle total sobre a query                       | Menos controle fino (embora seja possível usar `@Query` ou `NativeQuery`) |
| Mais rápido em cenários simples (sem overhead)     | Mais produtivo em sistemas complexos                                      |
| Requer gerenciar recursos e transações manualmente | ORM gerencia conexões, cache, transações                                  |

JDBC é como falar direto com o banco em “língua SQL crua”. ORM é como ter um tradutor automático entre seu mundo Java e o banco — você fala em objetos, ele cuida do SQL.

### ORM: Hibernate via JPA com Spring Data JPA

JPA (Java Persistence API) é uma especificação, ou seja, um contrato definido pela Oracle/Java EE (agora Jakarta EE). Ela não tem implementação própria: só diz como deve ser a API para persistência de objetos.

* Define anotações (@Entity, @Id, @OneToMany, etc.) e interfaces (EntityManager, Query, etc.).

* O código que você escreve usando somente JPA é portável: você pode trocar a implementação de ORM sem mudar a lógica.

```java
@Entity
public class Aluno {
    @Id
    private Long id;
    private String nome;
}
```

* Só usei JPA. Agora, para de fato persistir, preciso de uma implementação da JPA.

### Hibernate

Hibernate é um framework ORM concreto, ou seja, uma implementação (a mais popular) da especificação JPA. Ele adiciona extensões próprias além do JPA, como cache avançado, lazy loading mais flexível, criteria API própria, etc.

* Hibernate implementa JPA, mas você também pode usar APIs específicas dele (não-portáveis).

```java
Session session = sessionFactory.openSession();
Aluno aluno = session.get(Aluno.class, 1L);
```

* Esse código usa um recurso específico e é Hibernate nativo, não é JPA

### Spring Data JPA

* No código da aplicação, use JPA (interfaces, anotações)

* No `pom.xml` inclua o Hibernate como sua implementação do JPA

O Spring Data JPA faz exatamente isso: você usa JPA, mas por baixo dos panos ele usa Hibernate por padrão (ou outra implementação se quiser trocar)

* JPA -- Especificação

* Hibernate -- Implementação

---

### Templates

Templates no Spring são pontos de acesso a dados que abstraem a verbosidade das APIs originais, simplificando integrações com bancos e recursos externos.

São classes utilitárias prontas que encapsulam a verbosidade e repetição de código (boilerplate) da API original.

* Fazem parte do programming model do Spring para acesso a recursos externos

ex) `JdbcTemplate`: com JDBC puro, um select fica enorme: abrir conexão, criar statement, executar query, percorrer ResultSet, fechar tudo.

Com `JdbcTemplate`

```java
@Autowired
private JdbcTemplate jdbcTemplate;

public List<String> findNomes() {
    return jdbcTemplate.query(
        "SELECT nome FROM aluno",
        (rs, rowNum) -> rs.getString("nome")
    );
}
```

* Abre e fecha a conexão automaticamente

* Trata `SQLException` e converte em exceções unchecked do Spring (`DataAcessException`)

* Permite mapear resultados via lambda ou RowMapper

Os templates foram o primeiro nível da abstração do Spring, depois veio o Spring Data, que deu um salto ainda maior: ao invés de usar `JdbcTemplate` ou `HibernateTemplate`, você só define uma interface `Repository` e o Spring implementa o acesso.

* Templates = nível mais baixo (você escreve queries, mas com menos boilerplate).

* Spring Data = nível mais alto (o Spring até escreve as queries pra você, baseado em convenções).

