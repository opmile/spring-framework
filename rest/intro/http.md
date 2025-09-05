# HTTP

## Métodos HTTP

Quando você acessa ou interage com um recurso via REST, você **escolhe a ação** que quer realizar sobre ele.
Essa ação é representada por um **método HTTP** (também chamado de *verbo HTTP*).

No REST, esses métodos são usados de forma **semântica**: cada um tem um significado específico e padronizado.

---

1. **GET**

   * **Usado para:** buscar dados.
   * **Exemplo:** `GET /usuarios` → lista de usuários.
   * **Obs:** nunca deve alterar dados (é seguro e idempotente).

2. **POST**

   * **Usado para:** criar um recurso novo.
   * **Exemplo:** `POST /usuarios` com um JSON no corpo → cria um usuário novo.
   * **Obs:** não é idempotente (enviar duas vezes cria dois recursos).

3. **PUT**

   * **Usado para:** atualizar **um recurso inteiro**.
   * **Exemplo:** `PUT /usuarios/10` com um JSON → substitui os dados do usuário 10.
   * **Obs:** é idempotente (mesmo update enviado várias vezes gera sempre o mesmo estado).

4. **PATCH**

   * **Usado para:** atualizar **parcialmente** um recurso.
   * **Exemplo:** `PATCH /usuarios/10` com `{ "email": "novo@email.com" }` → altera só o email.
   * **Obs:** também é idempotente na prática, mas menos formal que PUT.

5. **DELETE**

   * **Usado para:** remover um recurso.
   * **Exemplo:** `DELETE /usuarios/10` → apaga o usuário 10.
   * **Obs:** idempotente (deletar duas vezes ainda resulta em “não existe”).

---

### Outros métodos (menos comuns em APIs REST)

* **HEAD** → igual ao `GET`, mas retorna apenas os headers (sem corpo).
* **OPTIONS** → mostra quais métodos são permitidos em um endpoint.
* **TRACE** → retorna a requisição recebida (quase nunca usado em APIs).

---

| Método | Ação                   | Idempotente? | Altera estado? |
| ------ | ---------------------- | ------------ | -------------- |
| GET    | Buscar recurso         | ✅ Sim        | ❌ Não          |
| POST   | Criar recurso          | ❌ Não        | ✅ Sim          |
| PUT    | Atualizar recurso todo | ✅ Sim        | ✅ Sim          |
| PATCH  | Atualizar parcial      | ✅ Sim\*      | ✅ Sim          |
| DELETE | Remover recurso        | ✅ Sim        | ✅ Sim          |

> \*Idempotente = enviar a mesma requisição várias vezes gera o mesmo resultado.

---

Pense nos métodos HTTP como **os botões de ação em um CRUD**:

* `GET` = ler
* `POST` = criar
* `PUT` = atualizar tudo
* `PATCH` = atualizar parte
* `DELETE` = remover

---

## Status Code

* Sempre que você faz uma requisição (ex: `GET /usuarios`), o servidor responde não só com os dados, mas também com um **código numérico**.
* Esse código indica **se deu certo, se deu errado ou o que o cliente deve fazer**.
* Eles são divididos em **faixas numéricas (classes)**:

---

### Classes de status codes

1. **1xx — Informativo**

   * Raramente usado em APIs REST.
   * Exemplo: `100 Continue` → servidor aceitou o início da requisição.

2. **2xx — Sucesso**

   * Indicam que a requisição foi processada corretamente.
   * Mais usados:

     * `200 OK` → requisição bem-sucedida (ex: `GET /usuarios`).
     * `201 Created` → recurso criado com sucesso (ex: `POST /usuarios`).
     * `204 No Content` → operação deu certo, mas não há nada para retornar (ex: `DELETE /usuarios/10`).

3. **3xx — Redirecionamento**

   * Indicam que o cliente precisa buscar a resposta em outro lugar.
   * Mais usados:

     * `301 Moved Permanently` → recurso mudou de endereço. Normalmente se você usou `HTTP` e não `HTTPS`
     * `302 Found` → redirecionamento temporário.

