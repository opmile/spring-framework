# AOP: Aspect-Oriented Programming

---

### 1. Contexto: IoC e DI no Spring

* **IoC (Inversion of Control)** → quem controla o ciclo de vida e as dependências dos objetos não é mais você no código, mas o **container do Spring**.
* **DI (Dependency Injection)** → é a forma prática de implementar IoC: em vez de você criar objetos manualmente (`new`), o Spring **injeta** as dependências certas para você.

Isso resolve o problema do **acoplamento** entre classes, deixando o código mais flexível e fácil de testar.

---

### 2. Onde entra o AOP?

* **AOP (Aspect-Oriented Programming, ou Programação Orientada a Aspectos)** aparece como **uma extensão natural de IoC/DI**.
* Se DI resolve *quem depende de quem*, o AOP resolve **o que está espalhado por todo o sistema** (cross-cutting concerns).

Exemplos de *cross-cutting concerns*:

* Logging
* Auditoria
* Segurança/autorização
* Tratamento de exceções
* Transações

Cross-cutting concerns são responsabilidades transversais que atravessam várias partes do sistema e que precisam ser tratadas de forma centralizada para evitar repetição de código.

O que AOP faz é separar esses comportamentos comuns e encapsulá-los em um módulo independente (aspecto). Assim:

* Você não precisa poluir todas as classes com o mesmo código repetitivo.

* A lógica de negócio fica “limpa” e focada só no que interessa.

* O aspecto age como uma “camada invisível” aplicada pelo framework.

---

### 3. Como o Spring usa AOP

O Spring permite que você **extraia esses comportamentos repetitivos** e os implemente como **“aspects”**.

* O container de IoC cuida de **criar os beans**.
* AOP entra e diz: “beleza, mas antes ou depois de chamar este método, rode também esse pedaço de lógica extra (o *aspect*)”.

Exemplo simplificado:

```java
@Aspect
public class LoggingAspect {
    @Before("execution(* com.exemplo.servico.*.*(..))")
    public void logBefore() {
        System.out.println("Método chamado!");
    }
}
```

Aqui, sem tocar na lógica principal, você intercepta todos os métodos de `com.exemplo.servico` e adiciona logging automático.

---

### Ligação

* **IoC/DI** → cuidam de **quem depende de quem** e de como os objetos são criados.
* **AOP** → cuida de **como esses objetos se comportam em aspectos transversais**, adicionando funcionalidades sem mexer no código central.

Em outras palavras:

* IoC/DI deixam seu código **modular e desacoplado**.
* AOP deixa seu código **limpo e livre de duplicação** nos pontos transversais.

---

#### Obs: "Transversal"?

Quando a gente fala de funcionalidade transversal (aspecto), significa que ela não fica restrita a uma única camada ou módulo, mas atravessa várias partes diferentes da aplicação.

* Logging → você precisa registrar logs em praticamente todos os serviços, repositórios e controladores.

* Segurança/autenticação → a verificação de permissão acontece em vários pontos diferentes do fluxo.

Essas funcionalidades “cortam” o sistema inteiro, aparecendo em diversas classes e camadas. É por isso que usamos o termo transversal — porque o aspecto atravessa o código principal, em vez de estar concentrado em um único lugar.

Em resumo:

* **Lógica de negócio = vertical** (focada em um fluxo específico, como cadastro de cliente).
* **Aspectos = transversais** (se espalham por todos os fluxos, como logging, segurança, auditoria).

---

## Funcionamento

A AOP surge como um **complemento à Programação Orientada a Objetos (POO)**.
Na POO você encapsula comportamento e estado em classes.
Na AOP, você encapsula **comportamentos transversais** (cross-cutting concerns) — funcionalidades que se espalham por várias classes, como **logging, segurança, transações, métricas**.

Em vez de poluir seu código principal repetindo esses pedaços, você os centraliza em um **aspecto**.

---

### 1. **Aspecto**

Um **aspecto** é a unidade central da AOP.
Pensa nele como uma **classe especial** que contém:

* **Pointcuts** → definem onde o comportamento extra deve ser inserido (ex.: "antes de qualquer método anotado com @Transactional").
* **Advice** → o que vai ser executado naquele ponto (ex.: "logar antes da execução").

