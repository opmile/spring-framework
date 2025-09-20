# `ApplicationContext`: o container Spring

Vamos separar em **duas abordagens bem claras**, mostrando o **“modo clássico XML”** e o **“modo moderno annotations/JavaConfig”**. 

Isso vai te dar a visão completa de como se instancia e configura um `ApplicationContext`.

---

## **Raiz clássica – XML**

### Passo 1: Criar o arquivo XML de configuração

Suponha que temos duas classes:

```java
public class PaymentService {
    public void pay() { System.out.println("Pagamento realizado"); }
}

public class OrderService {
    private PaymentService paymentService;

    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void processOrder() { paymentService.pay(); }
}
```

Arquivo `beans.xml`:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="paymentService" class="com.example.PaymentService"/>
    
    <bean id="orderService" class="com.example.OrderService">
        <property name="paymentService" ref="paymentService"/>
    </bean>
</beans>
```

---

### Passo 2: Instanciar o `ApplicationContext`

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");

        OrderService orderService = context.getBean(OrderService.class);
        orderService.processOrder();
    }
}
```

* `ClassPathXmlApplicationContext` lê o XML do classpath e cria os beans declarados.
* O container faz **setter injection** (ou constructor injection, se declarado via `<constructor-arg>`).

---

## **Abordagem moderna – Annotations / JavaConfig**

### Passo 1: Criar classes com annotations

```java
import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;

@Component
public class PaymentService {
    public void pay() { System.out.println("Pagamento realizado"); }
}

@Component
public class OrderService {

    private final PaymentService paymentService;

    @Autowired
    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void processOrder() { paymentService.pay(); }
}
```

---

### Passo 2: Criar classe de configuração

```java
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example") // escaneia os @Components
public class AppConfig {
}
```

---

### Passo 3: Instanciar o `ApplicationContext`

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

        OrderService orderService = context.getBean(OrderService.class);
        orderService.processOrder();
    }
}
```

* `AnnotationConfigApplicationContext` usa **JavaConfig + annotations**.
* O Spring detecta automaticamente os beans anotados com `@Component` e injeta dependências com `@Autowired`.
* Dependências obrigatórias → via construtor.
* Dependências opcionais → via setter ou `@Autowired(required=false)`.

---

## **Resumo das diferenças**

| Aspecto                | XML clássico                       | Moderno (Annotations/JavaConfig)        |
| ---------------------- | ---------------------------------- | --------------------------------------- |
| Configuração           | `beans.xml`                        | `@Configuration` + `@ComponentScan`     |
| Instanciação           | `ClassPathXmlApplicationContext`   | `AnnotationConfigApplicationContext`    |
| Injeção de dependência | `<property>` / `<constructor-arg>` | `@Autowired` (constructor/setter/field) |
| Flexibilidade          | Menos dinâmica, verboso            | Mais concisa, tipo-safe, refatorável    |
| Padrão recomendado     | Legacy / manutenção de sistemas    | Novo projeto moderno                    |

---

## Quando faz sentido usar **id/nome**

* Em configuração **XML clássica**, cada bean normalmente recebe um `id` ou `name`:

```xml
<bean id="orderService" class="com.example.OrderService"/>
```

Aí você busca assim:

```java
OrderService os = (OrderService) context.getBean("orderService");
```

* Também funciona em **annotations** se você der um nome ao bean:

```java
@Component("orderService")
public class OrderService { ... }
```

Mas hoje em dia, usar string pra buscar bean é considerado **frágil** (menos seguro, refatoração quebra fácil).

---

## Quando faz sentido usar **classe**

* Quando os beans foram registrados via **anotações** (`@Component`, `@Service`, `@Bean` etc.), você quase sempre acessa por **classe**:

```java
OrderService os = context.getBean(OrderService.class);
```

* Também funciona em XML se você não quiser depender de `id`:

```xml
<bean class="com.example.OrderService"/>
```

A busca por tipo (`getBean(OrderService.class)`) resolve direto.

---

## Problemas possíveis

* **Se houver mais de um bean do mesmo tipo**, o `getBean(OrderService.class)` quebra com `NoUniqueBeanDefinitionException`.
* Nesse caso, você precisaria:

  * Usar **qualificador** (`@Qualifier("nomeDoBean")`).
  * Ou buscar por id/nome mesmo (`getBean("orderService")`).

---

* **XML + beans nomeados** → `getBean("id")`.
* **Annotations / JavaConfig** → `getBean(Classe.class)` é o natural.
* **Vários beans do mesmo tipo** → você precisa desambiguar (qualificador, nome ou `ObjectProvider`).

---

* Se você ainda está em XML **com id/nome**, `getBean("id")` faz sentido.
* No estilo moderno (annotations), a boa prática é sempre `getBean(Classe.class)` porque é **type-safe** (mais seguro, refatorável).

---

## `ApplicationContext` de forma moderna

### **1. Classe de configuração com `@Configuration` + métodos `@Bean`**

* Você escreve os beans manualmente, declarando **como eles são instanciados**.
* Escopo, qualifiers e demais metadados são definidos **direto no método do bean**.

```java
@Configuration
public class AppConfig {

