# Vocabulário 

## Pool de Conexões

Em vez de abrir/fechar uma conexão ao banco cada vez que você precisa, o sistema mantém um "pool" (um conjunto e conexões já abertas) que ficam disponíveis para reutilização

* Abrir conexão = caro

* Reusar = muito mais eficiente

O Spring Boot usa por padrão o HikariCP como pool de conexões. Você configura parâmetros como `maximumPoolSize`, `idleTimeout`, etc. no `application.properties`.

Cada vez que sua aplicação precisar de uma conexão, ela pega uma do pool. Quando terminar, devolve. Isso aumenta desempenho e escalabilidade.

---

## Driver

É um software (biblioteca) que permite que uma aplicação Java "fale" com um banco de dados específico.

Funciona como uma ponte para permitir que uma aplicação se comunique com um sistema de gerenciamento de banco de dados (SGBD) específico. Ele traduz os comandos e consultas da aplicação para um formato que o SGBD entenda e vice-versa, facilitando a troca de dados e a execução de operações de forma padronizada. 

* `mysql-connector-j` (MySQL)

* `postgresql-42.x` (Postgres)

Você adiciona a dependência no pom.xml (Maven) ou build.gradle. Sem o driver, a aplicação nem sabe como se conectar ao banco.

## Sistema Gerenciador de Banco de Dados (SGBD)

O software responsável por gerenciar o banco em si: armazenar, processar, aplicar transações, controlar concorrência.

* MySQL, PostgreSQL, Oracle DB, SQL Server.

O Spring Boot não lida diretamente com o disco onde os dados estão. Ele conversa com o SGBD via driver. O SGBD é quem realmente executa a query e retorna os resultados.

## DataSource

É uma abstração que representa a fonte de dados (o "caminho" até o banco). No Spring, o DataSource é quem sabe como obter conexões (seja direto, seja via pool).

Quando você configura (no `.properties` ou `.yml`) `spring.datasource.url`, `spring.datasource.username` e `spring.datasource.password`, o Spring cria automaticamente um DataSource.

## Entity

Uma classe Java que representa uma tabela no banco.
Marcada com `@Entity` e geralmente com `@Table(name = "minha_tabela")`.

Com Spring Data JPA, cada entidade (classe) corresponde a uma linha de uma tabela — e você manipula objetos em vez de SQL direto.

## Repository

Um DAO, Interface no Spring Data que abstrai as operações de CRUD.

Ex.: `UserRepository extends JpaRepository<User, Long>`

Se você escolhe não usar JDBC, você não escreve SQL: só chama repository.findAll(), save(), etc., e o Spring gera as queries.

## ORM (Object-Relacional Mapping)

Técnica que mapeia objetos Java ↔ tabelas no banco.

* O Spring Boot normalmente usa Hibernate (implementação JPA).

Permite escrever menos SQL manual, delegando a persistência ao framework.

## JPA (Java Persistence API)

Uma especificação que define como objetos Java devem ser persistidos em bancos relacionais.

* Não é uma implementação, mas um contrato.

* Implementação mais usada: Hibernate

Quando você usa `@Entity`, `@Repository`, `@PersistenceContext`, isso é JPA.
O Hibernate cuida do “trabalho pesado” por baixo.

## Transação

Um conjunto de operações que devem ser executadas como uma unidade atômica (ou tudo acontece, ou nada).

No Spring, você anota métodos com `@Transactional`. O Spring garante que a conexão ao banco seja usada dentro de uma transação consistente.

## Dialect

Um “tradutor” do Hibernate para o SQL específico do seu SGBD.

* Ex.: `org.hibernate.dialect.PostgreSQLDialect`.

Mesmo que você use JPA, o Hibernate precisa saber “qual sotaque de SQL” falar.

## Como tudo se conecta

1. Você configura o DataSource (com `url`, `username`, `password` e **driver**).

2. O `DataSource` usa um **pool de conexões** (ex.: HikariCP).

3. O **Driver** conecta sua aplicação ao **SGBD**.

4. O **SGBD** executa as queries.

5. Se estiver usando **JPA/Hibernate**:

    * Suas **Entities** representam tabelas.

    * Os **Repositories** geram queries automaticamente.

    * O Hibernate traduz suas operações em SQL usando um **Dialect**.

    * O Spring gerencia **Transações** para garantir consistência.




