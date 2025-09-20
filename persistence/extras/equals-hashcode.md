# Uso de `equals` e `hashCode`

Quando você usa `Set` (principalmente `HashSet`), o funcionamento interno depende **diretamente** de `equals()` e `hashCode()`.

---

### O problema

* O `HashSet` precisa saber **como identificar se dois objetos são iguais** para:

  * evitar duplicatas
  * garantir consistência nas operações de adicionar, remover e buscar

Se você não sobrescrever `equals()` e `hashCode()`, a comparação vai usar o comportamento padrão da classe `Object`, que considera apenas a **referência em memória**.
Ou seja:

```java
Pedido p1 = new Pedido(1L, "Notebook");
Pedido p2 = new Pedido(1L, "Notebook");

set.add(p1);
set.add(p2);
```

Mesmo que `p1` e `p2` representem a **mesma linha no banco (id=1)**, o `HashSet` vai aceitar os dois, porque são objetos diferentes em memória.

---

### Funcinamento do `Set`

`Set` (e especificamente o `HashSet`) não permite duplicados.

Para decidir se um objeto já existe lá dentro, ele faz duas etapas:

1. Calcula o `hashCode` do objeto → isso indica em qual “caixinha” do `HashSet` ele deve ir.

2. Se ocorre colisão, e já existe algo naquela caixinha, compara usando o `equals()`.

Se você **não sobrescreve** esses métodos, o comportamento padrão do `Set` é:

* `hashCode()`: retorna um número baseado no endereço de memória.

* `equals()`: só retorna true se for a mesma referência de objeto.

Então, se você buscar ou tentar inserir outro objeto que representa a mesma linha no banco, mas que foi instanciado em outro momento, o `Set` vai tratar como objeto diferente.
Resultado: **duplicatas lógicas no `Set`**.

Quando você sobrescreve `equals()` e `hashCode()` (baseando-se no id, por exemplo), o Set passa a reconhecer que esses objetos representam a mesma entidade → evita duplicatas.

---

### Implementação Correta

A prática recomendada é basear `equals()` e `hashCode()` em um **atributo imutável que identifica unicamente a entidade** (= chave primária).
Na maioria dos casos → o campo `id`.

Exemplo com Lombok:

```java
@Entity
public class Pedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String descricao;

    @ManyToOne
    private Cliente cliente;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Pedido)) return false;
        Pedido pedido = (Pedido) o;
        return id != null && id.equals(pedido.id);
    }

    @Override
    public int hashCode() {
        return 31;
    }
}
```

Agora dois objetos `Produto` com o **mesmo id são considerados iguais**, mesmo que tenham sido instanciados separadamente.

---

* **Necessário** porque `Set`/`HashSet` usa `equals()` e `hashCode()` para evitar duplicatas.
* **Sem sobrescrita:** o JPA pode encher a coleção com entidades duplicadas (mesmo id).
* **Com sobrescrita correta (baseada em id):** garante consistência e evita bugs sutis.

---

## Sobre o valor de retorno do `hashCode`

O `hashCode()` retorna sempre o mesmo número.

Isso pode parecer estranho, mas tem lógica: o contrato do Java exige que se dois objetos são iguais (equals retorna true), **eles devem ter o mesmo `hashCode`**.

Então, ao garantir o retorno da função como 31, estamos garantindo isso de forma simplificada: todo mundo tem o mesmo hash.

Isso não é o mais eficiente (pode causar colisões), mas funciona e evita bugs.


Não precisa ser exatamente 31. O valor 31 virou padrão histórico no Java porque:

* É um número primo, o que ajuda a distribuir melhor os valores no cálculo do hash.

* Multiplicar por 31 pode ser otimizado pelo compilador (31 * x == (x << 5) - x).

```java
@Override
public int hashCode() {
    return 31;
}
```
Esse é um “fallback seguro”: todo mundo cai no mesmo balde, mas o `equals()` depois diferencia.

`hashCode()` com 31 é convenção otimizada, mas poderia ser outro primo, ou ainda:

```java
@Override
public int hashCode() {
    return Objects.hash(id);
}
```

O `31` no `hashCode()` é só um atalho para não ter que pensar muito. É seguro, mas se você quiser ser mais correto e performático, use o id no cálculo do hash.



