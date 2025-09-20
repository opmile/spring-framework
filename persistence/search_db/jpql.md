# JPQL

JPQL é uma **linguagem de consultas orientada a objetos** que funciona em cima da JPA.
Ao invés de consultar diretamente tabelas e colunas (como no SQL), você consulta **entidades e seus atributos**, ou seja, o modelo do seu código.

Por exemplo, suponha que você tem uma entidade `Pedido` que possui um relacionamento com `Cliente`.
No SQL, você escreveria algo como:

```sql
SELECT * FROM pedido p
JOIN cliente c ON p.cliente_id = c.id
WHERE c.nome = 'Milena';
```

Em **JPQL**, você escreve assim:

```jpql
SELECT p FROM Pedido p
JOIN p.cliente c
WHERE c.nome = 'Milena'
```

Repare que você não fala em `pedido` (tabela) ou `cliente_id` (coluna). Você fala em **entidades (`Pedido`, `Cliente`) e seus atributos (`cliente`, `nome`)**.

---

### Por que usar JPQL?

1. **Portabilidade** – como você não usa SQL específico de um banco, sua consulta funciona em qualquer SGBD que a JPA suporte.
2. **Orientação a objetos** – você raciocina no nível de entidades e relacionamentos, sem precisar se preocupar com nomes de tabelas/colunas.
3. **Integração natural com a JPA** – o resultado já vem mapeado como objetos (`Pedido`, `Cliente`), não como linhas ou colunas cruas.
4. **Relacionamentos mais claros** – é onde JPQL brilha. Consultas em relacionamentos N:1, 1\:N ou N\:N ficam expressivas e bem próximas do modelo que você desenhou em Java.

---

### Necessidade de uso em relacionamentos

No dia a dia, o uso de **JPQL se destaca quando você precisa fazer buscas que atravessam relacionamentos**, especialmente quando:

* Você precisa buscar entidades filtrando por atributos de outra entidade relacionada.
* Quer carregar informações de tabelas associadas sem escrever joins SQL manuais.
* Deseja maior controle do que os métodos automáticos do `JpaRepository` oferecem.

Exemplo clássico em relacionamento N\:N (`Aluno` ↔ `Curso`):

```jpql
SELECT a FROM Aluno a
JOIN a.cursos c
WHERE c.nome = 'Banco de Dados'
```

Esse tipo de consulta seria chato de expressar com apenas **derived queries** e muito mais verboso em **SQL nativo**, mas no JPQL fica claro e legível.

---

### Exemplo de modelo com relacionamento N\:N

```java
@Entity
public class Aluno {
    @Id @GeneratedValue
    private Long id;
    private String nome;

    @ManyToMany(mappedBy = "alunos")
    private List<Curso> cursos = new ArrayList<>();
}

@Entity
public class Curso {
    @Id @GeneratedValue
    private Long id;
    private String nome;

    @ManyToMany
    @JoinTable(
        name = "curso_aluno",
        joinColumns = @JoinColumn(name = "curso_id"),
        inverseJoinColumns = @JoinColumn(name = "aluno_id")
    )
    private List<Aluno> alunos = new ArrayList<>();
}
```

Aqui temos o relacionamento **N\:N** entre `Aluno` e `Curso`.

---

### Derived Query (limitação)

Com derived queries você até consegue algo assim:

```java
public interface AlunoRepository extends JpaRepository<Aluno, Long> {
    List<Aluno> findByCursosNome(String nome);
}
```

Funciona para casos simples, mas quando você precisa de condições mais ricas, começa a travar.

---

### JPQL em ação

Agora, com JPQL, dá para expressar consultas atravessando relacionamentos de forma bem mais flexível:

```java
public interface AlunoRepository extends JpaRepository<Aluno, Long> {

    @Query("SELECT a FROM Aluno a JOIN a.cursos c WHERE c.nome = :nomeCurso")
    List<Aluno> buscarAlunosPorCurso(@Param("nomeCurso") String nomeCurso);

    @Query("SELECT c FROM Curso c JOIN c.alunos a WHERE a.nome LIKE %:nomeAluno%")
    List<Curso> buscarCursosPorNomeDeAluno(@Param("nomeAluno") String nomeAluno);
}
```

---

### O que acontece aqui?

* `JOIN a.cursos c` → você está navegando pelo relacionamento **direto do objeto** (`Aluno` → `cursos`).
* A JPA cuida de traduzir isso para o **SQL com join** apropriado (usando a tabela intermediária `curso_aluno`).
* O resultado já vem como entidades prontas (`Aluno` ou `Curso`), sem precisar mapear linha a linha.

---

## Parâmetros Nomeados 

A annotation `@Param` é necessária **quando se utiliza parâmetros nomeados em JPQL**.

* Em JPQL você pode escrever queries com **placeholders nomeados**, por exemplo:

```java
@Query("SELECT p FROM Product p WHERE p.name LIKE :pattern")
List<Product> findByNamePattern(@Param("pattern") String pattern);
```

