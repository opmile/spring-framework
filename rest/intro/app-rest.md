## Controller REST?

No **Spring MVC** (módulo usado pelo Spring Boot), um **controller** é uma classe responsável por **receber requisições HTTP** do cliente, processar (chamando serviços, acessando banco de dados etc.) e devolver uma resposta.

Quando falamos em **REST Controller**, significa que:

* Ele é um **controller especializado em APIs REST**,
* Suas respostas normalmente são **dados (JSON, XML)**, e não páginas HTML.

No Spring Boot, usamos a anotação `@RestController` para marcar que aquela classe é um controlador REST.

---

## Diferença entre `@Controller` e `@RestController`

* `@Controller` → usado em apps web tradicionais (MVC), geralmente retorna **views** (HTML renderizado).
* `@RestController` → é um `@Controller` **+** `@ResponseBody` automático, ou seja:

  * Não retorna views.
  * Retorna dados direto no corpo da resposta (geralmente JSON).

---

## Criando um RestController no Spring Boot

Exemplo mínimo de aplicação Spring Boot com um endpoint REST:

### 1. Estrutura da classe principal

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MinhaApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(MinhaApiApplication.class, args);
    }
}
```

---

### 2. Criando um RestController

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Olá, mundo!";
    }
}
```

* `@RestController` → diz que a classe é um controlador REST.
* `@GetMapping("/hello")` → mapeia requisições **GET /hello** para esse método.
* Retorno (`String`) vai direto como resposta HTTP (nesse caso `"Olá, mundo!"`).

Se você rodar essa aplicação e acessar `http://localhost:8080/hello`, verá:

```http
HTTP/1.1 200 OK
Content-Type: text/plain;charset=UTF-8

Olá, mundo!
```

---

### 3. Retornando JSON

Na prática, você quer retornar **objetos em JSON**, não só texto.
Spring Boot faz isso automaticamente com a biblioteca **Jackson** (serialização JSON).

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

class Usuario {
    private String nome;
    private String email;

    public Usuario(String nome, String email) {
        this.nome = nome;
        this.email = email;
    }

    // getters obrigatórios para Jackson funcionar
    public String getNome() { return nome; }
    public String getEmail() { return email; }
}

@RestController
public class UsuarioController {

    @GetMapping("/usuario")
    public Usuario getUsuario() {
        return new Usuario("Milena", "milena@email.com");
    }
}
```

Se acessar `http://localhost:8080/usuario`, a resposta será:

```json
{
  "nome": "Milena",
  "email": "milena@email.com"
}
```

---

* `@RestController` é a base de uma API REST no Spring Boot.
* Ele recebe requisições HTTP e responde com dados (geralmente JSON).
* Spring Boot cuida automaticamente da serialização/deserialização.

---

## Mapeando Endpoints

* O **`DispatcherServlet`** já vimos que atua como front controller.
* Ele precisa de uma forma de saber **qual método do seu código atende a qual requisição HTTP**.
* Isso é feito por meio das **annotations de mapeamento** (`@RequestMapping`, `@GetMapping`, `@PostMapping`, etc.) que participaram do processo de execução do `DispatcherServlet` no ponto do `Handler Mapping`.

---

### 1. `@RequestMapping` (a “genérica”)

* É a mais **antiga e genérica**.
* Permite mapear:

  * **caminho** (ex: `/usuarios`),
  * **método HTTP** (GET, POST, PUT, DELETE, etc.),
  * além de outras restrições (headers, params...).

Exemplo:

```java
@RestController
@RequestMapping("/usuarios") // prefixo comum para todos os métodos
public class UsuarioController {

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public String buscarPorId(@PathVariable Long id) {
        return "Usuário com ID: " + id;
    }
}
```

---

### 2. `@GetMapping`, `@PostMapping`, etc. (os “atalhos modernos”)

Para facilitar, o Spring Boot trouxe **atalhos semânticos** para cada verbo HTTP:

