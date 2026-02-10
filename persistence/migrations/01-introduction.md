# Migrations com Flyway

---

## 1. Por que migrations existem (o problema real)

Em uma API Spring Boot **o banco evolui junto com o código**:

* Novas entidades → novas tabelas
* Novos campos → `ALTER TABLE`
* Regras mudam → índices, constraints, normalização

Sem migration, o fluxo costuma ser esse:

* Cada dev cria o banco “do seu jeito”
* Ambientes ficam diferentes (local ≠ teste ≠ prod)
* Deploy quebra por causa de schema inconsistente
* “Funciona na minha máquina”

**Migration é versionamento do banco**, do mesmo jeito que Git versiona código.

---

## 2. O papel do Flyway

Flyway é um **gerenciador de migrações SQL (ou Java)** que:

* Executa scripts **em ordem**
* Garante que cada script rode **uma única vez**
* Mantém histórico do que já foi aplicado
* Bloqueia inconsistências

No fundo, ele cria uma tabela de controle:

```sql
flyway_schema_history
```

Ali ele registra:

* versão
* descrição
* checksum
* data de execução

Isso é o que garante **reprodutibilidade**.

---

## 3. Flyway + Spring Boot: por que é tão usado

Spring Boot + JPA **não são suficientes sozinhos** para controle de schema.

`spring.jpa.hibernate.ddl-auto`:

* `update` → perigoso
* `create` → apaga tudo
* `create-drop` → só pra teste
* `none` → o mais seguro

`ddl-auto=update` **não é determinístico, não é versionado e não é auditável**.

Hibernate **não é ferramenta de migração**, **não descreve o banco**, ele apenas tenta **transformar o estado atual em algo compatível com o mapeamento atual**.

* O resultado depende do estado prévio do banco

* Dois bancos partindo de históricos diferentes não convergem

* Não existe garantia de schema final único

> O schema não é função apenas do código, mas do histórico implícito de alterações.

E isso por si só já inviabiliza uso em ambientes controlados.

### *"Cada dev cria o banco do seu jeito"*

Com `ddl-auto=update`:

* Um dev cria a entidade hoje

* Outro cria amanhã com campo extra

* Um terceiro remove um atributo

O Hibernate:

* não remove colunas

* não normaliza estruturas antigas

* não cria índices corretamente

* não aplica constraints com segurança

Resultado:

* Bancos semanticamente diferentes

* Mesmo código, schemas divergentes

Ou seja:

O banco deixa de ser artefato versionado e vira efeito colateral local.

> Obs: DDL (Data Definition Language) é um subconjunto de comandos SQL usado para definir, modificar e estruturar objetos de banco de dados, como tabelas, índices e views. Comandos principais incluem CREATE (criar), ALTER (modificar) e DROP (excluir), alterando a estrutura, não os dados. 

---

## 4. Como o Flyway funciona na prática

### Estrutura padrão

```text
src/main/resources/db/migration
```

Flyway varre esse diretório automaticamente.

### Nome dos arquivos (isso é crucial)

```text
V1__create_usuario_table.sql
V2__add_email_to_usuario.sql
V3__create_pedido_table.sql
```

Formato:

```
V<versão>__<descrição>.sql
```

* `V` = versioned migration
* `__` = separador obrigatório
* Ordem importa

---

## 5. Exemplo real de uso

### V1 – criação inicial

```sql
CREATE TABLE usuario (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL
);
```

### V2 – evolução do domínio

```sql
ALTER TABLE usuario
ADD COLUMN email VARCHAR(150) UNIQUE;
```

Quando a API sobe:

* Flyway vê que V1 e V2 ainda não rodaram
* Executa em ordem
* Registra na tabela de histórico

Nenhum script roda duas vezes. Nunca.

---

## 6. Configuração básica no Spring Boot

### Dependência

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

### `application.yml`

```yaml
spring:
  flyway:
    enabled: true
  jpa:
    hibernate:
      ddl-auto: none
```

### `application.properties`

```properties
spring.flyway.enabled=true
spring.jpa.hibernate.ddl-auto=none
```

---

## 7. Uso proativo 

> **Toda mudança no modelo/esquema → nova migration.**

Mesmo que seja:

* adicionar coluna
* mudar tamanho
* criar índice
* adicionar constraint
* normalizar tabela

Nada de:

* “vou só ajustar no banco”
* “depois a gente cria a migration”

---

## 8. Migration não é só CREATE / ALTER

Use Flyway também para:

### Índices

```sql
CREATE INDEX idx_usuario_email ON usuario(email);
```

### Constraints

```sql
ALTER TABLE pedido
ADD CONSTRAINT chk_valor_pos
CHECK (valor > 0);
```

### Dados iniciais (com cuidado)

```sql
INSERT INTO perfil (nome) VALUES ('ADMIN'), ('USER');
```

---

## 9. O que NÃO fazer (armadilhas comuns)

### ❌ Alterar migration já aplicada

Isso quebra checksum e o Flyway **vai bloquear a aplicação**.

Se errou:

* Crie **uma nova migration**

### ❌ Confiar em `ddl-auto=update`

