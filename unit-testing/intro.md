# Uma Introdução aos Testes Unitários

---

### 1. Por que existem testes automatizados

Testar código é garantir que ele faz o que promete. Manualmente, isso significa rodar o programa, digitar entradas e checar saídas. Funciona, mas cansa e erra fácil. Testes automatizados são só programas que testam outros programas: você escreve um conjunto de verificações que rodam sozinhas, sempre que quiser.

Com isso, você ganha:

* **Confiança em mudanças:** pode refatorar sem medo, sabendo que o teste vai te avisar se algo quebrou.
* **Documentação viva:** o teste mostra como o código é esperado se comportar.
* **Menos regressões:** evita aquele clássico “corrigi um bug e criei outro”.

### 2. Onde entra o *JUnit*

JUnit é o **framework de testes unitários** do ecossistema Java.
Ele te dá a estrutura para:

* definir o que é um teste (`@Test`),
* preparar e limpar o ambiente (`@BeforeEach`, `@AfterEach`),
* e verificar resultados (`Assertions.assertEquals`, `assertThrows`, etc).

Exemplo bobo:

```java
class Calculadora {
    int somar(int a, int b) {
        return a + b;
    }
}

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculadoraTest {
    @Test
    void deveSomarCorretamente() {
        Calculadora calc = new Calculadora();
        assertEquals(5, calc.somar(2, 3));
    }
}
```

Isso é **JUnit puro**: você testa uma unidade (função, método ou classe) de forma isolada, sem dependências externas.

### 3. Onde entra o *Mockito*

Mockito é um **framework de mock/stub**, usado **junto com o JUnit**, não no lugar dele.
Ele serve quando a unidade que você quer testar depende de outra — por exemplo, um serviço que chama um repositório ou API. Você não quer chamar o banco de verdade em um teste unitário; quer apenas simular a resposta.

Exemplo:

```java
class UserService {
    private final UserRepository repo;

    UserService(UserRepository repo) {
        this.repo = repo;
    }

    User buscarPorId(long id) {
        return repo.findById(id).orElseThrow();
    }
}
```

Agora o teste com **JUnit + Mockito**:

```java
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class UserServiceTest {
    @Test
    void deveRetornarUsuarioQuandoEncontrado() {
        UserRepository repo = mock(UserRepository.class);
        User user = new User("Milena", "milena@email.com");
        when(repo.findById(1L)).thenReturn(Optional.of(user));

        UserService service = new UserService(repo);
        User resultado = service.buscarPorId(1L);

        assertEquals("Milena", resultado.getNome());
        verify(repo).findById(1L);
    }
}
```

Aqui:

* **JUnit** define e executa o teste.
* **Mockito** cria o “fake” do `UserRepository` para não depender de um banco real.

### 4. Quando usar cada um

| Situação                                                                                | Ferramenta principal                           | Observação                             |
| --------------------------------------------------------------------------------------- | ---------------------------------------------- | -------------------------------------- |
| Testar uma classe isolada, sem dependências                                             | **JUnit**                                      | Teste puro, direto.                    |
| Testar comportamento de uma classe que depende de outras (serviços, repositórios, APIs) | **JUnit + Mockito**                            | Mockito simula as dependências.        |
| Testar integração real (banco, API real)                                                | **Spring Boot Test**, **Testcontainers**, etc. | São testes de integração, outro nível. |

### 5. Linha de raciocínio

1. Aprenda **JUnit** primeiro — só com lógica pura e métodos simples.
2. Depois traga **Mockito**, quando seus métodos começarem a depender de outros componentes.
3. Por fim, veja testes de integração com Spring Boot e bancos reais.

---

## Integração `@SpringBootTest` vs. Unitário `Mockito`

Boa — essa é uma dúvida que separa quem só “roda o teste” de quem começa a **entender** o que está sendo testado.

A diferença é de **intenção e escopo**:

---

### 1. Quando você usa `@Autowired`

Field injection (`@Autowired `direto no atributo) é tolerada em testes unitários, porque o objetivo ali é rapidez e isolamento, não necessariamente manter o código limpo ou facilmente testável.

Você está **pedindo que o Spring injete o bean real** no seu teste.
Isso significa:

* o Spring vai subir o *contexto da aplicação* (ou parte dele);
* as dependências reais serão injetadas;
* e, portanto, seu teste não é mais **unitário**, e sim **de integração**.

Exemplo:

```java
@SpringBootTest
class UserServiceIntegrationTest {

    @Autowired
    private UserService service; // injeta o bean real

    @Test
    void deveBuscarUsuarioDoBancoReal() {
        User user = service.buscarPorId(1L);
        assertEquals("Milena", user.getNome());
    }
}
```

