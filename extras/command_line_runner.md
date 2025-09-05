# O que é o `CommandLineRunner`?

É **uma interface funcional** do Spring Boot que permite rodar código **logo após a inicialização da aplicação**, quando o contexto do Spring já está carregado e todos os beans já estão disponíveis.

A assinatura dela é bem simples:

```java
@FunctionalInterface
public interface CommandLineRunner {
    void run(String... args) throws Exception;
}
```

Ou seja, você implementa o método `run` e ali coloca o que quer que aconteça **assim que a aplicação sobe**.

---

## Como usar na prática?

Você pode implementar a interface diretamente numa classe marcada com `@Component`, ou criar um **bean** que retorna um `CommandLineRunner`.

Exemplo com `@Component`:

```java
@Component
public class StartupRunner implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        System.out.println("Aplicação inicializada com sucesso!");
        // aqui você pode inicializar dados, chamar serviços, etc.
    }
}
```

* Essa implementação também aceita que sera declarada dentro da classe de aplicação (`@SpringBootApplication`)


Ou como `@Bean` no `@SpringBootApplication`:

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Bean
    CommandLineRunner initDatabase(UserRepository userRepository) {
        return args -> {
            // Exemplo: populando o banco com dados iniciais
            userRepository.save(new User("Milena", "milena@email.com"));
            System.out.println("Usuário inicial criado!");
        };
    }
}
```

---

## Qual a relevância do `CommandLineRunner`?

Ele é útil em cenários como:

1. **Seed de dados**
   Popular o banco com registros iniciais (útil em dev/teste).

2. **Tarefas automáticas na inicialização**
   Rodar scripts, limpar caches, iniciar jobs.

3. **Testar rapidamente lógica**
   Executar código sem precisar criar endpoints ou front-end para chamar.

4. **Prototipagem rápida**
   Se você só quer ver se um repositório, serviço ou configuração funciona.

O ideal é usar para inicializações pontuais, sempre mantendo a clareza de que é código de *startup*.

---

## Diferença para `ApplicationRunner`

Existe também o `ApplicationRunner`, que é quase igual, mas recebe um objeto `ApplicationArguments` em vez de `String... args`, oferecendo uma API mais rica para lidar com argumentos da linha de comando.

Se você só precisa dos `args` crus, use `CommandLineRunner`. Se precisa parsear e trabalhar melhor com eles, use `ApplicationRunner`.

---


