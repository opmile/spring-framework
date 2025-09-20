# Relacionamentos Uni e Bidirecionais

---

## 1. O que é direção em um relacionamento?

No JPA, **direção** significa:
**qual lado da relação conhece o outro?**

* O lado da relação que conhece o outro possui uma referência (atributo) que identifica a chave estrangeira (a partir da annotation `@JoinColumn`). Esse lado é o dominante da relação.

---

## 2. Relacionamento Unidirecional

* Apenas uma entidade tem referência para a outra.
* O “lado A” conhece o “lado B”, mas o “lado B” não sabe nada sobre o “lado A”.
* No banco de dados, a chave estrangeira existe do mesmo jeito, mas no modelo Java só existe um ponteiro em uma direção.

Exemplo unidirecional 1\:N (Pedido → ItemPedido):

```java
@Entity
public class Pedido {
    @Id @GeneratedValue
    private Long id;

    @OneToMany
    @JoinColumn(name = "pedido_id") // cria a FK em ItemPedido
    private List<ItemPedido> itens = new ArrayList<>();
}

@Entity
public class ItemPedido {
    @Id @GeneratedValue
    private Long id;

    private String produto;
}
```

Aqui, `Pedido` conhece seus `itens`, mas `ItemPedido` não conhece o `Pedido`.

---

## 3. Relacionamento Bidirecional

* As duas entidades têm referências entre si.
* O “lado A” conhece o “lado B” e vice-versa.
* No Java, isso significa que você consegue navegar nos dois sentidos: de `Pedido` para `ItemPedido` e de `ItemPedido` para `Pedido`.

Exemplo bidirecional 1\:N:

```java
@Entity
public class Pedido {
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "pedido")
    private List<ItemPedido> itens = new ArrayList<>();
}

@Entity
public class ItemPedido {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "pedido_id")
    private Pedido pedido;
}
```

Agora:

* `Pedido` sabe seus `itens`.
* Cada `ItemPedido` sabe qual é o `Pedido` dono dele.

---

## 4. Diferença na prática

* **Unidirecional**

  * Mais simples de modelar.
  * Útil quando você só precisa navegar em uma direção.
  * Evita manter coleções desnecessárias.

* **Bidirecional**

  * Mais completo, pois permite navegação nos dois sentidos.
  * Requer mais cuidado (precisa manter a consistência dos dois lados no código).
  * Exemplo: quando você adiciona um `ItemPedido` ao `Pedido`, também precisa setar o `pedido` no `ItemPedido`.

---

## 5. No banco de dados

Não existe diferença entre unidirecional e bidirecional no modelo relacional.
A chave estrangeira vai existir de qualquer jeito no banco de dados.
A diferença é apenas no modelo de objetos (Java), que pode conhecer em um ou nos dois sentidos.

---

**Resumo:**

* **Unidirecional:** um lado conhece o outro. Mais simples.
* **Bidirecional:** os dois lados se conhecem. Mais completo, mas exige consistência no código.

---

Observe o trecho de código a seguir que aborda o relacionamento bidirecional entre `Serie` e `Episodio`

```java
@Entity
public class Serie {
    @Id
    @GeneretedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "serie", cascade = CascadeType.ALL)
    private List<Episodio> episodios = new ArrayList<>();

    public void setEpisodios(List<Episodio> episodios) {
        episodios.forEach(e -> e.setSerie(this));
        this.episodios = episodios;
    }
} 

@Entity
public class Episodio {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Stirng nome;

    @ManyToOne
    @JoinColumn(name = "serie_id")
    private Serie serie;

    public void setSerie(Serie serie) {
        this.serie = serie;
    }
}
```

### Por que o `serie_id` estava ficando `NULL` no banco de dados?

Vamos imaginar o fluxo sem aquele trecho:

```java
Serie s = new Serie();
s.setName("Breaking Bad");

Episodio e1 = new Episodio();
e1.setNome("Pilot");

Episodio e2 = new Episodio();
e2.setNome("Cat's in the Bag...");

s.setEpisodios(List.of(e1, e2));
serieRepository.save(s);
```

O que acontece:

* O `Serie` é salvo.
* O `CascadeType.ALL` tenta salvar os `Episodio`s.
* Mas no objeto `Episodio`, o campo `serie` **nunca foi setado**.
* Ou seja, quando o Hibernate vai gerar o `INSERT` de `Episodio`, ele não tem valor para a coluna `serie_id`, porque sua refer"encia nunca foi setada → resultado: `NULL`.

---

### Como o `episodios.forEach(e -> e.setSerie(this))` resolve?

Esse trecho está dentro do setter de `Serie`:

```java
public void setEpisodios(List<Episodio> episodios) {
    episodios.forEach(e -> e.setSerie(this));
    this.episodios = episodios;
}
```

O que ele faz:

* Para cada episódio que você passa na lista…
* Ele seta a referência **de volta** para a própria `Serie` (`this`).

Assim, quando o Hibernate for persistir:

* O `Episodio` já tem o campo `serie` preenchido.
* O `INSERT` gerado vai incluir corretamente o `serie_id`.

---

### Bidirecionalidade em JPA

Esse detalhe mostra um ponto crucial:
Em JPA, relacionamentos **bidirecionais não são mágicos**.
Você tem duas variáveis (`Serie.episodios` e `Episodio.serie`) e precisa mantê-las consistentes manualmente.

É por isso que muitos guias recomendam criar **métodos auxiliares** em vez de depender de `setters` simples:

```java
public void addEpisodio(Episodio episodio) {
    episodio.setSerie(this); // garante consistência
    this.episodios.add(episodio);
}
```

Esse padrão é mais seguro que sobrescrever o `setEpisodios`.

---

### Sobre "não precisar de setters"

De fato, **setters comuns não são obrigatórios** para o JPA funcionar.
O que é obrigatório é:

* **Construtor padrão sem argumentos** (para o Hibernate instanciar o objeto).
* **Getters ou acesso direto aos campos** (para popular os dados).

Mas, quando falamos de **relacionamentos bidirecionais**, você precisa de algum mecanismo para **manter as duas pontas sincronizadas** — seja:

* via `setEpisodios` com aquele `forEach`,
* ou via `addEpisodio`/`removeEpisodio`.

Então não é que "todo setter é necessário". É que, em casos de relacionamento bidirecional, você **precisa de lógica extra** para manter coerência.

---

* O `serie_id` estava `NULL` porque você nunca setou o `Episodio.serie`.
* O `episodios.forEach(e -> e.setSerie(this))` corrige isso, mantendo os dois lados do relacionamento sincronizados.
* Não é que **todos os setters** sejam obrigatórios, mas em relacionamentos bidirecionais você precisa de **métodos auxiliares** para garantir consistência.

