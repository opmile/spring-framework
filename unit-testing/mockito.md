# Mockito

---

## 1. **Contexto: o papel do Mockito**

O Mockito é usado para **simular dependências externas** da classe que você quer testar.
Ele cria “mocks” — objetos falsos que se comportam como os reais, mas com comportamento controlado.

A ideia é isolar **apenas a lógica da classe sob teste**, sem depender de banco de dados, rede, ou outros serviços.

---

## 2. **Quando usar**

Use mocks quando:

* a classe tem dependências externas (ex: repositórios, serviços, APIs);
* você quer garantir que **a interação** com a dependência foi correta (quantas vezes foi chamada, com quais parâmetros, etc.);
* você **não precisa do comportamento real** da dependência.

Não use mocks para:

* testar integração com bancos reais (aí entra H2, Testcontainers, etc.);
* testar comportamento de frameworks (ex: se o Spring funciona, isso é problema dele).

---

## 3. **Exemplo sem annotations (forma manual)**

Vamos começar com a forma explícita, pra entender o que o Mockito faz nos bastidores.

```java
class UserServiceTest {

    @Test
    void shouldReturnUserName() {
        // Arrange
        UserRepository repository = Mockito.mock(UserRepository.class);
        Mockito.when(repository.findNameById(1L)).thenReturn("Milena");

        UserService service = new UserService(repository);

        // Act
        String result = service.getUserName(1L);

        // Assert
        Assertions.assertEquals("Milena", result);
        Mockito.verify(repository).findNameById(1L);
    }
}
```

Aqui o `UserRepository` é falso — ele não consulta o banco, só responde o que você mandou (`"Milena"`).
O `verify()` checa se o método foi chamado.

---

### Sobre `verify(...)`

`verify()` serve pra confirmar que um mock foi **usado da forma esperada**.
Enquanto `when(...).thenReturn(...)` define **o que o mock deve fazer**,
`verify(...)` checa **se e como ele foi chamado**.


```java
when(repo.findById(1L)).thenReturn(user);

service.getUser(1L);

verify(repo).findById(1L); // confirma que o método foi chamado
```

Você também pode verificar **quantas vezes** ou com **quais argumentos**:

```java
verify(repo, times(2)).findById(anyLong());
verify(repo, never()).delete(any());
```

* `when(...)` → define o comportamento simulado;
* `verify(...)` → garante que a interação aconteceu como esperado.

Se o `verify()` **passar**, ele **não imprime nada** — o teste simplesmente continua e termina com sucesso.

Se o `verify()` **falhar**, o JUnit lança uma exceção (`org.mockito.exceptions.verification.junit.ArgumentsAreDifferent` ou similar) e o relatório de teste mostra o erro.

Por exemplo:

```
Wanted but not invoked:
repo.findById(1L);
Actually, there were zero interactions with this mock.
```

ou, se foi chamado com argumento diferente:

```
Argument(s) are different! Wanted:
repo.findById(1L);
Actual invocation has different arguments:
repo.findById(2L);
```

Então:

* sucesso → silêncio;
* falha → mensagem detalhada dizendo o que esperava e o que realmente aconteceu.

O `verify()` é feito pra ser usado **em testes automatizados**, não pra logar no console — a ideia é que o próprio framework de testes interprete o resultado.

---

## 4. **Usando annotations: forma mais elegante**

### **@ExtendWith(MockitoExtension.class)**

Essa annotation “liga” o Mockito ao JUnit 5.
Ela inicializa os mocks antes de cada teste.

### **@Mock**

Cria um mock automaticamente (em vez de `Mockito.mock()` manual).

### **@InjectMocks**

Cria uma instância da classe sob teste e **injeta automaticamente** os mocks nas dependências dela.

---

