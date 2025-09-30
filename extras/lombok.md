# **Lombok**

A ideia central é: ele tira da sua frente o “boilerplate code” — getters, setters, construtores, equals/hashCode, toString… tudo aquilo que você escreve sempre igual e que só deixa a classe poluída.

O Lombok funciona via **anotações**. Você coloca no código, o plugin da IDE e o compilador se viram para gerar os métodos. No fim, o `.class` tem os métodos como se você tivesse escrito à mão, mas seu `.java` fica limpo.

---

### As anotações mais usadas (e que você vai usar direto no Spring)

**1. @Getter e @Setter**

* Gera automaticamente os métodos `get` e `set` para os atributos da classe.
* Você pode aplicar na classe (gera para todos os atributos) ou só em um campo específico.
* Útil em DTOs, entidades JPA, etc.

```java
@Getter
@Setter
public class User {
    private String name;
    private String email;
}
```

---

**2. @ToString**

* Cria o método `toString()`.
* Dá pra excluir campos sensíveis (`exclude = "password"`) ou incluir só alguns (`of = {"id", "name"}`).

---

**3. @EqualsAndHashCode**

* Gera os métodos `equals` e `hashCode`.
* Pode ser configurado para comparar só certos campos (`of = "id"`).
* Muito útil em entidades, principalmente quando você quer que duas instâncias sejam “iguais” pelo ID, não por todos os atributos.

---

**4. @NoArgsConstructor, @AllArgsConstructor, @RequiredArgsConstructor**

* Criam construtores:

  * `@NoArgsConstructor`: sem parâmetros.
  * `@AllArgsConstructor`: com todos os parâmetros.
  * `@RequiredArgsConstructor`: só para atributos `final` ou `@NonNull`.

Isso casa bem com Spring, que injeta dependências pelo construtor.

```java
@RequiredArgsConstructor
public class UserService {
    private final UserRepository repository; // Lombok gera construtor com esse parâmetro
}
```

---

**5. @Data**

* Combina: `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode` e `@RequiredArgsConstructor`.
* É um canivete suíço para classes de modelo (DTOs, entities).
* Mas cuidado: nem sempre você quer todos os setters públicos, então avalie se vale usar em cada caso.

---

**6. @Value**

* É tipo um `@Data` imutável.
* Atributos viram `private final`, gera apenas getters, construtor com todos os args e `equals/hashCode/toString`.
* Bom para objetos de valor, configs, respostas imutáveis de API.

---

**7. @Builder**

* Cria o padrão *builder* para instanciar objetos de forma fluida.
* Muito útil em DTOs e no padrão de construção em testes.

```java
@Builder
public class User {
    private String name;
    private String email;
}
```

Uso:

```java
User user = User.builder()
    .name("Milena")
    .email("milena@email.com")
    .build();
```

---

**8. @Slf4j (e variantes)**

* Injeta automaticamente um logger (`private static final Logger log = LoggerFactory.getLogger(...)`).
* Você só chama `log.info("mensagem")` sem precisar declarar o logger manualmente.

```java
@Slf4j
@Service
public class UserService {
    public void process() {
        log.info("Processando usuário...");
    }
}
```

---

### Onde Lombok mais brilha no Spring

* **Entities e DTOs**: menos código repetitivo.
* **Services e configs**: `@RequiredArgsConstructor` ajuda no *constructor injection*.
* **Builders em testes**: criação de objetos mais legível.
* **Logging**: `@Slf4j` economiza aquela linha chata de logger.

---

O Lombok é um “atalho inteligente”. Ele limpa o código, mas também pode mascarar demais se você não souber o que está sendo gerado. O ideal é usar, mas com consciência do que cada anotação realmente escreve por baixo dos panos.

