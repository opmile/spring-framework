# Versionamento e Documentação

---

## Versionamento

### 1. Por que versionar

Imagine que você tem um endpoint `/usuarios` e decide mudar a estrutura do JSON ou adicionar campos obrigatórios.

* Clientes antigos ainda usam o endpoint antigo.
* Sem versionamento, você **quebra a compatibilidade**, o que pode gerar erros em produção.

Versionamento permite **coexistir múltiplas versões da API**.

---

### 2. Principais formas de versionar

#### 2.1 Via URL (mais comum)

* Adiciona a versão **no path da URL**:

```java
@RestController
@RequestMapping("/api/v1/usuarios")
public class UsuarioControllerV1 {
    // endpoints
}

@RestController
@RequestMapping("/api/v2/usuarios")
public class UsuarioControllerV2 {
    // endpoints com mudanças
}
```

**Exemplo de chamada:**

* `GET /api/v1/usuarios` → retorna versão 1
* `GET /api/v2/usuarios` → retorna versão 2

**Prós:** simples e visível para o cliente
**Contras:** URLs podem ficar longas, mantém código duplicado se não houver abstração

---

#### 2.2 Via Header HTTP

* Cliente envia um **header customizado** indicando a versão:

```http
GET /usuarios
Accept: application/vnd.minhaapi.v1+json
```

* Controller pode usar `@RequestMapping` ou **content negotiation** para decidir a versão.

**Prós:** URL limpa, fácil de evoluir
**Contras:** menos óbvio, exige documentação clara

---

#### 2.3 Via Query Parameter

* Cliente indica versão na query string:

```
GET /usuarios?version=1
```

**Contras:** menos usada, pode poluir endpoints complexos

---

### 3. Boas práticas

1. **Sempre inclua a versão na URL ou header** desde o início.
2. **Não modifique contratos existentes** de versões antigas.

   * Novos campos → opcionais
   * Mudanças incompatíveis → nova versão
3. **Use semântica de versão incremental** (v1, v2, v3).
4. **Documente todas as versões** (Swagger/OpenAPI ajuda bastante).
5. **Mantenha código compartilhado** em serviços/serviços auxiliares, para não duplicar lógica entre versões.
6. **Depreque versões antigas com aviso**, mas não as remova abruptamente.

---

#### 4. Exemplo em Spring Boot usando URL

```java
@RestController
@RequestMapping("/api/v1/usuarios")
public class UsuarioControllerV1 {
    @GetMapping
    public List<Usuario> findAllV1() {
        // campos básicos
    }
}

@RestController
@RequestMapping("/api/v2/usuarios")
public class UsuarioControllerV2 {
    @GetMapping
    public List<Usuario> findAllV2() {
        // campos adicionais
    }
}
```

Aqui, os clientes escolhem a versão **pela URL**, e você consegue evoluir a API sem quebrar quem está usando a versão antiga.

---

Perfeito, Milena! Documentar uma API REST é essencial para que **clientes e outros desenvolvedores entendam rapidamente como usar sua API**. No ecossistema Spring Boot, a maneira mais popular de fazer isso é usando **Swagger / OpenAPI**. Vamos passo a passo.

---

## Documentação

---

### 1. Dependência necessária

Com Spring Boot, o mais comum é usar a biblioteca **springdoc-openapi**, que gera documentação OpenAPI automaticamente a partir de seus controllers e anotações.

No `pom.xml` (Maven):

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.1.0</version> <!-- verifique a versão mais recente -->
</dependency>
```

Essa dependência já inclui o **Swagger UI**, que permite ver a documentação via navegador.

---

### 2. Configuração básica

Na maioria dos casos, não é necessário criar nenhuma classe de configuração.
Basta ter seus controllers anotados normalmente, e o Spring Boot + SpringDoc já gera a documentação.

* Acesse a interface Swagger UI padrão:

```
http://localhost:8080/swagger-ui.html
```

ou

```
http://localhost:8080/swagger-ui/index.html
```

---

### 3. Anotando controllers para enriquecer a documentação

Você pode usar anotações do **OpenAPI** para deixar os endpoints claros.

### Exemplo:

```java
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;

@RestController
@RequestMapping("/api/v1/usuarios")
@Tag(name = "Usuários", description = "Endpoints para gerenciar usuários")
public class UsuarioController {

    @GetMapping("/{id}")
    @Operation(summary = "Buscar usuário por ID", description = "Retorna os dados de um usuário específico")
    public Usuario buscarPorId(@PathVariable Long id) {
        return new Usuario(id, "Milena", "milena@email.com");
    }

    @PostMapping
    @Operation(summary = "Criar usuário", description = "Cria um novo usuário com os dados fornecidos")
    public Usuario criar(@RequestBody Usuario usuario) {
        return usuario; // mock
    }
}
```

### O que acontece:

* `@Tag` → organiza endpoints por categoria.
* `@Operation` → adiciona resumo e descrição de cada endpoint.
* Swagger UI vai mostrar todas essas informações de forma interativa.

---

### 4. Boas práticas

1. **Use `@Operation` em todos os endpoints importantes**, descrevendo comportamento, parâmetros e respostas.
2. **Documente `@RequestBody` e `@PathVariable`/`@RequestParam`**, para que o Swagger entenda os tipos e nomes.
3. **Use códigos HTTP corretos no retorno** (ex.: `ResponseEntity`), para que o Swagger reflita os status corretamente.
4. **Versão da API** → pode ser colocada no path (`/api/v1/...`) e será refletida na documentação.
5. **Teste interativo** → a UI permite que você envie requisições direto do navegador para testar endpoints.

---

### 5. Resultado final

Depois de rodar a aplicação:

* Acesse `http://localhost:8080/swagger-ui/index.html`
* Você verá todos os endpoints documentados, com parâmetros, descrições e exemplos de JSON de entrada/saída.
* Dá para gerar **OpenAPI JSON/YAML** para integração com outras ferramentas, como Postman ou geração de clientes.

---