4. **4xx — Erro do cliente**

   * Indicam que o problema está na **requisição enviada**.
   * Mais usados:

     * `400 Bad Request` → requisição malformada (ex: JSON inválido).
     * `401 Unauthorized` → não autenticado (precisa de login/token).
     * `403 Forbidden` → autenticado, mas sem permissão.
     * `404 Not Found` → recurso não existe.
     * `409 Conflict` → conflito de estado (ex: tentar cadastrar usuário já existente).
     * `422 Unprocessable Entity` → dados válidos em formato, mas sem lógica (ex: data de nascimento no futuro).

5. **5xx — Erro do servidor**

   * Indicam que o problema está no **lado do servidor**.
   * Mais usados:

     * `500 Internal Server Error` → erro genérico interno.
     * `502 Bad Gateway` → problema em comunicação entre serviços.
     * `503 Service Unavailable` → servidor temporariamente indisponível.
     * `504 Gateway Timeout` → servidor demorou demais para responder.

---

* `GET /usuarios/10` → retorna `200 OK` com o JSON do usuário.
* `GET /usuarios/999` (usuário não existe) → retorna `404 Not Found`.
* `POST /usuarios` com JSON válido → retorna `201 Created` com o novo usuário.
* `POST /usuarios` com JSON malformado → retorna `400 Bad Request`.
* `DELETE /usuarios/10` → retorna `204 No Content` (apagado, sem resposta no corpo).

---

## Recursos

Em REST, tudo gira em torno de **recursos**.

* Um **recurso** é **qualquer entidade que sua API disponibiliza**: usuários, produtos, pedidos, relatórios etc.
* Cada recurso deve ter um **identificador único (URI)**.

Se a Web é feita de páginas, o REST é feito de recursos.

---

### Como modelar URIs (boas práticas)

1. **Nomes de recursos no plural**

   * `GET /usuarios`
   * `GET /getUsuario` (isso mistura com verbo, o método HTTP já diz a ação)

2. **Hierarquia de recursos**

   * Relacionamentos são representados de forma hierárquica.
   * Exemplo: pedidos de um usuário →

     * `GET /usuarios/10/pedidos` → lista de pedidos do usuário 10
     * `GET /usuarios/10/pedidos/5` → pedido específico

3. **Usar path params para identificar recursos**

   * `GET /produtos/123` (produto com ID 123)
   * `GET /produtos?id=123` (pode usar query param, mas path é melhor para identidade única)

4. **Query params para filtros, paginação e ordenação**

   * `GET /produtos?categoria=eletronicos&pagina=2&ordem=asc`

5. **Sem verbos nos endpoints** (já que o método HTTP é o verbo)

   * `POST /usuarios` (cria usuário)
   * `POST /criarUsuario`

6. **Snake case ou kebab case?**

   * Geralmente, APIs usam **kebab-case** ou **camelCase** para parâmetros, mas **recursos no plural** são mais comuns em minúsculo:
   * `/usuarios`, `/produtos-em-promocao`

7. **Versão da API** (quando necessário)

   * `GET /api/v1/usuarios`
   * Permite evoluir sem quebrar clientes antigos.

---

### Exemplo de modelagem REST bem feita

Um e-commerce:

* **Usuários**

  * `GET /usuarios` → lista usuários
  * `GET /usuarios/10` → detalhes do usuário 10
  * `POST /usuarios` → cria usuário

* **Produtos**

  * `GET /produtos` → lista produtos
  * `GET /produtos/123` → detalhes do produto 123
  * `POST /produtos` → cria produto

* **Pedidos** (relacionados a usuários)

  * `GET /usuarios/10/pedidos` → pedidos do usuário 10
  * `POST /usuarios/10/pedidos` → cria um pedido para usuário 10
  * `GET /usuarios/10/pedidos/5` → detalhes do pedido 5 do usuário 10

---

Se REST fosse um condomínio, cada recurso seria um **apartamento identificado por uma porta (URI)**. Você usa o **método HTTP** como a ação (bater na porta para ler, adicionar móveis, reformar, ou até demolir).

---

## Headers e Body

Perfeito, vamos sem emojis então.

---

## Headers e Body em REST

Quando cliente e servidor se comunicam em REST, a requisição HTTP não contém apenas o método (GET, POST etc.) e a URI. Ela também pode ter **informações adicionais (headers)** e **conteúdo principal (body)**.

### 1. Headers (cabeçalhos HTTP)