Exemplo Spring:

```java
@Aspect
@Component
public class LoggingAspect {
    
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore() {
        System.out.println("Método chamado!");
    }
}
```

Esse aspecto intercepta todos os métodos de classes em `com.example.service`.

---

### 2. **Objeto alvo (Target Object)**

O **objeto alvo** é a sua classe de verdade, aquela que contém a **lógica de negócio principal**.
Exemplo: um `UserService` com métodos `criarUsuario()`, `buscarUsuario()` etc.

Na ausência da AOP, o sistema chamaria diretamente esse objeto.

---

### 3. **Objeto proxied (Proxy Object)**

O Spring não altera diretamente sua classe-alvo.
Em vez disso, ele cria um **proxy** (um "objeto intermediário") que **envolve o objeto alvo**.

1. Você pede uma instância do `UserService` ao Spring.
2. O que você recebe **não é o objeto real**, mas um **proxy** que:

   * Decide quando chamar o **objeto alvo**.
   * Injeta o comportamento extra do **aspecto** no caminho.

Isso significa que quando você chama `userService.criarUsuario()`, na verdade:

* O proxy intercepta a chamada.
* Executa os **advices** associados (ex.: logar, verificar permissões).
* Só então delega a execução ao **objeto alvo**.

---

### Como isso roda no Spring?

O Spring AOP funciona principalmente via **proxys dinâmicos**:

* Se o objeto alvo implementa uma **interface**, Spring usa **JDK Dynamic Proxy**.
* Se não implementa, ele usa **CGLIB Proxy** (gera uma subclasse em runtime).

Isso garante que sua aplicação nunca precise mexer manualmente nos objetos. O Spring injeta proxys de forma **transparente**.

---

### Exemplo Visual Simplificado

Imagina o fluxo:

```
[Cliente] → [Proxy de UserService] → [Aspectos aplicados] → [UserService real]
```

Exemplo prático com logs:

```java
// Você acha que está chamando o UserService real:
userService.criarUsuario();

// Mas na real acontece:
ProxyUserService.criarUsuario() {
   loggingAspect.before();   // loga "Entrando no método"
   userService.criarUsuario(); // chama o target object real
   loggingAspect.after();    // loga "Saindo do método"
}
```

---

### Aplicação em projetos Spring

No mundo real, AOP com Spring é muito usado para:

* **Transações (@Transactional)** → Spring intercepta o método e inicia/commita/rollback da transação.
* **Segurança (@Secured, @PreAuthorize)** → antes de rodar o método, verifica as credenciais.
* **Logs (@Slf4j + AOP custom)** → registra métricas de performance ou auditoria.
* **Cache (@Cacheable)** → verifica se o resultado já está no cache antes de executar.

Esses recursos funcionam porque o Spring gera **proxys em torno dos beans** declarados no contexto.

---

* **Aspecto**: classe que contém lógica transversal (logging, segurança etc).
* **Objeto alvo**: o bean "real" da sua aplicação, com a lógica principal.
* **Objeto proxied**: wrapper criado pelo Spring que intercepta chamadas e injeta a lógica dos aspectos.

---

## Conceitos-Chave

### 1. **Join Point (Ponto de Junção)**

* É um **ponto específico da execução do programa** onde o Spring **poderia** inserir um comportamento extra.
* Exemplos de join points no Spring:

  * A chamada de um método público (`service.save()`).
  * A execução de um construtor.
  * O tratamento de uma exceção (dependendo da AOP usada).

No Spring AOP (baseado em proxy), **join points são basicamente chamadas de métodos públicos de beans gerenciados pelo Spring**.

> Então pensa: “um join point é uma oportunidade de interceptar algo”.

---

### 2. **Pointcut (Corte)**

* Um **filtro** que seleciona quais join points você realmente quer interceptar.
* Você **não quer** interceptar todos os métodos do sistema, certo? Então usa **expressões** para escolher.

Exemplo de pointcut em Spring:

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

Isso significa: *“todos os métodos de qualquer classe dentro do pacote `service`”*.