### **Exemplo prático**

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository userRepository; // simulação

    @InjectMocks
    private UserService userService; // classe que será testada

    @Test
    void shouldReturnUserName() {
        // Arrange
        when(userRepository.findNameById(1L)).thenReturn("Milena");

        // Act
        String result = userService.getUserName(1L);

        // Assert
        assertEquals("Milena", result);
        verify(userRepository).findNameById(1L);
    }
}
```

Agora o Mockito cuida de tudo:

* cria o mock (`userRepository`),
* injeta ele dentro de `userService`,
* e você só precisa focar na lógica e nas asserções.

---

## 5. **Quando usar cada um**

| Annotation                            | Função                                         | Onde usar                      |
| ------------------------------------- | ---------------------------------------------- | ------------------------------ |
| `@Mock`                               | Cria um objeto falso, controlado pelo Mockito  | Nas dependências simuladas     |
| `@InjectMocks`                        | Cria a classe sob teste e injeta os mocks nela | Na classe que você quer testar |
| `@ExtendWith(MockitoExtension.class)` | Conecta o Mockito com o JUnit 5                | No topo da classe de teste     |

---

## 6. **Exemplo com múltiplas dependências**

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private PaymentGateway paymentGateway;

    @Mock
    private InventoryService inventoryService;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldPlaceOrderSuccessfully() {
        when(paymentGateway.processPayment(anyDouble())).thenReturn(true);
        when(inventoryService.reserveItem(anyString())).thenReturn(true);

        boolean result = orderService.placeOrder("item-123", 100.0);

        assertTrue(result);
        verify(paymentGateway).processPayment(100.0);
        verify(inventoryService).reserveItem("item-123");
    }
}
```

O Mockito injeta **todos os mocks compatíveis com o construtor ou atributos** de `OrderService`.

---

Evite misturar:

* `@Mock` + `@Autowired` (são opostos: um simula, o outro injeta real)
* Mockito + contexto Spring (a não ser que use `@MockBean` com `@SpringBootTest`)

---

## Dintinguindo Mocks Tradicionais de BDD

---

### **1. Mesma ideia, dois estilos**

Os dois fazem **exatamente a mesma coisa** em termos de funcionalidade:

* ambos configuram o comportamento de um mock;
* ambos dizem: “quando esse método for chamado, devolva esse valor”.

A diferença está no **estilo** e na **semântica**.

---

### **2. O estilo tradicional: `when(...).thenReturn(...)`**

Esse é o estilo original do Mockito, mais “procedural”:

```java
when(repository.findById(1L)).thenReturn(Optional.of(user));
```

Lê-se como:

> “Quando `findById(1L)` for chamado, então retorne `Optional.of(user)`”.

É direto e muito usado, mas mistura um pouco o inglês com a estrutura de código — fica um pouco menos fluido quando o teste é descritivo.

---

### **3. O estilo BDD: `given(...).willReturn(...)`**

Esse vem do **BDD (Behavior-Driven Development)**, um jeito mais narrativo de escrever testes.
Ele enfatiza a ideia de **cenário e comportamento**, não apenas de chamada de método.

```java
given(repository.findById(1L)).willReturn(Optional.of(user));
```

Lê-se como:

> “Dado que `findById(1L)` retorna `Optional.of(user)`”.

É uma forma mais natural de escrever **Given / When / Then**,
que combina com a estrutura de teste BDDMockito e frameworks como Cucumber ou JBehave.

---

### **4. Comparação direta**

| Aspecto     | `when(...).thenReturn(...)`             | `given(...).willReturn(...)`            |
| ----------- | --------------------------------------- | --------------------------------------- |
| Estilo      | Tradicional (procedural)                | BDD (narrativo)                         |
| Leitura     | “Quando X acontecer, então retorne Y”   | “Dado que X retornará Y”                |
| Melhor uso  | Testes unitários simples                | Testes no formato BDD (Given/When/Then) |
| Biblioteca  | `org.mockito.Mockito`                   | `org.mockito.BDDMockito`                |
| Significado | Idêntico — só muda a forma de expressar |                                         |

---

### **5. Exemplo lado a lado**

```java
// Estilo tradicional
when(userRepository.findById(1L)).thenReturn(Optional.of(user));
User result = service.findUser(1L);
assertEquals(user, result);
verify(userRepository).findById(1L);

// Estilo BDD
given(userRepository.findById(1L)).willReturn(Optional.of(user));
User result = service.findUser(1L);
then(userRepository).should().findById(1L);
assertThat(result).isEqualTo(user);
```

Percebe como o segundo exemplo **lê como uma frase**?

> *Given* ... *When* ... *Then* ...

Essa coerência ajuda em times que usam uma linguagem mais descritiva nos testes.

---

* Se você está escrevendo testes unitários comuns → use `when(...).thenReturn(...)` sem culpa.
* Se quer que o teste soe mais “história de comportamento” → use `given(...).willReturn(...)` + `then(...).should()`.

---

## `ArgumentCaptor`

---

### **1. A ideia geral**

O `ArgumentCaptor` serve pra verificar **com quais valores** um mock foi chamado.
Enquanto `verify()` confirma que a interação aconteceu, o captor permite **analisar o conteúdo** dessa interação.

