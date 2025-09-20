# Interface `Set` em Entidades JPA

---

### `List` vs `Set` em relacionamentos do JPA

* **`List`** → mantém **ordem** de inserção (ou outra definida). Pode conter **duplicados**.
* **`Set`** → **não permite duplicados** e não garante ordem (a menos que seja `LinkedHashSet` ou `TreeSet`).

No caso de relacionamentos N\:N (como `Produto` ↔ `Tag`):

* Usar `List` permite que, por descuido, você acabe com **duplicatas** na coleção em memória.
* Usar `Set` já garante, a nível de Java, que não haverá a mesma `Tag` repetida no mesmo `Produto` (ou vice-versa).

Por isso muitos devs e projetos preferem **`Set` como padrão em relacionamentos N\:N**, pois faz sentido do ponto de vista do modelo de domínio: um mesmo produto não deveria ter a mesma tag duas vezes.

Em relacionamentos 1:N também faz sentido o uso da abordagem de não conter elementos duplicados:

```java
@Entity
public class Cliente {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nome;

    @OneToMany(mappedBy = "cliente")
    private Set<Pedido> pedidos = new HashSet<>();
}
```

```java
@Entity
public class Pedido {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String descricao;

    @ManyToOne
    @JoinColumn(name = "cliente_id")
    private Cliente cliente;
}
```

---

### Por que `Set` é mais comum em 1\:N?

1. **Duplicatas não fazem sentido**

   * Um mesmo `Pedido` não pode aparecer duas vezes na lista de `pedidos` de um `Cliente`. O `Set` garante isso automaticamente, enquanto o `List` aceitaria duplicatas.

2. **Mais fiel ao banco de dados**

   * No banco, a relação é representada por uma **chave estrangeira (`cliente_id`)** em `Pedido`. Ou seja: cada pedido pertence a exatamente um cliente → não tem como o mesmo pedido estar duas vezes.

3. **Prevenção de bugs**

   * Usando `List`, se você sem querer fizer `cliente.getPedidos().add(pedido)` duas vezes, terá duas referências ao mesmo objeto em memória (mesmo que o banco não reflita isso). Com `Set`, isso não acontece: ele mantém apenas uma referência.

---

### Quando usar `List` em 1\:N?

* Quando a **ordem dos elementos importa**.

  * Ex.: `Pedido` → `ItensPedido` (talvez você precise garantir que os itens sejam listados na ordem em que foram inseridos).

* Quando a entidade precisa de **índice posicional** (acessar pelo índice da lista).

  ```java
  @OneToMany(mappedBy = "pedido")
  @OrderColumn(name = "ordem")
  private List<ItemPedido> itens = new ArrayList<>();
  ```

---

Tanto em **1\:N** quanto em **N\:N**, você vai ver a recomendação de usar `Set` inicializado (`new HashSet<>()`) na maioria dos casos, e só migrar pra `List` se o domínio realmente exigir ordem.

---

## Inicializar ou não a coleção

* Quando você escreve:

  ```java
  private List<Tag> tags;
  ```

  Esse campo começa como `null`. Se você esquecer de inicializar antes de usar (`tags.add(...)`), vai ter `NullPointerException`.

* Quando você escreve:

  ```java
  private Set<Tag> tags = new HashSet<>();
  ```

  Você garante que a coleção **nunca vai ser nula**, e já pode manipular (`add`, `remove`) sem precisar de `if (tags == null)`.

Isso é um **padrão de boas práticas em JPA**: inicializar coleções que representam relacionamentos logo na declaração, para evitar bugs sutis.

---

* Prefira **`Set` + inicialização com `HashSet<>()`** em mapeamentos N\:N ou 1\:N.

* Use `List` quando:

  * A **ordem dos elementos importa** (e.g., playlist de músicas, etapas de um processo).
  * Você quer permitir **duplicados** (bem raro em modelagem de banco).

---
