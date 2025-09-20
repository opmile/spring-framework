# Annotations de Persistência

---

## Diferenciando Tipos de Acessos

Na **JPA** (e, consequentemente, no Hibernate / Spring Data JPA), você pode mapear sua entidade de **duas formas**:

---

### 1. **Field Access (anotação nos atributos)**

```java
@Entity
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String nome;

    private Double preco;

    // getters e setters...
}
```

Nesse caso, o provedor JPA acessa os **campos diretamente** (mesmo que sejam `private`, usando reflexão).

* Todas as anotações (`@Id`, `@Column`, `@ManyToOne` etc.) ficam **no atributo**.
* O Hibernate ignora se você anotar o getter.

---

### 2. **Property Access (anotação nos getters)**

```java
@Entity
public class Produto {

    private Long id;
    private String nome;
    private Double preco;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public Long getId() {
        return id;
    }

    @Column(nullable = false, length = 100)
    public String getNome() {
        return nome;
    }

    public Double getPreco() {
        return preco;
    }

    // setters...
}
```
Aqui o provedor JPA não olha os atributos diretamente:

* Ele usa os **getters e setters** para ler/gravar os valores.
* Todas as anotações devem ficar nos **métodos acessores**.

---

### Regras importantes

1. **Misturar os dois estilos não é permitido** dentro da mesma entidade.

   * Se você anotou `@Id` em um campo → JPA assume **field access** para toda a classe.
   * Se você anotou `@Id` em um getter → JPA assume **property access** para toda a classe.

2. O estilo mais comum em projetos Spring Boot é **field access** (anotando os atributos).

   * Mais direto e menos verboso.
   * Hibernate acessa via reflexão.

3. **Property access** pode ser útil quando:

   * Você precisa aplicar lógica dentro dos getters/setters.
   * Ou quer manter os campos totalmente encapsulados.


---

### 1. `@Entity`

**Exemplo:**

```java
@Entity
public class Produto {
    // ...
}
```

**Explicação:**
Marca a classe como uma entidade persistente → o Hibernate vai mapear essa classe para uma tabela do banco. Cada entidade corresponde a uma tabela no banco de dados.

* Por padrão, o nome da tabela é o mesmo da classe (`Produto`).
* Você pode mudar usando `@Table`.

---

### 2. `@Table`

**Exemplo:**

```java
@Entity
@Table(name = "produtos")
public class Produto {
    // ...
}
```

**Explicação:**
Personaliza o mapeamento da tabela, com ela você pode especificar o nome da tabela, o esquema e as restrições de chave primária.

* `name` → nome da tabela no banco.
* Útil quando a tabela já existe com outro nome.

---

### 3. `@Id`

**Exemplo:**

```java
@Id
private Long id;
```

**Explicação:**
Indica a **chave primária** da entidade/tabela no banco.

* Obrigatório em toda entidade.
* Sempre combinado com alguma estratégia de geração (`@GeneratedValue`) ou atribuído manualmente.

---

### 4. `@GeneratedValue`

Quando você tem uma entidade com uma chave primária (@Id), em algum momento essa chave precisa ser gerada.

Você pode:

* **gerar manualmente** (por exemplo, pedindo para o usuário informar um código), ou

* **delegar ao banco de dados ou ao provedor JPA/Hibernate** a geração dessa chave.

> "Ei, Hibernate/JPA, eu não vou me preocupar em gerar o valor do ID. Você que cuide disso."

**Exemplo:**

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

**Explicação:**
Define como o valor da chave primária será gerado.

* `IDENTITY` → banco gera automaticamente.

    * Cada vez que você insere uma nova linha, o banco de dados gera o ID automaticamente (em PostgreSQL, normalmente com `SERIAL` ou `BIGSERIAL`).

    * Desvantagem: o Hibernate precisa executar o INSERT imediatamente para recuperar o ID gerado → pode atrapalhar se você quiser batch inserts.

```sql
CREATE TABLE produto (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco NUMERIC NOT NULL
);
```

* `SEQUENCE` → usa uma sequência.
    
    * Usa uma sequência no banco de dados (um contador). O Hibernate consulta a sequência para pegar o próximo valor antes de inserir.

