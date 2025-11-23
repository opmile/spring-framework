# Uso de `BigDecimal` em Aplicações Financeiras

---

### 1. O problema com `double` e `Double`

O tipo `double` (e seu wrapper `Double`) é **binário e impreciso** para representar números decimais. Ele usa o padrão IEEE 754, que é ótimo pra cálculos científicos e engenharia numérica, mas não pra dinheiro.

Por exemplo:

```java
System.out.println(0.1 + 0.2);
```

Vai imprimir algo como:

```
0.30000000000000004
```

Isso acontece porque 0.1 e 0.2 não têm representação exata em binário — são aproximados.
Agora imagine isso acumulando em um sistema financeiro: centavos começam a sumir.

Além disso, `double` é **imutável**, mas não **exato**. E `Double` sofre do mesmo mal, só que com *autoboxing* e *nullability*.

---

### 2. O papel do `BigDecimal`

`BigDecimal` é uma classe da API Java feita justamente pra **representar números decimais com precisão arbitrária**.
Ele guarda o número como uma sequência de dígitos e uma escala (quantas casas decimais), sem arredondar implicitamente.

Exemplo:

```java
BigDecimal valor1 = new BigDecimal("0.1");
BigDecimal valor2 = new BigDecimal("0.2");
BigDecimal soma = valor1.add(valor2);

System.out.println(soma); // 0.3 certinho
```

Repare que o construtor usa **string**. Isso evita a conversão imprecisa de `double` pra binário antes de criar o objeto.
`new BigDecimal(0.1)` seria um erro clássico — herdaria a mesma imprecisão do double.


O que causa a imprecisão não é o `BigDecimal` em si, mas o **momento em que o número é convertido para binário**.

* O `double` representa números em **base 2** (potências de 2), enquanto nós humanos usamos **base 10**. Alguns números decimais — como 0.1, 0.2, 0.3 — não têm representação finita em binário, do mesmo jeito que 1/3 não tem representação finita em decimal.

```java
new BigDecimal(0.1)
```
o `0.1` já virou um número binário aproximado antes de chegar ao construtor.

* O `BigDecimal` apenas copia essa aproximação. Ele não tem como “consertar” — já recebeu o número errado.

Agora, quando você faz:
```java
new BigDecimal("0.1")
```
O `BigDecimal` não passa pelo sistema de ponto flutuante do Java.

Ele lê a string caractere por caractere, interpreta os dígitos diretamente em base 10 e armazena dois valores internos:

* Um `BigInteger`, que guarda todos os dígitos (sem ponto decimal).

* Um `scale`, que diz onde o ponto fica (expoente).

Então "0.1" vira internamente algo como:
```java
unscaledValue = 1
scale = 1
```
Isso significa “1 com uma casa decimal” — não há nenhuma conversão para binário nesse processo.

O número é exato, porque a base 10 é mantida do início ao fim.

---

### 3. Operações básicas

`BigDecimal` é **imutável**. Toda operação retorna um novo objeto.

```java
BigDecimal a = new BigDecimal("10.50");
BigDecimal b = new BigDecimal("2");

BigDecimal soma = a.add(b);             // 12.50
BigDecimal sub = a.subtract(b);         // 8.50
BigDecimal mult = a.multiply(b);        // 21.00
BigDecimal div = a.divide(b);           // 5.25
```

Mas atenção: divisão pode gerar resultado infinito (ex: 1/3).
Então o ideal é especificar **escala e modo de arredondamento**:

```java
BigDecimal c = new BigDecimal("1");
BigDecimal d = new BigDecimal("3");
BigDecimal resultado = c.divide(d, 2, RoundingMode.HALF_UP); // 0.33
```

Quando você faz uma operação (`add`, `multiply`, etc.), o `BigDecimal` combina os `BigInteger`s e ajusta as escalas conforme necessário, sem jamais transformar o valor em `double`.

Tudo é feito por aritmética de inteiros arbitrariamente grandes, então a precisão depende só da memória disponível.

---

### Estratégias de Arredondamento

O `BigDecimal` oferece várias estratégias, cada uma com um comportamento bem definido.
Todas estão no enum `java.math.RoundingMode`.

