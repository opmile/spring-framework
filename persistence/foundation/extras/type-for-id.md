# Modelagem do Campo ID

---

### Qual tipo usar para o campo `id`?

Os mais comuns são:

* **`Long` (wrapper de `long`)** → muito usado.
* **`Integer` (wrapper de `int`)** → também aparece bastante.
* **UUID** → alternativa moderna, especialmente em sistemas distribuídos.

---

### Por que preferem *wrapper* (`Long`, `Integer`) ao invés de primitivo (`long`, `int`)?

1. **Nullability**

   * Tipos primitivos (`long`, `int`) **não aceitam `null`**.
   * Já os wrappers (`Long`, `Integer`) podem ser `null`.
   * Isso é essencial no JPA, porque o `id` de uma entidade **não existe antes de ser salvo no banco** → ou seja, começa como `null` e só depois recebe valor.

   Exemplo:

   ```java
   @Id
   @GeneratedValue(strategy = GenerationType.IDENTITY)
   private Long id; // pode ser null até o insert
   ```

   Se fosse `long id`, teria que ter um valor inicial (tipo `0`), o que poderia confundir o JPA/Hibernate.

---

### Quando usar `Long` em vez de `Integer`?

* **`Integer`**:

  * Suporta até **2.147.483.647** (`2^31-1`).
  * Pode ser suficiente para tabelas pequenas ou bancos que não vão crescer muito.

* **`Long`**:

  * Suporta até **9.223.372.036.854.775.807** (`2^63-1`).
  * Mais indicado para sistemas que podem acumular **muitos registros ao longo do tempo**.
  * Ex.: sistemas de e-commerce, redes sociais, logs de eventos — onde é fácil ultrapassar bilhões de linhas.

---

### Regra de bolso (para projetos reais)

* Prefira **`Long`** para o campo `id`.

  * É padrão de mercado, evita limitações futuras.
* Use `Integer` só se tiver certeza que o volume de dados será **sempre baixo** (tipo tabelas auxiliares pequenas).

---

```java
@Entity
public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // escolha segura, cresce muito

    private String nome;
}
```

Se você tivesse usado `int`/`Integer`, estaria limitado a pouco mais de 2 bilhões de produtos. Com `long`/`Long`, a chance de overflow na prática é nula.

---

* Use sempre o **wrapper** (`Long` ou `Integer`) → por causa do `null`.
* Prefira **`Long`** para ids em entidades → é escalável e padrão em projetos reais.
* `Integer` pode ser usado em tabelas pequenas ou de apoio, mas é exceção.

---