* `@GetMapping` → para GET
* `@PostMapping` → para POST
* `@PutMapping` → para PUT
* `@DeleteMapping` → para DELETE
* `@PatchMapping` → para PATCH

O exemplo anterior, em estilo moderno:

```java
@RestController
@RequestMapping("/usuarios")
public class UsuarioController {

    @GetMapping("/{id}")
    public String buscarPorId(@PathVariable Long id) {
        return "Usuário com ID: " + id;
    }

    @PostMapping
    public String criar(@RequestBody String usuario) {
        return "Usuário criado: " + usuario;
    }
}
```

Se quisessemos um `GET` para receber todos os usuários do sistema, só criamos um endpoint que responda a um `GET` sem parâmetros (ou seja, para `/usuarios`).

```java
@RestController
@RequestMapping("/usuarios")
public class UsuarioController {

    @GetMapping("/{id}")
    public String buscarPorId(@PathVariable Long id) {
        return "Usuário com ID: " + id;
    }

    @PostMapping
    public String criar(@RequestBody String usuario) {
        return "Usuário criado: " + usuario;
    }

    @GetMapping
    public List<String> findAll() {
        // Exemplo: lista mockada, em uma app real viria do banco de dados
        return List.of("Usuário 1", "Usuário 2", "Usuário 3");
    }
}
```

* Como você está em um `@RestController`, o Spring converte automaticamente essa lista em JSON usando o Jackson.

O `@RequestMapping("/usuarios")` na classe já define a **raiz do endpoint**.

Quando você coloca `@GetMapping` sem nada, ele se aplica exatamente a `/usuarios`. Se tivesse `@GetMapping("/ativos")`, por exemplo, ficaria `/usuarios/ativos`.


---

### 3. Principais parâmetros

* `@PathVariable` → captura parte do caminho (`/usuarios/{id}`).
* `@RequestParam` → captura query params (`/usuarios?ativo=true`).
* `@RequestBody` → pega o corpo da requisição (JSON enviado via POST, PUT, etc.).
* `@ResponseBody` (implícito em `@RestController`) → converte retorno Java em JSON automaticamente via Jackson.

---

### 4. Fluxo prático

1. Cliente chama: `GET http://localhost:8080/usuarios/10`
2. Tomcat recebe e entrega ao `DispatcherServlet`.
3. `DispatcherServlet` consulta o **HandlerMapping** → vê que `@GetMapping("/{id}")` do `UsuarioController` atende.
4. Executa o método e converte a resposta em JSON.

---

## Parâmetros via Path Variables e Query Params

No Spring MVC (e portanto no Spring Boot) você pode receber parâmetros de duas formas principais:

---

### 1. **Path Variables** (`@PathVariable`)

São parâmetros que fazem **parte do caminho da URL**.
Exemplo: `/usuarios/10` → aqui o `10` é o `id` do usuário.

```java
@GetMapping("/{id}")
public String buscarPorId(@PathVariable Long id) {
    return "Usuário com ID: " + id;
}
```

### Chamada:

```
GET http://localhost:8080/usuarios/10
```

### Por que usar?

* Quando o parâmetro **identifica um recurso específico**.
* Segue o padrão REST: `/usuarios/{id}` para buscar 1 usuário.

---

### 2. **Query Parameters** (`@RequestParam`)

São parâmetros passados após `?` na URL.
Exemplo: `/usuarios?ativo=true&page=2`

```java
@GetMapping
public String listar(
        @RequestParam(required = false) Boolean ativo,
        @RequestParam(defaultValue = "0") int page) {

    return "Listando usuários. Ativo: " + ativo + ", Página: " + page;
}
```

### Chamada:

```
GET http://localhost:8080/usuarios?ativo=true&page=2
```

### Por que usar?

* Quando você quer **filtrar, ordenar ou paginar** resultados.
* Não são obrigatórios (pode usar `required=false` ou `defaultValue`).

---

