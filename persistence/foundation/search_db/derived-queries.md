# Derived Queries

*Derived Queries* são consultas que o Spring Data JPA gera **automaticamente** a partir do **nome do método** no repositório.

Aqui trabalhamos com métodos específicos que consultam o banco de forma personalizada. Esses métodos são criados na interface que herda de `JpaRepository`. Neles, utilizaremos palavras-chave (em inglês) para indicar qual a busca que queremos fazer.

Exemplo simples:

```java
public interface SerieRepository extends JpaRepository<Serie, Long> {
    List<Serie> findByName(String name);
}
```

Você nunca escreve SQL ou JPQL. O Spring Data lê `findByName`, entende que quer filtrar pela coluna `name` e gera a query:

```sql
SELECT * FROM serie WHERE name = ?;
```

---

### 2. Como funciona?

O mecanismo lê o nome do método e o quebra em partes:

* `findBy` → operação de busca.
* `Name` → o campo da entidade.

E você pode compor condições:

```java
List<Serie> findByNameAndAno(String name, Integer ano);
```

Gera:

```sql
SELECT * FROM serie WHERE name = ? AND ano = ?;
```

Ou ainda operadores:

```java
List<Serie> findByAnoGreaterThanEqual(Integer ano);
```

Gera:

```sql
SELECT * FROM serie WHERE ano >= ?;
```

---

### 3. Por que usar *Derived Queries*?

* **Produtividade**: você não precisa escrever JPQL ou SQL manualmente.
* **Legibilidade**: o nome do método já descreve o que a query faz.
* **Integração forte com o modelo**: trabalha diretamente com atributos da entidade.

---

### 4. Quando usar?

* **Consultas simples e frequentes**:

  * `findByEmail(String email)`
  * `findByStatus(Status status)`
  * `findByTituloContaining(String palavra)`

* **Cenários em que a query é trivial** e não compensa escrever manualmente.

---

### 5. Quando **não** usar?

* Quando a consulta é **complexa demais** (múltiplos JOINs, subqueries, agregações).
* Quando o nome do método ficaria enorme e pouco legível.

  * Exemplo absurdo:

    ```java
    findByNomeContainingAndAnoBetweenAndGeneroOrderByEpisodiosDesc(...)
    ```

Nesses casos, é melhor usar:

* `@Query` com JPQL/SQL customizado, ou
* `Criteria API` se quiser algo mais dinâmico.

---

* *Derived Queries* são consultas **geradas pelo nome do método**.
* Use para buscas **simples** e comuns.
* Elas deixam o código **mais limpo e rápido** de escrever, mas não substituem JPQL/SQL customizado em consultas complexas.

---

### 6. Modificadores

No **Spring Data JPA**, os **modificadores** em **Derived Queries** são palavras-chave que ajustam o comportamento da consulta sem precisar escrever SQL.

### Exemplos mais usados:

* **`IgnoreCase`**
  Torna a busca **case-insensitive**.

  ```java
  List<Serie> findByNomeIgnoreCase(String nome);
  ```

  → Equivalente a `LOWER(nome) = LOWER(:nome)`

* **`OrderBy`**
  Define a **ordenação** do resultado.

  ```java
  List<Serie> findByGeneroOrderByNomeAsc(String genero);
  List<Serie> findByGeneroOrderByNomeDesc(String genero);
  ```

  → Gera `ORDER BY nome ASC` ou `ORDER BY nome DESC`

* **Combinação de ambos**

  ```java
  List<Serie> findByNomeIgnoreCaseOrderByIdDesc(String nome);
  ```

  → Filtro sem diferenciar maiúsculas/minúsculas + resultado ordenado por id.

---

* **Predicados** → definem **condições de filtro** (`Like`, `Between`, `IsNull`, `Regex` etc.)
* **Modificadores** → controlam **como os resultados são tratados** (`IgnoreCase`, `OrderBy`, `Distinct`, `First`, `TopN` etc.).

---

## Exemplos

### Entidade base (exemplo)

```java
@Entity
public class Serie {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private Integer ano;
    private String genero;
    private String status; // ex: "Em exibição", "Finalizada"
}
```

---

### 1. Consultas simples

```java
List<Serie> findByNome(String nome);
```