```sql
CREATE SEQUENCE produto_seq START WITH 1 INCREMENT BY 1;
```

No código:

```java
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "produto_seq")
@SequenceGenerator(name = "produto_seq", sequenceName = "produto_seq", allocationSize = 1)
private Long id;
```

* `TABLE` → Cria uma tabela auxiliar só para armazenar e gerenciar os IDs.

    * É um fallback para bancos que não têm suporte a IDENTITY ou SEQUENCE. Pouco usado hoje, porque é mais lento.

* `AUTO` → o provedor JPA (Hibernate) escolhe qual estratégia usar com base no banco. 

    * No PostgreSQL, geralmente acaba virando SEQUENCE.

    * Pode ser bom para projetos portáveis (vários bancos), mas você perde controle.

---

### 5. `@Column`

**Exemplo:**

```java
@Column(nullable = false, length = 100, unique = true)
private String nome;
```

**Explicação:**
Configura detalhes da coluna no banco:

* `nullable = false` → não pode ser nulo.
* `length = 100` → limita tamanho (em strings). Funciona como um `varchar(n)` no padrão SQL.
* `unique = true` → valor exclusivo na tabela.

---

### 6. `@OneToMany` e `@ManyToOne`

**Exemplo (categoria → produtos):**

```java
@Entity
public class Categoria {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "categoria")
    private List<Produto> produtos;
}

@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
}
```

**Explicação:**
Relacionamentos entre tabelas:

* `@OneToMany` → uma categoria tem muitos produtos.
* `@ManyToOne` → muitos produtos pertencem a uma categoria.
* `@JoinColumn` → define a chave estrangeira no banco.

No mundo relacional:

* Uma **categoria** pode ter **muitos produtos**.
* Cada **produto** pertence a **uma categoria**.

Ou seja: **1\:N (um para muitos)**.

No banco, isso vira uma **chave estrangeira (FK)** em `produto` apontando para `categoria`.

---

### `@ManyToOne`

```java
@Entity
public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    private Double preco;

    @ManyToOne
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
}
```

* **`@ManyToOne`** → indica que **muitos produtos** estão associados a **uma categoria**.
* **`@JoinColumn(name = "categoria_id")`** → cria a coluna `categoria_id` em `produto`, que será a **FK** apontando para `categoria`.

  * Se não especificar, o Hibernate gera algo como `categoria_id` automaticamente.

Esse lado é o **lado dono da relação** (“owning side”), porque é quem tem a **foreign key** no banco.

---

### `@OneToMany`

```java
@Entity
public class Categoria {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToMany(mappedBy = "categoria")
    private List<Produto> produtos = new ArrayList<>();
}
```

* **`@OneToMany`** → indica que **uma categoria** pode ter **muitos produtos**.
* **`mappedBy = "categoria"`** → diz que essa relação já está **mapeada pelo atributo `categoria` na classe Produto**. O parâmetro `mappedBy` é o nome do atributo que detém a chave estrangeira (o lado “dono” da relação).

  * Ou seja, `Categoria` não cria uma coluna extra no banco.
  * Ele apenas "enxerga" o que foi definido do lado de `Produto`.

Esse é o **lado inverso da relação** (“inverse side” ou “mappedBy side”).

---

* `@ManyToOne` → lado **muitos**, sempre contém a **chave estrangeira** (FK).
* `@OneToMany(mappedBy = "...")` → lado **um**, apenas referencia o outro lado, **não cria coluna nova**.

* Dono da relação (owning side):
É a entidade que tem a **chave estrangeira** na tabela.
→ Geralmente é o `@ManyToOne`.
→ Quem tem a `@JoinColumn`? → Esse é o dono da relação.

Lado inverso (inverse side):
É a entidade que não tem a FK diretamente, mas que mapeia a relação pelo atributo do outro lado.
→ Geralmente é o `@OneToMany`, usando `mappedBy`.

---

### Banco gerado (exemplo PostgreSQL)

```sql
CREATE TABLE categoria (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255)
);

CREATE TABLE produto (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(255),
    preco NUMERIC,
    categoria_id BIGINT,
    CONSTRAINT fk_categoria FOREIGN KEY (categoria_id) REFERENCES categoria(id)
);
```

