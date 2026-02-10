# O que acontece quando o projeto não é parado ao criar uma migration

Durante o desenvolvimento, o Flyway é executado **automaticamente na inicialização da aplicação**.
Se você cria um arquivo de migration **com o projeto em execução**, o Flyway pode tentar executá-lo imediatamente, **antes do código estar pronto ou correto**.

Isso pode causar:

* execução de SQL incompleto
* erro de sintaxe
* alteração parcial do schema

Quando isso ocorre, o Flyway **marca a migration como falha** e **bloqueia a inicialização da aplicação** nas próximas tentativas.

---

## Como o erro se manifesta

Ao subir o projeto novamente, o Spring Boot interrompe a inicialização com um erro semelhante a:

```
Validate failed: Migrations have failed validation
```

Esse erro indica que:

* uma migration falhou anteriormente
* o Flyway detectou inconsistência
* a aplicação foi impedida de iniciar para evitar corrupção do banco

Esse mesmo erro também pode ocorrer se:

* o SQL da migration estiver inválido
* houver conflito de schema
* o script estiver incompleto

---

## Por que o Flyway bloqueia a aplicação

Esse comportamento é **intencional e desejado**.

O Flyway trabalha com o princípio de que:

> um banco em estado inconsistente **não deve ser utilizado**

Permitir a aplicação subir ignorando migrations com erro significaria:

* banco parcialmente alterado
* histórico inválido
* risco real de dados corrompidos

---

## Como corrigir o problema (caso padrão)

### 1. Acesse o banco de dados

Conecte-se ao banco utilizado pela aplicação.

### 2. Remova o registro da migration que falhou

Execute:

```sql
DELETE FROM flyway_schema_history
WHERE success = 0;
```

Esse comando:

* remove apenas migrations marcadas como falha
* não afeta migrations executadas com sucesso
* libera o Flyway para tentar novamente

### 3. Corrija o arquivo da migration

* ajuste o SQL
* garanta que o script esteja completo e válido

### 4. Suba novamente a aplicação

O Flyway executará a migration corrigida normalmente.

---

## Caso especial: alteração parcial do schema

Em alguns casos, a migration falha **depois** de criar:

* tabelas
* colunas
* índices

O Flyway **não desfaz automaticamente** essas alterações.

Se isso acontecer, o erro pode persistir mesmo após remover o registro da migration.

### Solução em ambiente de desenvolvimento

A abordagem mais simples é:

```sql
DROP DATABASE vollmed_api;
CREATE DATABASE vollmed_api;
```

Depois disso:

* o banco volta ao estado limpo
* todas as migrations serão reaplicadas do zero
* o ambiente fica consistente novamente

---

## Boas práticas para evitar o problema

* Sempre **pare a aplicação** antes de criar ou editar migrations
* Trate migrations como código crítico
* Nunca edite uma migration já aplicada com sucesso
* Prefira errar em ambiente local, nunca em produção

---

Esse tipo de erro não indica falha do Flyway, mas sim:

* **proteção contra inconsistência**
* **garantia de integridade do banco**

Aprender a lidar com ele faz parte do uso correto de migrations e da maturidade no desenvolvimento de APIs.

Esse cuidado evita exatamente os problemas clássicos de:

* bancos diferentes entre ambientes
* deploys quebrados
* “funciona na minha máquina”.