São pares **chave\:valor** enviados na requisição ou na resposta.
Eles **não são o conteúdo em si**, mas sim **metadados sobre a comunicação**.

**Exemplos comuns em APIs REST:**

* **`Content-Type`** → indica o formato do corpo da mensagem.

  * Ex: `application/json`, `application/xml`.
* **`Accept`** → indica o formato que o cliente aceita na resposta.

  * Ex: `Accept: application/json`.
* **`Authorization`** → usado para autenticação.

  * Ex: `Authorization: Bearer <token>`.
* **`Cache-Control`** → define regras de cache.
* **`Location`** → na resposta de `POST`, indica o endereço do recurso criado.

### 2. Body (corpo da requisição/resposta)

É o **conteúdo principal** que está sendo enviado ou recebido.

* Usado em `POST`, `PUT`, `PATCH` (porque esses métodos enviam dados).
* Normalmente é em **JSON** em APIs REST modernas.

**Exemplo de body em JSON (requisição):**

```http
POST /usuarios
Content-Type: application/json

{
  "nome": "Milena",
  "email": "milena@email.com"
}
```

**Exemplo de body em JSON (resposta):**

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /usuarios/123

{
  "id": 123,
  "nome": "Milena",
  "email": "milena@email.com"
}
```

### 3. Relação entre URI, Headers e Body

* **URI** → diz *o que* você está acessando (`/usuarios/123`).
* **Método HTTP** → diz *a ação* que você quer (`GET`, `POST`, `DELETE`).
* **Headers** → dão *informações adicionais* sobre como processar a requisição (`Content-Type`, `Authorization`).
* **Body** → contém os dados que estão sendo enviados ou recebidos.

---

* **Headers** = instruções extras sobre a comunicação.
* **Body** = dados principais da requisição ou resposta.

---

## JSON e XML em Rest

Boa! Então vamos para **JSON e XML em REST**.

---

## JSON e XML em REST

APIs REST precisam de um formato para enviar e receber dados entre cliente e servidor. Os dois principais formatos históricos são **XML** e **JSON**.

### 1. XML (Extensible Markup Language)

* Foi muito usado nos anos 2000.
* Estrutura baseada em **tags aninhadas**, parecida com HTML.
* Exemplo de um usuário em XML:

```xml
<usuario>
  <id>123</id>
  <nome>Milena</nome>
  <email>milena@email.com</email>
</usuario>
```

**Prós:**

* Muito flexível e padronizado.
* Permite validação com XSD (schemas).
* Suporta metadados via atributos.

**Contras:**

* Verboso (muito texto para pouca informação).
* Mais difícil de manipular em JavaScript (precisa de parser extra).

---

### 2. JSON (JavaScript Object Notation)

* Hoje é o padrão **de facto** para REST.
* Estrutura baseada em pares **chave\:valor**, muito mais simples.
* Exemplo do mesmo usuário em JSON:

```json
{
  "id": 123,
  "nome": "Milena",
  "email": "milena@email.com"
}
```

**Prós:**

* Mais leve e menos verboso que XML.
* Manipulado nativamente em JavaScript e facilmente em outras linguagens.
* Melhor para trafegar em rede (menos bytes).
* Leitura mais simples para humanos.

**Contras:**

* Menos formal em validação que XML (embora existam JSON Schemas).
* Não possui comentários nativamente.

---

### 3. Como REST lida com JSON e XML

REST não obriga um formato específico, mas:

* **Hoje a maioria das APIs REST usa JSON**.
* Algumas ainda oferecem XML como opção.
* O cliente pode dizer o que aceita usando o header **`Accept`**:

  * `Accept: application/json` → quero resposta em JSON.
  * `Accept: application/xml` → quero resposta em XML.

Exemplo de resposta em JSON:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "produto": "Camiseta",
  "preco": 49.90
}
```

Exemplo de resposta em XML:

```http
HTTP/1.1 200 OK
Content-Type: application/xml

<produto>
  <nome>Camiseta</nome>
  <preco>49.90</preco>
</produto>
```

---

### 4. Conclusão

* REST é **agnóstico de formato** → pode ser JSON, XML, YAML, até texto puro.
* Mas **JSON virou o padrão dominante** por ser mais simples, leve e fácil de integrar com aplicações web modernas.

---