    @Bean
    @Scope("singleton")
    @Qualifier("mysql")
    public DataSource mysqlDataSource() {
        return new MysqlDataSource();
    }

    @Bean
    @Scope("prototype")
    @Qualifier("postgres")
    public DataSource postgresDataSource() {
        return new PostgresDataSource();
    }
}
```

Aqui, você controla explicitamente como cada bean é criado.

E para levantar o container manualmente:

```java
ApplicationContext context = 
    new AnnotationConfigApplicationContext(AppConfig.class);

DataSource ds = context.getBean("mysqlDataSource", DataSource.class);
```

---

### **2. Escaneamento automático com `@ComponentScan`**

* Você anota diretamente suas classes com `@Component`, `@Service`, `@Repository`, etc.
* O Spring descobre e registra automaticamente.
* Escopo, qualifiers e tudo mais ficam **na própria classe**.

```java
@Service
@Scope("singleton")
@Qualifier("mysql")
public class MysqlDataSource implements DataSource { ... }

@Service
@Scope("prototype")
@Qualifier("postgres")
public class PostgresDataSource implements DataSource { ... }
```

E sua config pode ser:

```java
@Configuration
@ComponentScan(basePackages = "com.exemplo.projeto")
public class AppConfig { }
```

Ou, em uma aplicação Spring Boot, basta `@SpringBootApplication`, que já engloba o `@ComponentScan`.

---

* `@Configuration + @Bean` = você escreve **receita explícita** de como criar os beans.
* `@ComponentScan + estereótipos` = Spring faz **descoberta automática**, você só anota as classes.

No fundo, os dois levam ao mesmo destino: um `ApplicationContext` cheio de beans prontos.
A diferença é **o nível de controle versus automação**.

---

Em ambos os casos, se você não estiver no Spring Boot, precisa instanciar o `ApplicationContext` manualmente com `AnnotationConfigApplicationContext(AppConfig.class)`.

Existem **dois cenários principais**:

---

#### 1. **Sem Spring Boot (Spring “puro”)**

Você mesmo precisa instanciar o contexto manualmente.
Exemplo:

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = 
            new AnnotationConfigApplicationContext(AppConfig.class);

        MyService service = context.getBean(MyService.class);
        service.doSomething();
    }
}
```

* Aqui você diz: “Spring, crie um container com base nessa classe de configuração `AppConfig`”.
* A partir daí, o Spring varre `@Configuration`, `@Bean`, `@ComponentScan`, etc., e monta o contexto.

---

#### 2. **Com Spring Boot**

O Boot já **esconde essa criação manual**.
No lugar, você anota sua classe principal com `@SpringBootApplication` e roda:

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

O `SpringApplication.run(...)` internamente cria um `ApplicationContext` pra você.
Ou seja, você **não precisa mais escrever `new AnnotationConfigApplicationContext(...)`**.

---

* **Spring puro** → você instancia o contexto manualmente (`new AnnotationConfigApplicationContext(AppConfig.class)`).
* **Spring Boot** → o Boot faz isso nos bastidores quando você roda `SpringApplication.run(...)`.

---