SQL gerado:

```sql
SELECT * FROM serie WHERE nome = ?;
```

Uso: buscar por nome exato.

---

### 2. Condições compostas

```java
List<Serie> findByNomeAndAno(String nome, Integer ano);
```

SQL gerado:

```sql
SELECT * FROM serie WHERE nome = ? AND ano = ?;
```

Uso: buscar série específica de um determinado ano.

---

### 3. Comparações

```java
List<Serie> findByAnoGreaterThanEqual(Integer ano);
```

SQL:

```sql
SELECT * FROM serie WHERE ano >= ?;
```

Uso: séries lançadas a partir de determinado ano.

---

### 4. Operadores de texto

```java
List<Serie> findByNomeContaining(String palavra);
```

SQL:

```sql
SELECT * FROM serie WHERE nome LIKE %?%;
```

Uso: buscar séries cujo nome contém a palavra.

---

### 5. Listas de valores

```java
List<Serie> findByGeneroIn(List<String> generos);
```

SQL:

```sql
SELECT * FROM serie WHERE genero IN (?, ?, ...);
```

Uso: buscar por vários gêneros ao mesmo tempo.

---

### 6. Ordenação

```java
List<Serie> findByStatusOrderByAnoDesc(String status);
```

SQL:

```sql
SELECT * FROM serie WHERE status = ? ORDER BY ano DESC;
```

Uso: séries finalizadas, ordenadas da mais nova para a mais antiga.

---

### 7. Limite de resultados

```java
List<Serie> findTop3ByOrderByAnoDesc();
```

SQL:

```sql
SELECT * FROM serie ORDER BY ano DESC LIMIT 3;
```

Uso: pegar as 3 séries mais recentes.

---

### 8. Combinando operadores

```java
List<Serie> findByGeneroAndAnoBetween(String genero, Integer inicio, Integer fim);
```

SQL:

```sql
SELECT * FROM serie WHERE genero = ? AND ano BETWEEN ? AND ?;
```

Uso: buscar séries de determinado gênero em um intervalo de anos.

---

### 9. Consultas booleanas

```java
boolean existsByNome(String nome);
```

SQL:

```sql
SELECT COUNT(*) > 0 FROM serie WHERE nome = ?;
```

Uso: verificar se já existe uma série com aquele nome.

---

### 10. Contagem

```java
long countByStatus(String status);
```

SQL:

```sql
SELECT COUNT(*) FROM serie WHERE status = ?;
```

Uso: quantas séries estão em exibição, por exemplo.

---

### 11. Regex

```java
public interface SerieRepository extends JpaRepository<Serie, Long> {

    // Busca séries cujo nome corresponde a uma expressão regular
    List<Serie> findByNomeRegex(String regex);
}

// No código
@Autowired
private SerieRepository serieRepository;

public void buscarSeries() {
    // Exemplo: todas as séries que começam com "The"
    List<Serie> series = serieRepository.findByNomeRegex("^The.*");

    series.forEach(s -> System.out.println(s.getNome()));
}
```

---

* Do **simples** (`findByNome`) ao **complexo** (`findByGeneroAndAnoBetween`).
* Você começa com igualdade → múltiplos campos → operadores de comparação → LIKE/IN → ordenação → limites → agregações (`count`, `exists`).

---

## Mapa mental Derived Queries