A **chave estrangeira** fica em `produto` (lado `@ManyToOne`).
O lado `@OneToMany` (`Categoria`) só “aponta de volta” no Java, mas no banco não gera nada novo.

---

### E o `fetch`?

* Por padrão:

  * `@ManyToOne` → `FetchType.EAGER` (traz junto a categoria sempre que buscar um produto).
  * `@OneToMany` → `FetchType.LAZY` (só carrega a lista de produtos quando você acessar).

Isso impacta **performance**:

* Se você carregar uma categoria com 1000 produtos, e o fetch for `EAGER`, pode vir uma enxurrada de queries.
* É comum configurar manualmente o `fetch` de acordo com o caso de uso.

---

```java
// Criando categoria
Categoria cat = new Categoria();
cat.setNome("Eletrônicos");
categoriaRepository.save(cat);

// Criando produto
Produto p1 = new Produto();
p1.setNome("Notebook");
p1.setPreco(3500.0);
p1.setCategoria(cat);
produtoRepository.save(p1);

// Agora posso acessar os produtos da categoria
List<Produto> produtos = cat.getProdutos(); // pode disparar query (lazy)
```

---

* `@ManyToOne` → lado dono, cria a FK, está sempre na tabela do lado “muitos”.
* `@OneToMany(mappedBy = "...")` → lado inverso, não cria coluna, apenas referencia o atributo do outro lado.
* Sempre que você tem um relacionamento 1\:N no banco → no JPA você modela com `@OneToMany` de um lado e `@ManyToOne` do outro.

---

### 7. `@ManyToMany`

**Exemplo (produto ↔ tag):**

`@ManyToMany` descreve um relacionamento N\:N entre duas entidades: muitos A podem se relacionar com muitos B.
Ex.: um **Produto** pode ter muitas **Tag**s; uma **Tag** pode pertencer a muitos **Produtos**.

No banco relacional isso vira **uma tabela de junção** (join table) com duas FKs: `produto_id` e `tag_id`.

```java
@Entity
public class Tag {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany(mappedBy = "tags")
    private List<Produto> produtos;
}

@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany
    @JoinTable(
        name = "produto_tag",
        joinColumns = @JoinColumn(name = "produto_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private List<Tag> tags;
}
```

**Explicação:**
Cria um relacionamento N\:N → uma tabela intermediária (`produto_tag`) é gerada.

* `@JoinTable` → define o nome da tabela de junção.
* `joinColumns` → referência para a entidade atual.
* `inverseJoinColumns` → referência para a entidade do outro lado.

*Qual lado da relação leva a `@JoinTable?`* Tecnicamente, pode ser qualquer um dos lados, mas:

* O lado que você coloca a `@JoinTable` é chamado de lado dono (owning side).
* O outro lado deve apenas usar `mappedBy` e não cria a tabela de junção de novo.

**Escolha o lado que faz mais sentido de leitura no seu domínio.** 

* Ex.: “um produto tem várias tags” → natural ser o dono em `Produto`.

#### a) Bidirecional (mais comum — você pode navegar dos dois lados)

**Produto (lado “owning”)**

```java
@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @ManyToMany
    @JoinTable(
        name = "produto_tag",
        joinColumns = @JoinColumn(name = "produto_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id"),
        uniqueConstraints = @UniqueConstraint(columnNames = {"produto_id","tag_id"})
    )
    private Set<Tag> tags = new HashSet<>();
    // getters/setters
}
```

**Tag (lado inverso)**

```java
@Entity
public class Tag {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @ManyToMany(mappedBy = "tags")
    private Set<Produto> produtos = new HashSet<>();
    // getters/setters
}
```

* O lado **owning** é o que define `@JoinTable` (aqui: `Produto`).
* O lado **inverso** usa `mappedBy` apontando para o nome do atributo do owning side (`"tags"`).
* Uso de `Set` evita duplicatas por natureza.

#### b) Unidirecional (apenas um lado navega)

**Produto**

```java
@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany
    @JoinTable(name = "produto_tag",
      joinColumns = @JoinColumn(name = "produto_id"),
      inverseJoinColumns = @JoinColumn(name = "tag_id"))
    private Set<Tag> tags = new HashSet<>();
}
```

