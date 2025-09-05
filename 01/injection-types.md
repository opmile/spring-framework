# Tipos de Injeção

---

## 1. **Injeção por Construtor**

* O Spring injeta dependências via **parâmetros do construtor**.
* É considerada a forma **mais recomendada** porque:

  * Garante **imutabilidade** (dependências final).
  * Obriga a passar todas as dependências necessárias na criação do objeto (evita objetos “meio construídos”).


```java
@Component
public class OrderService {

    private final PaymentService paymentService;

    // Injeção via construtor
    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Ou, desde o Spring 4.3, se a classe só tiver **um construtor**, o `@Autowired` é opcional.

* Declarar o campo como `final` ao usar injeção por construtor não é uma exigência do Spring, mas uma boa prática de design orientado a objetos que garante imutabilidade da dependência.

    * Como é final, ele só pode ser inicializado uma vez, no construtor. Depois disso, ninguém pode trocar a referência (não existe setter, nem reatribuição possível).

    * Se não for declarado como `final` a classe ainda funciona, mas alguém poderia criar um `setPaymentService(...)` e trocar a dependência depois e não existe garantia de que o objeto foi corretamente inicializado

---

## 2. **Injeção por Setter**

* O Spring injeta dependências chamando os **métodos setters** da classe.
* Útil quando:

  * A dependência é **opcional**.
  * Você quer dar flexibilidade para mudar dependências após a construção (como o setter é um método público, você pode chamá-lo manualmente em qualquer ponto do código, substituindo a dependência que já estava lá).

* Repare aqui que como o campo pode ser mutável, não o declaramos como `final`

```java
@Component
public class OrderService {

    private PaymentService paymentService;

    // Injeção via setter
    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

```java
@Component
public class NotificationService {

    private MessageSender sender;

    @Autowired
    public void setSender(MessageSender sender) {
        this.sender = sender;
    }

    public void notify(String msg) {
        sender.send(msg);
    }
}
```
* Modificando a dependência: 
```java
// Em algum ponto do código
NotificationService service = context.getBean(NotificationService.class);

// Inicialmente o Spring injetou, digamos, um EmailSender
service.notify("Olá!"); // envia por e-mail

// Mas eu posso trocar manualmente depois
service.setSender(new SmsSender());
service.notify("Olá!"); // agora envia por SMS
```

---

## 3. **Injeção por Campo (Field Injection)**

* O Spring injeta dependências diretamente nos atributos usando `@Autowired`.
* É a forma **mais simples** e “menos código”, mas tem pontos negativos:

  * Viola princípio de inversão de dependência (a classe depende do framework pra instanciar).
  * Dificulta testes unitários (não dá pra mockar sem usar reflexão).
  * É menos explícito sobre o que a classe realmente precisa.

```java
@Component
public class OrderService {

    @Autowired
    private PaymentService paymentService;
}
```

---

## 4. **Injeção por Interface (pouco comum)**

* O Spring permite que você crie uma interface que será usada como “contrato” para injeção.
* Exemplo clássico é `ApplicationContextAware`, onde o container injeta o próprio contexto.
* Pouco usada em aplicações modernas, pois cria **acoplamento ao Spring**.

---

| Tipo              | Prós                                                       | Contras                                   |
| ----------------- | ---------------------------------------------------------- | ----------------------------------------- |
| **Construtor**    | Imutabilidade, claro, ideal para dependências obrigatórias | Pode ficar verboso se muitas dependências |
| **Setter**        | Flexível, bom para dependências opcionais                  | Objeto pode nascer incompleto             |
| **Campo (field)** | Simples, pouco código                                      | Dificulta testes, acopla ao framework     |
| **Interface**     | Permite integração especial com o Spring                   | Muito acoplado ao framework, pouco usado  |

---

* **Construtor** → padrão ouro (claro, seguro, testável).
* **Setter** → bom pra dependências opcionais.
* **Field** → serve em protótipo/exemplo rápido, mas não em projeto sério.
* **Interface** → só quando precisa de comportamento especial do Spring.

---

## Tipos de Dependência

Quando falamos em **dependência obrigatória vs opcional**, estamos falando de **quanto a classe depende daquele outro objeto para funcionar**.

---

## Dependência Obrigatória

É aquela sem a qual o objeto **não funciona**.

* Deve ser passada no **construtor** → garante que o objeto sempre vai nascer “completaço”.
* Exemplo:

```java
@Component
public class OrderService {

    private final PaymentService paymentService; // sem isso não há pedido

    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

Se `PaymentService` não for injetado, o `OrderService` não faz sentido, porque não dá pra fechar pedidos sem processar pagamento.

---

## Dependência Opcional

É aquela que **melhora** ou **enriquece** a classe, mas **não é essencial**.

* Boa candidata para **injeção via setter**.

* A classe deve ter um comportamento padrão caso essa dependência não esteja presente.

```java
@Component
public class NotificationService {

    private EmailService emailService; // opcional

    @Autowired(required = false)
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }

    public void notifyUser(String message) {
        if (emailService != null) {
            emailService.send(message);
        } else {
            System.out.println("Fallback: " + message);
        }
    }
}
```

Se o `EmailService` não for configurado, o sistema **ainda funciona**, só que notifica pelo console ao invés de e-mail.

---

* Se a dependência é **vital para o core da classe** → **Obrigatória → Construtor**.
* Se a dependência é **acessória, opcional ou plugin** → **Opcional → Setter**.

---