Esse tipo de teste verifica se o Spring configurou tudo direito, se o repositório conversa com o banco, se as anotações JPA estão certas — mas roda mais lento e depende do ambiente.

---

### 2. Quando você faz o `mock(...)`

Você **não quer o bean real**, quer apenas **simular o comportamento** de uma dependência.
Isso mantém o teste **100% unitário** e sem precisar do Spring ou de um banco.

Exemplo:

```java
class UserServiceTest {

    @Test
    void deveBuscarUsuarioSimulado() {
        UserRepository repo = mock(UserRepository.class);
        when(repo.findById(1L)).thenReturn(Optional.of(new User("Milena", "email")));

        UserService service = new UserService(repo);

        User resultado = service.buscarPorId(1L);

        assertEquals("Milena", resultado.getNome());
    }
}
```

Nenhum contexto do Spring é carregado. É só o Java puro + Mockito.
Rápido, isolado e totalmente controlado.

---

### 3. O meio-termo: `@MockBean`

Quando você precisa de um teste Spring (por exemplo, pra testar o `@Service` com injeção funcionando), mas **quer substituir uma dependência específica**, você usa:

```java
@SpringBootTest
class UserServiceWithMockTest {

    @Autowired
    private UserService service;

    @MockBean
    private UserRepository repo;

    @Test
    void deveBuscarUsuarioComMockBean() {
        when(repo.findById(1L)).thenReturn(Optional.of(new User("Milena", "email")));

        User resultado = service.buscarPorId(1L);

        assertEquals("Milena", resultado.getNome());
    }
}
```

Aqui o Spring ainda sobe, mas ele injeta **o mock** no lugar do bean real do `UserRepository`.

---

### Em resumo

| Tipo         | Usa Spring? | Velocidade | Testa integração real? | Ideal pra                                     |
| ------------ | ----------- | ---------- | ---------------------- | --------------------------------------------- |
| `mock(...)`  |   Não       |   Rápido   |   Não                  | Teste **unitário puro**                       |
| `@Autowired` |   Sim       |   Lento   |   Sim                  | Teste **de integração**                       |
| `@MockBean`  |   Sim       |   Médio   |   Parcial              | Teste **de serviço isolado** dentro do Spring |

---

## `Mockito` e Banco `H2`

A diferença entre **Mockito** e **H2** é o **nível de realidade** que você quer no teste.

---

### **Mockito** → simulação total

O Mockito não cria um banco, nem acessa nada externo. Ele **finge o comportamento** do seu repositório.
Você diz exatamente o que ele deve responder:

```java
when(repo.findById(1L)).thenReturn(Optional.of(user));
```

Ou seja: o repositório “acha” que existe esse usuário, mas na verdade é só você dizendo pra ele agir assim.

Isso serve pra **testes unitários**, quando:

* você quer testar **só a lógica da sua classe**,
* e não se importa se o banco funciona ou não.

**Vantagem:** rápido, previsível, não depende de nada externo.
**Limite:** não testa se o repositório ou o banco realmente funcionam — só o comportamento esperado.

---

### **Banco H2** → simulação realista

O H2 é um **banco de dados de verdade**, só que **em memória** (ou em arquivo temporário).
Quando você usa ele em testes, o Spring cria o contexto e o banco inteiro, com tabelas e SQL executando normalmente.

Serve pra **testes de integração**, quando:

* você quer garantir que seu `UserRepository` funciona de fato,
* que as queries, mapeamentos JPA e transações estão certas.

Exemplo:

```java
@DataJpaTest
class UserRepositoryTest {
    @Autowired
    private UserRepository repo;

    @Test
    void deveSalvarERecuperarUsuario() {
        repo.save(new User("Milena", "email"));
        assertEquals(1, repo.findAll().size());
    }
}
```

Aqui o teste usa **o repositório real**, e o H2 é só o banco que o sustenta durante o teste.

**Vantagem:** garante que o repositório e as entidades estão corretos.
**Limite:** mais lento, depende de contexto Spring, e não serve pra testar lógica de negócio pura.

---

### Em termos simples

| Ferramenta  | Nível           | O que testa                         | O que não testa             |
| ----------- | --------------- | ----------------------------------- | --------------------------- |
| **Mockito** | Simulação       | Lógica de negócio (sem banco)       | Configuração, JPA, SQL      |
| **H2**      | Integração real | Persistência, repositórios, queries | Lógica pura sem dependência |

---

