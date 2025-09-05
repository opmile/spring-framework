**Trilha de aprendizado guiada por perguntas** (prompts). Se você me fizer cada um deles na sequência, vai fechar o ciclo completo de como conectar e usar bancos de dados em Spring Boot.

---

(ordem sugerida para que seu aprendizado seja progressivo)

### 🔹 Etapa 1 – Fundamentos

1. **"O que é uma fonte de dados no Spring Boot e como o framework cria e gerencia DataSources automaticamente?"**
2. **"Quais são as formas de configurar um DataSource no Spring Boot (application.properties, JNDI, múltiplas fontes) e quando usar cada uma?"**
3. **"Quais são as diferenças práticas entre configurar banco de dados no `application.properties` e via JNDI em servidores de aplicação?"**

---

### 🔹 Etapa 2 – Conexão e Configuração

4. **"Como configurar minha aplicação Spring Boot para se conectar a um banco de dados PostgreSQL (ou outro banco que eu escolher)?"**
5. **"O que é um pool de conexões e como o Spring Boot utiliza HikariCP por padrão?"**
6. **"Como configurar múltiplas conexões com bancos de dados diferentes no mesmo projeto Spring Boot?"**

---

### 🔹 Etapa 3 – Acesso a Dados no Spring

7. **"Quais são as opções de acesso a dados que o Spring Boot oferece (JdbcTemplate, Spring Data JPA, MyBatis, etc.)?"**
8. **"Como usar JdbcTemplate para executar queries SQL diretamente em uma aplicação Spring Boot?"**
9. **"Como funciona o Spring Data JPA e como ele se integra com Hibernate para mapear entidades?"**

---

### 🔹 Etapa 4 – Criando e Manipulando Dados

10. **"Como criar uma entidade JPA e um repositório no Spring Boot para salvar, buscar e deletar registros?"**
11. **"Como personalizar queries no Spring Data JPA usando métodos derivados e a anotação @Query?"**
12. **"Como usar o schema.sql e data.sql para popular o banco automaticamente no startup da aplicação?"**

---

### 🔹 Etapa 5 – Boas Práticas e Casos Especiais

13. **"Como configurar migrações de banco de dados em Spring Boot com Flyway ou Liquibase?"**
14. **"Quais são as melhores práticas para separar credenciais do banco em produção (Vault, variáveis de ambiente, Config Server)?"**
15. **"Quando faz sentido usar JNDI DataSource em aplicações Spring Boot modernas?"**
16. **"Como implementar testes de integração com banco de dados em projetos Spring Boot?"**

---

Se você me fizer todos esses prompts (nessa ordem), no fim você vai:

* Saber **como o Spring Boot gerencia conexões de banco**.
* Saber configurar banco **direto via properties** e também **via JNDI**.
* Entender **pools de conexão** e como o Boot usa HikariCP.
* Usar **JdbcTemplate e JPA/Hibernate** para CRUD real.
* Dominar **migrations e boas práticas de credenciais**.
* Estar pronto para **projetos reais Spring Boot + banco de dados**.

---

