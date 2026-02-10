# JUnit

Os métodos de teste usam `package-private` por padrão, isso significa usar o modificador de visibilidade `default` para métodos.

O modificador `package-private` é o padrão em Java e permite que membros de uma classe (métodos) sejam acessados por qualquer outra classe dentro do mesmo pacote, mas não por classes fora desse pacote, diferentemente do `public` e `private`. Ele define uma **visibilidade intermediária**, mais restritiva que public e menos que private. 

* Reduz poluição visual (menos `public` repetitivo)

* Mantém encapsulamento natural, os métodos de teste não precisam ser públicos para fora do pacote.

* Evita a falsa impressão de que métodos de teste fazem parte da API pública da classe.

---

## Ciclo de Vida

O **JUnit 5** organiza o ciclo de vida de um teste de forma que você possa preparar recursos antes de cada teste, limpar depois, ou mesmo fazer preparações gerais uma vez só.

---

Para cada classe de teste:

1. **Antes de tudo (`@BeforeAll`)** → executa **uma vez** antes de qualquer teste da classe. Ideal para configurar recursos caros ou globais (como conexões com banco ou arquivos de configuração).

2. **Antes de cada teste (`@BeforeEach`)** → executa **antes de cada método de teste**. Útil para preparar o estado do objeto ou mocks de forma limpa.

3. **Teste (`@Test`)** → o método que efetivamente testa uma funcionalidade.

4. **Depois de cada teste (`@AfterEach`)** → executa **após cada método de teste**. Bom para limpar estado ou recursos criados no `@BeforeEach`.

5. **Depois de tudo (`@AfterAll`)** → executa **uma vez** depois de todos os testes. Para liberar recursos globais, fechar conexões etc.

---

* `@BeforeAll` e `@AfterAll` **devem ser estáticos**, porque ainda não existe instância da classe de teste quando são executados.

* `@BeforeEach` e `@AfterEach` **são instanciados por teste**, então você sempre começa com objetos “limpos” em cada método.

* Essa separação garante **isolamento entre testes**, evitando efeitos colaterais de estado compartilhado.

---

### **1. @BeforeAll** – executa **uma vez antes de todos os testes**

**Quando usar:**

* Configuração de recursos **custosos** que podem ser compartilhados por todos os testes.
* Ex.: criar uma conexão com banco H2 em memória, carregar arquivos de configuração, inicializar dados globais.

```java
import org.junit.jupiter.api.*;

class BancoTest {

    static BancoH2 banco;

    @BeforeAll
    static void setupGlobal() {
        banco = new BancoH2();
        banco.conectar();
        System.out.println("Conexão com banco criada uma vez antes de todos os testes");
    }

    @Test
    void testeInserir() {
        banco.inserir("usuario1");
        Assertions.assertTrue(banco.contem("usuario1"));
    }

    @Test
    void testeRemover() {
        banco.inserir("usuario2");
        banco.remover("usuario2");
        Assertions.assertFalse(banco.contem("usuario2"));
    }

    @AfterAll
    static void cleanupGlobal() {
        banco.desconectar();
        System.out.println("Banco desconectado após todos os testes");
    }
}
```

**Por que faz sentido:** criar/destruir conexão de banco **uma vez só** é muito mais eficiente que abrir e fechar antes/depois de cada teste.

---

### **2. @BeforeEach** – executa **antes de cada teste**

**Quando usar:**

* Preparar **estado limpo** para cada teste, evitando efeitos colaterais.
* Ex.: instanciar objetos, resetar mocks, limpar listas temporárias.

```java
import org.junit.jupiter.api.*;
import java.util.ArrayList;
import java.util.List;

class ListaTest {

    List<String> lista;

    @BeforeEach
    void setup() {
        lista = new ArrayList<>(); // lista nova a cada teste
        System.out.println("Lista reiniciada antes de cada teste");
    }

    @Test
    void testeAdicionar() {
        lista.add("Milena");
        Assertions.assertEquals(1, lista.size());
    }

    @Test
    void testeVazia() {
        Assertions.assertTrue(lista.isEmpty());
    }
}
```