No estilo BDD, isso vira:

```java
then(mock).should().metodo(argumentCaptor.capture());
```

Depois você usa `argumentCaptor.getValue()` pra verificar o que foi capturado.

---

### **2. Exemplo prático**

Imagine o mesmo serviço:

```java
class UserService {
    private final UserRepository repo;

    UserService(UserRepository repo) {
        this.repo = repo;
    }

    void createUser(String name) {
        User user = new User(name.toUpperCase());
        repo.save(user);
    }
}
```

---

### **3. Teste com BDDMockito**

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.mockito.BDDMockito.*;
import static org.junit.jupiter.api.Assertions.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository repo;

    @InjectMocks
    private UserService service;

    @Captor
    private ArgumentCaptor<User> userCaptor; // vai capturar o User passado ao repo

    @Test
    void shouldSaveUserWithUppercaseName() {
        // Given
        String name = "milena";

        // When
        service.createUser(name);

        // Then
        then(repo).should().save(userCaptor.capture());
        User captured = userCaptor.getValue();

        assertEquals("MILENA", captured.getName());
    }
}
```

---

### **4. O fluxo em BDD**

O teste lê de forma mais natural:

* **Given:** cenário inicial configurado (mock pronto, dados definidos)
* **When:** ação sendo testada (`service.createUser(name)`)
* **Then:** verificação de comportamento (`then(repo).should()...`)

A chamada

```java
then(repo).should().save(userCaptor.capture());
```

é funcionalmente idêntica a

```java
verify(repo).save(userCaptor.capture());
```

mas soa mais fluida e narrativa, ideal pra testes de comportamento.

---

### **5. Extras**

Você também pode capturar **múltiplas chamadas**:

```java
then(repo).should(times(2)).save(userCaptor.capture());
List<User> all = userCaptor.getAllValues();
```

Ou verificar que **nada mais foi chamado**:

```java
then(repo).shouldHaveNoMoreInteractions();
```

---

### **6. Em resumo**

| Função                  | Mockito tradicional                     | BDDMockito                                     |
| ----------------------- | --------------------------------------- | ---------------------------------------------- |
| Verificar interação     | `verify(mock)`                          | `then(mock).should()`                          |
| Capturar argumento      | `verify(mock).metodo(captor.capture())` | `then(mock).should().metodo(captor.capture())` |
| Acessar valor capturado | `captor.getValue()`                     | `captor.getValue()` (igual)                    |

---

## `@Spy`

---

### Antes de tudo: o que o Mock “não consegue” fazer

Um **Mock** é completamente vazio. Ele não tem comportamento real — ele só **responde o que você configurar** com `when(...).thenReturn(...)`.

Isso é ótimo quando:

* Você quer isolar **totalmente** a classe que está testando;
* E não precisa nem quer que a lógica interna da dependência rode.

Mas… o Mock **não serve** quando você quer **testar parcialmente o comportamento real** de um objeto, mantendo parte do seu funcionamento.

E é aí que o **Spy** entra.

---

### 1. Quando o Spy é útil: testando métodos que chamam outros métodos internos da mesma classe

Um `Mock` não consegue interceptar chamadas **dentro da própria classe**.
Veja esse exemplo clássico:

```java
public class PaymentService {

    public void processPayment(String userId, double value) {
        validateUser(userId);
        System.out.println("Processando pagamento de " + value);
    }

    void validateUser(String userId) {
        if (userId == null) throw new IllegalArgumentException("Usuário inválido");
    }
}
```

Agora imagine que você quer testar **somente o comportamento de `processPayment()`**,
mas **não quer que `validateUser()` seja executado** (por exemplo, porque ele acessa o banco de dados).

Com `@Mock`, isso não funciona, porque o mock **não executa nada real**, e você não conseguiria testar `processPayment()` de verdade.
Mas com `@Spy`, funciona perfeitamente:

```java
@ExtendWith(MockitoExtension.class)
class PaymentServiceTest {

    @Spy
    private PaymentService service;

    @Test
    void shouldSkipValidateUser() {
        doNothing().when(service).validateUser(anyString());

        service.processPayment("user123", 100.0);

        then(service).should().validateUser("user123");
    }
}
```

Aqui o método `processPayment()` **rodou de verdade**,
mas a chamada interna a `validateUser()` foi **interceptada e silenciada**.
Isso é impossível com Mock.

---

### 2. Quando você precisa verificar chamadas internas (self-invocation)

Suponha uma classe de negócio mais complexa:

```java
public class UserService {

