## MVC e Spring MVC

**MVC** significa **Model-View-Controller** — é um **padrão de arquitetura de software** que organiza o código em três camadas distintas:

| Camada                       | Função                                                    | Exemplo                               |
| ---------------------------- | --------------------------------------------------------- | ------------------------------------- |
| **Model (Modelo)**           | Representa os dados e a lógica de negócio                 | Classes como `Aluno`, `PedidoService` |
| **View (Visão)**             | Interface com o usuário, apresentação                     | HTML, JSP, Thymeleaf                  |
| **Controller (Controlador)** | Recebe requisições, interage com o Model e escolhe a View | `AlunoController`, `PedidoController` |

**Objetivo do MVC:**

* Separar responsabilidades → facilita manutenção e testes.
* Evitar que lógica de negócio fique misturada com interface do usuário.

---

O **Spring MVC** é **uma implementação do padrão MVC dentro do Spring Framework**.

#### Como ele funciona:

1. **DispatcherServlet** → é o “controlador frontal” do Spring.

   * Recebe todas as requisições HTTP.
   * Decide qual Controller vai tratar cada requisição.
2. **Controller** → classes anotadas com `@Controller` ou `@RestController`.

   * Chamam serviços (Model) e retornam dados ou escolhem uma View.
3. **Model** → objetos Java que carregam dados que serão exibidos na View.
4. **View Resolver** → decide **qual template/renderização usar** para mostrar os dados.

```java
@Controller
public class AlunoController {

    @Autowired
    private AlunoService alunoService;

    @GetMapping("/alunos")
    public String listarAlunos(Model model) {
        model.addAttribute("alunos", alunoService.buscarTodos());
        return "listaAlunos"; // nome da View (Thymeleaf, JSP etc.)
    }
}
```

**Fluxo do Spring MVC:**

1. Cliente faz requisição GET `/alunos`.
2. `DispatcherServlet` direciona para `AlunoController.listarAlunos()`.
3. Controller consulta o **Model** (`AlunoService`).
4. Dados são adicionados ao **Model** e a **View** é escolhida.
5. Resultado final é renderizado e enviado ao cliente.

---

* **MVC** = padrão que separa Model, View e Controller.
* **Spring MVC** = implementação concreta do MVC no Spring, com DispatcherServlet, Controllers, Views e integração com todo o ecossistema Spring (DI, AOP, segurança, etc.).

---

## Padrão Font Controller

O **coração do Spring MVC** é o **`DispatcherServlet`**, e ele segue o padrão de design chamado **Front Controller**. Vamos por partes.

---

### 1. O que é o padrão **Front Controller**

Antes do Spring, uma aplicação web Java típica tinha várias **Servlets** mapeadas em um `web.xml`.
Exemplo (bem feio e trabalhoso):

* `/login` → LoginServlet
* `/usuarios` → UsuarioServlet
* `/produtos` → ProdutoServlet

Ou seja, você precisava criar e configurar uma **servlet para cada URL**. Isso escalava muito mal.

O padrão **Front Controller** resolve isso dizendo:

* Teremos **apenas um ponto de entrada** (um servlet central).
* Esse servlet vai **receber todas as requisições** da aplicação.
* Ele decide para qual componente (controller, serviço etc.) a requisição deve ser encaminhada.

Isso facilita:

* centralização de lógica comum (segurança, logging, exceções, encoding, etc.),
* evita repetição de código,
* torna o sistema mais extensível.

---

### 2. Onde entra o **DispatcherServlet**

No Spring MVC, o **DispatcherServlet** é justamente o **Front Controller**.

**Papel dele:**

1. Intercepta todas as requisições HTTP (geralmente mapeado em `/` ou `/*`).
2. Decide **para qual controller** a requisição deve ir.
3. Invoca o método do controller correto.
4. Recebe a resposta e escolhe como transformá-la (view resolver ou JSON, por exemplo).
5. Envia a resposta final para o cliente.