Aqui vai um resumo das mais usadas (e quando usar):

---

### **1. RoundingMode.HALF_UP**

> Arredonda para o vizinho mais próximo; em empate, sobe.
> Exemplo:

* 1.25 → 1.3
* 1.24 → 1.2

É o modo “matemático clássico” — o mesmo que aprendemos na escola.
É o padrão mais usado em valores monetários no Brasil.

---

### **2. RoundingMode.HALF_EVEN**

> Arredonda para o vizinho mais próximo; em empate, vai pro número par.
> Exemplo:

* 1.25 → 1.2 (porque 2 é par)
* 1.35 → 1.4 (porque 4 é par)

Esse é o chamado **“arredondamento bancário”** (*bankers’ rounding*).
Evita viés estatístico quando se arredonda grandes volumes de dados — por isso é o padrão em contabilidade e sistemas financeiros em vários países.

---

### **3. RoundingMode.DOWN**

> Corta as casas excedentes, sem arredondar.
> Exemplo:

* 1.29 → 1.2
* 1.21 → 1.2

É como fazer `floor()` para positivos e `ceil()` para negativos.
Útil quando você quer **sempre truncar** (ex: exibir apenas duas casas sem alterar o valor).

---

### **4. RoundingMode.UP**

> Sempre arredonda pra cima, independentemente do sinal.
> Exemplo:

* 1.21 → 1.3
* -1.21 → -1.3

Usado quando **não pode haver subestimação** (ex: cálculos fiscais, juros, tarifas).

---

### **5. RoundingMode.FLOOR / CEILING**

* `FLOOR`: sempre vai em direção ao **-∞**.

  * 1.29 → 1.2
  * -1.29 → -1.3
* `CEILING`: sempre vai em direção ao **+∞**.

  * 1.29 → 1.3
  * -1.29 → -1.2

São versões matemáticas mais explícitas de “pra baixo” e “pra cima”.

---

### **6. RoundingMode.HALF_DOWN**

> Igual ao HALF_UP, mas em empate arredonda pra baixo.

* 1.25 → 1.2

Pouco usado, mas útil em cálculos específicos de juros simples.

---

Um exemplo pra ver na prática:

```java
BigDecimal valor = new BigDecimal("1.255");

System.out.println(valor.setScale(2, RoundingMode.HALF_UP));   // 1.26
System.out.println(valor.setScale(2, RoundingMode.HALF_EVEN)); // 1.26
System.out.println(valor.setScale(2, RoundingMode.DOWN));      // 1.25
System.out.println(valor.setScale(2, RoundingMode.UP));        // 1.26
```

---

Em geral:

* **Aplicações financeiras:** `HALF_EVEN` (profissional) ou `HALF_UP` (mais intuitivo).
* **Corte de casas:** `DOWN`.
* **Sempre pra cima:** `UP` ou `CEILING`.

---

### 4. Comparação

Outro ponto importante: **não use `equals()` para comparar valores**.
`BigDecimal` considera a escala — ou seja, `new BigDecimal("2.0")` ≠ `new BigDecimal("2.00")` pelo `equals()`.
Use `compareTo()`:

```java
BigDecimal x = new BigDecimal("2.0");
BigDecimal y = new BigDecimal("2.00");

System.out.println(x.equals(y));     // false
System.out.println(x.compareTo(y));  // 0 (iguais em valor numérico)
```

---

### 5. Boas práticas

* Sempre **use o construtor com `String`** ou `BigDecimal.valueOf(double)` (que corrige parte da imprecisão).
* Defina **escala padrão e arredondamento** em operações financeiras.
* Prefira **constantes estáticas** (`BigDecimal.ZERO`, `BigDecimal.ONE`, etc.).
* Não misture com `double` nas contas — periga perder precisão.

---

* `double`: rápido, mas impreciso.
* `BigDecimal`: mais pesado, mas matematicamente correto.

É por isso que todo sistema bancário, contábil ou de e-commerce sério usa `BigDecimal`.
Se o resultado “errado por 0.0001” te causa dano, `BigDecimal` é o caminho.

