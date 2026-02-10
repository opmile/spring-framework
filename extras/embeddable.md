# `@Ebedded` e `@Embeddable`

---

Beleza, esse par de annotations é simples na ideia, mas **muito mal explicado na maioria dos materiais** — então vou direto ao ponto e com opinião forte:
`@Embeddable` + `@Embedded` existem pra **modelar valor**, não entidade. Se você usa isso como “atalho” de entidade, está usando errado.

---

## O problema que elas resolvem

Imagine uma entidade `Usuario` que tem um **endereço**:

* Rua
* Número
* Cidade
* CEP

Pergunta-chave:
👉 **Endereço tem identidade própria no sistema?**
👉 Ele vive sozinho no banco? Tem `id`? É buscado isoladamente?

Se a resposta for **não**, então **Endereço NÃO é uma entidade**.
É um **value object**. E é exatamente aí que entram `@Embeddable` e `@Embedded`.

---

## `@Embeddable` — “essa classe não vive sozinha”

Você usa `@Embeddable` **na classe que será incorporada**.

```java
@Embeddable
public class Endereco {

    private String rua;
    private String cidade;
    private String cep;
}
```

O que isso significa conceitualmente?

* ❌ Não vira tabela
* ❌ Não tem `@Id`
* ❌ Não existe sem alguém “dono”
* ✅ Seus atributos serão **colunas da tabela da entidade que a usar**

Pense assim:

> `@Embeddable` diz: *“eu sou parte de algo maior”*.

---

## `@Embedded` — “essa entidade contém esse objeto”

Agora, na entidade:

```java
@Entity
public class Usuario {

    @Id
    @GeneratedValue
    private Long id;

    private String nome;

    @Embedded
    private Endereco endereco;
}
```

O que acontece no banco?

Tabela `usuario`:

| id | nome | rua | cidade | cep |
| -- | ---- | --- | ------ | --- |

💥 **Nenhuma tabela `endereco` é criada**.

O JPA simplesmente **“espalha”** os campos do `Endereco` dentro da tabela `usuario`.

---

## Importante: não confunda com `@OneToOne`

Essa confusão é clássica.

### `@Embedded`

* Sem tabela própria
* Sem identidade
* Dependente da entidade
* Modela **composição**
* Sem `JOIN`

### `@OneToOne`

* Tabela separada
* Possui identidade
* Pode existir sozinha
* Modela **associação**
* Com `JOIN`

Regra prática (guarde isso):

> **Se faz sentido ter `EnderecoRepository`, então NÃO é `@Embedded`.**

---

## `@AttributeOverrides` (quando os nomes colidem)

Se você usar dois objetos embutidos do mesmo tipo:

```java
@Embedded
private Endereco enderecoResidencial;

@Embedded
private Endereco enderecoComercial;
```

Vai dar conflito de colunas. A solução:

```java
@Embedded
@AttributeOverrides({
    @AttributeOverride(name = "rua", column = @Column(name = "res_rua")),
    @AttributeOverride(name = "cidade", column = @Column(name = "res_cidade"))
})
private Endereco enderecoResidencial;
```

Isso é feio? Um pouco.
Mas é **honesto com o modelo**.

---

## Quando usar (e quando NÃO usar)

### Use `@Embeddable` quando:

* O objeto **não tem identidade**
* É conceitualmente parte de outro
* Não faz sentido existir isolado
* Você quer um modelo mais expressivo e limpo

### NÃO use quando:

* Precisa consultar separadamente
* Precisa de `@Id`
* Possui ciclo de vida próprio
* Representa uma entidade de negócio

---

## Minha opinião direta

`@Embedded` é **subutilizado** porque muita gente modela tudo como entidade.
Isso gera:

* JOIN desnecessário
* Modelo inchado
* Complexidade artificial

Se pensa em DDD (mesmo sem saber), `@Embeddable` é o caminho natural para **Value Objects**.