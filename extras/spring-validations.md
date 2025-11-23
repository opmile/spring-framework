# Spring Validations

Se você gerar o projeto no Spring Initializr e selecionar **“Validation”**, ele já adiciona o seguinte no `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Esse é o pacote que puxa o Bean Validation (Hibernate Validator por baixo dos panos) e integra com o Spring MVC.

Depois disso, você já pode:

* usar anotações como `@NotBlank`, `@Email`, `@Size`, `@Valid` etc.
* receber erros formatadinhos no Spring (`MethodArgumentNotValidException`).
* devolver mensagens customizadas com `@ControllerAdvice`.

---

Beleza, vamos montar isso de um jeito direto, técnico na medida e sem frescura. A ideia é você realmente *entender* como tudo se encaixa no fluxo de uma API Spring — desde a entrada no controller até a lógica de domínio no service.

---

### O que é o **Spring Validation**

Spring Validation é o mecanismo do Spring que integra:

* **Bean Validation (Jakarta Validation)** — as anotações tipo `@NotBlank`, `@Email`, `@Size`, `@Min`, etc.
* **Hibernate Validator** — implementação padrão dessas validações.
* **Spring MVC** — pra interceptar requisições, rodar validações e devolver erros no formato certo.

Em resumo: é o jeito “oficial” de validar dados de entrada automaticamente, antes mesmo da sua lógica de negócio rodar.

---

### Validações *estruturais* (Bean Validation)

Essas são as validações que você coloca **no DTO** ou **no modelo**, usando as anotações padrão.

Elas servem para garantir que **a estrutura dos dados** está correta:

* formato
* tamanho
* obrigatoriedade
* tipos válidos
* constraints simples

Exemplos:

```java
public record UserRequest(
    @NotBlank(message = "Nome é obrigatório")
    String name,

    @Email(message = "E-mail inválido")
    String email,

    @Min(value = 18, message = "Idade mínima é 18")
    Integer age
) {}
```

Essas validações são sempre “estáticas”: elas não dependem do banco, do service, de nada. É a forma mais básica de filtrar dados ruins.

---

### Validações *de domínio* (lógica de negócio)

Agora vem a parada mais séria.

As validações de domínio pertencem ao **core da lógica** do seu sistema, e por isso **sempre ficam na camada de serviço** (ou no domínio, se você for mais purista).

Coisas como:

* e-mail já existe no banco?
* um pedido só pode ser cancelado se estiver no estado X?
* estoque suficiente?
* data do início do evento não pode estar no passado?
* um usuário não pode se cadastrar se estiver suspenso?

Essas regras **não pertencem ao DTO** porque:

1. Dependem de lógica.
2. Dependem de banco de dados.
3. Dependem de contexto.
4. Podem mudar conforme regras do produto.

Exemplo:

```java
@Service
public class UserService {

    public User create(UserRequest request) {
        if (userRepository.existsByEmail(request.email())) {
            throw new DomainException("E-mail já está cadastrado");
        }

        // outras regras de negócio...

        var user = new User(request.name(), request.email(), request.age());
        return userRepository.save(user);
    }
}
```

---

### Como usar `@Valid` nos controllers

Quando o controller recebe um DTO com `@Valid`, o Spring intercepta a requisição **antes do método rodar**.

Se algo estiver inválido → ele lança `MethodArgumentNotValidException`.

Exemplo:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @PostMapping
    public ResponseEntity<UserResponse> create(@RequestBody @Valid UserRequest request) {
        var user = userService.create(request);
        return ResponseEntity.ok(new UserResponse(user));
    }
}
```

Simples: colocou `@Valid`, o Spring liga o modo “babá” dos dados.

---

### Como tratar erros de validação usando `GlobalExceptionHandler`

Agora vem a parte do trato fino: interceptar esses erros e devolver respostas amigáveis.

Você cria um `@ControllerAdvice` (geralmente chamado de `GlobalExceptionHandler`) e trata:

* `MethodArgumentNotValidException` → erros de Bean Validation
* sua `DomainException` → erros de regra de negócio