**Por que faz sentido:** cada teste começa com **estado previsível**, evitando que um teste altere o resultado do outro.

---

### **3. @AfterEach** – executa **depois de cada teste**

**Quando usar:**

* Limpar recursos temporários criados no teste (arquivos, conexões, dados de teste).
* Ex.: apagar arquivos temporários, resetar mocks, limpar tabelas de banco de dados em memória.

```java
import org.junit.jupiter.api.*;
import java.io.File;

class ArquivoTest {

    File tempFile;

    @BeforeEach
    void setup() throws Exception {
        tempFile = File.createTempFile("teste", ".txt");
        System.out.println("Arquivo temporário criado");
    }

    @Test
    void testeEscrever() throws Exception {
        Assertions.assertTrue(tempFile.exists());
    }

    @AfterEach
    void cleanup() {
        tempFile.delete();
        System.out.println("Arquivo temporário deletado");
    }
}
```

**Por que faz sentido:** garante que o teste **não deixa lixo** que possa afetar outros testes.

---

### **4. @AfterAll** – executa **uma vez depois de todos os testes**

**Quando usar:**

* Liberar **recursos globais** ou fechar serviços que foram inicializados em `@BeforeAll`.
* Ex.: desconectar banco, encerrar servidor de teste, limpar cache global.

```java
import org.junit.jupiter.api.*;

class ServidorTest {

    static ServidorFake servidor;

    @BeforeAll
    static void iniciarServidor() {
        servidor = new ServidorFake();
        servidor.start();
        System.out.println("Servidor iniciado");
    }

    @Test
    void testeConexao() {
        Assertions.assertTrue(servidor.estaAtivo());
    }

    @AfterAll
    static void pararServidor() {
        servidor.stop();
        System.out.println("Servidor parado após todos os testes");
    }
}
```

**Por que faz sentido:** não faz sentido iniciar e parar o servidor antes/depois de **cada teste**, isso seria muito lento.

---

### **Resumo visual da necessidade**

| Annotation    | Quando usar                                           | Exemplo prático                            |
| ------------- | ----------------------------------------------------- | ------------------------------------------ |
| `@BeforeAll`  | Configuração **global** que vale para todos os testes | Conectar banco, iniciar servidor           |
| `@BeforeEach` | Preparar **estado limpo** antes de cada teste         | Instanciar objetos, resetar mocks          |
| `@AfterEach`  | Limpar recursos do teste                              | Apagar arquivos temporários, limpar listas |
| `@AfterAll`   | Limpeza global após todos os testes                   | Desconectar banco, parar servidor          |

---

## Teste JUnit Puro

---

### **1. Estrutura de uma classe de teste JUnit 5**

