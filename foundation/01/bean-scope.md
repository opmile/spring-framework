# Escopo de Beans

## Singleton

### 1. O que significa escopo *singleton* no Spring?

Quando você anota um bean com `@Component` (sem `@Scope`), ele é por padrão **singleton**.

Isso quer dizer:

* O container cria **uma única instância** desse objeto no ciclo de vida da aplicação.
* Sempre que alguém pedir esse bean (`@Autowired`, `getBean()`, etc.), recebe a **mesma referência**.

---

### 2. Motivação do *singleton*: eficiência

* Criar objetos pode ser **caro** (ex.: conexão, serviço pesado, parser de config).
* Se o objeto não precisa mudar de estado a cada uso, faz sentido **compartilhar a mesma instância** entre vários clientes.

```java
@Service
public class CalculadoraImpostos {
    public BigDecimal calcular(BigDecimal valor) {
        return valor.multiply(new BigDecimal("0.1"));
    }
}
```

Esse serviço não precisa de estado interno, só aplica uma regra.
Logo, **uma instância só basta para todos os usuários**.

---

### 3. Estado do objeto no singleton

* **Ideal**: o objeto é **stateless** (sem estado mutável).
  → Assim, não importa que múltiplos clientes compartilhem a mesma instância.
* **Perigoso**: se o objeto mantém **estado interno mutável**, então esse estado também será compartilhado entre todos que usam o mesmo bean.
  → Isso gera **problemas de concorrência**.

---

### 4. Então por que singleton é padrão?

Porque a maioria dos serviços em aplicações corporativas são **stateless**:

* Serviços de negócio (`PedidoService`, `ProdutoService`)
* Repositórios de dados (`ClienteRepository`)
* Adaptadores de integração (`EmailSender`, `ApiClient`)

Ou seja: o escopo singleton **não foi escolhido para compartilhar estado entre usuários**, mas sim para **reaproveitar uma única instância quando o estado não importa**.

---

### 5. Conclusão

* O escopo **singleton** no Spring **implica que o estado interno (se houver)** será compartilhado entre todos os clientes do bean.
* Mas a ideia do Spring não é que você **compartilhe estado entre usuários**, e sim que você **não mantenha estado mutável** nesses beans.
* Por isso: singleton é ótimo para **serviços stateless**, mas inadequado para objetos que precisam de contexto específico (aí entram `prototype`, `request`, `session`).

---

### 1. Serviço **stateless** (a maioria)

```java
@Service
public class PedidoService {

    // Nenhum atributo mutável, apenas dependências injetadas
    private final PedidoRepository pedidoRepository;

    public PedidoService(PedidoRepository pedidoRepository) {
        this.pedidoRepository = pedidoRepository;
    }

    // Cada chamada recebe dados, processa e retorna — sem guardar estado
    public Pedido criarPedido(Long clienteId, List<Item> itens) {
        Pedido pedido = new Pedido(clienteId, itens);
        return pedidoRepository.save(pedido);
    }

    public BigDecimal calcularTotal(Pedido pedido) {
        return pedido.getItens().stream()
                .map(Item::getPreco)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

`PedidoService` pode ser um **singleton** no Spring porque:

* O estado do pedido vive no **objeto Pedido** (ou no banco),
* O serviço só **orquestra** a lógica.
* Se 1.000 usuários chamarem `criarPedido()`, eles não “brigam” por memória dentro do `PedidoService`.

---

### 2. Serviço **stateful** (minoria)

```java
@Service
@Scope("prototype") // precisa ser um novo objeto a cada uso
public class CarrinhoDeCompras {

    // Estado mutável: muda de acordo com o usuário
    private final List<Item> itens = new ArrayList<>();

    public void adicionarItem(Item item) {
        itens.add(item);
    }

    public BigDecimal calcularTotal() {
        return itens.stream()
                .map(Item::getPreco)
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }

    public void limpar() {
        itens.clear();
    }
}
```

Esse serviço **NÃO pode ser singleton**:

* O estado (`itens`) ficaria **compartilhado entre todos os usuários**.
* Usuário A adicionaria um produto e o Usuário B veria no carrinho também.
* Aqui, ou você cria um `prototype`, ou usa escopo `session` (dependendo da app).

---

### 3. Comparação prática

* `PedidoService` (**stateless**):

  * pode ser `singleton`
  * é thread-safe
  * escala bem (um objeto atende milhares de requisições)

* `CarrinhoDeCompras` (**stateful**):

  * não pode ser `singleton`
  * cada usuário precisa de sua própria instância
  * difícil de escalar se não tomar cuidado

---

Se você olhar para serviços típicos (cálculo de imposto, orquestração de pedido, envio de e-mail, consulta no DB), todos seguem o padrão do `PedidoService` → **não guardam estado interno entre requisições**.

Isso explica por que **a maioria esmagadora dos serviços em aplicações corporativas é stateless**.

---

Boa, Milena 😎
Essa é uma daquelas afirmações que sempre aparecem quando falamos de Spring/Java EE, mas vale **provar tecnicamente**. Vamos quebrar:

---

#### 1. **Serviços de negócio (Domain Services)**

* Ex.: `PedidoService`, `PagamentoService`, `ProdutoService`.
* O que eles fazem?

  * Validam regras de negócio
  * Coordenam entidades do domínio
  * Chamam repositórios
* Esses serviços **não precisam guardar estado entre chamadas**.
  → O estado está nas **entidades (Pedido, Cliente, Produto)**, que vivem no banco de dados.

Você pode instanciar um `PedidoService` uma vez só e usá-lo em mil requisições diferentes sem problema, porque o "estado" do pedido não está no `PedidoService`, mas sim nos objetos `Pedido` carregados do DB.

---

#### 2. **Repositórios (DAO / Repository Pattern)**

* Ex.: `ClienteRepository`, `ProdutoRepository`.
* O que fazem?

  * Encapsulam acesso a dados.
* Eles não têm estado mutável; apenas **executam queries**.
* O que muda? O **resultado da query**, não o estado do repositório.

Um `ClienteRepository` pode ser singleton, porque a mudança de estado (cadastro de cliente, exclusão) ocorre no **banco de dados**, não no repositório em si.

---

#### 3. **Adaptadores de Integração**

* Ex.: `EmailService`, `PagamentoGateway`, `ApiClient`.
* Função: disparar requisições externas.
* Eles não precisam guardar estado. O "estado" está na mensagem ou no payload enviado.

Um `EmailService` pode ser reutilizado infinitamente. O que muda é a mensagem enviada, não o objeto.

---

#### 4. **Serviços utilitários**

* Ex.: `ConversorDeMoeda`, `ValidadorDeCPF`, `GeradorDeToken`.
* 100% sem estado.
* Recebem entrada → produzem saída → não lembram de nada depois.

---

### E os casos *stateful*?

Existem, mas são **minoria**:

* Objetos que representam **sessão do usuário**
* Objetos que fazem **cache interno mutável**
* Objetos de **processamento com contexto contínuo** (ex.: fluxo de ETL que carrega pedaços em memória)

Nesses casos, você evita `singleton` e usa outro escopo (`prototype`, `request`, `session`).

---

## Como um \*\*bean `prototype` pode virar um "pseudo-singleton" sem querer?

No Spring, o **escopo do bean é respeitado pelo container**.

* Um bean `singleton` é instanciado **uma vez** e compartilhado.
* Um bean `prototype` deveria ser instanciado **sempre que pedido**.

Mas há um detalhe:

Se você tem um **singleton** que recebe um `prototype` via **injeção direta**, o Spring resolve a dependência **no momento da criação do singleton**. Ou seja, o `prototype` é instanciado **uma vez só** e fica preso dentro do singleton → perdendo o comportamento esperado.

```java
@Component
@Scope("prototype")
class Tarefa {
    private final UUID id = UUID.randomUUID();
    public UUID getId() { return id; }
}

@Service
class GerenciadorDeTarefas {
    private final Tarefa tarefa; // problema aqui!

    public GerenciadorDeTarefas(Tarefa tarefa) {
        this.tarefa = tarefa;
    }

    public void executar() {
        System.out.println("Executando tarefa " + tarefa.getId());
    }
}
```

Aqui, `GerenciadorDeTarefas` é singleton.

O Spring injeta **uma única instância de `Tarefa`** e pronto.
Mesmo sendo `prototype`, sempre será o **mesmo objeto**.
Resultado: `prototype` virou "singleton disfarçado".

---

Você precisa que o `GerenciadorDeTarefas` **peça um novo `prototype` toda vez que usar**.

### Solução com `ApplicationContextAware`

```java
@Component
@Scope("prototype")
class Tarefa {
    private final UUID id = UUID.randomUUID();
    public UUID getId() { return id; }
}

@Service
class GerenciadorDeTarefas implements ApplicationContextAware {
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        this.context = applicationContext;
    }

    public void executar() {
        // Aqui cada vez que chamamos, pedimos um novo bean prototype
        Tarefa tarefa = context.getBean(Tarefa.class);
        System.out.println("Executando tarefa " + tarefa.getId());
    }
}
```

Agora, cada chamada a `executar()` pega um **novo `Tarefa`** do container.

O `prototype` se comporta como esperado, mesmo sendo usado dentro de um `singleton`.

---

## Outras formas de resolver (modernas)

Além de `ApplicationContextAware`, o Spring recomenda alternativas mais elegantes:

1. **`ObjectFactory<T>` ou `Provider<T>`**

   ```java
   @Service
   class GerenciadorDeTarefas {
       private final ObjectFactory<Tarefa> tarefaFactory;

       public GerenciadorDeTarefas(ObjectFactory<Tarefa> tarefaFactory) {
           this.tarefaFactory = tarefaFactory;
       }

       public void executar() {
           Tarefa tarefa = tarefaFactory.getObject(); // novo a cada chamada
           System.out.println("Executando tarefa " + tarefa.getId());
       }
   }
   ```

2. **`@Lookup` method injection**

   ```java
   @Service
   abstract class GerenciadorDeTarefas {
       public void executar() {
           Tarefa tarefa = criarTarefa();
           System.out.println("Executando tarefa " + tarefa.getId());
       }

       @Lookup
       protected abstract Tarefa criarTarefa();
   }
   ```

---

* `prototype` **injetado em singleton = cai na armadilha de virar singleton**.
* Para evitar: ou usa **`ApplicationContextAware`**, ou as alternativas (`ObjectFactory`, `Provider`, `@Lookup`).
* `ApplicationContextAware` é mais "manual", mas funciona bem em casos onde você quer total controle.

---






