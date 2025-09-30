## 1. **JUnit – a base de testes em Java**

O **JUnit** é o framework padrão de testes unitários em Java. Ele dá:

* **Estrutura**: fornece anotações (`@Test`, `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`) para organizar a execução dos testes.
* **Assertions**: métodos para verificar o comportamento esperado (`assertEquals`, `assertTrue`, `assertThrows` etc.).
* **Ciclo de vida**: você consegue isolar cada teste (resetar dados, abrir/fechar conexões, etc.) para que um teste não “contamine” o outro.
* **Runner**: é quem executa seus testes e mostra o resultado (passou/falhou). IDEs como IntelliJ ou Eclipse já trazem integração nativa.

Exemplo básico:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class CalculadoraTest {

    @Test
    void deveSomarDoisNumeros() {
        Calculadora calc = new Calculadora();
        int resultado = calc.somar(2, 3);
        assertEquals(5, resultado);
    }
}
```

Aqui:

* `@Test` marca o método como teste.
* `assertEquals` compara o esperado (5) com o resultado real.

---

## 2. **Mockito – simulando dependências**

O **Mockito** entra quando a classe que você está testando **depende de outra coisa** (um banco, uma API externa, um serviço caro ou lento). Em vez de acionar o banco de verdade, você cria um *mock* (um objeto “falso” que simula o comportamento da dependência).

Principais recursos:

* `@Mock`: cria o objeto simulado.
* `when(...).thenReturn(...)`: define o que o mock deve responder quando chamado.
* `verify(...)`: checa se o mock foi chamado de determinada forma.
* `@InjectMocks`: injeta os mocks automaticamente dentro da classe que você está testando.

Exemplo:

```java
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

class UsuarioServiceTest {

    @Mock
    private UsuarioRepository usuarioRepository; // simulação do banco

    @InjectMocks
    private UsuarioService usuarioService; // classe que depende do repo

    public UsuarioServiceTest() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void deveSalvarUsuario() {
        Usuario user = new Usuario("Milena");

        when(usuarioRepository.save(user)).thenReturn(user); // define comportamento simulado

        Usuario salvo = usuarioService.salvar(user);

        assertEquals("Milena", salvo.getNome());
        verify(usuarioRepository).save(user); // confirma se o método save foi chamado
    }
}
```

Aqui:

* Não acessamos um banco real.
* O `UsuarioRepository` foi simulado.
* O foco está em **testar a lógica do `UsuarioService`**, não o banco.

---

## 3. **JUnit + Mockito juntos**

Na prática, quase sempre você combina os dois:

* JUnit organiza, executa e valida.
* Mockito cria as dependências falsas.

Com isso você consegue testar métodos isolados sem precisar levantar servidor, chamar banco, nem criar contexto web ainda.

---

## 4. **E quando chega na aplicação web (Spring Boot API)?**

Quando integrar com uma API:

* Você vai usar **Spring Boot Test** junto, que estende o JUnit para rodar testes em contexto Spring.
* Aí entram coisas como `@SpringBootTest`, `@WebMvcTest`, `MockMvc` e até `@DataJpaTest`.
* Nessa etapa, alguns testes deixam de ser puramente unitários e passam a ser **testes de integração** (ex.: testar se um endpoint `/usuarios` realmente salva no banco).

---

* **JUnit** = organiza, executa e valida testes.
* **Mockito** = cria dependências simuladas para focar na lógica da classe.
* **Integração com API** = envolve Spring Boot Test, que é tipo “JUnit+Mockito turbinados para o ecossistema Spring”.