Um teste JUnit puro significa: **sem Spring, sem Mockito, só JUnit** — direto no raciocínio e na lógica.

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculadoraTest {

    @Test
    void deveSomarCorretamente() {
        Calculadora calc = new Calculadora();
        int resultado = calc.somar(2, 3);
        assertEquals(5, resultado);
    }
}
```

### **Pontos-chave**

* `@Test`: marca o método como um teste a ser executado.
* A classe **não precisa ser pública**, e o método **não precisa ser público**.
* Cada teste deve ser **independente** — não dependa de testes anteriores.
* Use `asserts` para verificar resultados (é o “coração” do teste).

---

### **2. Principais *assertions* do JUnit**

Os *asserts* comparam o que o código **fez** com o que você **esperava que fizesse**.
Se não bate, o teste falha.

#### **assertEquals(expected, actual)**

Verifica se dois valores são iguais.

```java
assertEquals(5, calc.somar(2, 3));
assertEquals("Java", "Java");
```

Também aceita comparação com margem de erro (útil pra `double`):

```java
assertEquals(98.6, conversor.celsiusParaFahrenheit(37), 0.01);
```

---

#### **assertTrue(condition)** / **assertFalse(condition)**

Checa se a condição booleana é verdadeira ou falsa.

```java
assertTrue(validadorCPF.validar("12345678909"));
assertFalse(validadorCPF.validar("11111111111"));
```

---

#### **assertThrows(Class<T>, Executable)**

Verifica se um método lança uma exceção esperada.

```java
assertThrows(ArithmeticException.class, () -> calc.dividir(10, 0));
```

---

#### **assertNotNull(object)** / **assertNull(object)**

Confere se um objeto não é ou é nulo.

```java
User user = repo.buscarPorId(1L);
assertNotNull(user);
assertNull(repo.buscarPorId(99L));
```

---

#### **assertSame(expected, actual)** / **assertNotSame(expected, actual)**

Verifica se são **o mesmo objeto** (não apenas iguais em valor).

```java
User u1 = new User("Milena");
User u2 = u1;
assertSame(u1, u2);
```

---

#### **assertAll(description, ...assertions)**

Permite agrupar várias verificações sem parar no primeiro erro.

```java
assertAll("valores calculados",
    () -> assertEquals(10, calc.somar(5, 5)),
    () -> assertEquals(0, calc.subtrair(5, 5)),
    () -> assertThrows(ArithmeticException.class, () -> calc.dividir(5, 0))
);
```

---

#### **assertArrayEquals(expected[], actual[])**

Compara arrays.

```java
int[] esperado = {1, 2, 3};
int[] atual = {1, 2, 3};
assertArrayEquals(esperado, atual);
```

---

#### Diferenciando imports

---

1. **Forma “normal”**

```java
Assertions.assertEquals(5, resultado);
```

Aqui você está **chamando o método diretamente da classe** `Assertions`.
Pra isso funcionar, você precisa importar a classe:

```java
import org.junit.jupiter.api.Assertions;
```

Essa é a forma **explícita** — útil quando você quer deixar claro de onde vem o método, ou se estiver misturando frameworks (por exemplo, `Assert` do JUnit 4 e `Assertions` do JUnit 5 no mesmo projeto).

---

2. **Forma “estática”**

```java
assertEquals(5, resultado);
```

Aqui, o método foi **importado estaticamente**:

```java
import static org.junit.jupiter.api.Assertions.*;
```

Esse `import static` diz ao compilador:

> “todos os métodos estáticos da classe `Assertions` podem ser usados diretamente”.

---

**Diferença prática:** Nenhuma no comportamento — os dois chamam o mesmo método.
A diferença é apenas **de estilo e legibilidade**:

| Forma                       | Vantagem                                             | Desvantagem                                                            |
| --------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------- |
| `Assertions.assertEquals()` | explícita, clara pra quem lê código fora de contexto | mais verbosa                                                           |
| `assertEquals()`            | limpa, padrão em testes JUnit                        | pode confundir se houver imports de outros frameworks com o mesmo nome |

Na prática, o padrão é usar **`import static`** e escrever apenas `assertEquals()`.
É o estilo usado em praticamente todos os exemplos, tutoriais e projetos Spring Boot.

---

## Tipos de Cenários

Você **não precisa** se limitar a apenas dois cenários (sucesso e falha). Essa dicotomia serve bem pra testar validações simples, mas **a lógica de negócio raramente é binária**.

Em vez disso, o ideal é pensar em **cenários de comportamento** — o que acontece quando o mundo se apresenta de jeitos diferentes.

---

### **1. Cenário de sucesso**

O caso comum, o “caminho feliz”.
Serve pra garantir que o comportamento básico funciona:

```java
@Test
void deveSalvarUsuarioComDadosValidos() { ... }
```

---

### **2. Cenários de falha previsível**

Erros esperados — dados inválidos, estado incorreto, exceções de negócio.

```java
@Test
void deveLancarExcecaoQuandoEmailJaCadastrado() { ... }
```

---

### **3. Cenários de borda**

Limites matemáticos, coleções vazias, valores nulos, strings extremas.

```java
@Test
void deveTratarNomeVazioComoEntradaInvalida() { ... }
@Test
void deveAceitarValorZeroComoLimiteMinimo() { ... }
```

Esses casos pegam erros sutis — *off by one*, nulls esquecidos, divisões indevidas.

---

### **4. Cenários alternativos válidos**

Variações do sucesso — situações diferentes que ainda são consideradas corretas.

```java
@Test
void deveCalcularDescontoParaClientePremium() { ... }
@Test
void deveCalcularDescontoParaClienteComum() { ... }
```

Isso garante que sua lógica cobre **todas as ramificações legítimas**.

---

### **5. Cenários de consistência**

Quando o resultado de uma operação deve refletir um estado consistente.

```java
@Test
void deveReduzirEstoqueAposVenda() { ... }
@Test
void naoDeveAlterarEstoqueEmVendaCancelada() { ... }
```

Esses são mais próximos de testes de integração, mas ainda podem ser feitos em unidade, se você isolar bem a lógica.

---


| Tipo de cenário    | Exemplo                       | Valor do teste                 |
| ------------------ | ----------------------------- | ------------------------------ |
| Sucesso simples    | salvar usuário válido         | garante o fluxo base           |
| Falha previsível   | exceção por email duplicado   | valida tratamento de erro      |
| Borda              | string vazia, número negativo | captura casos extremos         |
| Alternativo válido | cliente premium vs comum      | cobre ramificações da regra    |
| Consistência       | estoque antes/depois          | verifica integridade do estado |

---

Crie quantos cenários fizer sentido **para a história que o código conta**.
Quanto mais nuances você cobrir, mais confiança você tem de que a lógica se sustenta fora do “mundo ideal”.

---

## Padrões de Nomenclatura de Métodos

O `@DisplayName` entra justamente quando você quer **deixar o método com um nome técnico**, mas ainda **mostrar uma descrição legível** nos relatórios de teste.

Em outras palavras:

* o nome do método serve ao **código** (curto, limpo, consistente),
* o `@DisplayName` serve ao **humano** (expressivo, narrativo, quase uma frase).

---

### **1. Estrutura mais comum**

```java
@DisplayName("Should return total price including discount")
@Test
void shouldReturnTotalPriceWithDiscount() {
    ...
}
```

Assim, o método continua seguindo a convenção em inglês e camelCase,
mas o relatório de teste mostra algo bonito, legível, com espaços e letras maiúsculas.

---

### **2. Quando o DisplayName substitui o “nome falante”**

Se você adota `@DisplayName` com consistência, pode deixar o método bem mais enxuto:

```java
@DisplayName("Should throw exception when user is not found")
@Test
void userNotFound() {
    ...
}
```

Ou até mais técnico:

```java
@DisplayName("Should calculate discount for premium customer")
@Test
void calculateDiscount_premiumCustomer() {
    ...
}
```

A ideia é que o nome do método sirva apenas como **identificador interno**,
enquanto o `@DisplayName` se encarrega de **contar a história**.

---

### **3. Boas práticas**

* Sempre escreva o `@DisplayName` em **inglês natural**, com espaços e pontuação quando fizer sentido.
* Evite repetir o nome do método no `DisplayName` — ele é complementar, não redundante.
* Use-o principalmente quando o nome do método seria grande demais pra manter legível.
* Pode usar emojis ou símbolos se quiser destacar categorias de testes (alguns times fazem isso).

---

### **4. Padrão equilibrado**

Um formato limpo e profissional que aparece muito em projetos Java modernos:

```java
class OrderServiceTest {

