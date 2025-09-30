# Padrão Strategy

---

## A motivação do Strategy

Imagina que você tem um método que precisa executar uma “ação” que varia dependendo da situação.
Um jeito ruim seria encher o método de **`if`** ou **`switch`**, algo assim:

```java
public double calcularFrete(String tipoEntrega, double peso) {
    if (tipoEntrega.equals("Sedex")) {
        return peso * 10;
    } else if (tipoEntrega.equals("PAC")) {
        return peso * 5;
    } else if (tipoEntrega.equals("RetiradaNaLoja")) {
        return 0;
    }
    return -1;
}
```

Isso quebra o **princípio do aberto/fechado (OCP)**: cada vez que surge um novo cálculo de frete, você precisa alterar essa função, criando uma bagunça.

---

### Obs: princípio do aberto/fechado (OCP)

É um dos pilares do design orientado a objetos e parte dos princípios SOLID, afirma que as entidades de software (como classes, módulos e funções) devem ser abertas para extensão, mas fechadas para modificação.

* É possível adicionar novas funcionalidades ou modificar o comportamento do sistema sem ter que alterar o código-fonte já existente. 

#### **Funcionamento:**

* **Aberto para extensão:** Quando surge um novo requisito, em vez de alterar o código original, você cria novas classes ou módulos que estendem o comportamento da classe ou funcionalidade existente.

* **Fechado para modificação:** O código que já está funcionando e foi testado não deve ser modificado para adicionar novas funcionalidades, pois isso pode introduzir bugs e impactar outras partes do sistema. 

#### Exemplificação

Imagine um sistema de cálculo de impostos que precisa calcular diferentes tipos de impostos (ICMS, ISS, IPI). 

Em vez de modificar a classe principal de impostos para cada novo tipo, o OCP sugere criar *novas classes para cada imposto* que *herdam de uma interface* ou classe abstrata *comum*. 

Assim, a classe original permanece fechada, enquanto novas funcionalidades são adicionadas através de novas classes. 

---

## A ideia do Strategy

O Strategy propõe que você extraia cada **algoritmo/estratégia** para uma classe separada, mas todas elas obedecem a uma **mesma interface**.

* Uma **interface Strategy** define a operação que pode variar.
* As **classes concretas** implementam essa interface cada uma com sua lógica.
* Uma **classe de contexto** recebe um Strategy e o utiliza sem se preocupar com qual implementação está rodando.

Assim, o código que usa a estratégia fica desacoplado da lógica interna de cada variação.

---

## Estrutura (bem canônica)

1. **Strategy (interface/abstração)** – define a operação.
2. **ConcreteStrategies** – implementações específicas do algoritmo.
3. **Context** – usa a estratégia, delegando a execução sem precisar saber qual delas é.

---

## Exemplo em Java (cálculo de frete)

### 1. Interface Strategy

```java
public interface CalculadoraFrete {
    double calcular(double peso);
}
```

### 2. Estratégias concretas

```java
public class FreteSedex implements CalculadoraFrete {
    @Override
    public double calcular(double peso) {
        return peso * 10;
    }
}

public class FretePAC implements CalculadoraFrete {
    @Override
    public double calcular(double peso) {
        return peso * 5;
    }
}

public class FreteRetiradaLoja implements CalculadoraFrete {
    @Override
    public double calcular(double peso) {
        return 0;
    }
}
```

### 3. Contexto

```java
public class ServicoFrete {
    private final CalculadoraFrete calculadora;

    public ServicoFrete(CalculadoraFrete calculadora) {
        this.calculadora = calculadora;
    }

    public double calcularFrete(double peso) {
        return calculadora.calcular(peso);
    }
}
```

### 4. Uso

```java
public class Main {
    public static void main(String[] args) {
        ServicoFrete servico1 = new ServicoFrete(new FreteSedex());
        System.out.println(servico1.calcularFrete(2)); // 20.0

        ServicoFrete servico2 = new ServicoFrete(new FretePAC());
        System.out.println(servico2.calcularFrete(2)); // 10.0
    }
}
```

---

## Onde isso brilha em APIs REST com Spring

No mundo Spring Boot, você pode injetar essas estratégias usando o próprio **Spring IoC**.

Exemplo:

* Você define todas as implementações como **`@Component`**.
* Injeta uma lista ou um mapa de estratégias no seu **Service**.
* Seleciona dinamicamente qual usar com base em algum parâmetro (ex.: tipo de pagamento, forma de autenticação, cálculo de desconto etc.).

Isso evita aquele **Service inchado** cheio de `if/else` e deixa as regras encapsuladas em classes específicas.

---

## Quando usar Strategy

* Quando você tem **múltiplos algoritmos intercambiáveis**.
* Quando a lógica pode crescer e mudar com frequência.
* Quando não quer poluir uma classe com um mar de condicionais.

---

## Vantagens

* Respeita o **OCP** e o **SRP**.
* Deixa o código **extensível** (adicione novas estratégias sem alterar código existente).
* Facilita **testes unitários** (você testa cada estratégia isolada).

---

## Desvantagens

* Gera mais classes no projeto (a tal "explosão de classes").
* Se usado para algo muito simples, pode parecer burocracia desnecessária.

---

