# Comportamentos Padrão Esperados pela JPA

---

## 1. Construtor padrão (sem argumentos)

👉 **Obrigatório**.

A especificação JPA exige que toda entidade tenha um **construtor público ou protegido sem argumentos**.

* Isso é necessário porque o Hibernate (ou qualquer provider JPA) precisa instanciar o objeto por **reflexão** (sem saber os parâmetros de construtor).
* Se você só tiver construtores com argumentos, o Hibernate não vai conseguir criar o objeto.

```java
@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;

    // Construtor padrão exigido pelo JPA
    public Produto() {}

    // Construtor útil para você
    public Produto(String nome) {
        this.nome = nome;
    }
}
```

---

## 2. Getters e Setters

**Setters não são obrigatórios em todos os casos.**

O que a JPA precisa é de **acesso aos campos**. Isso pode ser feito de duas formas:

* **Acesso por campo (field access):**
  Se as anotações (`@Id`, `@Column`, etc.) estão nos atributos, o Hibernate acessa os atributos **diretamente via reflexão**, sem precisar de getters/setters.

* **Acesso por propriedade (property access):**
  Se as anotações estão nos **getters**, aí sim você é obrigado a ter getters e setters, porque o Hibernate vai usá-los para manipular os valores.

Exemplo **field access** (sem setters):

```java
@Entity
public class Produto {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // anotação no campo → field access
    
    private String nome;

    public Produto() {}

    public String getNome() {
        return nome;
    }
}
```

Exemplo **property access** (precisa de setters):

```java
@Entity
public class Produto {
    private Long id;
    private String nome;

    public Produto() {}

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public Long getId() { // anotação no getter → property access
        return id;
    }
    public void setId(Long id) {
        this.id = id;
    }

    @Column(name = "nome")
    public String getNome() {
        return nome;
    }
    public void setNome(String nome) {
        this.nome = nome;
    }
}
```

---

* **Construtor sem argumentos** → sempre obrigatório.
* **Getters/Setters** → só obrigatórios se você estiver usando **property access**. Se usar **field access** (que é o mais comum no Spring Data JPA), não precisa de setters para dar build, mas é uma boa prática ter pelo menos os getters para expor os dados.

---

