# Glossário

## O Core Container Spring

Boa! Essa pergunta é bem central pra entender o **Spring Framework** — o **Core Container** é a espinha dorsal dele, aquilo que sustenta a inversão de controle e a injeção de dependências. Vamos quebrar cada módulo:

O **Core Container** é composto por quatro módulos principais:

1. **spring-core**
2. **spring-beans**
3. **spring-expression (SpEL)**
4. **spring-context**

Ele fornece a infraestrutura fundamental para criar e gerenciar beans, que são os objetos controlados pelo Spring IoC Container.

---

### 1. **Core (spring-core)**

* É a fundação do framework.
* Fornece as classes utilitárias e o **IoC (Inversion of Control)** de baixo nível.
* Implementa o mecanismo de injeção de dependências (via construtor, setter ou campo).
* Lida com recursos como:

  * `Resource` (abstração para acesso a arquivos, URLs, etc.)
  * `ApplicationContextAware` e interfaces auxiliares
  * Manipulação de tipos (Type Conversion)

Sem esse módulo, o Spring não teria a base para criar e gerenciar os beans.

---

### 2. **Beans (spring-beans)**

* Constrói em cima do **Core**.
* É responsável por **definir, criar e gerenciar os beans** no container.
* Suporta:

  * **BeanFactory** → container básico e leve para gerenciar beans.
  * Ciclo de vida dos beans (init, destroy).
  * Injeção de dependências declarada (via XML, annotations ou JavaConfig).

Pensa nele como o "motor de fábrica de beans".

---

### 3. **Expression Language (spring-expression / SpEL)**

* Uma mini linguagem própria do Spring para **avaliar expressões em tempo de execução**.
* Sintaxe parecida com EL do JSP, mas muito mais poderosa.
* Permite:

  * Acessar propriedades de beans (`#{myBean.property}`).
  * Chamar métodos (`#{myService.calculateSomething()}`).
  * Fazer operações matemáticas e lógicas (`#{2 + 2}`, `#{user.age > 18}`).
  * Acessar coleções (`#{list[0]}`, `#{map['key']}`).

É o **“motor dinâmico”**: você injeta valores e comportamentos flexíveis sem precisar hardcodar.

---

### 4. **Context (spring-context)**

* É um **superset do BeanFactory**, fornecendo um container mais rico.
* Implementa a interface `ApplicationContext`.
* Fornece recursos adicionais, como:

  * **Internacionalização (i18n)** → mensagens em diferentes idiomas.
  * **Eventos do Spring** → publish/subscribe interno (`ApplicationEvent`).
  * **Suporte a recursos corporativos** → JNDI, EJB, serviços de email, scheduler etc.
  * Integração com Annotations (`@Component`, `@Autowired`, `@Configuration`).

---

## Factory

O **Factory** é um padrão de projeto que **centraliza a criação de objetos** em uma "fábrica" (um método ou classe), em vez de espalhar `new` pelo código.

Assim, você pede para a fábrica um objeto e ela decide qual implementação criar e te devolver.

---

### Estrutura

Existem duas variações comuns:

1. **Factory Method**

   * Define um **método** que as subclasses implementam para criar objetos.
   * Foca em permitir que subclasses decidam qual classe instanciar.
   * Exemplo:

     ```java
     abstract class Dialog {
         abstract Button createButton(); // método fábrica
     }

     class WindowsDialog extends Dialog {
         @Override
         Button createButton() {
             return new WindowsButton();
         }
     }

     class LinuxDialog extends Dialog {
         @Override
         Button createButton() {
             return new LinuxButton();
         }
     }
     ```

2. **Abstract Factory**

   * Define uma **fábrica de fábricas**: cria **famílias de objetos relacionados** sem especificar suas classes concretas.
   * Útil quando você precisa garantir que um conjunto de objetos trabalhe junto.
   * Exemplo:

     ```java
     interface GUIFactory {
         Button createButton();
         Checkbox createCheckbox();
     }

     class WindowsFactory implements GUIFactory {
         public Button createButton() { return new WindowsButton(); }
         public Checkbox createCheckbox() { return new WindowsCheckbox(); }
     }

     class LinuxFactory implements GUIFactory {
         public Button createButton() { return new LinuxButton(); }
         public Checkbox createCheckbox() { return new LinuxCheckbox(); }
     }
     ```

---

### No Spring

O Spring usa a ideia de **Factory** em vários pontos:

* O **BeanFactory** é literalmente uma aplicação do padrão Factory.
* Você nunca dá `new` em um bean, você pede para o container criar e entregar.
* Isso permite **injeção de dependência**: você não se preocupa **com qual implementação** está vindo, só com a interface.

---