* Neste caso, `Tag` não conhece/tem coleção de `Produto`. Isso gera a mesma join table, mas é mais simples quando só precisa navegar de um lado.

---

#### Tabela no banco:

SQL gerado esperado:

```sql
CREATE TABLE produto (
  id BIGSERIAL PRIMARY KEY,
  nome VARCHAR(...)
);

CREATE TABLE tag (
  id BIGSERIAL PRIMARY KEY,
  nome VARCHAR(...)
);

CREATE TABLE produto_tag (
  produto_id BIGINT NOT NULL,
  tag_id BIGINT NOT NULL,
  CONSTRAINT pk_produto_tag PRIMARY KEY (produto_id, tag_id),
  CONSTRAINT fk_produto FOREIGN KEY (produto_id) REFERENCES produto(id),
  CONSTRAINT fk_tag FOREIGN KEY (tag_id) REFERENCES tag(id)
);
```

(Obs.: você pode definir explicitamente `uniqueConstraints` e indices na `@JoinTable` para reforçar unicidade/performance.)

---

#### Fetch type e performance

* **Por padrão `@ManyToMany` é `FetchType.LAZY`** — isso é bom. Ele só carrega a coleção quando você acessa.
* **Evite `EAGER`** — se não controlar, pode gerar joins grandes e problemas de performance (ou N+1 quando mal usado).
* Problema comum: carregar uma entidade que carrega centenas de associações por EAGER → travamento de memória/queries enormes.

Solução prática: manter `LAZY`, e usar `JOIN FETCH` em consultas JPQL quando precisar trazer junto.

---

#### Duplicatas e unicidade

* Use `Set` para evitar duplicatas na coleção Java.
* Adicione `uniqueConstraints` na `@JoinTable` se quiser garantir no DB que não existam pares repetidos:

  ```java
  @JoinTable(..., uniqueConstraints = @UniqueConstraint(columnNames = {"produto_id","tag_id"}))
  ```

---

#### Equals / hashCode e coleções

* Se usar `Set`, **equals() e hashCode()** das entidades devem estar bem definidos para evitar comportamento errado (duplicatas aparentemente iguais ou falha em remoção).
* **Padrão recomendado**:

  * Implementar `equals` que use uma **chave de negócio imutável** (p.ex. um código único).
  * Se não tiver chave natural, usar `id` com cuidado: comparar por `id` **apenas** quando `id != null` (entidade ainda transitória tem id null).
  * Exemplo simples (suficiente na maioria dos casos):

    ```java
    @Override
    public boolean equals(Object o) {
      if (this == o) return true;
      if (!(o instanceof Produto)) return false;
      Produto other = (Produto) o;
      return id != null && id.equals(other.getId());
    }

    @Override
    public int hashCode() {
      return 31;
    }
    ```

    -> Essa técnica assume que o `id` só fica disponível após persistência; `hashCode` constante evita problemas quando o objeto muda de estado, embora exista trade-off (menos distribuição).
* Alternativa elegante: usar Lombok com `@EqualsAndHashCode(onlyExplicitlyIncluded = true)` e incluir apenas campos estáveis.

---

#### Quando **não** usar ManyToMany — usar entidade de junção (associative entity)

Use uma entidade de junção quando:

* você precisa armazenar **atributos adicionais** naquela relação (ex.: `criadoEm`, `autoridade`, `peso`),
* ou quando a associação tem **ciclo de vida** próprio,
* ou quando precisa consultar a associação como entidade (ex.: paginar ligações, ordenar por data da associação).

**Exemplo: associação com atributos (ProdutoTag)**

```java
@Entity
public class ProdutoTag {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "produto_id")
    private Produto produto;

    @ManyToOne
    @JoinColumn(name = "tag_id")
    private Tag tag;

    private LocalDateTime criadoEm;
    private Integer relevancia;
    // getters/setters
}
```

* Aqui você substitui ManyToMany por duas relações ManyToOne → é uma tabela com colunas extras.
* Muito mais flexível para lógica de negócio.

---

### 8. `@OneToOne`