Então pensa:

* **Join points** = todos os possíveis lugares.
* **Pointcut** = os lugares que você escolheu.

---

### 3. **Advice (Conselho)**

* É o **código que será rodado** quando um pointcut encontrar um join point.
* É a **ação concreta**: logar, iniciar transação, validar segurança etc.

Tipos de advice no Spring:

* `@Before` → executa antes do método.
* `@After` → executa depois (independente de sucesso ou erro).
* `@AfterReturning` → executa depois, mas só se não der erro.
* `@AfterThrowing` → executa se o método lançar exceção.
* `@Around` → envolve toda a execução (pode até substituir a chamada do método alvo).

---

### Como eles se conectam

Imagina o fluxo:

```
Join Point: execução do método UserService.save()
     ↓
Pointcut: "quero apenas métodos do pacote com.example.service"
     ↓
Advice: "logar antes e depois"
```

Ou seja:

* O **join point** diz *“há um método executando aqui”*.
* O **pointcut** diz *“é exatamente esse tipo de método que eu quero interceptar”*.
* O **advice** diz *“o que eu vou fazer nesse momento”*.

---

### Exemplo Spring

```java
@Aspect
@Component
public class LoggingAspect {

    // Definindo o pointcut
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}

    // Advice antes da execução do join point
    @Before("serviceLayer()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("Chamando método: " + joinPoint.getSignature().getName());
    }

    // Advice depois da execução do join point
    @After("serviceLayer()")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("Finalizou método: " + joinPoint.getSignature().getName());
    }
}
```

Aqui:

* **Join points**: todos os métodos públicos dos serviços.
* **Pointcut**: a expressão `execution(* com.example.service.*.*(..))`.
* **Advices**: `logBefore()` e `logAfter()`.

---

* **Join point** = onde *poderia* acontecer (pontos de execução).
* **Pointcut** = onde *de fato escolhi* que vai acontecer (filtro).
* **Advice** = o *que vai acontecer* (código adicional).

---

## Exemplos

### 1) Serviço que processa pedidos

```java
@Service
public class OrderService {
    public void placeOrder(String product) {
        System.out.println("Pedido realizado para: " + product);
    }

    public String confirmOrder(int orderId) {
        System.out.println("Pedido confirmado: " + orderId);
        return "Pedido #" + orderId + " confirmado!";
    }

    public void cancelOrder(int orderId) {
        throw new RuntimeException("Erro ao cancelar pedido: " + orderId);
    }
}
```

Aspecto com todos os Advices:

```java
@Aspect
@Component
public class LoggingAspect {

    // Pointcut: todas as execuções de métodos em OrderService
    @Pointcut("execution(* com.example.demo.OrderService.*(..))")
    public void orderMethods() {}

    // BEFORE: executa antes do método
    @Before("orderMethods()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("[BEFORE] Chamando: " + joinPoint.getSignature());
    }

    // AFTER: executa sempre após o método (com ou sem exceção)
    @After("orderMethods()")
    public void logAfter(JoinPoint joinPoint) {
        System.out.println("[AFTER] Finalizou: " + joinPoint.getSignature());
    }

    // AFTER RETURNING: só quando o método retorna sem erro
    @AfterReturning(pointcut = "orderMethods()", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("[AFTER RETURNING] Método: " + joinPoint.getSignature() +
                           " retornou: " + result);
    }

    // AFTER THROWING: só quando o método lança exceção
    @AfterThrowing(pointcut = "orderMethods()", throwing = "ex")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable ex) {
        System.out.println("[AFTER THROWING] Método: " + joinPoint.getSignature() +
                           " lançou: " + ex.getMessage());
    }

    // AROUND: controla a execução (pode modificar parâmetros, decidir executar ou não)
    @Around("orderMethods()")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("[AROUND - antes] Executando: " + pjp.getSignature());
        Object result = null;
        try {
            result = pjp.proceed(); // executa o método original
            System.out.println("[AROUND - depois] Sucesso!");
        } catch (Throwable ex) {
            System.out.println("[AROUND - exceção] " + ex.getMessage());
            throw ex;
        }
        return result;
    }
}
```