    public void registerUser(String email) {
        sendWelcomeEmail(email);
        saveToDatabase(email);
    }

    void sendWelcomeEmail(String email) {
        System.out.println("Enviando email para: " + email);
    }

    void saveToDatabase(String email) {
        System.out.println("Salvando no banco: " + email);
    }
}
```

Com um `@Spy`, você pode garantir que o método `registerUser()` **chama internamente** os outros métodos:

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Spy
    private UserService userService;

    @Test
    void shouldCallEmailAndDatabaseMethods() {
        userService.registerUser("mila@example.com");

        then(userService).should().sendWelcomeEmail("mila@example.com");
        then(userService).should().saveToDatabase("mila@example.com");
    }
}
```

Isso é **muito útil para verificar o fluxo interno** de métodos *sem precisar depender de mocks externos.*

---

### 3. Quando você quer testar *parte* da lógica real + mockar dependências externas

Imagine que `OrderService` usa um `PaymentGateway`:

```java
public class OrderService {

    private final PaymentGateway gateway;

    public OrderService(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    public String processOrder(String userId, double value) {
        boolean success = gateway.pay(userId, value);
        if (success) {
            sendConfirmation(userId);
            return "OK";
        }
        return "FAIL";
    }

    void sendConfirmation(String userId) {
        System.out.println("Enviando confirmação para: " + userId);
    }
}
```

Agora queremos:

* Rodar **a lógica real** de `processOrder()`
* Mas **mockar o gateway**, porque ele acessa uma API externa.

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private PaymentGateway gateway;

    @Spy
    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldSendConfirmationWhenPaymentSucceeds() {
        given(gateway.pay(anyString(), anyDouble())).willReturn(true);

        orderService.processOrder("u123", 500.0);

        then(orderService).should().sendConfirmation("u123");
    }
}
```

Aqui o `@Spy` permite testar **a lógica real da classe** com **mock das dependências** — o que é bem comum em `@Service` do Spring.

---

### 4. Quando você quer “espiar” o comportamento de uma dependência real (ex: cache, DAO)

Um uso avançado: às vezes você quer medir *quantas vezes* um método é chamado,
mas precisa que ele execute o comportamento real.

Exemplo:

```java
@Spy
private UserRepository repository; // Repositório real, não mockado
```

Você pode testar:

```java
then(repository).should(times(1)).findById(any());
```

Assim, o teste roda de verdade, mas você ainda **espiona as interações internas** — algo que não seria possível com Mock.

---

| Cenário                                                    | Use Mock | Use Spy |
| ---------------------------------------------------------- | -------- | ------- |
| Quer simular dependência externa (banco, API, repositório) | ✅        | ❌       |
| Quer rodar lógica real da classe testada                   | ❌        | ✅       |
| Quer interceptar chamadas internas da mesma classe         | ❌        | ✅       |
| Quer evitar executar partes específicas (ex: log, email)   | ❌        | ✅       |
| Quer testar só o contrato (input/output previsível)        | ✅        | ❌       |

---

## `Asserts` com `Mockito`

---

### **1. Quando você usava só JUnit**

Você testava **valores de retorno** ou **efeitos observáveis**.
Algo assim:

```java
int resultado = calculadora.somar(2, 3);
assertEquals(5, resultado);
```

Simples: você quer saber **o que a função devolve**.

---

### **2. Quando entra o Mockito, o foco muda**

Mocks entram quando você **não quer testar a implementação interna** de algo, mas **como ele se comporta com dependências**.
Você para de verificar **“o que volta”** e passa a verificar **“o que foi chamado”**.

Exemplo:

```java
class UserService {
    private final UserRepository repository;
    UserService(UserRepository repository) { this.repository = repository; }

    void register(String name, String email) {
        if (!repository.existsByEmail(email)) {
            repository.save(new User(name, email));
        }
    }
}
```

Agora, no teste, você **não quer testar o `repository.save()`**, porque ele é de outro sistema.
Você só quer garantir **que o `Service` faz a chamada certa** quando deve.

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    private UserRepository repository;

    @InjectMocks
    private UserService service;

    @Test
    void shouldSaveWhenEmailDoesNotExist() {
        given(repository.existsByEmail("a@b.com")).willReturn(false);

        service.register("Milena", "a@b.com");

        // Aqui não tem assert. O foco é verificar interação:
        then(repository).should().save(any(User.class));
    }
}
```