* O `:pattern` dentro da query é um **parâmetro nomeado**.
* O Spring precisa saber **qual argumento do método corresponde a esse parâmetro**, e é aí que entra `@Param("pattern")`.

Sem `@Param`, o Spring **não consegue mapear o argumento `pattern` do método para o `:pattern` da query**, e você receberá um erro de execução.

* Necessária sempre que a query JPQL usar **parâmetros nomeados** (`:nomeDoParametro`).
* Não é necessária se você usar **parâmetros posicionais** (`?1`, `?2`) ou se a query não tiver parâmetros.

---

## Comparação de Padrões em Strings

Perfeito, vamos focar só na **palavra-chave `LIKE` e `ILIKE`** em consultas de banco usando JPQL:

---

### **LIKE**

* Usada para **comparação de padrões** em strings.
* Sensível a **maiúsculas e minúsculas** (case-sensitive).
* Aceita **wildcards**:

  * `%` → representa zero ou mais caracteres.
  * `_` → representa exatamente um caractere.
* Exemplo JPQL:

```java
@Query("SELECT p FROM Product p WHERE p.name LIKE :pattern")
List<Product> findByNamePattern(@Param("pattern") String pattern);
```

* Uso típico:

  * `pattern = "Smart%"` → todos os produtos cujo nome começa com “Smart”.
  * `pattern = "%Phone%"` → todos os produtos cujo nome contém “Phone”.

---

### **ILIKE**

* Usada para **comparação de padrões sem diferenciar maiúsculas e minúsculas** (case-insensitive).
* Funciona apenas em **bancos que suportam ILIKE**, como PostgreSQL.
* Exemplo JPQL (PostgreSQL):

```java
@Query("SELECT p FROM Product p WHERE LOWER(p.name) LIKE LOWER(:pattern)")
List<Product> findByNamePatternIgnoreCase(@Param("pattern") String pattern);
```

* Nota: JPQL não possui nativamente `ILIKE`, mas você pode simular usando `LOWER()` para tornar a comparação case-insensitive.

---

### **Resumo rápido**

| Palavra-chave | Case-sensitive | Wildcards | Observações                                                 |
| ------------- | -------------- | --------- | ----------------------------------------------------------- |
| LIKE          | Sim            | `%` e `_` | Padrões simples, sensível a maiúsculas/minúsculas           |
| ILIKE         | Não            | `%` e `_` | Case-insensitive; não é padrão JPQL, simulado com `LOWER()` |


---

## Mais Exemplos

Aqui vai uma lista de exemplos **onde JPQL é necessário** (adaptando minha lista anterior):

```java
// Buscar pedidos que contenham um produto específico (JOIN entre Order e Product)
@Query("SELECT o FROM Order o JOIN o.products p WHERE p.name = :productName")
List<Order> findOrdersByProductName(@Param("productName") String productName);

// Contar quantos pedidos um cliente já fez (uso de COUNT + GROUP BY)
@Query("SELECT COUNT(o) FROM Order o WHERE o.client = :client")
long countOrdersByClient(@Param("client") String client);

// Trazer os produtos de uma categoria que estejam em pelo menos 1 pedido (JOIN + DISTINCT)
@Query("SELECT DISTINCT p FROM Product p JOIN p.orders o WHERE p.category = :category")
List<Product> findProductsInOrdersByCategory(@Param("category") Category category);

// Buscar os clientes que já compraram determinado produto (JOIN + SELECT em campo relacionado)
@Query("SELECT DISTINCT o.client FROM Order o JOIN o.products p WHERE p.id = :productId")
List<String> findClientsByProduct(@Param("productId") Long productId);

// Buscar pedidos cujo valor total (soma dos preços dos produtos) ultrapasse um valor X
@Query("SELECT o FROM Order o JOIN o.products p GROUP BY o HAVING SUM(p.price) > :minValue")
List<Order> findOrdersAboveTotal(@Param("minValue") Double minValue);

// Buscar produtos mais caros que a média de preço de todos os produtos (uso de subquery)
@Query("SELECT p FROM Product p WHERE p.price > (SELECT AVG(p2.price) FROM Product p2)")
List<Product> findProductsAboveAveragePrice();

// Trazer o top 3 produtos mais caros de uma categoria (ORDER BY + LIMIT)
@Query("SELECT p FROM Product p WHERE p.category = :category ORDER BY p.price DESC")
List<Product> findTop3MostExpensiveByCategory(@Param("category") Category category, Pageable pageable);

// Buscar clientes que compraram em mais de um pedido (uso de HAVING COUNT > 1)
@Query("SELECT o.client FROM Order o GROUP BY o.client HAVING COUNT(o) > 1")
List<String> findClientsWithMultipleOrders();

// Buscar o pedido mais caro (MAX em agregação)
@Query("SELECT o FROM Order o JOIN o.products p GROUP BY o.id ORDER BY SUM(p.price) DESC")
List<Order> findMostExpensiveOrder(Pageable pageable);
```






