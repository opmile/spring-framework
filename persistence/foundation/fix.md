# Exercícios de Fixação 01

---

1. Um cliente pode ter apenas um endereço, e cada endereço pertence a um cliente. Modele as entidades e indique o SQL correspondente.

* Caso 1:1

```java
@Embeddable
public class Address {
    private String road;
    private String number;
    private String county;
    private String city;
    private String state;
    private String country;
}

@Entity
@Table(name = "tb_clients")
public class Client {

    @Id
    @GeneratedValue(stategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Embedded
    private Address address;
}
```

```java
// todo endereço pertence a um cliente
@Entity
public class Endereco {
    @Id
    @GerenetedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private int numero;
    private String cep;    

    @OneToOne(mappedBy = "endereco")
    private Cliente cliente;
}

// lado dominante -> recebe a fk de endereço
// um cliente só possui um endereço
@Entity
public class Cliente {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToOne
    @JoinColumn(name = "endereco_id")
    private Endereco endereco;
}
````

```java
// relação dominante -> recebe a fk
// uma pessoa pode ter um passsaporte
@Entity
public class Pessoa {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToOne
    @JoinColumn(name = "passaporte_id")
    private Passaporte passaporte;
}

// um passaporte sempre vai pertencer a uma pessoa
@Entity
public class Passaporte {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private int numero;
    
    @OneToOne(mappedBy = "passaporte")
    private Pessoa pessoa;
}
```

```sql
CREATE TABLE endereco (
    id BIGSERIAL PRIMARY KEY,
    numero INT NOT NULL,
    cep VARCHAR(9)
);  

CREATE TABLE client (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100),
    endereco_id REFERENCES endereco(id)
);
```

2. Um produto pertence a uma categoria. Uma categoria pode ter vários produtos. Adicione campos extras: nome, preco para `Produto`, e nome para `Categoria`.

```java
// uma categoria pode conter vários produtos
public class Categoria {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToMany(mappedBy = "categoria")
    private List<Produto> produtos;
}

public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private double preco;

    @ManyToOne
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;
}
```

```sql
CREATE TABLE categoria (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100)
);

CREATE TABLE produto (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100),
    preco NUMERIC(10,2),
    categoria_id INT,
    CONSTRAINT fk_produto FOREIGN KEY (categoria_id) REFERENCES categoria(id)
);
```

EXTRA)  Um departamento pode ter muitos funcionários, mas cada funcionário pertence a um único departamento. 

```java
@Entity
public class Departamento {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String nome;

    @OneToMany(mappedBy = "departamento")
    private List<Funcionario> funcionarios;
}

@Entity
public class Funcionario {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private String cpf;
    private int idade;
    private String cargo;
    private double salario;

    @ManyToOne
    @JoinColumn(name = "departamento_id")
    private Departamento departamento;
}
```

```sql
CREATE TABLE departamento (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100)
);

CREATE TABLE funcionario (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100),
    cpf VARCHAR(14),
    idade INT,
    cargo VARCHAR(100),
    salario NUMERIC(10,2),
    departamento_id INT,
    CONSTRAINT (departamento_id) FOREIGN KEY departamento(id)
);
```

3. Produtos podem ter várias tags, e uma tag pode estar em vários produtos.

```java
public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;
    private double preco;

    @ManyToMany
    @JoinTable(
        name = "produto_tag"
        joinColumn = @JoinColumn(name = "produto_id")
        inverseJoinColumn = @JoinColumn(name = "tag_id")
    )
    private List<Tag> tags;
}

public class Tag {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @ManyToMany(mappedBy = "tags")
    private List<Produto> produtos
}
```

```sql
CREATE TABLE produto (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100),
    preco NUMERIC(10,2)
);

CREATE TABLE tag (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100)
);

CREATE TABLE produto_tag (
    produto_id INT,
    tag_id INT,
    PRIMARY KEY (produto_id, tag_id),
    FOREIGN KEY (produto_id) REFERENCES produto(id),
    FOREIGN KEY (tag_id) REFERENCES tag(id)
);
```

4. Um curso pode ter vários alunos, e um aluno pode estar matriculado em vários cursos.

```java
@Entity
public class Curso {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String nome;
    private String cargaHoraria;
    
    @ManyToMany(mappedBy = "cursos")
    private List<Aluno> alunos;
}

@Entity
public class Aluno {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTY)
    private Long id;

    private String nome;
    private int matricula;
    
    @ManyToMany
    @JoinTable(
        name = "aluno_curso",
        joinColumn = @JoinColumn(name = "aluno_id"),
        inverseJoinColumn = @JoinColumn(name = "curso_id"))
    private List<Curso> cursos;
}
```

```sql
CREATE TABLE curso (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100),
    carga_horaria VARCHAR(100)
);

CREATE TABLE aluno (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100),
    matricula INT
);

CREATE TABLE aluno_curso (
    aluno_id INT,
    curso_id INT,
    PRIMARY KEY (aluno_id, curso_id),
    FOREIGN KEY (aluno_id) REFERENCES aluno(id)
    FOREIGN KEY (curso_id) REFERENCES curso(id)
);
```

5. Um pedido pode ter vários itens (`ItemPedido`). Cada ItemPedido pertence a um pedido. Cada item tem `quantidade` e `precoUnitario`.

```java
@Entity
public class Pedido {
    @Id
    @GeneretedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToMany(mappedBy = "pedidos")
    private Set<ItemPedido> pedidos = new HashSet<>();
}

@Entity
public class ItemPedido {
    @Id
    @GeneretedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private int quantidade;
    private double precoUnitario;

    @ManyToOne
    @JoinColumn(name = "pedidos_id")
    private Pedido pedidos;
}
```

