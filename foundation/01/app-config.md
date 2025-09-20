# Tipos de Configuração

---

## 1. Configuração por *Component Scanning*

É o mais comum hoje. Você coloca `@Component`, `@Service`, `@Repository` ou `@Controller` nas classes, e o Spring as detecta automaticamente.

```java
@Component
public class RepositorioJpa implements RepositorioCliente {
    // implementação
}

@Service
public class ServicoCliente {
    private final RepositorioCliente repo;

    @Autowired
    public ServicoCliente(RepositorioCliente repo) {
        this.repo = repo;
    }
}
```

E no `@SpringBootApplication` ou `@ComponentScan`, o Spring varre o pacote e registra os beans.
---

### O que é o `CoponentScan`

`@ComponentScan` é a anotação que diz ao Spring **em quais pacotes ele deve procurar classes anotadas com `@Component`, `@Service`, `@Repository`, `@Controller`** (ou qualquer estereótipo que o Spring reconheça como bean).

Sem ele, o Spring **não tem como saber onde estão as classes que devem virar beans**.

---

### Como funciona

Quando você inicia a aplicação, o Spring cria o **ApplicationContext**.
Se você marcou sua classe principal com `@SpringBootApplication`, dentro dela já existe implicitamente:

```java
@SpringBootApplication
@ComponentScan
public class MinhaAplicacao {
    public static void main(String[] args) {
        SpringApplication.run(MinhaAplicacao.class, args);
    }
}
```

Aqui, por padrão, o `@ComponentScan` vai **procurar no mesmo pacote da classe principal e em todos os subpacotes**.

Exemplo:

* Classe principal em `com.exemplo.app`
* Beans em `com.exemplo.app.service`, `com.exemplo.app.repository`

O Spring encontra tudo de boa.

---

### Escaneando pacotes específicos

Se os beans estão em outro pacote, você precisa indicar isso no `@ComponentScan`:

```java
@SpringBootApplication
@ComponentScan(basePackages = {"com.exemplo.servicos", "com.exemplo.repositorios"})
public class MinhaAplicacao { ... }
```

Assim o Spring varre esses pacotes e registra os beans encontrados.

---

### Diferença para `@Configuration + @Bean`

* `@ComponentScan`: **descoberta automática** (você deixa o Spring encontrar os beans baseado nas anotações).
* `@Configuration + @Bean`: **declaração explícita** (você mesmo registra os beans manualmente).

---

* `@ComponentScan` = scanner que varre pacotes e cria beans automaticamente.
* Funciona junto com `@Component`, `@Service`, `@Repository`, `@Controller`.
* Por padrão, escaneia o pacote da classe onde está aplicado.
* Você pode mudar os pacotes com `basePackages`.

---

## 2. Configuração explícita com `@Configuration` + `@Bean`

Aqui entra a **classe `AppConfig`**.
Você não usa `@Component`, mas **declara manualmente cada bean**:

```java
@Configuration
public class AppConfig {

    @Bean
    public RepositorioCliente repositorioJpa() {
        return new RepositorioJpa();
    }

    @Bean
    public RepositorioCliente repositorioMemoria() {
        return new RepositorioMemoria();
    }

    @Bean
    public ServicoCliente servicoCliente(@Qualifier("repositorioJpa") RepositorioCliente repo) {
        return new ServicoCliente(repo);
    }
}
```

Aqui o Spring instancia o que você retornou no método `@Bean` e registra no contexto.
Você tem **mais controle** e pode resolver conflitos de dependências diretamente dentro da configuração.

---

## 3. Configuração via XML (*legado*)

Antes do Spring moderno, os beans eram definidos em XML:

```xml
<beans>
    <bean id="repositorioJpa" class="com.exemplo.RepositorioJpa"/>
    <bean id="servicoCliente" class="com.exemplo.ServicoCliente">
        <constructor-arg ref="repositorioJpa"/>
    </bean>
</beans>
```

Hoje quase não se usa mais, mas ainda é suportado.

---

## Diferença prática do `AppConfig` (Java Config)

* Com `@Component`: o Spring descobre os beans sozinho.
* Com `@Configuration`: você mesmo diz **como** e **qual** implementação será usada.
* Com XML: bem verboso, pouco usado.

O `AppConfig` é muito útil quando:

* Você tem mais de uma implementação da mesma interface.
* Precisa instanciar um bean com lógica de criação (fábricas, construtores complexos).
* Quer centralizar a configuração dos beans para maior clareza.

---

### O que é o `@Autowired`?

É uma **anotação** que instrui o container do Spring a **resolver e injetar uma dependência automaticamente**, sem você precisar escrever `new MinhaClasse()`.

Na prática, o Spring olha para o tipo do campo, parâmetro do construtor ou setter anotado e tenta **procurar um bean compatível no ApplicationContext**.
Ele é parte do mecanismo de **IoC (Inversion of Control)** que já falamos.

---

### Onde usar

1. **Em construtores (recomendado)**

   * A forma mais moderna e recomendada.
   * Garante imutabilidade (você pode até marcar os campos como `final`).
   * Se tiver só um construtor, o `@Autowired` nem é obrigatório.

```java
@Component
class ServicoCliente {
    private final RepositorioCliente repositorio;

    @Autowired  // opcional aqui
    public ServicoCliente(RepositorioCliente repositorio) {
        
    }
}
```

---

