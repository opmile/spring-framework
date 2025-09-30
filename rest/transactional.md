# Usando `@Transactional`

---

### 1. Estrutura básica de um `@RestController` com POST

Um endpoint POST recebe dados do cliente (geralmente em JSON), transforma isso em um objeto Java e repassa para a camada de serviço:

```java
@RestController
@RequestMapping("/pets")
public class PetController {

    private final PetService petService;

    public PetController(PetService petService) {
        this.petService = petService;
    }

    @PostMapping
    public ResponseEntity<PetResponseDTO> cadastrar(@RequestBody PetRequestDTO dto) {
        PetResponseDTO petCriado = petService.cadastrar(dto);
        return ResponseEntity.ok(petCriado);
    }
}
```

* `@PostMapping` → indica que o método responde a requisições POST.
* `@RequestBody` → mapeia o JSON recebido para o objeto `PetRequestDTO`.
* `ResponseEntity` → te dá controle sobre status HTTP e corpo da resposta.

---

### 2. A camada de serviço com `@Transactional`

Aqui entra o `@Transactional`.
Ele garante que **toda a operação de escrita no banco seja atômica**: ou tudo dá certo, ou nada é persistido.

```java
@Service
public class PetService {

    private final PetRepository petRepository;

    public PetService(PetRepository petRepository) {
        this.petRepository = petRepository;
    }

    @Transactional
    public PetResponseDTO cadastrar(PetRequestDTO dto) {
        Pet pet = new Pet(dto.nome(), dto.idade());
        petRepository.save(pet);
        return new PetResponseDTO(pet);
    }
}
```

* `@Transactional` → começa uma transação no banco antes do método rodar.

  * Se o método terminar sem erro: `commit`.
  * Se lançar uma exception `RuntimeException` ou `Error`: `rollback`.
* Sem `@Transactional`, cada chamada ao `save()` pode ser persistida de forma isolada. Se algo falhar no meio, você pode acabar com o banco em estado inconsistente.

---

### 3. Quando colocar o `@Transactional`?

* **Na camada de serviço, não no controller.**
  O controller deve só orquestrar requisição e resposta; a lógica transacional pertence à regra de negócio.
* Use sempre que o método **faz alterações no banco** (inserts, updates, deletes).
* Para métodos só de leitura, não precisa. Há até a opção `@Transactional(readOnly = true)` para otimizar queries.

---

### 4. Fluxo completo do POST

1. Cliente manda um JSON para `POST /pets`.
2. O controller transforma em DTO e chama `petService.cadastrar(dto)`.
3. O serviço roda dentro de uma transação (`@Transactional`).
4. Se tudo rolar bem: pet é salvo no banco e a transação é commitada.
5. O controller devolve o resultado no corpo da resposta.

---