* `@PathVariable` → identifica **quem** (ex.: `/usuarios/15`)
* `@RequestParam` → indica **como buscar** ou **como filtrar** (ex.: `/usuarios?ativo=true&page=2`)

---

## Receber e Enviar JSON

### 1. **Receber JSON (`@RequestBody`)**

Quando um cliente envia um JSON no corpo da requisição, o Spring pode **desserializar** esse JSON para um objeto Java automaticamente, usando o **Jackson** (que já vem configurado por padrão no Spring Boot).

Exemplo de classe DTO simples:

```java
public class Usuario {
    private Long id;
    private String nome;
    private String email;

    // getters e setters
}
```

Controller recebendo o JSON:

```java
@RestController
@RequestMapping("/usuarios")
public class UsuarioController {

    @PostMapping
    public String criar(@RequestBody Usuario usuario) {
        return "Usuário criado: " + usuario.getNome() + " - " + usuario.getEmail();
    }
}
```

### Requisição:

```
POST http://localhost:8080/usuarios
Content-Type: application/json

{
  "id": 1,
  "nome": "Milena",
  "email": "milena@email.com"
}
```

### O que acontece?

* O **Jackson** transforma esse JSON em um objeto `Usuario`.
* O Spring injeta esse objeto no método via `@RequestBody`.

---

### 2. **Enviar JSON (`@ResponseBody`)**

Toda classe anotada com `@RestController` já assume que os métodos retornam JSON por padrão, então não é necessário colocar `@ResponseBody` manualmente (isso seria necessário apenas com `@Controller`).

Exemplo:

```java
@GetMapping("/{id}")
public Usuario buscarPorId(@PathVariable Long id) {
    Usuario u = new Usuario();
    u.setId(id);
    u.setNome("Exemplo");
    u.setEmail("exemplo@email.com");
    return u; // retornado como JSON
}
```

#### Resposta:

```json
{
  "id": 10,
  "nome": "Exemplo",
  "email": "exemplo@email.com"
}
```

---

* `@RequestBody` → transforma **JSON → Objeto Java**.
* `@ResponseBody` (implícito em `@RestController`) → transforma **Objeto Java → JSON**.
* O **Jackson** faz essa conversão automática.

---

* Cliente envia objeto → precisa serializar antes de mandar
* Servidor recebe JSON → precisa desserializar para objeto Java (Jackson faz com `@RequestBody`).
* Servidor responde com objeto Java → precisa serializar de novo em JSON.

---

## Exceptions

### 1. O que é `@ControllerAdvice`?

É uma classe que o Spring escaneia para **tratar exceções de forma centralizada**.
Pensa assim: em vez de cada controller ter sua lógica de erro, você concentra tudo num "advogado dos controllers".

---

### 2. O que é `@ExceptionHandler`?

É um método dentro do `@ControllerAdvice` (ou dentro do próprio controller, se você quiser tratar localmente) que diz **"quando tal exceção ocorrer, execute esse método"**.

---

### 3. Exemplo prático

### Exceção customizada

```java
public class UsuarioNaoEncontradoException extends RuntimeException {
    public UsuarioNaoEncontradoException(Long id) {
        super("Usuário com ID " + id + " não encontrado.");
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/usuarios")
public class UsuarioController {

    @GetMapping("/{id}")
    public Usuario buscarPorId(@PathVariable Long id) {
        if (id == 99L) { // só de exemplo
            throw new UsuarioNaoEncontradoException(id);
        }
        return new Usuario(id, "Milena", "milena@email.com");
    }
}
```