Define um relacionamento 1 para 1 entre duas entidades. Ou seja, cada instância de A se relaciona com no máximo uma instância de B, e vice-versa.

* Uma Pessoa pode ter um único Passaporte.
* Um Carro pode ter um único Documento de Registro.

1. Owning Side (quem contém a chave estrangeira (FK) no banco)

```java
@Entity
public class Pessoa {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToOne
    @JoinColumn(name = "passaporte_id") // cria a coluna de FK em Pessoa
    private Passaporte passaporte;
}
```

2. Outro lado (opcional, apenas se quiser bidirecional)

```java
@Entity
public class Passaporte {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String numero;

    @OneToOne(mappedBy = "passaporte") // nome do atributo em Pessoa
    private Pessoa pessoa;
}
```

---

### 9. `JoinColumn`

* Define a coluna de chave estrangeira (FK) no banco.

* Usada em relacionamentos` @OneToOne` e `@ManyToOne`, sempre no lado dono da relção (já que possui a FK).

> “Guarde o id da outra entidade nesta coluna aqui.”

```java
@Entity
public class Produto {
    @Id @GeneratedValue
    private Long id;

    private String nome;

    @ManyToOne
    @JoinColumn(name = "categoria_id") // cria a FK para Categoria
    private Categoria categoria;
}
```

* Tabela `produto` terá uma coluna `categoria_id` apontando para `categoria.id`.

### 10. `@JoinTable`

* Define uma tabela de junção para relacionamentos. Cria uma tabela intermediária com duas FKs (uma para cada entidade)

* Usada em relacionamentos `@ManyToMany` (ou em `@OneToOne`/`@OneToMany` se você quiser forçar join table, mas é raro).

```java
@Entity
public class Produto {
    @Id @GeneratedValue
    private Long id;

    private String nome;

    @ManyToMany
    @JoinTable(
        name = "produto_tag", // nome da tabela de junção
        joinColumns = @JoinColumn(name = "produto_id"),        // FK para Produto
        inverseJoinColumns = @JoinColumn(name = "tag_id")      // FK para Tag
    )
    private Set<Tag> tags = new HashSet<>();
}
```

No banco:

* `produto` (id, nome)

* `tag` (id, nome)

* `produto_tag` (produto_id, tag_id) ← tabela extra criada por @JoinTable.

---

Diferença `@JoinTable` e `@JoinColumn`

* `@JoinColumn` → coluna FK em uma tabela existente.

* `@JoinTable` → cria uma tabela nova só para mapear a relação (normalmente em N:N).

---

### 11. `@Enumerated`

**Exemplo:**

```java
public enum Status {
    ATIVO, INATIVO;
}

@Entity
public class Usuario {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)
    private Status status;
}
```

**Explicação:**
Diz ao Hibernate como salvar enums:

* `EnumType.ORDINAL` → salva posição (0, 1, 2...).
* `EnumType.STRING` → salva o nome (`ATIVO`, `INATIVO`).

Sempre prefira `STRING` para legibilidade e evitar problemas se a ordem mudar.

---

### 12. `@Lob`

**Exemplo:**

```java
@Lob
private String descricao;
```

**Explicação:**
Indica que o campo é um **Large Object** (texto longo ou binário).

* Pode ser usado para `CLOB` (Character Large Object) → textos grandes (artigos, logs, JSONs).
* Ou `BLOB` (Binary Large Object) → imagens, vídios, pdfs, arquivos, etc.

O Hibernate decide se usa CLOB ou BLOB de acordo com o **tipo Java** do atributo.

1. Armazenar imagens de perfil do usuário

```java
@Entity
public class Usuario {
    @Id @GeneratedValue
    private Long id;

    private String nome;

    @Lob
    private byte[] fotoPerfil; // BLOB (dados binários)
}
```

* No banco: coluna do tipo `BYTEA` (PostgreSQL), `BLOB` (MySQL/Oracle).

2. Guardar artigos ou descrições longas

```java
@Entity
public class Artigo {
    @Id @GeneratedValue
    private Long id;

    private String titulo;

    @Lob
    private String conteudo; // CLOB (texto muito grande)
}
```

* No banco: vira `TEXT` (PostgreSQL/MySQL), `CLOB` (Oracle).

