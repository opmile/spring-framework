# Flyway com Spring Boot + JPA + PostgreSQL

Ao configurar uma aplicação Spring Boot utilizando **JPA + PostgreSQL + Flyway**, é importante entender que **o Flyway não funciona apenas com o starter do Spring Boot** nas versões mais recentes.

### Contexto

O Flyway passou por uma mudança estrutural:
o **core do Flyway** foi separado do **suporte específico a cada banco de dados**.

Isso significa que:

* `flyway-core` → apenas o motor de versionamento
* Suporte ao PostgreSQL → módulo separado

Sem o módulo do banco, o Flyway **não reconhece o PostgreSQL** e **não executa nenhuma migration**, mesmo que a aplicação suba normalmente.

---

## Dependências necessárias

### 1. Starter do Flyway (via Spring Initializr)

Este starter integra o Flyway ao ciclo de inicialização do Spring Boot:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-flyway</artifactId>
</dependency>
```

Ele:

* inicializa o Flyway no startup
* gerencia configuração via `application.yml` / `properties`
* **não inclui** suporte específico a banco de dados

---

### 2. Módulo do Flyway para PostgreSQL (obrigatório)

Este módulo permite que o Flyway reconheça e execute migrations no PostgreSQL:

```xml
<dependency>
  <groupId>org.flywaydb</groupId>
  <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

Sem essa dependência:

* a tabela `flyway_schema_history` **não é criada**
* scripts em `db/migration` **não são executados**
* o Flyway não apresenta erro explícito
* a aplicação sobe normalmente, mascarando o problema

---

## Resultado da configuração correta

Com ambas as dependências:

* o Flyway reconhece o PostgreSQL
* a tabela `flyway_schema_history` é criada automaticamente
* migrations versionadas são executadas no startup
* o versionamento do banco passa a ser controlado corretamente

---

## Observação importante

Sempre que utilizar Flyway, verifique se o **módulo do banco correspondente** está presente:

* PostgreSQL → `flyway-database-postgresql`
* MySQL → `flyway-database-mysql`
* Oracle → `flyway-database-oracle`
* etc.

O starter do Spring Boot **não adiciona automaticamente** esses módulos.