2. **Em setters**

   * Útil se a dependência for opcional ou se você quiser poder alterar depois.
   * Mais usado em configurações antigas (especialmente com XML).

```java
@Component
class ServicoCliente {
    private RepositorioCliente repositorio;

    @Autowired
    public void setRepositorio(RepositorioCliente repositorio) {
        this.repositorio = repositorio;
    }
}
```

---

3. **Em campos**

   * É o mais direto, mas menos elegante.
   * Não favorece testes unitários porque você não consegue passar mocks sem mexer em reflexão ou `@MockBean`.

```java
@Component
class ServicoCliente {
    @Autowired
    private RepositorioCliente repositorio;
}
```

---

### Como o Spring escolhe qual bean injetar?

1. **Por tipo**: se houver só um bean do tipo pedido → ele injeta direto.
2. **Conflito (mais de um bean do mesmo tipo)** → você pode usar:

   * `@Primary` → define a implementação padrão.
   * `@Qualifier("nomeDoBean")` → escolhe explicitamente.

```java
@Component
@Qualifier("repositorioMemoria")
class RepositorioMemoria implements RepositorioCliente {}

@Component
@Qualifier("repositorioJpa")
class RepositorioJpa implements RepositorioCliente {}

@Component
class ServicoCliente {
    @Autowired
    @Qualifier("repositorioJpa")
    private RepositorioCliente repositorio;
}
```

---

### Controle de obrigatoriedade

Por padrão, `@Autowired` **exige** que o bean exista.
Se a dependência for opcional:

```java
@Autowired(required = false)
private Notificador notificador;
```

Ou, mais moderno:

```java
@Autowired
private Optional<Notificador> notificador;
```

---

* `@Autowired` → diz ao Spring: “resolve isso pra mim no container”.
* Pode ser usado em **construtores, setters ou campos**.
* Resolve por **tipo**, podendo usar `@Qualifier` para conflitos.
* Favorece **IoC + DIP**, tirando a responsabilidade de instanciar dependências.
* Hoje em dia, a **injeção por construtor** é considerada **boa prática**.

---

### O cenário do conflito

Imagine que temos uma interface:

```java
public interface RepositorioCliente {
    void salvar(String cliente);
}
```

E **duas implementações**:

```java
@Component
public class RepositorioMemoria implements RepositorioCliente {
    @Override
    public void salvar(String cliente) {
        System.out.println("Salvando em memória: " + cliente);
    }
}

@Component
public class RepositorioJpa implements RepositorioCliente {
    @Override
    public void salvar(String cliente) {
        System.out.println("Salvando no banco JPA: " + cliente);
    }
}
```

---

### O problema

Agora criamos um serviço que depende da interface:

```java
@Component
public class ServicoCliente {
    private final RepositorioCliente repositorio;

    @Autowired
    public ServicoCliente(RepositorioCliente repositorio) {
        this.repositorio = repositorio;
    }
}
```

O Spring vai falhar na inicialização com exceção:
`NoUniqueBeanDefinitionException: expected single matching bean but found 2: repositorioMemoria, repositorioJpa`

Isso porque ele não sabe **qual dos dois beans** usar.

---

### Soluções para o conflito

#### 1. **Marcar um como `@Primary`**

Se você tem uma implementação “padrão”, marque-a com `@Primary`:

```java
@Component
@Primary
public class RepositorioJpa implements RepositorioCliente {
    @Override
    public void salvar(String cliente) {
        System.out.println("Salvando no banco JPA: " + cliente);
    }
}
```

Assim, se ninguém disser nada, o Spring sempre vai escolher o `RepositorioJpa`.

---

#### 2. **Usar `@Qualifier` para escolher explicitamente**

Quando você precisa de uma implementação específica em determinado ponto:

```java
@Component
public class ServicoCliente {
    private final RepositorioCliente repositorio;

    @Autowired
    public ServicoCliente(@Qualifier("repositorioMemoria") RepositorioCliente repositorio) {
        this.repositorio = repositorio;
    }
}
```

O nome passado em `@Qualifier("...")` corresponde ao **nome do bean**, que por padrão é o **nome da classe com a primeira letra minúscula** (`repositorioMemoria`, `repositorioJpa`).

---

#### 3. **Declarar Beans no `@Configuration` (AppConfig)**

Ao invés de deixar o Spring detectar com `@Component`, você pode definir manualmente num `@Configuration`:

```java
@Configuration
public class AppConfig {

    @Bean
    public RepositorioCliente repositorioMemoria() {
        return new RepositorioMemoria();
    }

    @Bean
    public RepositorioCliente repositorioJpa() {
        return new RepositorioJpa();
    }

    @Bean
    public ServicoCliente servicoCliente(@Qualifier("repositorioJpa") RepositorioCliente repositorio) {
        return new ServicoCliente(repositorio);
    }
}
```

Aqui, o Spring também ficaria confuso se não usarmos `@Qualifier` ou `@Primary`, porque ambos os beans expostos (`repositorioMemoria` e `repositorioJpa`) são do mesmo tipo.

---

* Quando temos **uma só implementação**, o Spring injeta sem problemas.
* Quando temos **mais de uma**, ocorre **conflito** → resolvemos com:

  * `@Primary` → define a implementação padrão.
  * `@Qualifier` → escolhe explicitamente qual bean usar.
  * `@Configuration` (`AppConfig`) → declarando os beans com `@Bean` e também controlando manualmente a escolha.

---