3. Armazenar docs pdfs

```java
@Entity
public class Documento {
    @Id @GeneratedValue
    private Long id;

    private String nome;

    @Lob
    private byte[] arquivoPdf; // BLOB
}
```

* Você poderia receber esse `byte[]` de um upload, salvar no banco, e depois servir de volta via REST.

#### Considerações Importantes

* **Performance**: salvar arquivos binários grandes diretamente no banco pode deixar consultas lentas. Muitas vezes é melhor armazenar no sistema de arquivos ou em serviços como S3, e no banco guardar só a URL/caminho.

* **Boa prática**: usar `@Lob` para textos grandes é bem tranquilo (ex.: descrição longa, JSONs). Para binários grandes (imagens, PDFs pesados), avalie se o banco é realmente o melhor lugar.

* **Lazy Loading**: combine com `@Basic(fetch = FetchType.LAZY)` para não carregar o campo automaticamente em todas as consultas:

```java
@Lob
@Basic(fetch = FetchType.LAZY)
private byte[] fotoPerfil;
```

---

### 13. `@Transient`

**Exemplo:**

```java
@Transient
private String precoFormatado;
```

**Explicação:**
Ignora o campo → ele **não será persistido no banco**.

* Útil para atributos calculados ou auxiliares. Quando você tem um atributo que faz sentido na lógica da aplicação, mas não deve virar coluna na tabela.

Exemplos: atributos calculados, valores temporários, dados sensíveis que não precisam ser persistidos.

1. Campo calculado

```java
@Entity
public class Produto {
    @Id @GeneratedValue
    private Long id;

    private String nome;
    private double preco;
    private double desconto;

    @Transient
    public double getPrecoFinal() {
        return preco - desconto;
    }
}
```
* `precoFinal` é só calculado em memória, não persiste

2. Campo auxiliar para lógica de negócio

```java
@Entity
public class Usuario {
    @Id @GeneratedValue
    private Long id;

    private String nome;
    private String senhaHash;

    @Transient
    private String senhaEmTexto; // usado só na lógica antes de criptografar
}
```
---

### 14. `@NamedQuery`

* **O que é**: uma forma de definir consultas **JPQL fixas** (estáticas) diretamente na entidade.

* **Onde se coloca**: na classe de entidade (geralmente em cima da declaração da classe).

* **Como funciona**: você “batiza” uma query com um nome e depois chama esse nome no seu código, em vez de escrever a consulta toda hora.

* **Exemplo**:

  ```java
  @Entity
  @NamedQuery(
      name = "Cliente.buscarPorNome",
      query = "SELECT c FROM Cliente c WHERE c.nome = :nome"
  )
  public class Cliente {
      @Id
      private Long id;
      private String nome;
  }
  ```

  E no repositório/DAO:

  ```java
  TypedQuery<Cliente> q = em.createNamedQuery("Cliente.buscarPorNome", Cliente.class);
  q.setParameter("nome", "Milena");
  List<Cliente> resultado = q.getResultList();
  ```

* **Quando usar**: quando você tem queries **repetitivas e fixas** no projeto (ex.: buscar por nome, listar ativos).

---

### 15. `@Query`

* **O que é**: usada geralmente com **Spring Data JPA**, permitindo escrever queries diretamente no **repositório**.

* **Diferença chave**: mais flexível, você não precisa definir nada na entidade; a query fica próxima do método que a usa.

* **Exemplo**:

  ```java
  public interface ClienteRepository extends JpaRepository<Cliente, Long> {
      
      @Query("SELECT c FROM Cliente c WHERE c.nome = :nome")
      List<Cliente> buscarPorNome(@Param("nome") String nome);
  }
  ```

* **Quando usar**: quando você precisa de consultas **personalizadas** que não dá para resolver só com os métodos automáticos do `JpaRepository`.

---

### 16. `@Cascade`

* É uma **annotation específica do Hibernate** (não faz parte do JPA puro).
* Serve para **controlar o comportamento em cascata** de operações feitas em uma entidade em relação às entidades relacionadas.

