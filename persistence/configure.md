# Configurando PostgreSQL com JPA/Hibernate

---

### Dependências Maven

No seu `pom.xml`, você precisa do **starter JPA** e do **driver do PostgreSQL**:

```xml
<dependencies>
    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- PostgreSQL JDBC Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Spring Boot Starter Web (opcional, se for API REST) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

---

### Configuração no `application.properties` ou `application.yml`

No **PostgreSQL**, você precisa informar URL, usuário e senha. Exemplo usando `application.properties`:

```properties
# Conexão com o banco
spring.datasource.url=jdbc:postgresql://localhost:5432/meubanco
spring.datasource.username=meu_user
spring.datasource.password=minha_senha
spring.datasource.driver-class-name=org.postgresql.Driver

# Configurações do Hibernate
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

Observações importantes:

* `spring.jpa.hibernate.ddl-auto=update` → cria/atualiza tabelas automaticamente. Para produção, geralmente se usa `validate` ou `none`.
* `spring.jpa.show-sql=true` → imprime no console os SQLs gerados pelo JPA/Hibernate.

Se você preferir `application.yml`:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/meubanco
    username: meu_user
    password: minha_senha
    driver-class-name: org.postgresql.Driver
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    hibernate:
      ddl-auto: update
    show-sql: true
```

---

### Criar a **entidade JPA**

Exemplo de uma tabela `Produto`:

```java
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;

@Entity
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private double preco;

    // Getters e setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    public double getPreco() { return preco; }
    public void setPreco(double preco) { this.preco = preco; }
}
```

---

### Criar o **Repository**

Com Spring Data JPA, você não precisa implementar DAO manualmente:

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    // Aqui você pode criar métodos customizados
    // Exemplo: List<Produto> findByNomeContaining(String nome);
}
```

O Spring Boot gera automaticamente **implementações de CRUD** para você.

---

### Usando o Repository no serviço ou controller

```java
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class ProdutoService {

    private final ProdutoRepository produtoRepository;

    public ProdutoService(ProdutoRepository produtoRepository) {
        this.produtoRepository = produtoRepository;
    }

    public List<Produto> listarProdutos() {
        return produtoRepository.findAll();
    }

    public Produto salvarProduto(Produto produto) {
        return produtoRepository.save(produto);
    }
}
```

---

1. Spring Boot detecta dependências (`spring-boot-starter-data-jpa` + driver PostgreSQL).
2. Configura automaticamente o **DataSource** com as propriedades informadas.
3. Configura JPA/Hibernate usando esse DataSource.
4. Cria beans do Repository que usam automaticamente o DataSource.

---

## Sobre as implementações de `JpaRepository`

---

### O que acontece quando você estende `JpaRepository`

Quando você escreve:

```java
public interface ProdutoRepository extends JpaRepository<Produto, Long> { }
```

você **não está implementando nada**, mas está **herdando uma interface cheia de métodos padrão** definidos pelo Spring Data JPA.

* `JpaRepository` já **herda de outras interfaces**, criando uma hierarquia:

```
JpaRepository<T, ID> 
   ↳ PagingAndSortingRepository<T, ID>
       ↳ CrudRepository<T, ID>
```

* Então, os métodos que você vê como `findAll()`, `save()`, `deleteById()`, `findById()` **vêm dessa hierarquia**:

| Método                       | Da onde vem                | O que faz                     |
| ---------------------------- | -------------------------- | ----------------------------- |
| `save()`                     | CrudRepository             | Insere ou atualiza a entidade |
| `findById(ID id)`            | CrudRepository             | Busca por ID                  |
| `findAll()`                  | JpaRepository              | Retorna todas as entidades    |
| `deleteById(ID id)`          | CrudRepository             | Deleta a entidade pelo ID     |
| `count()`                    | CrudRepository             | Conta o número de registros   |
| `findAll(Sort sort)`         | PagingAndSortingRepository | Retorna todos, com ordenação  |
| `findAll(Pageable pageable)` | PagingAndSortingRepository | Retorna página de resultados  |

---

### E como o Spring cria a implementação real?

Aqui que está a **parte mágica do Spring**:

1. No **runtime**, o Spring **analisa suas interfaces que estendem `JpaRepository`**.
2. Ele cria **uma implementação dinâmica por meio de proxies** usando Reflection + bytecode.
3. O proxy **chama o EntityManager do JPA/Hibernate** por trás, fazendo os CRUDs de forma automática.

Então, quando você faz:

```java
produtoRepository.findAll();
```

* Na verdade, você está chamando **um método implementado dinamicamente pelo Spring**, que gera a query SQL equivalente (`SELECT * FROM produto`) e executa no banco.

---

### Métodos customizados

Além dos métodos herdados, você pode criar consultas personalizadas só declarando **a assinatura do método**.
Exemplo:

```java
List<Produto> findByNomeContaining(String nome);
```

* O Spring **traduz o nome do método em uma query SQL** automaticamente (`SELECT * FROM produto WHERE nome LIKE %?%`).
* Não precisa escrever `@Query` nem implementar nada manualmente.

---

* Você herda de `JpaRepository` → ganha métodos CRUD prontos.
* Spring cria **uma implementação automática** por proxy.
* O DAO “real” existe, mas você **não precisa ver nem escrever**.

---