    @DisplayName("Should create a new order successfully")
    @Test
    void shouldCreateOrder() { ... }

    @DisplayName("Should throw exception when stock is insufficient")
    @Test
    void shouldThrowExceptionWhenStockIsInsufficient() { ... }

    @DisplayName("Should apply discount when customer is premium")
    @Test
    void shouldApplyDiscountForPremiumCustomer() { ... }
}
```

---

* sem `@DisplayName` → o nome do método precisa carregar o sentido completo.
* com `@DisplayName` → o método pode ser mais técnico, o texto explica o comportamento.

---

## Padrão de Organização de Testes

Os dois padrões de organização de testes — **Triple A (Arrange–Act–Assert)** e **GWT (Given–When–Then)** — têm o mesmo objetivo: **organizar a estrutura lógica de um teste unitário** para deixá-lo limpo, previsível e fácil de entender.
Mas eles vêm de **culturas diferentes** de desenvolvimento.

---

### **1. Triple A (Arrange – Act – Assert)**

Esse é o padrão **clássico e mais técnico**, muito usado em Java e C#.
Ele foca na **sequência lógica de execução** do teste:

1. **Arrange** → preparar o cenário (dados, mocks, objetos, dependências).
2. **Act** → executar a ação que você quer testar (chamar o método-alvo).
3. **Assert** → verificar o resultado (com `assertEquals`, `assertThrows`, etc).

---

```java
@Test
void shouldCalculateTotalWithDiscount() {
    // Arrange
    OrderService service = new OrderService();
    Order order = new Order(100.0, true); // true = cliente premium

    // Act
    double total = service.calculateTotal(order);

    // Assert
    Assertions.assertEquals(90.0, total);
}
```

O foco aqui é **estrutural**: cada etapa faz uma coisa só e na ordem certa.
Ideal quando você quer deixar o raciocínio direto e limpo — “setup → execução → verificação”.

---

### **2. GWT (Given – When – Then)**

O padrão **Given–When–Then** nasceu do **BDD (Behavior-Driven Development)**,
e é mais **semântico**, mais próximo de linguagem natural.
Ele descreve o **comportamento esperado** do sistema, não só a execução do código.

1. **Given** → contexto inicial (“dado que o cliente é premium”).
2. **When** → ação sob teste (“quando o cliente faz um pedido de 100 reais”).
3. **Then** → resultado esperado (“então o valor total deve ter 10% de desconto”).

---

```java
@Test
@DisplayName("Should apply 10% discount for premium customers")
void shouldApplyDiscountForPremiumCustomer() {
    // Given
    OrderService service = new OrderService();
    Order order = new Order(100.0, true);

    // When
    double total = service.calculateTotal(order);

    // Then
    Assertions.assertEquals(90.0, total);
}
```

Perceba que é quase idêntico ao Triple A, mas o tom é **mais narrativo e expressivo**.
Isso facilita muito a leitura de testes de comportamento (e integra melhor com frameworks de BDD, como Cucumber).

---

### **3. Diferenças conceituais**

| Aspecto    | Triple A                                 | GWT                                 |
| ---------- | ---------------------------------------- | ----------------------------------- |
| Origem     | Test-Driven Development (TDD)            | Behavior-Driven Development (BDD)   |
| Ênfase     | Estrutura do código                      | Clareza do comportamento            |
| Linguagem  | Técnica (arranjo, execução, verificação) | Narrativa (dado, quando, então)     |
| Sintaxe    | Comentários ou blocos técnicos           | Frases quase “naturais”             |
| Uso típico | Testes unitários e de integração         | Testes de comportamento e aceitação |

---

### **4. Qual usar em projetos com Spring Boot / JUnit**

Em **testes unitários Java**, o mais comum é o **Triple A**,
porque ele é direto, fácil de ler e combina com o estilo do JUnit.

Mas você pode usar **GWT como estilo de escrita** — com comentários ou `@DisplayName` —
para dar uma pegada mais semântica.

Por exemplo:

```java
@DisplayName("Given a premium customer, when making a $100 purchase, then total should be $90")
@Test
void shouldApplyDiscountForPremiumCustomer() {
    // Given
    Order order = new Order(100.0, true);
    OrderService service = new OrderService();

    // When
    double total = service.calculateTotal(order);

    // Then
    Assertions.assertEquals(90.0, total);
}
```

Isso é totalmente compatível com o JUnit moderno e ajuda muito na documentação dos testes.

---

* **Triple A** → mais técnico, mais usado em testes unitários de baixo nível.
* **GWT** → mais expressivo, ótimo pra testes que contam “histórias” (regras de negócio, BDD).
* Ambos funcionam bem; o importante é **ser consistente dentro do projeto**.

---
