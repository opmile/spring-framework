# Implementação Teste em Controller

---

# 1. DTO com Bean Validation

```java
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;

public class UsuarioRequest {

    @NotBlank
    private String nome;

    @Email
    @NotBlank
    private String email;

    // getters e setters
}
```

```java
public class UsuarioResponse {

    private Long id;
    private String nome;
    private String email;

    // construtores, getters e setters
}
```

---

# 2. Controller (contexto mínimo)

```java
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/usuarios")
public class UsuarioController {

    private final UsuarioService service;

    public UsuarioController(UsuarioService service) {
        this.service = service;
    }

    @PostMapping
    public ResponseEntity<UsuarioResponse> criar(
            @Valid @RequestBody UsuarioRequest request) {

        UsuarioResponse response = service.criar(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }

    @GetMapping("/{id}")
    public ResponseEntity<UsuarioResponse> buscarPorId(@PathVariable Long id) {
        return ResponseEntity.ok(service.buscarPorId(id));
    }

    @PutMapping("/{id}")
    public ResponseEntity<UsuarioResponse> atualizar(
            @PathVariable Long id,
            @Valid @RequestBody UsuarioRequest request) {

        return ResponseEntity.ok(service.atualizar(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        service.deletar(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

# 3. Service (apenas contrato)

```java
public interface UsuarioService {

    UsuarioResponse criar(UsuarioRequest request);

    UsuarioResponse buscarPorId(Long id);

    UsuarioResponse atualizar(Long id, UsuarioRequest request);

    void deletar(Long id);
}
```

---

# 4. Classe de teste do Controller

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@SpringBootTest
@AutoConfigureMockMvc
class UsuarioControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @MockBean
    private UsuarioService service;
```

---

## 4.1 POST — sucesso (201 Created)

```java
    @Test
    void deveCriarUsuarioComDadosValidos() throws Exception {

        UsuarioRequest request = new UsuarioRequest();
        request.setNome("Milena");
        request.setEmail("milena@email.com");

        UsuarioResponse response =
                new UsuarioResponse(1L, "Milena", "milena@email.com");

        when(service.criar(any())).thenReturn(response);

        mockMvc.perform(post("/usuarios")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
               .andExpect(status().isCreated())
               .andExpect(jsonPath("$.id").value(1L))
               .andExpect(jsonPath("$.nome").value("Milena"));
    }
```

### O que esse teste garante

* `@RequestBody` funciona
* `@Valid` aceita dados corretos
* Status HTTP correto
* JSON de resposta correto

---

## 4.2 POST — erro de validação (400 Bad Request)

```java
    @Test
    void deveRetornar400QuandoEmailForInvalido() throws Exception {

        UsuarioRequest request = new UsuarioRequest();
        request.setNome("Milena");
        request.setEmail("email-invalido");

        mockMvc.perform(post("/usuarios")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
               .andExpect(status().isBadRequest());

        verify(service, never()).criar(any());
    }
```

### Ponto importante

* O service **não é chamado**
* A falha ocorre antes, no pipeline de validação

---

## 4.3 GET — sucesso (200 OK)

```java
    @Test
    void deveBuscarUsuarioPorId() throws Exception {

        UsuarioResponse response =
                new UsuarioResponse(1L, "Milena", "milena@email.com");

        when(service.buscarPorId(1L)).thenReturn(response);

        mockMvc.perform(get("/usuarios/{id}", 1L))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.email").value("milena@email.com"));
    }
```

---

## 4.4 PUT — sucesso (200 OK)

```java
    @Test
    void deveAtualizarUsuario() throws Exception {

        UsuarioRequest request = new UsuarioRequest();
        request.setNome("Milena Atualizada");
        request.setEmail("milena@novo.com");

        UsuarioResponse response =
                new UsuarioResponse(1L, "Milena Atualizada", "milena@novo.com");

        when(service.atualizar(eq(1L), any())).thenReturn(response);

        mockMvc.perform(put("/usuarios/{id}", 1L)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
               .andExpect(status().isOk())
               .andExpect(jsonPath("$.nome").value("Milena Atualizada"));
    }
```

---

## 4.5 PUT — erro de validação (400)

```java
    @Test
    void deveRetornar400AoAtualizarComNomeVazio() throws Exception {

        UsuarioRequest request = new UsuarioRequest();
        request.setNome("");
        request.setEmail("milena@email.com");

        mockMvc.perform(put("/usuarios/{id}", 1L)
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
               .andExpect(status().isBadRequest());

        verify(service, never()).atualizar(any(), any());
    }
```

---

## 4.6 DELETE — sucesso (204 No Content)

```java
    @Test
    void deveDeletarUsuario() throws Exception {

        doNothing().when(service).deletar(1L);

        mockMvc.perform(delete("/usuarios/{id}", 1L))
               .andExpect(status().isNoContent());

        verify(service).deletar(1L);
    }
```

---