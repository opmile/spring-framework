# O que é CORS

CORS significa **Cross-Origin Resource Sharing**.
Traduzindo: é a forma como os navegadores controlam pedidos feitos de um **site A** para recursos que estão em um **site B**.

* **Origem** é definida pelo **protocolo + domínio + porta**.
  Exemplo:

  * `http://localhost:3000` (origem A — seu front-end React/Angular/etc.)
  * `http://localhost:8080` (origem B — seu back-end Spring MVC).

Mesmo que ambos estejam no *localhost*, o navegador enxerga **portas diferentes = origens diferentes**.

Então, quando o front (`3000`) faz uma requisição para o back (`8080`), o navegador entra no meio e pensa:

> "Opa, isso é *cross-origin*. Só libero se o servidor disser que é permitido."

Se o servidor **não responder com os cabeçalhos corretos de CORS**, o navegador barra. Esse é o famoso erro de **CORS policy**.

---

### Sobre o `localhost`

O termo "dominio localhost" não se refere a um domínio na internet, mas sim a um endereço de loopback reservado para o próprio computador, que aponta para o endereço IP 127.0.0.1 (para IPv4) ou ::1 (para IPv6).

Desenvolvedores utilizam o localhost para testar e depurar aplicativos e websites localmente, sem a necessidade de conexão com a internet ou de um servidor público. 

O `localhost` é um nome de domínio de nível superior (TLD) que, de acordo com as normas de rede, sempre aponta para o seu próprio computador. 

Ele cria um ambiente de rede interno onde um programa pode se comunicar com ele mesmo, servindo como um servidor simulado no seu próprio sistema operacional. 

Quando um navegador ou aplicativo tenta acessar `localhost`, o sistema o interpreta como uma solicitação para o seu próprio computador. Em vez de enviar a solicitação para a internet, o sistema a roteia internamente, fazendo com que o aplicativo e o servidor "conversem" dentro do mesmo dispositivo. 

---

### Quando acontece

* Geralmente quando você está desenvolvendo **front separado do back** (React, Angular, Vue no `3000`, Spring no `8080`).
* Pode rolar também em produção, se a API e o site rodam em domínios diferentes, tipo:

  * Front: `https://meusite.com`
  * Back: `https://api.meusite.com`

Em ambos os casos, o navegador não deixa a requisição seguir sem autorização explícita.

---

### Como o navegador decide

Quando você faz uma requisição **"sensível"** (POST com JSON, PUT, DELETE, headers customizados, etc.), o navegador primeiro dispara um **preflight request** (uma requisição `OPTIONS`).
Esse `OPTIONS` pergunta para o servidor:

> "Ei, você aceita pedidos vindos de `http://localhost:3000`?"

Se o servidor não responder com os cabeçalhos certos (`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, etc.), o navegador bloqueia a requisição.

---

### Como resolver no Spring

No Spring MVC/Spring Boot você precisa **ensinar o back-end a responder que aceita o front**. Existem várias formas:

#### 1. Anotação no controller

```java
@CrossOrigin(origins = "http://localhost:3000")
@RestController
@RequestMapping("/api")
public class MeuController {

    @GetMapping("/dados")
    public String getDados() {
        return "OK";
    }
}
```

* Só libera CORS para esse controller.
* Dá pra liberar para múltiplas origens passando array.
* Se quiser liberar geral (não recomendado em produção):

  ```java
  @CrossOrigin(origins = "*")
  ```

---

#### 2. Configuração global

Se não quiser ficar colocando `@CrossOrigin` em todo lugar, pode criar uma config:

* Aqui é interessante a criação de mais um pacote, caso usando o modelo Package-by-Layer, denominado `config`

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**") // todos os endpoints
                        .allowedOrigins("http://localhost:3000") // libera só o front local
                        .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS") // métodos permitidos
                        .allowedHeaders("*"); // cabeçalhos permitidos
            }
        };
    }
}
```

Isso resolve de forma mais centralizada ao retornar um `WebMvcConfigurer` via `@Bean`, mas você também pode implementar diretamente a interface `WebMvcConfigurer`.

```java
package br.com.alura.screenmatch.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class CorsConfiguration implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://127.0.0.1:5501") // liberando live server
                .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS", "HEAD", "TRACE", "CONNECT");
    }
}
```

---

#### 3. Usando propriedades do Spring Security (se tiver security no projeto)

Se estiver usando `Spring Security`, ele também precisa permitir CORS.
Exemplo com config moderna:

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .cors().and() // habilita CORS
            .csrf().disable() // em dev, desabilita CSRF pra facilitar
            .authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
        return http.build();
    }
}
```

E você ainda usa a `CorsConfig` que mostrei acima.

---

* O erro de CORS não é um bug do seu código, é **o navegador protegendo o usuário**.
* Sempre que front e back estão em origens diferentes, você precisa configurar o back para **responder que aceita essa origem**.
* No Spring MVC:

  * Use `@CrossOrigin` em controllers específicos.
  * Ou crie uma configuração global com `WebMvcConfigurer`.
  * Se tiver Spring Security, habilite `.cors()`.

O CORS é configurado no backend para informar ao navegador quais origens estão autorizadas a acessar os recursos do servidor. Por exemplo, é comum configurar o backend para permitir acesso apenas do domínio do frontend durante o desenvolvimento.

---