Perfeito, você descreveu um problema clássico: **um método de serviço virando um monstro cheio de `if`s, validações e regras espalhadas**. É aí que Strategy e Chain of Responsibility fazem sentido em APIs REST com Spring Boot.

---

## Strategy em APIs REST

A ideia central do Strategy é **definir uma interface comum para algoritmos intercambiáveis**. No seu caso, cada “algoritmo” é uma **regra de validação**.

* Você cria uma interface genérica:

```java
public interface ValidadorAdocao {
    void validar(AdocaoRequestDTO dto);
}
```

* Cada validação específica vira uma classe concreta:

```java
@Component
public class ValidadorPetDisponivel implements ValidadorAdocao {
    private final PetRepository petRepository;

    public ValidadorPetDisponivel(PetRepository petRepository) {
        this.petRepository = petRepository;
    }

    @Override
    public void validar(AdocaoRequestDTO dto) {
        Pet pet = petRepository.findById(dto.petId())
                .orElseThrow(() -> new IllegalArgumentException("Pet não encontrado"));
        if (pet.isAdotado()) {
            throw new RegraNegocioException("Pet já foi adotado");
        }
    }
}
```

```java
@Component
public class ValidadorTutorAtivo implements ValidadorAdocao {
    private final TutorRepository tutorRepository;

    public ValidadorTutorAtivo(TutorRepository tutorRepository) {
        this.tutorRepository = tutorRepository;
    }

    @Override
    public void validar(AdocaoRequestDTO dto) {
        Tutor tutor = tutorRepository.findById(dto.tutorId())
                .orElseThrow(() -> new IllegalArgumentException("Tutor não encontrado"));
        if (!tutor.isAtivo()) {
            throw new RegraNegocioException("Tutor inativo não pode adotar");
        }
    }
}
```

* No **service**, você injeta todos eles de uma vez:

```java
@Service
public class AdocaoService {

    private final AdocaoRepository adocaoRepository;

    private final List<ValidadorAdocao> validadores;

    public AdocaoService(AdocaoRepository adocaoRepository,
                         List<ValidadorAdocao> validadores) {
        this.adocaoRepository = adocaoRepository;
        this.validadores = validadores;
    }

    @Transactional
    public void adotar(AdocaoRequestDTO dto) {
        validadores.forEach(v -> v.validar(dto));

        // se passou por todos os validadores, salva
        Adocao adocao = new Adocao(dto.petId(), dto.tutorId());
        adocaoRepository.save(adocao);
    }
}
```

Aqui você aplicou **Strategy**: cada regra é um strategy diferente, selecionado dinamicamente pelo loop.
Isso substitui `if`s no service por **injeção polimórfica**.

---

## **Chain of Responsibility** nesse cenário

A diferença é sutil, mas importante. Enquanto o Strategy é mais sobre **escolher entre implementações intercambiáveis**, o Chain of Responsibility é sobre **passar uma requisição por uma cadeia de manipuladores, onde cada um decide se lida com ela ou passa adiante**.

No seu caso de validações:

* Se cada regra **precisar decidir se continua ou interrompe a cadeia**, você está mais próximo de **Chain of Responsibility**.
* Você poderia estruturar com uma **classe abstrata** que tem a lógica de “chamar o próximo validador”:

```java
public abstract class ValidadorChain {
    private ValidadorChain proximo;

    public ValidadorChain linkarProximo(ValidadorChain proximo) {
        this.proximo = proximo;
        return proximo;
    }

    public void validar(AdocaoRequestDTO dto) {
        executar(dto);
        if (proximo != null) {
            proximo.validar(dto);
        }
    }

    protected abstract void executar(AdocaoRequestDTO dto);
}
```

* Implementações concretas:

```java
@Component
public class ValidadorPetDisponivelChain extends ValidadorChain {
    @Override
    protected void executar(AdocaoRequestDTO dto) {
        // valida pet disponível
    }
}
```

* No service, você monta a corrente (ou deixa o Spring montar):

```java
@Service
public class AdocaoServiceChain {

    private final ValidadorChain primeiroValidador;

    public AdocaoServiceChain(ValidadorPetDisponivelChain pet,
                              ValidadorTutorAtivoChain tutor) {
        // monta a corrente
        pet.linkarProximo(tutor);
        this.primeiroValidador = pet;
    }

    @Transactional
    public void adotar(AdocaoRequestDTO dto) {
        primeiroValidador.validar(dto);
        // se passou em todos, salva
    }
}
```

Aqui, cada validador tem a chance de decidir: **lança exception** (quebra a cadeia) ou **passa adiante**.

---

## 3. Qual usar?

* **Strategy com lista injetada**: perfeito quando todas as validações devem ser aplicadas, sem exceção. É limpo, simples e aproveita o poder da injeção de dependências do Spring.

* **Chain of Responsibility**: útil quando cada regra **pode parar o fluxo** ou quando você quer que **apenas um manipulador seja responsável** (ex.: autenticação com múltiplas estratégias, filtros em pipeline).

---

## 4. No contexto de API REST com Spring Boot

* **Strategy com lista injetada** é o mais comum para **validações de DTO**. Mantém o service enxuto e escalável.
* **Chain of Responsibility** aparece mais em **processamentos complexos** (ex.: pipeline de autorização, filtros de auditoria, manipulação de eventos).

---



