# Fetching

---

### 1. O que é *fetching*?

*Fetching* significa: **quando e como os dados de uma associação (relacionamento) são carregados do banco**.

Imagine o cenário da série e seus episódios (`@OneToMany`).
Quando você busca uma `Serie`, o Hibernate precisa decidir:

* **Carrego junto todos os `Episodio`s dessa série imediatamente?**
* **Ou deixo para buscar só quando (e se) o código realmente acessar `getEpisodios()`?**

Isso é controlado pelo `fetch`.

---

### 2. `FetchType.EAGER`

**Conceito:**
Carrega a associação **imediatamente**, no mesmo momento em que a entidade principal é carregada.

Exemplo:

```java
@OneToMany(mappedBy = "serie", fetch = FetchType.EAGER)
private List<Episodio> episodios;
```

O que acontece no banco:

* Você faz: `Serie s = serieRepository.findById(1L).get();`
* O Hibernate já dispara JOIN ou SELECT adicional para buscar todos os episódios dessa série.
* Ou seja, mesmo que você nunca chame `s.getEpisodios()`, a lista já está carregada.

**Prós:**

* Mais simples de usar: evita `LazyInitializationException` (aquele erro de acessar coleções fora do contexto da sessão).
* Útil quando você **sempre precisa** daquela associação junto.

**Contras:**

* Pode gerar consultas pesadas, com muitos `JOIN`s.
* Se houver várias associações EAGER, o Hibernate pode gerar um **cartesian product** (dados repetidos, consultas enormes).
* Afeta performance se você só precisava da entidade principal.

---

### 3. `FetchType.LAZY`

**Conceito:**
Carrega a associação **sob demanda** (lazy = preguiçoso).

Exemplo:

```java
@OneToMany(mappedBy = "serie", fetch = FetchType.LAZY)
private List<Episodio> episodios;
```

O que acontece no banco:

* Você faz: `Serie s = serieRepository.findById(1L).get();`
* O Hibernate só traz os dados da `Serie`.
* A lista de `episodios` fica como um **proxy** (estrutura vazia que "promete" carregar os dados depois).
* Só quando você chamar `s.getEpisodios().size()` ou iterar a lista, o Hibernate dispara uma nova consulta `SELECT * FROM episodio WHERE serie_id = ?`.

**Prós:**

* Performance melhor: você só carrega a associação se realmente precisar dela.
* Evita consultas enormes desnecessárias.
* Dá mais controle ao programador.

**Contras:**

* Se você tentar acessar os dados **fora do contexto da sessão** (ex: depois do fechamento da transação), gera `LazyInitializationException`.
* Pode exigir `JOIN FETCH` ou DTOs quando você quer carregar tudo de propósito.

---

### 4. Qual usar? (Aplicabilidade)

* **Regra de ouro:** sempre prefira **`LAZY`** por padrão.
* Use **`EAGER`** apenas quando:

  * a associação é **obrigatória** e você **sempre precisa** dela junto;
  * o conjunto de dados não é grande nem cresce descontroladamente.

Exemplo prático:

* `Serie → Episodio` → deve ser `LAZY`, porque você nem sempre precisa carregar todos os episódios só para listar as séries.
* `Pedido → Cliente` (N:1) → pode ser `EAGER`, porque sempre que você carrega um pedido, precisa saber o cliente.

---

### 5. Resumindo

* **EAGER**: “traz tudo junto já”.
* **LAZY**: “traz só quando eu pedir”.
* A boa prática é usar **LAZY por padrão** e controlar manualmente quando realmente precisa de carregamento antecipado (com `JOIN FETCH` em queries ou DTOs).

---