| **Categoria**     | **Palavra-chave** | **Exemplo de método**                       | **Tradução SQL gerada**             | **Uso típico**                |
| ----------------- | ----------------- | ------------------------------------------- | ----------------------------------- | ----------------------------- |
| **Igualdade**     | `Is`, `Equals`    | `findByNome(String nome)`                   | `WHERE nome = :nome`                | Comparar por igualdade direta |
|                   | `Not`             | `findByNomeNot(String nome)`                | `WHERE nome <> :nome`               | Valores diferentes            |
| **Similaridade**  | `Like`            | `findByNomeLike(String nome)`               | `WHERE nome LIKE :nome`             | Busca com curingas (`%`)      |
|                   | `Containing`      | `findByNomeContaining(String termo)`        | `WHERE nome LIKE %:termo%`          | Contém substring              |
|                   | `StartingWith`    | `findByNomeStartingWith(String prefixo)`    | `WHERE nome LIKE :prefixo%`         | Começa com                    |
|                   | `EndingWith`      | `findByNomeEndingWith(String sufixo)`       | `WHERE nome LIKE %:sufixo`          | Termina com                   |
|                   | `IgnoreCase`      | `findByNomeIgnoreCase(String nome)`         | `LOWER(nome) = LOWER(:nome)`        | Ignora maiúsc./minúsc.        |
| **Comparação**    | `GreaterThan`     | `findByDuracaoGreaterThan(Integer x)`       | `WHERE duracao > :x`                | Maior que                     |
|                   | `LessThan`        | `findByDuracaoLessThan(Integer x)`          | `WHERE duracao < :x`                | Menor que                     |
|                   | `Between`         | `findByAnoBetween(int start, int end)`      | `WHERE ano BETWEEN :start AND :end` | Intervalo                     |
|                   | `After`           | `findByDataAfter(LocalDate d)`              | `WHERE data > :d`                   | Depois de uma data            |
|                   | `Before`          | `findByDataBefore(LocalDate d)`             | `WHERE data < :d`                   | Antes de uma data             |
| **Nulidade**      | `IsNull`          | `findByDescricaoIsNull()`                   | `WHERE descricao IS NULL`           | Campo vazio/nulo              |
|                   | `IsNotNull`       | `findByDescricaoIsNotNull()`                | `WHERE descricao IS NOT NULL`       | Campo preenchido              |
| **Modificadores** | `Distinct`        | `findDistinctByGenero(String genero)`       | `SELECT DISTINCT ...`               | Elimina duplicatas            |
|                   | `OrderBy`         | `findByGeneroOrderByNomeAsc(String genero)` | `ORDER BY nome ASC`                 | Ordenação                     |
|                   | `Top`, `First`    | `findTop3ByOrderByDataDesc()`               | `LIMIT 3`                           | Retorna primeiros resultados  |
|                   | `Exists`          | `existsByNome(String nome)`                 | `SELECT CASE WHEN COUNT(*) ...`     | Verifica existência           |


* **Igualdade/Comparação** → filtros exatos ou numéricos.
* **Similaridade** → buscas flexíveis com `LIKE`.
* **Nulidade** → checagem de campos nulos.
* **Modificadores** → controle sobre duplicatas, ordenação e limite de resultados.

## Derived Queries em Relacionamentos

Quando você tem entidades relacionadas, você pode **navegar pelos atributos da entidade relacionada** usando o nome do atributo no método do repositório.

Exemplo com entidades `Order` e `Product`:

```java
@Entity
public class Order {
    @ManyToMany
    private Set<Product> products = new HashSet<>();
}

@Entity
public class Product {
    private String name;
    private Category category;
}
```

---

### **1. Buscar pedidos por nome de produto**

```java
List<Order> findByProducts_Name(String productName);
```

* `Products` → nome da coleção no `Order`.
* `Name` → atributo da entidade `Product`.
* Resultado: todos os pedidos que contenham pelo menos um produto com aquele nome.

SQL equivalente:

```sql
SELECT o.*
FROM "order" o
JOIN order_product op ON o.id = op.order_id
JOIN product p ON op.product_id = p.id
WHERE p.name = ?;
```

---

### **2. Buscar pedidos por categoria de produto**

```java
List<Order> findByProducts_Category(Category category);
```

* Navega de `Order` → `products` → `category`.
* Retorna todos os pedidos que tenham algum produto naquela categoria.

SQL equivalente:

```sql
SELECT o.*
FROM "order" o
JOIN order_product op ON o.id = op.order_id
JOIN product p ON op.product_id = p.id
WHERE p.category = ?;
```

---

### **3. Combinar filtros de relacionamento com filtros da entidade principal**

```java
List<Order> findByClientAndProducts_Category(String client, Category category);
```

* Filtra pedidos de um cliente específico **que contenham produtos de uma categoria específica**.
* Spring gera a junção necessária automaticamente.

---

* Use `EntidadeRelacionada_Atributo` no nome do método.
* Pode combinar com operadores (`GreaterThan`, `Containing`, etc.) e conectores (`And`, `Or`).
* Spring Data JPA cuida das **joins automaticamente** na tabela de relacionamento.

---