Exemplo completo:

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationErrors(MethodArgumentNotValidException ex) {

        var errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(error -> new ErrorField(error.getField(), error.getDefaultMessage()))
                .toList();

        return ResponseEntity
                .badRequest()
                .body(new ErrorResponse("Erro de validação", errors));
    }

    @ExceptionHandler(DomainException.class)
    public ResponseEntity<?> handleDomainErrors(DomainException ex) {
        return ResponseEntity
                .unprocessableEntity()
                .body(new ErrorResponse(ex.getMessage(), null));
    }
}

record ErrorField(String field, String message) {}

record ErrorResponse(String message, List<ErrorField> fields) {}
```

Agora sua API devolve respostas limpas e padronizadas.

Show, vamos direto ao ponto. Aqui está exatamente **a implementação pedida**, com:

* `LinkedHashMap` para preservar a ordem em que os erros foram encontrados.
* `Map<String, List<String>>` para guardar **todas** as mensagens de cada campo.
* Sem `extends ResponseEntityExceptionHandler`.

E logo abaixo eu explico rapidamente o fluxo desse handler.

---

# **GlobalExceptionHandler completo (como você pediu)**

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorDTO> handleValidationExceptions(MethodArgumentNotValidException ex) {

        Map<String, List<String>> errors = new LinkedHashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(error -> {
            String field = error.getField();
            String message = error.getDefaultMessage();

            // Inicializa o campo caso ainda não exista
            errors.computeIfAbsent(field, k -> new ArrayList<>());

            // Adiciona a mensagem do erro à lista daquele campo
            errors.get(field).add(message);
        });

        ErrorDTO response = new ErrorDTO("Validação inválida", errors);

        return ResponseEntity.badRequest().body(response);
    }
}
```

---

# **DTO de erro**

```java
public class ErrorDTO {
    private String message;
    private Map<String, List<String>> errors;

    public ErrorDTO(String message, Map<String, List<String>> errors) {
        this.message = message;
        this.errors = errors;
    }

    public String getMessage() {
        return message;
    }

    public Map<String, List<String>> getErrors() {
        return errors;
    }
}
```

---

# **Notas rápidas do porquê cada escolha faz sentido**

### **1) LinkedHashMap**

Preserva a ordem dos campos exatamente como as validações foram encontradas.
Erros chegam primeiro para campos definidos primeiro no DTO → a resposta mantém essa ordem.

### **2) Map<String, List<String>>**

* Se o campo `email` quebrar três validações (`@NotBlank`, `@Email`, `@Size`), você recebe **todas** as mensagens.
* Não perde nada, não sobrescreve nada.

### **3) computeIfAbsent**

Inicializa a lista só no momento em que aquele campo aparece pela primeira vez.
Evita `if(!errors.containsKey(...))`.

* The method takes two arguments: the `key` to check and a `mappingFunction` (a Function or lambda expression) that will compute the value if needed.

* It first checks if the `key` is already present in the map and associated with a non-null value.

* If the `key` is absent or the associated value is `null`, the `mappingFunction` is executed to compute a new value for that key.

* This newly computed value is then inserted into the map, associating it with the `key`.

* If the `key` was already present and had a non-null value, the `mappingFunction` is not executed, and **the existing value for that key is returned**.


---

1. **chega a requisição**
2. `@Valid` roda e valida constraints simples
3. se falhar → vai pro `MethodArgumentNotValidException` → `GlobalExceptionHandler` → 400
4. se passar → controller chama service
5. no service rodamos as **validações de domínio**
6. se falhar → lança `DomainException` → `GlobalExceptionHandler` → 422
7. se tudo ok → continua fluxo normal

E pronto: você tem uma API com validação “profissional”.

---

# Annotations

## 1) **Validações de texto (String)**

### `@NotBlank`

Garante que não seja nulo, vazio **nem só espaços**.
É a que você mais vai usar pra campos obrigatórios.

### `@NotEmpty`

Garante que não seja nulo nem vazio, mas permite espaços.
Mais usada com coleções do que com String.

### `@Size(min = X, max = Y)`

Tamanho mínimo e máximo.
Serve pra Strings, listas, arrays, etc.

### `@Email`

Valida formato de e-mail.
Ele é “nice but not perfect”, mas é o padrão.

### `@Pattern(regex = "...")`

Quando você precisa impor um formato específico: CPF, CEP, telefone, código, etc.

---

## 2) **Validações numéricas**