Funciona… até não funcionar.

### ❌ Misturar lógica de negócio em migration

Migration ≠ seed dinâmico ≠ regra de negócio.

### ❌ Não encerrar a aplicação antes de adicionar a nova migration

Encerrar a aplicação (ou qualquer processo que mantenha uma conexão ativa com o banco de dados) antes de criar ou rodar novas migrations é uma boa prática fundamental para evitar erros de bloqueio de banco de dados (database locks), falhas de migração e corrupção de estado

---

## 10. Estratégia madura de projeto

Fluxo saudável:

1. Modela entidade
2. Cria migration correspondente
3. Sobe aplicação
4. Flyway aplica
5. Código e banco evoluem juntos

Em time:

* Migration entra no PR
* Review avalia SQL
* Banco vira parte do versionamento

---

## 11. Flyway vs Liquibase (opinião honesta)

* Flyway → simples, direto, SQL puro
* Liquibase → mais verboso, XML/YAML/JSON

Para 90% das APIs Spring:

> **Flyway é a escolha certa.**

---

Se sua API não usa migrations:

* ela **não é reprodutível**
* ela **não escala bem em equipe**
* ela **vai quebrar em produção**

Flyway não é luxo.
É **infraestrutura básica**.

---

# Migração de Responsabilidade (Hibernate -> Flyway)

---

## 1. O que realmente mudou quando você adotou Flyway

Antes:

* Hibernate criava/alterava tabelas automaticamente
* Você **não controlava** exatamente o que ia pro banco
* O banco era um “efeito colateral” do código

Agora:

* Hibernate **para de mexer no schema**
* Flyway vira o **dono do schema**
* O banco passa a ser **contrato versionado**

Isso é maturidade de projeto, não trabalho extra desnecessário.

---

## 2. Você precisa escrever SQL para tudo? NÃO.

Aqui está a divisão mental correta:

### ❌ NÃO é Flyway

* `SELECT`
* `INSERT`
* `UPDATE`
* `DELETE`
* Queries JPA / JPQL
* Repositories
* Acesso a dados em runtime

👉 Isso continua **exatamente igual**, via JPA/Hibernate.

---

### ✅ É Flyway

Somente mudanças **estruturais** do banco:

* Criar tabelas
* Alterar colunas
* Criar índices
* Constraints
* Chaves estrangeiras
* Relacionamentos
* Dados estruturais (roles, perfis, enums de negócio)

Em resumo:

> **Flyway cuida do formato do banco, não do uso do banco.**

---

## 3. “Mas antes eu só criava a entidade e pronto…”

Sim. E isso escondia problemas sérios:

### Hibernate não sabe:

* Estratégia ideal de índices
* Constraints de negócio
* Performance real do banco
* Diferença entre ambientes
* Ordem correta de alterações complexas

Ele gera SQL **genérico**, não **intencional**.

Com Flyway:

* Você **decide**
* Você **assume controle**
* Você **previne surpresas**

---

## 4. Fluxo mental correto daqui pra frente

### Antes (ingênuo, mas confortável)

> “Criei a entidade, o banco se vira”

### Agora (profissional)

> “O modelo mudou, logo o schema precisa evoluir”

Isso vira um hábito natural bem rápido.

---

## 5. Exemplo real do dia a dia

Você adiciona um campo na entidade:

```java
private String email;
```

O que você faz agora?

### 1️⃣ Cria migration

```sql
ALTER TABLE usuario
ADD COLUMN email VARCHAR(150) UNIQUE;
```

### 2️⃣ Sobe a aplicação

* Flyway aplica
* Hibernate só mapeia

Fim.

Isso leva **30 segundos** depois que você pega o jeito.

---

## 6. “E se eu esquecer de criar a migration?”

Ótimo — o erro aparece cedo.

* Hibernate vai tentar mapear uma coluna que não existe
* A aplicação quebra **na inicialização**
* Você corrige antes de ir pra produção

Erro cedo > erro tarde.

---

## 7. Quando Hibernate ainda ajuda (e muito)

Você **não perde produtividade** no código:

* Mapeamento com `@Entity`
* Relacionamentos (`@OneToMany`, etc.)
* Validações (`@NotNull`)
* Geração de queries
* Cache
* Dirty checking

Flyway **não substitui Hibernate**, eles se complementam.

---

## 8. Estratégia prática para não sofrer

### Durante desenvolvimento local

Você pode:

* Apagar o banco
* Rodar todas as migrations do zero
* Garantir que o projeto sobe limpo

Se isso funciona:

> **Seu projeto está saudável.**

---

## 9. Opinião direta (sem anestesia)

Se um projeto depende de:

```yaml
ddl-auto: update
```

Ele:

* é frágil
* não escala em time
* vai quebrar em produção

Flyway te obriga a pensar — e isso **te torna uma desenvolvedora melhor**, não mais lenta.

---

* Você não escreve SQL para acessar dados
* Você escreve SQL para **evoluir o banco**
* Hibernate não deve controlar schema
* Flyway é o “Git do banco”
* Um pouco mais de trabalho, muita menos dor em produção