Se rodar o app e chamar:

```java
@Autowired
private OrderService orderService;

@PostConstruct
public void test() {
    orderService.placeOrder("Notebook");
    orderService.confirmOrder(123);
    try {
        orderService.cancelOrder(999);
    } catch (Exception e) {}
}
```

Essa será a saída esperada:

```java
[AROUND - antes] Executando: void com.example.demo.OrderService.placeOrder(String)
[BEFORE] Chamando: void com.example.demo.OrderService.placeOrder(String)
Pedido realizado para: Notebook
[AFTER] Finalizou: void com.example.demo.OrderService.placeOrder(String)
[AROUND - depois] Sucesso!

[AROUND - antes] Executando: String com.example.demo.OrderService.confirmOrder(int)
[BEFORE] Chamando: String com.example.demo.OrderService.confirmOrder(int)
Pedido confirmado: 123
[AFTER RETURNING] Método: String com.example.demo.OrderService.confirmOrder(int) retornou: Pedido #123 confirmado!
[AFTER] Finalizou: String com.example.demo.OrderService.confirmOrder(int)
[AROUND - depois] Sucesso!

[AROUND - antes] Executando: void com.example.demo.OrderService.cancelOrder(int)
[BEFORE] Chamando: void com.example.demo.OrderService.cancelOrder(int)
Erro ao cancelar pedido: 999
AFTER THROWING] Método: void com.example.demo.OrderService.cancelOrder(int) lançou: Erro ao cancelar pedido: 999
[AFTER] Finalizou: void com.example.demo.OrderService.cancelOrder(int)
[AROUND - exceção] Erro ao cancelar pedido: 999
```

---

Show! Bora dissecar isso sem enrolar 👌

---

## AspectJ Pointcut Expression Language (APEL)

É uma **linguagem declarativa** usada para descrever **em quais pontos do código (join points)** um **advice** deve ser aplicado.

No **AspectJ puro**, essas expressões podem atingir praticamente qualquer coisa no bytecode (atributos, construtores, inicializações, exceções…).
No **Spring Boot (Spring AOP)**, só temos join points que são **execuções de métodos públicos em beans Spring**.

* Ou seja, você usa a mesma sintaxe do AspectJ, mas o Spring Boot aplica apenas onde consegue via **proxies**.

---

### Uso no `@Pointcut`

No Spring Boot, você usa o `@Pointcut` para **declarar uma expressão AspectJ** que define onde um advice deve rodar.
Depois, você referencia esse pointcut nas anotações `@Before`, `@After`, `@Around`, etc.

Exemplo básico:

```java
@Aspect
@Component
public class LoggingAspect {

    // define um pointcut para qualquer método em OrderService
    @Pointcut("execution(* com.example.demo.OrderService.*(..))")
    public void orderMethods() {}

    @Before("orderMethods()")
    public void logBefore() {
        System.out.println("➡️ Antes do método!");
    }
}
```

---

### Expressões do APEL no Spring Boot

### 1. `execution()`

Intercepta chamadas a métodos.
Formato:

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```

Exemplos:

```java
execution(* com.example.dao.*.*(..))   // qualquer método em classes do pacote dao
execution(public * *(..))              // qualquer método público
execution(* save(..))                  // qualquer método chamado save
```

---

### 2. `within()`

Seleciona todos os métodos dentro de uma classe/pacote.

```java
within(com.example.service.*)    // todas as classes no pacote service
within(com.example.service.OrderService) // apenas nessa classe
```

---

### 3. `this()` / `target()`

* `this()` → bean atual implementando uma interface.
* `target()` → objeto real sendo chamado.
  Pouco usados em Spring Boot, mas servem para diferenciar **proxy** de **alvo real**.

---

### 4. `args()`

Seleciona métodos por tipo de parâmetro.

```java
args(java.lang.String)    // métodos que recebem String
args(.., Long)            // métodos cujo último parâmetro é Long
```

---

### 5. `@annotation()`

Seleciona métodos anotados com uma annotation específica.

```java
@Pointcut("@annotation(org.springframework.transaction.annotation.Transactional)")
public void transactionalMethods() {}
```

---

### Combinando expressões

Você pode compor com operadores lógicos:

* `&&` (AND)
* `||` (OR)
* `!` (NOT)

Exemplo:

```java
@Pointcut("execution(* com.example.service.*.*(..)) && @annotation(org.springframework.web.bind.annotation.GetMapping)")
public void getRequestsInServiceLayer() {}
```

---

### No Spring Boot

Como o Spring AOP é baseado em proxy:

* Só intercepta **métodos públicos** de beans gerenciados.
* Só aplica pointcuts em **execuções de métodos** (não em atributos, construtores, blocos estáticos, etc).

Portanto, dentro do Spring Boot, você vai usar basicamente:

* `execution()` → o mais comum
* `within()` → limitar a pacotes/classes
* `@annotation()` → interceptar métodos anotados

---

### Sintaxe geral do `execution()`

Normalmente o `execution()` será sempre o trigger mais usado, por isso usaremos ele. Mas a sintaxe APEL para os parâmetros de qualquer outro trigger continua o mesmo.

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```

