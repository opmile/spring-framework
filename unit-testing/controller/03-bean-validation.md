# Casos de Teste Bean Validation

---

## 1. O erro conceitual comum

Testar **cada combinação possível de falha de validação** leva a:

* Explosão de casos de teste
* Testes redundantes
* Baixo ganho de confiança
* Alto custo de manutenção

Isso **não é prática profissional**.

---

## 2. O que você realmente precisa garantir

Quando há muitas validações em um DTO, você precisa provar apenas **três coisas** no controller test:

1. **Que a validação está ativa**

   * `@Valid` está sendo aplicado
   * O pipeline de validação do Spring funciona

2. **Que uma falha gera o status HTTP correto**

   * Normalmente `400 Bad Request`

3. **Que o contrato de erro é respeitado**

   * Estrutura do JSON de erro (se houver)
   * Campos obrigatórios aparecem

Uma vez isso provado, **o framework já garante o resto**.

---

## 3. Estratégia correta de testes (padrão de mercado)

### 3.1 Um cenário de sucesso

```text
Dados válidos → 2xx
```

Isso prova que:

* Todas as validações podem ser satisfeitas simultaneamente

---

### 3.2 Um cenário de falha representativo por grupo de validação

Exemplos de grupos:

* Campo obrigatório (`@NotNull`, `@NotBlank`)
* Formato (`@Email`, `@Pattern`)
* Tamanho (`@Size`, `@Min`, `@Max`)
* Regra semântica customizada (`@AssertTrue`, validador próprio)

Você **não testa cada anotação**, testa **cada tipo de erro**.

---

## 4. Exemplo concreto

### DTO complexo

```java
public class PedidoRequest {

    @NotNull
    private Long clienteId;

    @NotBlank
    @Size(min = 5, max = 100)
    private String descricao;

    @Min(1)
    @Max(100)
    private int quantidade;

    @Email
    private String emailContato;

    @AssertTrue
    private boolean termosAceitos;
}
```

---

### Testes recomendados

#### 1️⃣ Sucesso

```java
@Test
void deveCriarPedidoComDadosValidos() { ... }
```

---

#### 2️⃣ Falha por campo obrigatório

```java
@Test
void deveRetornar400QuandoClienteIdForNulo() { ... }
```

---

#### 3️⃣ Falha por formato

```java
@Test
void deveRetornar400QuandoEmailForInvalido() { ... }
```

---

#### 4️⃣ Falha por regra semântica

```java
@Test
void deveRetornar400QuandoTermosNaoForemAceitos() { ... }
```

Isso já cobre **toda a superfície de risco real**.

---

## 5. Onde testar validação em profundidade (se necessário)

Se você tiver:

* Validações customizadas
* Lógica complexa
* Regras condicionais

O lugar correto é **teste de unidade do validador**, não do controller.

Exemplo:

* Testar um `ConstraintValidator` isoladamente
* Testar um método que monta o DTO antes do controller

---

## 6. Regra de ouro (guarde isso)

> Controller test **não valida Bean Validation**
> Ele valida **integração do controller com o mecanismo de validação**

O Hibernate Validator já é testado pelos próprios autores.

Você só precisa provar que:

* Está ligado
* Responde corretamente
* Não deixa passar dados inválidos

---