* **Mockito** finge que o banco respondeu “sim”.
* **H2** realmente responde, só que num banco temporário.

---

## Níveis de Teste de uma Aplicação

---

## **1. Nível superficial – Testes Unitários (só JUnit)**

**Objetivo:** testar *lógica pura* — uma classe, um método, uma função.
Nada de banco, nada de rede, nada de Spring.

**Quando usar:**

* Código que não depende de serviços, repositórios ou APIs.
* Classes utilitárias (ex: cálculos, validações, formatações, parsers).
* Métodos que recebem entrada e retornam saída previsível.

**Ferramentas:**

* `JUnit` puro.
* `Assertions` (`assertEquals`, `assertTrue`, etc).

**Exemplo típico:**

```java
@Test
void deveCalcularMediaCorretamente() {
    Calculadora calc = new Calculadora();
    assertEquals(5, calc.media(4, 6));
}
```

* Testa lógica. Nada mais.

---

## **2. Nível intermediário – Testes Unitários com Mocks (JUnit + Mockito)**

**Objetivo:** testar a *lógica de negócio* de uma classe que depende de outras,
mas **sem executar essas dependências reais**.

**Quando usar:**

* Testar classes de **serviço** (`@Service`) que usam repositórios (`@Repository`) ou outros serviços.
* Quando a lógica depende de resultados de chamadas externas, mas você quer isolar o código.

**Ferramentas:**

* `JUnit` + `Mockito`
* Mocks (`mock()`, `@Mock`) e comportamentos simulados (`when(...).thenReturn(...)`).

**Exemplo típico:**

```java
@Test
void deveRetornarUsuarioQuandoEncontrado() {
    UserRepository repo = mock(UserRepository.class);
    when(repo.findById(1L)).thenReturn(Optional.of(new User("Milena")));

    UserService service = new UserService(repo);
    assertEquals("Milena", service.buscarPorId(1L).getNome());
}
```

* Testa a lógica da **classe**, não do banco.
A dependência (`repo`) é só uma simulação.

---

## **3. Nível profundo – Testes de Integração**

**Objetivo:** verificar se os componentes da aplicação se comunicam corretamente.
Aqui o foco não é mais “lógica pura”, mas “as peças encaixam de verdade?”.

**Quando usar:**

* Verificar se o Spring cria e injeta beans corretamente (`@Autowired`).
* Testar **repositórios reais** com H2 ou Testcontainers.
* Testar **serviços reais** que interagem entre si.
* Testar **controllers** respondendo via HTTP.

**Ferramentas:**

* `@SpringBootTest` (carrega todo o contexto Spring)
* Ou alternativas leves:

  * `@DataJpaTest` → só JPA e banco
  * `@WebMvcTest` → só controllers e camada web

**Exemplo típico (serviço + H2):**

```java
@SpringBootTest
class UserServiceIntegrationTest {

    @Autowired
    private UserService service; // Bean real

    @Test
    void deveSalvarERecuperarUsuario() {
        User user = service.salvar(new User("Milena", "email"));
        assertNotNull(user.getId());
    }
}
```

* Testa o sistema funcionando “de verdade”, mas em ambiente controlado.

---

## **Comparativo rápido**

| Cenário                            | Tipo de teste       | Ferramentas                        | Contexto Spring | Testa banco real?   | Velocidade | Exemplo                      |
| ---------------------------------- | ------------------- | ---------------------------------- | --------------- | ------------------- | ---------- | ---------------------------- |
| Função ou classe isolada           | Unitário puro       | JUnit                              |   Não carrega   |        Não          |   Rápido   | `CalculadoraTest`            |
| Serviço com dependências simuladas | Unitário com mocks  | JUnit + Mockito                    |   Não carrega   |        Não          |   Rápido   | `UserServiceTest`            |
| Repositório JPA real               | Integração JPA      | JUnit + Spring (`@DataJpaTest`)    | ✅ Parcial      |         H2          |   Médio    | `UserRepositoryTest`         |
| Serviço real + banco real          | Integração completa | JUnit + Spring (`@SpringBootTest`) | ✅ Total        |  H2/Testcontainers  |    Lento   | `UserServiceIntegrationTest` |
| Controller HTTP                    | Integração Web      | JUnit + Spring (`@WebMvcTest`)     | ✅ Parcial      |        Não          |    Médio   | `UserControllerTest`         |

---

Em essência:

* **JUnit** → garante que *teu código funciona*.
* **Mockito** → garante que *tua lógica funciona mesmo sem as dependências*.
* **Spring Boot Test** → garante que *tudo funciona junto*.