```java
@Entity
public class Cliente {
    @Id
    private Long id;

    @OneToMany(mappedBy = "cliente")
    @Cascade(org.hibernate.annotations.CascadeType.ALL) 
    private List<Pedido> pedidos;
}
```

* Aqui, se você salvar ou remover o `Cliente`, automaticamente os `Pedidos` relacionados terão a mesma ação aplicada, conforme o tipo de cascade definido.

**Importante:**
No **JPA padrão**, você já tem o atributo `cascade` dentro de annotations como `@OneToMany`, `@ManyToOne`, `@ManyToMany` e `@OneToOne`.
Exemplo em JPA:

```java
@OneToMany(mappedBy = "cliente", cascade = CascadeType.ALL)
private List<Pedido> pedidos;
```

O Hibernate, além do `cascade` do JPA, oferece a **annotation própria** `@Cascade`, que dá **mais opções específicas**.

---

#### Diferença prática

* `cascade = CascadeType.ALL` (JPA) → cobre as operações básicas (`PERSIST`, `MERGE`, `REMOVE`, `REFRESH`, `DETACH`).
* `@Cascade` (Hibernate) → adiciona comportamentos extras que **não existem no JPA**, como:

  * `SAVE_UPDATE` → quando a entidade é salva/atualizada, as associadas também são.
  * `DELETE_ORPHAN` → remove entidades órfãs (sem relação com o pai).
  * `REPLICATE`, entre outros.

---

* **Prefira sempre o `cascade` do JPA** quando possível → mais **portável** (não amarra o código ao Hibernate), e já resolve a maioria dos casos comuns.
* Use `@Cascade` apenas se você realmente precisa de um comportamento **exclusivo do Hibernate** (ex.: `DELETE_ORPHAN`).

---

### 17. `@Embeddable` e `@Embedded`

São muito úteis quando você precisa **reaproveitar estruturas de dados dentro de uma entidade** sem criar uma tabela separada.

---

#### @Embeddable

* Marca uma **classe** como sendo **embutível** em outra entidade.
* Essa classe **não vira tabela própria** no banco — os atributos dela ficam armazenados como **colunas na tabela da entidade que a usa**.
* Geralmente usada para **agrupar atributos relacionados**.

```java
@Embeddable
public class Endereco {
    private String rua;
    private String cidade;
    private String cep;
}
```

Aqui, `Endereco` **não é uma entidade independente**. Ele só existe “dentro” de outra entidade.

---

#### @Embedded

* É usada **dentro de uma entidade** para indicar que aquele atributo é uma instância de uma classe marcada com `@Embeddable`.
* Os campos da classe embutida se transformam em colunas na mesma tabela da entidade.

```java
@Entity
public class Cliente {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @Embedded
    private Endereco endereco; // vai expandir em rua, cidade, cep
}
```

No banco de dados, a tabela `Cliente` ficaria assim:

| id | nome   | rua        | cidade    | cep       |
| -- | ------ | ---------- | --------- | --------- |
| 1  | Milena | Rua A, 123 | São Paulo | 01000-000 |

---

* **Organização**: evita que sua entidade principal fique cheia de atributos soltos.
* **Reuso**: a mesma classe `Endereco` pode ser usada em várias entidades (ex.: `Cliente`, `Fornecedor`, `Funcionario`).
* **Legibilidade**: código mais limpo e fácil de entender.

---

* `@Entity` → essa classe é uma tabela.
* `@Table` → nome da tabela.
* `@Id` → chave primária.
* `@GeneratedValue` → como gerar a PK.
* `@Column` → detalhes da coluna.
* `@OneToMany`, `@ManyToOne`, `@ManyToMany` → relacionamentos.
* `@Enumerated` → enums.
* `@Lob` → textos/arquivos grandes.
* `@Transient` → não salvar no banco.
* `@NamedQuery` → consultas **pré-definidas e fixas**, acopladas à **entidade**.
* `@Query` → consultas **específicas e flexíveis**, definidas no **repositório**.
* `@Embeddable` → marca uma classe para ser usada dentro de entidades, sem criar tabela própria.
* `@Embedded` → indica o uso dessa classe dentro da entidade.
* `@Cascade` → controla o comportamento em cascata de operações feitas em uma entidade em relação às entidades relacionadas.

---