### `@Min(x)`

Mínimo numérico permitido (int, long, BigDecimal).

### `@Max(x)`

Máximo numérico permitido.

### `@Positive` / `@PositiveOrZero`

Aceita apenas números positivos / positivos ou zero.

### `@Negative` / `@NegativeOrZero`

Mesma ideia, só que pra negativos.

---

## 3) **Validações para objetos e coleções**

### `@NotNull`

O campo precisa existir.
Muito útil com tipos complexos ou coleções.

### `@NotEmpty`

Garante que listas/coleções não estejam vazias.

### `@Valid`

Essa é crucial:
Ela obriga que **o objeto interno também passe pelas próprias validações**.

Exemplo:

```java
public record OrderRequest(
    @NotNull @Valid AddressRequest address,
    @NotEmpty List<@Valid ItemRequest> items
) {}
```

Sem `@Valid`, o Spring valida só o DTO externo e ignora as anotações internas.

---

## 4) **Validações de data**

### `@Past`

Tem que ser uma data no passado.

### `@PastOrPresent`

Data não pode ser futura.

### `@Future`

Só datas futuras.

### `@FutureOrPresent`

Data futura ou hoje.

Obs: usando `java.time` funciona lindamente.

---

## 5) **Validações para números decimais**

### `@DecimalMin("0.01")`

Versão decimal do `@Min`, mais usada quando envolve dinheiro.

### `@DecimalMax("999.99")`

### `@Digits(integer = x, fraction = y)`

Controla quantidade de dígitos antes e depois da vírgula.
Muito útil em preços.

Exemplo:

```java
@Digits(integer = 6, fraction = 2)
BigDecimal price;
```

---

## 6) **Validações compostas com listas, maps, etc.**

Você pode anotar elementos internos também:

```java
List<@Email String> emails;
```

O Spring valida **cada item** conforme o tipo.

---

## 7) **Validações personalizadas**

Quando nenhuma anotação padrão resolve, você cria sua própria constraint, com:

* anotação personalizada
* validator customizado
* regra complexa embutida (mas só se for estrutural, não de domínio)

Exemplo clássico:
Validar CPF, CNPJ, CEP, ou um formato não coberto por regex simples.

---

## Campos obrigatórios?

`@NotBlank` (texto)
`@NotNull` (objeto)
`@NotEmpty` (lista)

## Campos com formato?

`@Email`, `@Pattern`, `@Size`

## Campos numéricos?

`@Min`, `@Max`, `@Positive`, `@Negative`, `@Digits`

## Datas?

`@Past`, `@Future`, etc.

## Objetos aninhados?

`@Valid`

---

# Mais sobre Validações

Beleza, vamos destrinchar isso com calma e sem frescura. Você tem três jeitos de validar o domínio:

1. **Validação no domínio (métodos dentro da entidade)**
2. **Validação no serviço de domínio (DomainService)**
3. **Validação via Spring Validation (@NotBlank, @Size, etc.)**

Cada uma serve pra uma camada diferente.
Vou te mostrar **os dois tipos de solução do milestone** e **te explicar onde o Spring Validation entra — e onde ele NÃO deveria entrar**.

---

# 1) **Validações dentro da ENTIDADE (métodos de domínio)**

Essa abordagem diz:
*"Se essa entidade existe, ela precisa garantir as invariantes dela mesma."*

**Exemplo (abaixo é estilo DDD raiz, responsabilidade própria do domínio):**

```java
public class Book {

    private Long id;
    private String title;
    private int publicationYear;
    private String isbn;

    public Book(String title, int publicationYear, String isbn) {
        this.title = title;
        this.publicationYear = publicationYear;
        this.isbn = isbn;

        validate();
    }

    private void validate() {
        validateTitle();
        validatePublicationYear();
        validateIsbn();
    }

    private void validateTitle() {
        if (title == null || title.trim().isEmpty()) {
            throw new InvalidBookException("Título não pode ser vazio.");
        }
    }

    private void validatePublicationYear() {
        if (publicationYear > Year.now().getValue()) {
            throw new InvalidBookException("Ano de publicação não pode ser no futuro.");
        }
    }

    private void validateIsbn() {
        if (!ISBNValidator.isValid(isbn)) {
            throw new InvalidBookException("ISBN inválido.");
        }
    }
}
```

