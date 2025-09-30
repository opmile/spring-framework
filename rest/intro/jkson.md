# Jackson e Spring Boot

---

## 1. Quando você estudou Jackson isoladamente

No estudo “puro” do **Jackson**, você lidava diretamente com a API:

* Criava um objeto `ObjectMapper` manualmente:

  ```java
  ObjectMapper mapper = new ObjectMapper();
  String json = mapper.writeValueAsString(obj);
  MeuObjeto obj = mapper.readValue(json, MeuObjeto.class);
  ```
* Configurava módulos extras (JavaTimeModule, por exemplo, para datas).
* Lidava com exceções diretamente (`JsonProcessingException`).
* Escolhia manualmente como serializar/desserializar (inclusão de nulls, snake\_case vs camelCase, etc.).

Ou seja, **você controlava tudo**, mas também tinha que lidar com toda a verbosidade.

---

## 2. O que o Spring MVC já trazia (antes do Boot)

O **Spring MVC** já fazia uso interno do **Jackson** como uma implementação de `HttpMessageConverter` (quando ele estava no classpath).
Isso significa:

* Você não precisava chamar o `ObjectMapper` manualmente.
* Quando declarava um endpoint assim:

  ```java
  @PostMapping("/usuarios")
  public Usuario criar(@RequestBody Usuario usuario) {
      return usuario;
  }
  ```

  O Spring MVC pegava o **JSON recebido**, usava Jackson para **desserializar** em `Usuario` e, ao retornar o objeto, usava Jackson de novo para **serializar** em JSON.

Esse mecanismo é o **MessageConverter**.
No caso de JSON, quem cuida é o `MappingJackson2HttpMessageConverter`.

Mas veja: **essa integração era configurável, mas você ainda precisava registrar beans e mexer na configuração manualmente** se quisesse ajustar o `ObjectMapper`.

---

## 3. O que o Spring Boot acrescentou

É aqui que a mágica do Boot entra. O Boot traz **auto-configuration** para o Jackson:

* **Auto-detecção**:
  Se Jackson (`com.fasterxml.jackson.databind.ObjectMapper`) está no classpath, o Spring Boot automaticamente registra um `MappingJackson2HttpMessageConverter` com um `ObjectMapper`.

* **Bean pronto de ObjectMapper**:
  O Boot já cria e registra um `ObjectMapper` no contexto do Spring como um `@Bean`.
  Assim você pode **injetar** em qualquer lugar:

  ```java
  @Autowired
  private ObjectMapper mapper;
  ```

* **Configuração centralizada**:
  Você consegue customizar o Jackson via `application.properties` sem precisar mexer em código, exemplo:

  ```properties
  spring.jackson.serialization.indent-output=true
  spring.jackson.property-naming-strategy=SNAKE_CASE
  ```

  Isso já aplica automaticamente no `ObjectMapper` que o Boot mantém.

* **Customização avançada via beans**:
  Quer mexer ainda mais? Basta expor um bean `Jackson2ObjectMapperBuilderCustomizer` ou até mesmo um `ObjectMapper` próprio, e o Boot usa o seu no lugar:

  ```java
  @Bean
  public Jackson2ObjectMapperBuilderCustomizer customizer() {
      return builder -> builder
          .simpleDateFormat("yyyy-MM-dd")
          .serializationInclusion(JsonInclude.Include.NON_NULL);
  }
  ```

* **Módulos extras já plugados**:
  Exemplo: se você tem o módulo `jackson-datatype-jsr310` no classpath, o Boot já registra automaticamente para suporte a `LocalDate`, `Instant`, etc.
  Ou seja, não precisa mais chamar `mapper.registerModule(new JavaTimeModule())`.

---

## O que mudou e o que persiste?

**Persistiu**:

* O motor por baixo dos panos continua sendo o mesmo Jackson (`ObjectMapper` + anotações como `@JsonProperty`, `@JsonIgnore`, etc.).
* A lógica de serialização/desserialização é a mesma — nada de sintaxe nova.

**Mudou com o Boot**:

