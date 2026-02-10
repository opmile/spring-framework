# Configurando uma Data Source

---

### O que é uma **DataSource**?

Um **DAO (Data Access Object)** precisa de uma **fonte de dados** para se conectar a um banco. Essa fonte é justamente o que chamamos de `DataSource`, que encapsula as informações de conexão (URL do banco, usuário, senha, driver, pool de conexões etc).

No Java “cru”, você poderia criar isso manualmente com `DriverManager.getConnection(...)`.
Mas no Spring/Spring Boot, você **não cria conexões na unha**. Você injeta um **bean DataSource** e deixa o framework gerenciar isso para você.

---

### Como o Spring Boot te entrega uma `DataSource`

O Spring Boot detecta, a partir do seu `application.properties` (ou `application.yml`), **qual banco você quer usar** e configura automaticamente uma `DataSource`.

Exemplo no `application.properties`:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/minha_base
spring.datasource.username=meu_user
spring.datasource.password=minha_senha
spring.datasource.driver-class-name=org.postgresql.Driver
```

Com isso, o Spring Boot cria e registra no contexto um **bean `DataSource` pronto para uso**.

---

### Tipos de DataSource no Spring Boot

Aqui vem a parte legal: **nem todo DataSource é igual**, e o Spring Boot escolhe automaticamente um dependendo do que está no seu classpath.

* **HikariCP (default)**

  * Desde o Spring Boot 2.x, o **HikariCP** é o pool padrão.
  * É extremamente performático e leve.
  * Se você só colocar a dependência de um driver JDBC, o Spring Boot já monta um `HikariDataSource`.

* **Tomcat JDBC Pool**

  * Pool de conexões mantido pelo time do Tomcat.
  * Não é mais o default, mas você pode usar explicitamente adicionando a dependência.

* **Commons DBCP2**

  * Outro pool de conexões. Mais antigo e mais pesado, raramente usado hoje em dia.

* **DataSource Embedded (H2, HSQLDB, Derby)**

  * Se você **não configura nada** e só coloca uma dessas dependências, o Boot cria automaticamente um banco em memória (muito útil para testes).

* **JNDI DataSource**

  * Em ambientes corporativos, em vez de deixar sua aplicação conhecer usuário/senha, o servidor de aplicação (ex: Tomcat, WebLogic) expõe uma `DataSource` via JNDI, e o Spring Boot pode buscar ela via configuração (`spring.datasource.jndi-name=java:comp/env/jdbc/minhaDataSource`).

---

### Usando a DataSource no DAO

Uma vez que o Boot criou o `DataSource`, você pode injetar e usar no seu DAO.

Exemplo com `JdbcTemplate` (wrapper do Spring para JDBC):

```java
@Repository
public class ProdutoDAO {

    private final JdbcTemplate jdbcTemplate;

    public ProdutoDAO(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<Produto> listar() {
        return jdbcTemplate.query("SELECT * FROM produto",
            (rs, rowNum) -> new Produto(rs.getLong("id"), rs.getString("nome")));
    }
}
```

Aqui, você não mexeu com `DataSource` diretamente, mas o `JdbcTemplate` internamente usa o `DataSource` configurado pelo Boot.

Se quisesse injetar direto:

```java
@Autowired
private DataSource dataSource;
```

---

### Diferenciando as fontes de dados

Resumindo:

| Tipo de DataSource             | Quando usar                                                             |
| ------------------------------ | ----------------------------------------------------------------------- |
| **HikariCP** (default)         | Melhor opção para produção, rápido e eficiente.                         |
| **Tomcat JDBC Pool**           | Quando já existe legado ou necessidade de integração com Tomcat nativa. |
| **DBCP2**                      | Cenários legados ou compatibilidade.                                    |
| **Embedded (H2, HSQL, Derby)** | Testes, prototipagem, dev rápido.                                       |
| **JNDI DataSource**            | Ambientes corporativos/servidores de aplicação.                         |

---

* Todo DAO precisa de um **DataSource**.
* No Spring Boot, você raramente cria um DataSource manualmente: o Boot gera automaticamente com base nas dependências + `application.properties`.
* Você só precisa **dizer “qual banco quero”**. O resto (pool, driver, objeto) vem de graça.

---

### Auto-configuração do Spring Boot

O Spring Boot funciona em cima do conceito de **Auto Configuration**.

* Ele inspeciona o **classpath** do projeto, as dependências e as propriedades definidas (`application.properties` ou `application.yml`).
* A partir disso, decide **quais beans criar automaticamente**.

Para `DataSource`, o ponto-chave é a classe `DataSourceAutoConfiguration` (que faz parte do **spring-boot-autoconfigure**).

* Ela olha se existe **alguma classe de driver JDBC no classpath** (`org.postgresql.Driver`, `com.mysql.cj.jdbc.Driver` etc).
* Ela verifica se não existe **nenhum bean DataSource manual definido**.
* Se essas condições forem verdadeiras, o Boot cria um `DataSource` automaticamente.

---

### Escolha do pool de conexões

Spring Boot não cria um DataSource “cru” do JDBC, ele escolhe um **pool de conexões** para gerenciar as conexões com o banco:

* **HikariCP** → default desde Spring Boot 2.x.
* **Tomcat JDBC Pool** → usado se Hikari não estiver disponível, mas dependência estiver presente.
* **DBCP2** → só se nenhuma das anteriores estiver e você adicionar explicitamente.

O Boot configura esse pool **com base nas propriedades**: URL, username, password, tamanho máximo do pool, timeout, etc.

Exemplo de propriedades que o Boot usa para configurar Hikari:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/meubanco
spring.datasource.username=user
spring.datasource.password=senha
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=30000
```

O Spring Boot transforma essas configurações em **um bean `HikariDataSource` pronto**, gerenciando a criação e liberação de conexões para você.

---

### Registro do DataSource no contexto do Spring

Depois de criar o `DataSource`, o Boot **registra ele como um bean do Spring**.

* Isso significa que qualquer componente do seu projeto pode injetar `DataSource` (ou `JdbcTemplate`, `EntityManagerFactory`, etc.) sem precisar criar nada manualmente.
* O Boot também cuida do **ciclo de vida do DataSource**:

  * Inicializa conexões quando a aplicação sobe.
  * Fecha conexões quando a aplicação é finalizada.

---

### Beans relacionados que o Boot cria automaticamente

Quando você usa JPA/Hibernate com Spring Boot, além do DataSource, ele também cria automaticamente:

* `JdbcTemplate` (para consultas JDBC simplificadas)
* `EntityManagerFactory` (para JPA/Hibernate)
* `PlatformTransactionManager` (para gerenciar transações)

Tudo isso é possível **porque o DataSource já está disponível como bean**, e o Boot liga as pontas automaticamente.

---

1. Spring Boot detecta driver e propriedades → decide que precisa de DataSource.
2. Escolhe o pool de conexões disponível (Hikari, Tomcat, DBCP2).
3. Cria o DataSource como bean e configura com suas propriedades.
4. Gerencia o ciclo de vida das conexões automaticamente.
5. Outros beans (JdbcTemplate, EntityManager) usam esse DataSource automaticamente.

---