### **Quando vale usar?**

* Quando o domínio é “inteligente” (DDD, regras complexas)
* Quando o objeto deve garantir sua própria validade em QUALQUER contexto (serviço, CLI, API…)

### **Prós**

* Domínio forte e consistente
* Entidade 100% autônoma
* Garantia de invariantes em qualquer camada

### **Contras**

* Código da entidade cresce
* Pode ficar verboso
* Pode exigir mocks complicados em testes

---

# 2) **Validações num SERVICE de domínio**

Aqui você tira a lógica da entidade e coloca em um service que valida antes de criar livros.

**Exemplo:**

```java
public class BookDomainService {

    public void validate(Book book) {
        validateTitle(book.getTitle());
        validatePublicationYear(book.getPublicationYear());
        validateIsbn(book.getIsbn());
    }

    private void validateTitle(String title) {
        if (title == null || title.isBlank()) {
            throw new InvalidBookException("Título não pode ser vazio.");
        }
    }

    private void validatePublicationYear(int year) {
        if (year > Year.now().getValue()) {
            throw new InvalidBookException("Ano de publicação inválido.");
        }
    }

    private void validateIsbn(String isbn) {
        if (!ISBNValidator.isValid(isbn)) {
            throw new InvalidBookException("ISBN inválido.");
        }
    }
}
```

E no service de aplicação:

```java
var book = new Book(...);
bookDomainService.validate(book);
repository.save(book);
```

### **Quando vale usar?**

* Se você quer uma entidade magra
* Se regras variam por caso de uso
* Se você segue um estilo mais “anêmico” (arquitetura sem DDD hardcore)

### **Prós**

* Entidade fica mais simples
* Lógica agrupada e reutilizável
* Melhor quando as validações dependem de repositórios

### **Contras**

* O domínio não é tão sólido
* Possível criar objetos inválidos na memória se não tomar cuidado

---

# 3) **E o Spring Validation?**

### **Spring Validation NÃO é validação de domínio.**

Spring Validation é **validação de transporte**, validação *da entrada da API*, por exemplo:

```java
public record BookRequest(
    @NotBlank String title,
    @Min(1400) int publicationYear,
    @NotBlank @Size(min = 10, max = 17) String isbn
) {}
```

Ele resolve coisas como:

* “O cliente mandou um título vazio?”
* “O ano veio negativo?”
* “O JSON está no formato esperado?”

**Mas ele NÃO garante regras de negócio.**
Exemplos de domínio que Spring não deveria validar:

* ISBN único
* Ano futuro
* Autor não existir no sistema
* Livro já estar inativo
* Status não permitido
* Transições inválidas de estado

### **Resumo direto:**

* **Spring Validation → validação do REQUEST (entrada da API)**
* **Validação de domínio → regras e invariantes da entidade**

---

# Então qual é MELHOR?

### **Se seu sistema tem regras de negócio reais → use validação de domínio (entidade ou service).**

### **Spring Validation só complementa**, não substitui.

Ele evita request sujo, mas não protege seu núcleo.

---

Use **os dois**:

### 1) SPRING VALIDATION (na borda)

* evitar requisições inválidas
* garantir formato básico
* validar DTOs

### 2) VALIDAÇÃO DE DOMÍNIO dentro da entidade ou DomainService

* regras de negócio reais
* ISBN único
* ano não futuro
* título obrigatório
* status permitido

---

Na **service layer**, você faz **validações que dependem de infraestrutura ou contexto**, ou seja, coisas que a entidade sozinha não tem como saber.

Pensa assim:

**Validações na entidade/domain:**
Regras internas do próprio objeto.
Ex.: título vazio, ano futuro, ISBN malformado.

**Validações no service:**
Regras que precisam de repositório, de outros agregados ou de lógica de caso de uso.
Exemplos:

* Verificar se o ISBN já existe no banco
* Checar se o autor informado realmente existe
* Conferir se o livro está ativo antes de atualizar
* Validar que a operação é permitida (ex.: não remover livro emprestado)

**Resumo da lei prática:**

* **Coisas que a entidade sabe sobre ela mesma → no domínio.**
* **Coisas que exigem buscar dados ou validar contexto → no service.**