---

### 3. Fluxo interno do **DispatcherServlet**

Imagina que você acessa:

```
GET /usuarios/42
```

O fluxo seria:

1. **Cliente → DispatcherServlet**
   A requisição chega ao **DispatcherServlet** (único ponto de entrada).

2. **Handler Mapping**
   O DispatcherServlet consulta os **HandlerMappings**, que são responsáveis por descobrir **qual método do controller** deve atender a essa URL (`/usuarios/42`).

   * Ele vai olhar as anotações (`@GetMapping`, `@PostMapping`, etc.).

3. **Controller Execution**
   O DispatcherServlet chama o método do controller mapeado.
   Exemplo:

   ```java
   @GetMapping("/usuarios/{id}")
   public Usuario buscar(@PathVariable Long id) { ... }
   ```

4. **Handler Adapter**
   Ele usa um **HandlerAdapter** para invocar o método de forma correta (passando parâmetros, injetando objetos do request etc.).

5. **View Resolver / HttpMessageConverter**

   * Se o método retorna **um objeto** → um **HttpMessageConverter** serializa para JSON/XML.
   * Se o método retorna **uma String com nome de view** → o **ViewResolver** encontra o JSP/Thymeleaf correspondente.

6. **Resposta → Cliente**
   O DispatcherServlet monta a resposta final e devolve ao navegador/cliente HTTP.

---

### 4. Visualizando o fluxo

```
Cliente → DispatcherServlet → HandlerMapping → Controller → HandlerAdapter 
        → ViewResolver/MessageConverter → DispatcherServlet → Cliente
```

---

### 5. Por que isso importa?

* Graças ao `DispatcherServlet`, você não precisa criar várias Servlets.
* Só precisa criar controllers e mapear rotas com anotações.
* Ele é o **core do Spring MVC**, e entender isso ajuda a compreender porque `@RestController` funciona “magicamente”.

---

## Apache Tomcat

O **Apache Tomcat** é um **servidor de aplicações Java EE (Servlet Container)**.

* abre a porta (ex: 8080),
* aceita conexões HTTP,
* entende o protocolo HTTP,
* sabe como lidar com **Servlets** (especificação Java Servlet API).

Sem um servlet container, seu `DispatcherServlet` nem teria como ser chamado, porque não teria quem processasse o request HTTP.

---

### Relação entre Tomcat e DispatcherServlet

* O **DispatcherServlet** é apenas uma classe Java que implementa a especificação `javax.servlet.http.HttpServlet`.
* Para rodar, precisa ser registrado em um **Servlet Container** (como Tomcat, Jetty, Undertow\...).
* No **Spring Boot**, por padrão, o Tomcat vem **embutido** (embedded Tomcat).
  Isso significa que você não precisa fazer deploy manual de um `.war` em um Tomcat externo: o Tomcat já roda dentro da sua aplicação (`.jar`).

Fluxo simplificado:

```
Cliente → Tomcat (porta 8080)
        → DispatcherServlet (Front Controller do Spring MVC)
        → Controllers / Serviços / Views
        → DispatcherServlet
        → Tomcat
        → Cliente
```

---

### 3. O que o Tomcat faz **antes do Spring**

1. Recebe a requisição HTTP crua (ex: `GET /usuarios/42`).
2. Cria os objetos `HttpServletRequest` e `HttpServletResponse`.
3. Chama o `service()` do `DispatcherServlet` (porque ele está registrado como Servlet principal).
4. A partir daqui, o **Spring MVC assume** com seu fluxo interno (HandlerMapping, Controller, ViewResolver, etc.).

---

### 4. Analogia simples

* **Tomcat** → é o "porteiro" do prédio: recebe todo mundo que chega da rua (HTTP).
* **DispatcherServlet** → é a "recepção" que organiza quem vai para cada sala (controller).
* **Controllers** → são os "escritórios" específicos que resolvem os problemas dos visitantes.

---