Cada pedacinho é um **componente opcional** que você pode preencher ou usar curingas (`*`, `..`).

---

# 🔹 Componentes explicados

### 1. **`modifiers-pattern`** *(opcional)*

Permite restringir por modificadores de acesso:

* `public`, `protected`, `private`.
* No Spring AOP, como só intercepta **métodos públicos**, quase sempre você usa `public` ou simplesmente omite.

Exemplo:

```java
execution(public * *(..))   // todos métodos públicos
```

---

### 2. **`ret-type-pattern`** *(obrigatório)*

Define o tipo de retorno do método:

* `*` → qualquer tipo de retorno.
* `void` → apenas métodos void.
* `java.lang.String` → só métodos que retornam String.

Exemplo:

```java
execution(void save(..))          // só métodos void chamados save
execution(java.lang.String *())   // métodos que retornam String sem parâmetros
```

---

### 3. **`declaring-type-pattern`** *(opcional)*

Qual classe ou pacote contém o método.

* `com.example.dao.*` → todas as classes no pacote dao.
* `*Service` → qualquer classe cujo nome termina em "Service".

Exemplo:

```java
execution(* com.example.service.OrderService.*(..))   // qualquer método em OrderService
```

---

### 4. **`name-pattern`** *(obrigatório)*

Nome do método a ser interceptado.

* `save` → apenas métodos chamados `save`.
* `*` → qualquer nome.
* `get*` → qualquer método que começa com `get`.

Exemplo:

```java
execution(* *..*Service.get*(..))   // qualquer get* em classes terminando com Service
```

---

### 5. **`param-pattern`** *(obrigatório, mesmo que vazio)*

Define os tipos e quantidade de parâmetros.

* `()` → nenhum parâmetro.
* `(String)` → exatamente um parâmetro String.
* `(String, ..)` → começa com String e pode ter mais.
* `(..)` → qualquer número/tipo de parâmetros.

Exemplo:

```java
execution(* save(String))       // métodos save(String)
execution(* save(..))           // métodos save com qualquer parâmetro
execution(* *(Long, ..))        // métodos cujo primeiro param é Long
```

---

### 6. **`throws-pattern`** *(opcional, raramente usado)*

Filtra métodos pelo tipo de exceção declarada.
Exemplo:

```java
execution(* *(..) throws IOException)
```

---

### Exemplos práticos (Spring Boot)

```java
// Qualquer método público de qualquer classe no pacote service
execution(public * com.example.service.*.*(..))

// Qualquer método que começa com find, retorna List
execution(java.util.List find*(..))

// Qualquer método chamado save com parâmetro String
execution(* save(String))

// Qualquer método em classes terminando com Controller
execution(* *..*Controller.*(..))
```

---

* `execution()` é composto por:
  **\[modifiers] \[retorno] \[classe-alvo] \[nome-método] \[parâmetros] \[throws]**
* No Spring Boot você usa basicamente: **retorno, classe/pacote, nome, parâmetros**.
* O resto (modifiers, throws) quase nunca é usado porque Spring AOP só trabalha com **métodos públicos**.

---






