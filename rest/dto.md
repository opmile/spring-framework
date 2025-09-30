# Usando Data Transfer Objects (DTOs)

### 1. O que é DTO?

DTO (*Data Transfer Object*) é um objeto que serve só para **transportar dados** entre camadas ou sistemas.
Na prática em uma API REST com Spring, você vai ter:

* **Request DTO** → modela o que o cliente manda no corpo da requisição (`POST`, `PUT`).
* **Response DTO** → modela o que você devolve ao cliente como resposta.

Eles não têm regra de negócio dentro, só dados.

---

### 2. Por que não expor direto as entidades do JPA?

Parece tentador fazer isso:

```java
@PostMapping
public Pet salvar(@RequestBody Pet pet) {
    return petRepository.save(pet);
}
```

Funciona, mas dá problema:

* **Acoplamento**: a estrutura da entidade fica presa ao contrato público da API. Se amanhã você mudar a tabela, quebra o cliente.
* **Exposição indesejada**: você pode acabar revelando campos que não deveriam sair (senha, IDs internos, flags de sistema).
* **Validação bagunçada**: fica difícil validar entrada de dados separadamente.
* **Controle de resposta**: às vezes você quer devolver só parte dos dados (ex: `nome` e `idade`), não o objeto inteiro.

---

### 3. Onde DTOs entram no fluxo?

* **Cliente → API (Request)**:
  Cliente manda um JSON → Spring transforma em `PetRequestDTO` → service usa esse DTO para criar/atualizar entidades.

```java
public record PetRequestDTO(String nome, int idade) {}
```

* **API → Cliente (Response)**:
  Service retorna uma entidade → você monta um `PetResponseDTO` só com os campos relevantes → controller devolve esse DTO.

```java
public record PetResponseDTO(Long id, String nome) {
    public PetResponseDTO(Pet pet) {
        this(pet.getId(), pet.getNome());
    }
}
```

---

### 4. Quando são necessários?

* **Quase sempre em APIs públicas**. Eles isolam seu domínio interno do contrato externo.
* **Sobretudo em escrita/leitura complexa** (muitos campos, validações, exclusão de campos sensíveis).
* Em projetos pequenos/didáticos, você até consegue sobreviver sem DTOs. Mas quando a aplicação cresce, usar DTOs é o que mantém a API sustentável.

---

### 5. Padrão comum em APIs REST profissionais

* `XxxRequest` para entrada (com `@Valid` e anotações do Bean Validation).
* `XxxResponse` para saída (com campos já filtrados).
* Mapeamento feito manualmente ou com libs como **MapStruct**.

---

Exemplo final costurado:

```java
@RestController
@RequestMapping("/pets")
public class PetController {

    private final PetService petService;

    public PetController(PetService petService) {
        this.petService = petService;
    }

    @PostMapping
    public ResponseEntity<PetResponseDTO> cadastrar(@RequestBody @Valid PetRequestDTO dto) {
        PetResponseDTO petCriado = petService.cadastrar(dto);
        return ResponseEntity.status(HttpStatus.CREATED).body(petCriado);
    }
}
```

---

* DTOs não são burocracia à toa; são um **escudo** entre o seu modelo de banco (entidades) e o mundo externo (API).
* Eles viram necessários quando você quer **desacoplar**, **proteger** e **controlar** sua API.


## Usando `@Valid` em DTOs para validações

### 1. Para que serve `@Valid`

`@Valid` é uma **chamada automática ao Bean Validation** (javax.validation / jakarta.validation).
Quando você coloca `@Valid` num DTO, o Spring vai verificar se os campos obedecem às regras de validação antes de passar o objeto pro service. Se algo estiver errado, ele **lança automaticamente uma exception** (`MethodArgumentNotValidException`) que você pode tratar globalmente com `@RestControllerAdvice`.

---

### 2. Como se usa

Você coloca `@Valid` no parâmetro do controller, junto do `@RequestBody`:

```java
@PostMapping
public ResponseEntity<PetResponseDTO> cadastrar(@RequestBody @Valid PetRequestDTO dto) {
    PetResponseDTO petCriado = petService.cadastrar(dto);
    return ResponseEntity.status(HttpStatus.CREATED).body(petCriado);
}
```

---

### 3. Definindo as regras no DTO

O DTO recebe as anotações de validação nos campos:

```java
import jakarta.validation.constraints.*;

public record PetRequestDTO(
        @NotBlank(message = "O nome do pet é obrigatório")
        String nome,

        @Min(value = 0, message = "A idade do pet não pode ser negativa")
        int idade
) {}
```

Principais anotações:

* `@NotNull` → não pode ser nulo
* `@NotBlank` → não pode ser nulo ou vazio (para strings)
* `@Min/@Max` → valor mínimo/máximo
* `@Size` → tamanho de strings ou coleções
* `@Email` → valida e-mail
* `@Pattern` → regex

---

### 4. Fluxo completo com DTO + @Valid

1. Cliente envia JSON para o endpoint POST.
2. Spring converte para o DTO e aplica todas as validações de `@Valid`.
3. Se tudo passar → DTO vai pro service.
4. Se falhar → Spring lança `MethodArgumentNotValidException`.
5. Se você tiver um `@RestControllerAdvice`, pode capturar e devolver **resposta amigável** para o cliente:

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErroResponse> handleValidation(MethodArgumentNotValidException ex) {
    String mensagem = ex.getBindingResult().getFieldErrors()
                        .stream()
                        .map(err -> err.getField() + ": " + err.getDefaultMessage())
                        .collect(Collectors.joining(", "));
    return ResponseEntity.badRequest().body(new ErroResponse(400, mensagem));
}
```

---

### 5. Por que é interessante

* Mantém o controller limpo: você não precisa escrever `if (dto.nome == null || dto.nome.isEmpty())…`.
* Centraliza mensagens de erro.
* Funciona muito bem com DTOs, porque você valida **entrada** sem tocar nas entidades.
* Combina com `@Transactional` no service: se a entrada é inválida, nem chega a iniciar transação.

---