### Tratamento global

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UsuarioNaoEncontradoException.class)
    public ResponseEntity<String> handleUsuarioNaoEncontrado(UsuarioNaoEncontradoException ex) {
        return ResponseEntity
                .status(HttpStatus.NOT_FOUND) // 404
                .body(ex.getMessage());       // "Usuário com ID X não encontrado."
    }

    @ExceptionHandler(Exception.class) // fallback para qualquer exceção não tratada
    public ResponseEntity<String> handleException(Exception ex) {
        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR) // 500
                .body("Erro interno: " + ex.getMessage());
    }
}
```

---

### 4. O que acontece no fluxo

* O cliente faz `GET /usuarios/99`.
* O controller lança `UsuarioNaoEncontradoException`.
* O Spring intercepta → vê que existe um método no `@ControllerAdvice` para essa exceção.
* O `handleUsuarioNaoEncontrado` é chamado → devolve `404 + mensagem`.

Resposta JSON:

```json
"Usuário com ID 99 não encontrado."
```

---

### 5. Melhorando ainda mais (resposta JSON estruturada)

Em vez de devolver uma `String`, você pode criar um **DTO de erro**:

```java
public class ErrorResponse {
    private String message;
    private int status;
    private LocalDateTime timestamp;

    // construtores/getters/setters
}
```

E no handler:

```java
@ExceptionHandler(UsuarioNaoEncontradoException.class)
public ResponseEntity<ErrorResponse> handleUsuarioNaoEncontrado(UsuarioNaoEncontradoException ex) {
    ErrorResponse error = new ErrorResponse(
        ex.getMessage(),
        HttpStatus.NOT_FOUND.value(),
        LocalDateTime.now()
    );
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
}
```

Resposta JSON agora:

```json
{
  "message": "Usuário com ID 99 não encontrado.",
  "status": 404,
  "timestamp": "2025-08-30T14:32:01.123"
}
```

---

* `@ControllerAdvice` → centraliza o tratamento.
* `@ExceptionHandler` → mapeia exceções específicas para respostas HTTP.
* Você pode customizar tanto em texto simples quanto em JSON bonitão.

---

## O **`ResponseEntity`**

---

### 1. O que é `ResponseEntity`

* É uma classe do Spring que representa **toda a resposta HTTP** que você quer enviar para o cliente.
* Ela não é apenas o **corpo** (como o retorno normal de um `@RestController`), mas também permite configurar:

  1. **Status HTTP** (200, 201, 404, 500 etc.)
  2. **Headers** (Content-Type, Location, Authorization etc.)
  3. **Body** (o conteúdo da resposta, que pode ser String, objeto Java, lista, DTO etc.)

---

### 2. Por que usar `ResponseEntity`

* Com o retorno direto (`Usuario`, `List<Usuario>`), o Spring assume **200 OK** e o `Content-Type` é definido automaticamente.
* Mas muitas vezes você precisa:

  * Devolver **404 Not Found** se o recurso não existir.
  * Devolver **201 Created** ao criar um recurso.
  * Adicionar headers extras (como `Location` em POST).
* `ResponseEntity` te dá esse controle total sobre a resposta HTTP.

---

### 3. Exemplos

#### 3.1 Retorno simples com status 200

```java
@GetMapping("/{id}")
public ResponseEntity<Usuario> buscarPorId(@PathVariable Long id) {
    Usuario u = new Usuario(id, "Milena", "milena@email.com");
    return ResponseEntity.ok(u); // 200 OK + body
}
```

#### 3.2 Retorno 404 se não encontrado

```java
@GetMapping("/{id}")
public ResponseEntity<Usuario> buscarPorId(@PathVariable Long id) {
    Usuario u = repository.findById(id);
    if (u == null) {
        return ResponseEntity.notFound().build(); // 404 sem body
    }
    return ResponseEntity.ok(u); // 200 + body
}
```

#### 3.3 Retorno 201 Created com header Location

```java
@PostMapping
public ResponseEntity<Usuario> criar(@RequestBody Usuario usuario) {
    Usuario criado = repository.save(usuario);
    URI uri = URI.create("/usuarios/" + criado.getId());
    return ResponseEntity.created(uri).body(criado); // 201 Created + Location + body
}
```

---

* `ResponseEntity` = **controle total da resposta HTTP**.

* Permite:

  * Definir **status** (`ok()`, `created()`, `notFound()`, `badRequest()` etc.)
  * Adicionar **headers** (`header("X-Custom", "valor")`)
  * Definir **body** (objeto Java que será serializado em JSON ou XML)

---

### `@ControllerAdvice` funciona como Aspect

Ao criar um `@ControllerAdvice`, o Spring automaticamente:

* Intercepta todas as requisições que chegam aos controllers.

* Observa se algum método lança uma exceção.

* Redireciona essa exceção para o método adequado marcado com `@ExceptionHandler`.

Ou seja, você está **separando a preocupação do tratamento de erro do código de negócio**, que é exatamente o espírito do AOP: cross-cutting concern.

`@ControllerAdvice` é uma forma específica de interceptação inspirada em AOP, focada em tratamento de exceções e aplicada apenas para controllers.

* Você não escreve pointcuts manualmente, mas o Spring internamente faz algo parecido ao interceptar o fluxo de execução do controller.

---

## Mantendo o Controller limpo

O ponto chave: **deixar o controller limpo**. 

O controller não devia ficar se preocupando com `try/catch`, muito menos com mensagem de erro ou status HTTP. Isso é papel de um “tradutor” global de exceções.

---

### 1. O desenho ideal

* **Service**: aplica a regra de negócio e lança exceções (personalizadas ou não).
* **Controller**: só recebe requisição → chama service → devolve resposta.
* **@RestControllerAdvice**: intercepta exceções que subirem e traduz para uma resposta HTTP adequada.

---

### 2. Criando uma exception customizada

Exemplo: você tem uma regra onde não pode adotar um pet que já foi adotado.

```java
public class PetJaAdotadoException extends RuntimeException {
    public PetJaAdotadoException(String message) {
        super(message);
    }
}
```

---

### 3. Usando a exception no Service

No service, você lança essa exception quando a regra for violada:

```java
@Service
public class AdocaoService {