Nenhum `assertEquals`, porque **não há retorno para verificar**.
Você está testando *interação*, não *estado*.

---

### **3. Mas nem todo teste com Mockito dispensa asserts**

Se o método testado **retorna algo** ou **gera um resultado calculado**, você ainda usa assert normalmente:

```java
given(repository.existsByEmail("a@b.com")).willReturn(false);
given(repository.save(any(User.class))).willReturn(new User("Milena", "a@b.com"));

User result = service.register("Milena", "a@b.com");

assertEquals("Milena", result.getName()); // continua JUnit puro
```

Mockito **não substitui os asserts** — ele só adiciona a possibilidade de verificar **interações**:

* se um método foi chamado,
* quantas vezes,
* com quais parâmetros,
* em que ordem.

---

### **4. Resumindo a lógica mental**

| Tipo de teste                           | O que verificar                 | Como verificar                                     |
| --------------------------------------- | ------------------------------- | -------------------------------------------------- |
| **Lógica interna / cálculo / retorno**  | valores, estado, exceções       | `assertEquals`, `assertTrue`, `assertThrows`       |
| **Interação com dependências mockadas** | chamadas, argumentos, sequência | `verify()`, `then(...).should()`, `ArgumentCaptor` |

---

### **5. Regra prática**

Pensa assim:

> “O método que estou testando devolve algo que posso comparar?”
> → Usa `assert`.
>
> “Ou ele apenas **chama outros métodos** como efeito colateral?”
> → Usa `then(...).should()` ou `verify()`.

---

## `InOrder` 

---

### **1. O que é `InOrder`**

`InOrder` é uma classe do Mockito que serve para **verificar se chamadas a mocks aconteceram em uma ordem específica**.
Não se trata de se o método foi chamado ou não — isso o `then().should()` já faz —, mas **se as chamadas ocorreram na sequência correta entre múltiplos mocks ou múltiplas chamadas de um mesmo mock**.

---

### **2. Como você cria um `InOrder`**

Existem dois jeitos comuns:

#### a) Passando **um ou mais mocks**:

```java
InOrder ordem = inOrder(mock1, mock2);
```

* `mock1` e `mock2` são os objetos que você quer monitorar.
* O Mockito irá rastrear todas as chamadas desses mocks **na ordem em que ocorreram**.

Exemplo com o `PagamentoService`:

```java
InOrder ordem = inOrder(usuarioService, gateway);
```

Aqui você quer garantir que:

1. `usuarioService.buscarPorId()` seja chamado primeiro,
2. depois `gateway.processar()`,
3. depois `usuarioService.atualizarStatus()`.

---

#### b) Criando `InOrder` com apenas um mock:

```java
InOrder ordem = inOrder(mock1);
```

Isso é útil se você quer garantir a ordem **das chamadas dentro do mesmo mock**, por exemplo, dois métodos diferentes chamados no mesmo objeto.

---

### **3. Como usar o `verify` com `InOrder`**

Depois de criar o `InOrder`, você precisa usar **`ordem.verify(...)`** em vez de `then(mock).should()`.

```java
ordem.verify(usuarioService).buscarPorId(1L);
ordem.verify(gateway).processar(usuario, 100.0);
ordem.verify(usuarioService).atualizarStatus(usuario);
```

**Por que não podemos usar `then().should()` aqui?**

* `then(mock).should()` é **independente da ordem de chamadas**. Ele só verifica que uma chamada **aconteceu**.
* `InOrder` precisa **capturar a sequência exata de todas as chamadas**.
* O Mockito **não integra `then().should()` com `InOrder`**, então a única forma de garantir a ordem é usar `ordem.verify(...)`.

---

| Conceito              | Quando usar                                   | Sintaxe                                                                                 |
| --------------------- | --------------------------------------------- | --------------------------------------------------------------------------------------- |
| `then(mock).should()` | Verificar que um método foi chamado (BDD)     | `then(mock).should().metodo();`                                                         |
| `InOrder`             | Verificar a sequência de chamadas entre mocks | `InOrder ordem = inOrder(mock1, mock2); ordem.verify(mock1)...; ordem.verify(mock2)...` |

* Se você só quer saber **que foi chamado**, use `then().should()`.
* Se você quer saber **em que ordem foram chamadas**, use `InOrder` + `verify()`.

---







