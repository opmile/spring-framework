# `cascade = CascadeType.ALL`

---

### O que significa `cascade`?

O parâmetro `cascade` indica ao **JPA/Hibernate** que **operações realizadas em uma entidade "pai" devem ser propagadas para entidades relacionadas** automaticamente.

Exemplo típico:

```java
@OneToMany(mappedBy = "pedido", cascade = CascadeType.ALL)
private Set<ItemPedido> itens = new HashSet<>();
```

Aqui, a entidade `Pedido` é o “pai” e `ItemPedido` é o “filho”.

---

### O que faz `CascadeType.ALL`?

`CascadeType.ALL` é **um atalho** que engloba todos os tipos de cascata:

| Tipo      | O que faz                                                                     |
| --------- | ----------------------------------------------------------------------------- |
| `PERSIST` | Se você salvar o pedido, todos os itens também serão salvos automaticamente.  |
| `MERGE`   | Se você atualizar o pedido, todos os itens também serão atualizados.          |
| `REMOVE`  | Se você apagar o pedido, todos os itens também serão removidos do banco.      |
| `REFRESH` | Atualiza os itens quando você recarrega o pedido do banco.                    |
| `DETACH`  | Se o pedido for desconectado do contexto do Hibernate, os itens também serão. |

Ou seja, `ALL` = **todas essas operações propagadas**.

---

### Por que usar `CascadeType.ALL`?

1. **Evita código repetitivo**
   Sem cascata, você teria que salvar, atualizar ou deletar cada `ItemPedido` manualmente sempre que mexe no `Pedido`.

2. **Mantém consistência do modelo**
   Em relações 1\:N ou N\:N, faz sentido que a vida do filho dependa da vida do pai.
   Ex.: apagar um pedido → não quer deixar itens órfãos no banco.

3. **Simplifica o desenvolvimento**
   Especialmente útil em **sistemas CRUD**, onde a entidade pai é o ponto de entrada natural.

---

### Quando **não usar** `CascadeType.ALL`?

* Quando os objetos filhos **podem existir independentemente** do pai.

  * Ex.: `Produto` e `Tag` em N\:N → você provavelmente não quer que apagar um produto delete a tag do banco.
* Quando o filho é gerenciado por outro contexto ou serviço.

---

### Exemplo prático

```java
@Entity
public class Pedido {
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "pedido", cascade = CascadeType.ALL)
    private Set<ItemPedido> itens = new HashSet<>();
}

@Entity
public class ItemPedido {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    private Pedido pedido;
}
```

* Ao salvar um `Pedido` com `itens.add(...)`, o JPA salva **pedido e itens automaticamente**.
* Ao remover o `Pedido`, os `ItemPedido` são apagados também.

---

* `cascade = CascadeType.ALL` → propaga **todas as operações** do pai para o filho.
* Útil quando o filho **depende totalmente do pai**.
* Evita manipular cada objeto manualmente e mantém a integridade do modelo.

---

### Modelo sem cascade

Digamos que temos as entidades:

```java
@Entity
public class Serie {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titulo;

    @OneToMany(mappedBy = "serie")
    private List<Episodio> episodios = new ArrayList<>();
}
```

```java
@Entity
public class Episodio {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titulo;
    private Integer numero;

    @ManyToOne
    @JoinColumn(name = "serie_id")
    private Serie serie;
}
```

Em SQL, isso vira:

```sql
CREATE TABLE serie (
    id BIGSERIAL PRIMARY KEY,
    titulo VARCHAR(255)
);

CREATE TABLE episodio (
    id BIGSERIAL PRIMARY KEY,
    titulo VARCHAR(255),
    numero INT,
    serie_id BIGINT,
    CONSTRAINT fk_serie FOREIGN KEY (serie_id) REFERENCES serie(id)
);
```

Sem `cascade`, se você criar uma nova `Serie` e episódios em memória, precisa salvar **manualmente**:

```java
Serie s = new Serie();
s.setTitulo("Breaking Bad");

Episodio e1 = new Episodio();
e1.setTitulo("Pilot");
e1.setNumero(1);
e1.setSerie(s);

Episodio e2 = new Episodio();
e2.setTitulo("Cat's in the Bag...");
e2.setNumero(2);
e2.setSerie(s);

serieRepository.save(s);       // salva só a série
episodioRepository.save(e1);   // salva episódio
episodioRepository.save(e2);   // salva episódio
```

---

### Com cascade

Agora, se modelamos assim:

```java
@OneToMany(mappedBy = "serie", cascade = CascadeType.ALL)
private List<Episodio> episodios = new ArrayList<>();
```

Isso muda completamente:

* Agora, quando você **persistir** a `Serie`, o Hibernate automaticamente também persiste os `Episodios`.
* O mesmo vale para operações de **update** e **delete**, dependendo do tipo de `CascadeType`.

```java
Serie s = new Serie();
s.setTitulo("Breaking Bad");

Episodio e1 = new Episodio();
e1.setTitulo("Pilot");
e1.setNumero(1);
e1.setSerie(s);

Episodio e2 = new Episodio();
e2.setTitulo("Cat's in the Bag...");
e2.setNumero(2);
e2.setSerie(s);

s.getEpisodios().add(e1);
s.getEpisodios().add(e2);

serieRepository.save(s); // salva a série e TODOS os episódios
```

---



