# Ordenação

---

## 1. Por que ordenação é necessária

Sem ordenação explícita:

* o banco **não garante ordem**
* resultados podem variar entre requisições
* paginação perde consistência
* páginas podem repetir ou “pular” registros

Regra fundamental:

> **Paginação sem ordenação é conceitualmente incorreta.**

---

## 2. Como o Spring implementa ordenação

O Spring Data usa a abstração `Sort`, integrada ao `Pageable`.

Na prática, isso significa que:

* o cliente pode definir campos e direção
* o Spring traduz isso para `ORDER BY`
* a ordenação acontece **no banco**, não na aplicação

---

## 3. Ordenação via requisição (forma padrão)

Exemplo:

```http
GET /usuarios?sort=nome,asc
```

Ou múltiplos critérios:

```http
GET /usuarios?sort=nome,asc&sort=dataCadastro,desc
```

Interpretação:

* primeiro ordena por `nome`
* em caso de empate, ordena por `dataCadastro`

---

## 4. Integração com paginação

Ordenação e paginação **andam juntas**.

```http
GET /usuarios?page=0&size=20&sort=nome,asc
```

O SQL gerado será conceitualmente:

```sql
SELECT *
FROM usuario
ORDER BY nome ASC
LIMIT 20 OFFSET 0;
```

Sem `ORDER BY`, o banco pode retornar resultados em ordem arbitrária.

---

## 5. Ordenação padrão (boa prática)

Definir uma ordenação padrão evita comportamento imprevisível.

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

Se o cliente não enviar `sort`, essa será usada.

---

## 6. Restringindo campos de ordenação

Problema comum:

* cliente ordena por qualquer campo
* risco de performance ruim
* ordenação por campos não indexados

Solução:

* validar manualmente
* ou mapear ordenações permitidas

Conceitualmente:

> **nem todo campo deve ser ordenável via API pública**.

---

## 7. Impacto em performance

Ordenação:

* força `ORDER BY`
* pode invalidar uso de índices
* é cara em tabelas grandes

Boas práticas:

* ordenar por campos indexados
* evitar ordenação por colunas textuais grandes
* limitar múltiplos critérios

---

## 8. Ordenação e consistência

Para paginação estável, a ordenação deve:

* ser determinística
* incluir campo único quando possível (ex: `id`)

Exemplo ideal:

```text
ORDER BY dataCadastro DESC, id DESC
```

Isso evita:

* registros duplicados entre páginas
* “saltos” de dados

---

## 9. Ordenação vs regra de negócio

Ordenação:

* é responsabilidade da API
* não deve embutir regra de negócio complexa

Se a ordem tem significado de domínio:

* deixe explícito
* documente
* evite ordens implícitas

---