* Você **não instancia mais `ObjectMapper`** manualmente — ele já vem configurado como bean.
* Você **não precisa registrar manualmente os módulos** comuns — o Boot cuida disso.
* Configurações comuns ficam centralizadas em `application.properties` em vez de espalhadas no código.
* Extensões/correções podem ser feitas de forma declarativa via Beans ou Customizers.

---

### Annotations:

* Se você precisa mudar o **nome de uma propriedade** exposta no JSON, ainda usa:

  ```java
  public class Usuario {
      @JsonProperty("user_name")
      private String nome;
  }
  ```
* Se quiser **ignorar um campo**:

  ```java
  @JsonIgnore
  private String senha;
  ```
* Se quiser controlar serialização condicional, ordem, etc., continua usando `@JsonInclude`, `@JsonFormat`, `@JsonPropertyOrder`…

Essas anotações são da **própria biblioteca Jackson**, e o Spring Boot não substitui isso. Ele simplesmente **reconhece e respeita** essas anotações no `ObjectMapper` que ele mantém.

---

### Onde o Spring Boot te dá atalhos

O Boot permite que você defina **comportos globais** sem precisar anotar cada campo manualmente:

* Arquivo `application.properties`:

  ```properties
  spring.jackson.default-property-inclusion=non_null
  spring.jackson.property-naming-strategy=SNAKE_CASE
  ```
* Customização com `Jackson2ObjectMapperBuilderCustomizer`, se quiser aplicar regras a todos os objetos sem precisar anotar.

---

* Você continua usando as anotações quando precisa de **controle específico por classe/campo**.
* **Não**, você não é obrigado a anotar tudo: o Spring Boot te dá mecanismos globais para muitas dessas configurações (estilo de nomes, inclusão/exclusão de `nulls`, formato de datas etc.).

---

## Anotações Jackson vs Configuração Global no Spring Boot

| **Objetivo**                                               | **Quando usar Anotações (@Json… etc.)**                                                                         | **Quando usar Configuração Global (Spring Boot)**                                                                                         |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Renomear um campo específico**                           | `@JsonProperty("user_name")` em cima de um atributo. Útil quando só uma classe/campo precisa de nome diferente. | ❌ Não aplicável globalmente — cada campo teria nomes diferentes.                                                                          |
| **Ignorar um campo**                                       | `@JsonIgnore` para não serializar aquele campo específico.                                                      | ❌ Global não faz sentido — você quer isso só em alguns pontos.                                                                            |
| **Ignorar todos os `nulls`**                               | `@JsonInclude(Include.NON_NULL)` em uma classe/atributo.                                                        | ✅ `spring.jackson.default-property-inclusion=non_null` aplica em todos os objetos automaticamente.                                        |
| **Formatar datas em um campo**                             | `@JsonFormat(pattern = "dd/MM/yyyy")` para controlar aquele atributo de data.                                   | ✅ Configuração global de formato de data com `spring.jackson.date-format=yyyy-MM-dd` (afeta todas as datas, se você quiser consistência). |
| **Escolher estratégia de nomes (camelCase → snake\_case)** | `@JsonProperty` em cada campo (trabalhoso e repetitivo).                                                        | ✅ `spring.jackson.property-naming-strategy=SNAKE_CASE` resolve pra todo o projeto.                                                        |
| **Controlar ordem dos campos no JSON**                     | `@JsonPropertyOrder({"id","nome","email"})` em uma classe.                                                      | ❌ Global não faz sentido — ordem depende de cada DTO/entidade.                                                                            |
| **Ignorar campos desconhecidos no JSON**                   | `@JsonIgnoreProperties(ignoreUnknown = true)` em uma classe.                                                    | ✅ `spring.jackson.deserialization.fail-on-unknown-properties=false` aplica globalmente.                                                   |
| **Incluir apenas campos não vazios**                       | `@JsonInclude(Include.NON_EMPTY)` em uma classe.                                                                | ✅ `spring.jackson.default-property-inclusion=non_empty` globalmente.                                                                      |

---

* **Use anotações**: quando a configuração é **caso a caso** (nível de campo ou classe).
* **Use configuração global (Boot)**: quando a configuração é **padrão do sistema inteiro** (estilo, datas, inclusão/exclusão de valores).

---