    private final PetRepository petRepository;
    private final AdocaoRepository adocaoRepository;

    public AdocaoService(PetRepository petRepository, AdocaoRepository adocaoRepository) {
        this.petRepository = petRepository;
        this.adocaoRepository = adocaoRepository;
    }

    @Transactional
    public void adotar(Long petId, Long tutorId) {
        Pet pet = petRepository.findById(petId)
                .orElseThrow(() -> new IllegalArgumentException("Pet não encontrado"));

        if (pet.isAdotado()) {
            throw new PetJaAdotadoException("Este pet já foi adotado.");
        }

        // lógica de salvar adoção...
    }
}
```

---

### 4. Criando o “tradutor” global com `@RestControllerAdvice`

Aqui você centraliza o tratamento:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(PetJaAdotadoException.class)
    public ResponseEntity<ErroResponse> handlePetJaAdotado(PetJaAdotadoException ex) {
        ErroResponse erro = new ErroResponse(HttpStatus.BAD_REQUEST.value(), ex.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(erro);
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErroResponse> handleIllegalArgument(IllegalArgumentException ex) {
        ErroResponse erro = new ErroResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(erro);
    }
}
```

Classe de resposta de erro (pode ser um record, se preferir):

```java
public class ErroResponse {
    private int status;
    private String mensagem;

    public ErroResponse(int status, String mensagem) {
        this.status = status;
        this.mensagem = mensagem;
    }

    // getters e setters
}
```

---

### 5. Controller limpo

Repara como o controller agora só se preocupa com a requisição/resposta normal. Se algo der errado, a exception sobe e o `@RestControllerAdvice` resolve:

```java
@RestController
@RequestMapping("/adocoes")
public class AdocaoController {

    private final AdocaoService adocaoService;

    public AdocaoController(AdocaoService adocaoService) {
        this.adocaoService = adocaoService;
    }

    @PostMapping
    public ResponseEntity<Void> adotar(@RequestParam Long petId, @RequestParam Long tutorId) {
        adocaoService.adotar(petId, tutorId);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}
```

---
















