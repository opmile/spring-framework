# Teste em Controllers

---

## 1. O que é um teste de Controller

Um teste de controller **verifica o comportamento da camada web** da aplicação Spring:

* Se uma **requisição HTTP** chega corretamente ao controller
* Se os **parâmetros / body / headers** são interpretados
* Se o **status HTTP** retornado é o esperado
* Se o **JSON de resposta** está correto

Ele **não testa regra de negócio** em profundidade (isso é papel de service).

---

## 2. Por que usar MockMvc

`MockMvc` **simula chamadas HTTP reais**, mas **sem subir o servidor**.

Ele permite:

* Simular `GET`, `POST`, `PUT`, `DELETE`
* Enviar JSON no corpo da requisição
* Verificar status HTTP
* Verificar conteúdo da resposta

Na prática: você testa seu controller **como se fosse um cliente HTTP**, mas tudo roda em memória.

---

## 3. Por que usar `@SpringBootTest`

`@SpringBootTest`:

* Sobe **o contexto completo do Spring**
* Registra controllers, services, repositories, filtros, etc.

Com isso:

* O teste se aproxima muito do comportamento real da aplicação
* Ideal para **testes de integração da camada web**

Normalmente é usado junto com:

```java
@AutoConfigureMockMvc
```

* O **ApplicationContext** do Spring é carregado completamente
* Todos os beans são instanciados (controllers, services, filtros, etc.)
* O **MockMvc** intercepta as requisições **antes** da camada de rede

Ou seja:

* Não há Tomcat/Jetty escutando porta
* Não existe socket, porta, nem servidor web real
* Tudo acontece **em memória**

---

### Por que isso é possível

O Spring MVC é desacoplado do container HTTP.
O `MockMvc` conversa diretamente com o `DispatcherServlet`, que é o núcleo do Spring MVC.

Fluxo real do teste:

```
MockMvc
 → DispatcherServlet
   → Controller
     → Service
       → Repository
```

Sem passar por rede.

---

### Quando o servidor sobe de fato

O servidor **só sobe** se você usar:

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
// ou
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
```

Nesses casos:

* Um servidor real é iniciado
* A aplicação escuta uma porta
* Você testa via HTTP real (RestTemplate, WebTestClient, etc.)

---

## 4. Estrutura básica do teste

### Classe de teste

```java
@SpringBootTest
@AutoConfigureMockMvc
class UsuarioControllerTest {

    @Autowired
    private MockMvc mockMvc;

}
```

### O que está acontecendo aqui

* `@SpringBootTest`: carrega o contexto da aplicação
* `@AutoConfigureMockMvc`: configura o MockMvc automaticamente
* `MockMvc`: objeto que dispara as requisições HTTP simuladas

---

## 5. Testando um endpoint GET

### Controller (exemplo mental)

```java
@GetMapping("/usuarios/{id}")
public ResponseEntity<UsuarioDTO> buscarPorId(@PathVariable Long id) {
    return ResponseEntity.ok(service.buscar(id));
}
```

---

### Teste

```java
@Test
void deveBuscarUsuarioPorId() throws Exception {

    mockMvc.perform(get("/usuarios/{id}", 1L))
           .andExpect(status().isOk())
           .andExpect(jsonPath("$.id").value(1L));
}
```

### Leitura didática do teste

1. `perform(get(...))`

   * Simula um `GET /usuarios/1`

2. `andExpect(status().isOk())`

   * Verifica se o controller retorna HTTP 200

3. `jsonPath("$.id")`

   * Acessa o campo `id` do JSON de resposta
   * Verifica se o valor é `1`

Esse teste **não chama o controller diretamente**.
Ele passa por:

* Mapeamento da URL
* Conversão de parâmetros
* Serialização JSON
* Pipeline HTTP do Spring

---

## 6. Testando POST com body JSON

### Controller

```java
@PostMapping("/usuarios")
public ResponseEntity<UsuarioDTO> criar(@RequestBody UsuarioDTO dto) {
    return ResponseEntity.status(HttpStatus.CREATED)
                         .body(service.criar(dto));
}
```

---

### Teste

```java
@Test
void deveCriarUsuario() throws Exception {

    String json = """
        {
          "nome": "Milena",
          "email": "milena@email.com"
        }
        """;

    mockMvc.perform(post("/usuarios")
            .contentType(MediaType.APPLICATION_JSON)
            .content(json))
           .andExpect(status().isCreated())
           .andExpect(jsonPath("$.nome").value("Milena"));
}
```

### Pontos-chave

* `contentType(APPLICATION_JSON)`

  * Simula um cliente enviando JSON

* `content(json)`

  * Corpo da requisição

* `@RequestBody`

  * É realmente testado aqui: desserialização acontece de verdade

---

## 7. Testando erros (status 400, 404, etc.)

### Exemplo: validação com Bean Validation

```java
@PostMapping("/usuarios")
public ResponseEntity<UsuarioDTO> criar(@Valid @RequestBody UsuarioDTO dto) {
    ...
}
```

---

### Teste de erro

```java
@Test
void deveRetornarBadRequestQuandoNomeForNulo() throws Exception {

    String json = """
        {
          "email": "teste@email.com"
        }
        """;

    mockMvc.perform(post("/usuarios")
            .contentType(MediaType.APPLICATION_JSON)
            .content(json))
           .andExpect(status().isBadRequest());
}
```

Aqui você valida:

* Integração com `@Valid`
* Validação automática
* Mapeamento correto do erro HTTP

---

## 8. Controller vs Service nos testes

Com `@SpringBootTest`, você tem duas opções:

### 1. Teste mais realista (Service real)

* Usa o service real
* Pode bater em banco (ou H2)
* Teste mais lento

### 2. Isolar o controller (mais comum)

```java
@MockBean
private UsuarioService service;
```

Isso:

* Substitui o bean real por um mock
* Permite controlar o retorno do service
* Foca **exclusivamente no comportamento do controller**

Exemplo:

```java
when(service.buscar(1L)).thenReturn(new UsuarioDTO(1L, "Milena"));
```

---
