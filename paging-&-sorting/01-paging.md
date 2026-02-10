# Paginação

---

Paginação é um **recurso estrutural**, não cosmético. Em APIs Spring Boot ela existe para **controlar volume de dados, custo computacional e previsibilidade de resposta**. Vou explicar **o porquê**, **como funciona** e **como se implementa corretamente**.

---

## 1. Por que paginação é necessária em APIs

Sem paginação, uma API que expõe listas:

* retorna grandes volumes de dados
* consome mais memória
* aumenta tempo de resposta
* sobrecarrega rede e banco
* não escala

Exemplo clássico:

```http
GET /usuarios
```

Se houver 200 mil registros:

* a API tenta carregar tudo
* o banco sofre
* o cliente sofre
* o sistema quebra

Paginação resolve isso **limitando e organizando o acesso**.

---

## 2. Conceito de paginação (independente de framework)

Paginação divide um conjunto grande em **fatias ordenadas**.

Parâmetros básicos:

* **page** → qual página (base 0)
* **size** → quantos registros por página
* **sort** → ordenação

Exemplo:

```http
GET /usuarios?page=0&size=20&sort=nome,asc
```

Significa:

* primeira página
* 20 registros
* ordenados por nome crescente

---

## 3. Como o Spring implementa paginação (Spring Data)

O Spring Data abstrai paginação com duas interfaces centrais:

### `Pageable`

Representa o pedido de página:

* página
* tamanho
* ordenação

### `Page<T>`

Representa a resposta paginada:

* conteúdo
* metadados
* total de registros

Isso permite paginação **automática**, sem SQL manual.

---

## 4. Funcionamento interno (o que realmente acontece)

Quando você usa paginação:

1. O Spring cria um objeto `Pageable`
2. O repositório traduz isso em SQL
3. O banco executa algo como:

```sql
SELECT *
FROM usuario
ORDER BY nome ASC
LIMIT 20 OFFSET 0;
```

Além disso, o Spring executa uma **query extra**:

```sql
SELECT COUNT(*)
FROM usuario;
```

Essa segunda query é necessária para:

* saber quantas páginas existem
* preencher metadados (`totalElements`, `totalPages`)

Isso tem custo. Não é “de graça”.

---

## 5. Implementação básica

### Repository

```java
public interface UsuarioRepository
        extends JpaRepository<Usuario, Long> {
}
```

`JpaRepository` já expõe:

```java
Page<Usuario> findAll(Pageable pageable);
```

---

### Controller

```java
@GetMapping("/usuarios")
public Page<Usuario> listar(Pageable pageable) {
    return usuarioRepository.findAll(pageable);
}
```

Pronto.
O Spring já entende automaticamente parâmetros de paginação via query string.

---

## 6. Exemplo de resposta da API

```json
{
  "content": [
    { "id": 1, "nome": "Ana" },
    { "id": 2, "nome": "Bruno" }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 2
  },
  "totalElements": 50,
  "totalPages": 25,
  "last": false,
  "first": true
}
```

Isso dá ao cliente **controle total de navegação**.

---

## 7. Paginação com DTO (cenário real)

Nunca exponha entidade diretamente.

```java
@GetMapping("/usuarios")
public Page<UsuarioDTO> listar(Pageable pageable) {
    return usuarioRepository
        .findAll(pageable)
        .map(UsuarioDTO::new);
}
```

O método `map`:

* mantém os metadados
* transforma apenas o conteúdo

---

## 8. Configurações importantes

### Tamanho máximo de página

Evite abuso do cliente:

```yaml
spring:
  data:
    web:
      pageable:
        max-page-size: 100
```

```properties
spring.data.web.pageable.max-page-size=100
```

Ele não define o tamanho padrão da página.
Ele não impede paginação.
Ele não retorna erro HTTP se o valor for maior.

Ele apenas protege o backend contra abuso ou uso ingênuo.

---

### Ordenação padrão

```java
@GetMapping("/usuarios")
public Page<UsuarioDTO> listar(
    @PageableDefault(
        size = 20,
        sort = "nome",
        direction = Sort.Direction.ASC
    ) Pageable pageable
) {
    ...
}
```

---

## 9. Cuidado com performance

### Problema comum

Em tabelas grandes:

* `COUNT(*)` pode ser caro
* paginação profunda (`page=5000`) degrada

### Alternativas

* Limitar profundidade de páginas
* Usar `Slice<T>` quando não precisa do total
* Paginação baseada em cursor (casos avançados)

---

* Sempre pagine endpoints de listagem
* Nunca confie no `size` enviado pelo cliente
* Use índices nos campos de ordenação
* Evite paginação profunda
* Prefira DTOs
* Monitore custo do `COUNT`

---
