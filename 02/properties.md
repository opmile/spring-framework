# Properties: `application.properties`

O arquivo **`application.properties`** (ou `application.yml`) é o **ponto central de configuração da sua aplicação Spring Boot**. Ele substitui um monte de XML ou Java Config manual que você teria que escrever no Spring tradicional.

---

## Função principal

Ele serve para **configurar o comportamento do container Spring e das bibliotecas auto-configuradas pelo Boot**, sem precisar mexer no código Java.

---

## Exemplos de uso comuns

1. **Configuração de servidor**

```properties
server.port=8081   # Muda a porta do Tomcat embutido
server.servlet.context-path=/api
```

2. **Configuração de banco de dados**

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/minha_base
spring.datasource.username=root
spring.datasource.password=senha
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

3. **Configuração de logs**

```properties
logging.level.org.springframework=DEBUG
logging.file.name=app.log
```

4. **Configuração de beans customizados**
   Você pode criar suas próprias chaves:

```properties
app.mensagem.boasvindas=Olá, seja bem-vindo!
```

E no Java:

```java
@Value("${app.mensagem.boasvindas}")
private String mensagem;
```

---

## application.properties vs application.yml

* Os dois fazem a **mesma coisa**.

* `application.yml` é mais legível para estruturas hierárquicas (tipo JSON).

  Exemplo YAML equivalente ao banco de dados acima:

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/minha_base
    username: root
    password: senha
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

---

## Ambientes diferentes

Você pode ter vários arquivos, como:

* `application-dev.properties`
* `application-prod.properties`

E ativar via:

```properties
spring.profiles.active=dev
```

---

O `application.properties` é o **coração da configuração externa** da sua aplicação Spring Boot. Ele evita hardcode no código e permite mudar comportamentos (porta, banco, segurança, etc.) só alterando arquivo de configuração — sem recompilar nada.

---

## Múltiplos arquivos

Existem situações comuns onde você vai ver mais de um arquivo `.properties` ou `.yml` sendo usado em projetos Spring Boot. 

---

### 1. **Separação por responsabilidade (modularização)**

Em projetos grandes, é comum quebrar a configuração em vários arquivos para **organizar melhor**.
Exemplo:

* `application-db.yml` → configs de banco.
* `application-messaging.yml` → configs de Kafka/RabbitMQ.
* `application-security.yml` → configs de segurança.

Depois, no **application.yml principal**, você importa os outros:

```yaml
spring:
  config:
    import:
      - classpath:application-db.yml
      - classpath:application-messaging.yml
      - classpath:application-security.yml
```

---

### 2. **Configuração sensível ou externa**

Muitas vezes, configs **sensíveis** (senha de banco, tokens de API) ficam separadas em outro arquivo, para facilitar:

* Não versionar no Git.
* Carregar apenas em ambiente seguro.

Exemplo:

* `application.yml` → configs públicas/gerais.
* `secrets.yml` → credenciais (gerenciado no `.gitignore`).

Importando:

```yaml
spring:
  config:
    import: optional:file:./secrets.yml
```

---

### 3. **Overrides por ambiente específico**

Além de profiles (`application-dev.yml`), você pode ter overrides adicionais.
Exemplo:

* `application.yml` → configs default.
* `application-docker.yml` → ajustes quando roda em container.
* `application-k8s.yml` → configs específicas de Kubernetes.

Isso dá flexibilidade sem ficar “amarrado” só a `dev/test/prod`.

---

### 4. **Configuração multilíngue/multirregião**

Em sistemas globais, você pode ter arquivos diferentes para **locale/região**.
Exemplo:

* `application-br.yml`
* `application-us.yml`
* `application-eu.yml`

E ativar conforme variável de ambiente, parecendo profiles, mas com foco em **regionalização**.

---

### 5. **Externalização em diferentes diretórios**

O Spring Boot permite que configs sejam carregadas de fora do jar.
Isso é muito comum em **produção**:

* Um `application.properties` dentro do jar (default).
* Outro arquivo em `/etc/myapp/application.properties` sobrepõe os defaults.

Isso permite **rodar o mesmo binário em vários ambientes**, só mudando os arquivos externos.

---

### 6. **Feature toggles / configs experimentais**

Você pode usar arquivos separados para **ativar/desativar recursos** sem mexer no principal.
Exemplo:

* `application.yml` → configs base.
* `feature-flags.yml` → lista de recursos habilitados/desabilitados.

---

### 7. **Testes automatizados**

Em **testes de integração**, é comum ter um arquivo `.properties` separado:

* `application-test.yml` (profile `test`)
* ou até múltiplos arquivos com configs específicas de mocks, banco em memória, etc.

---

Múltiplos `.properties/.yml` aparecem em situações como:

1. Modularizar por responsabilidade (db, messaging, security).
2. Isolar configs sensíveis (`secrets.yml`).
3. Ajustar overrides para ambientes especiais (docker, k8s).
4. Regionalização (`application-br.yml`, `application-us.yml`).
5. Externalizar configs fora do jar (produção).
6. Feature toggles.
7. Testes de integração.

---

## Importando Arquivos de Config

A primeira abordagem segue usando de forma moderna com `@ConfigurationProperties` e criando uma classe com essa config, e a segunda com `@PropertySource` e consumindo os valores com `@Value`

---

### `@ConfigurationProperties`

### 1. Criar um arquivo de configuração separado

Suponha que você queira separar configs de banco em `db.properties` (ou `db.yml`).

`src/main/resources/db.properties`

```properties
db.url=jdbc:postgresql://localhost:5432/meuBanco
db.username=usuario
db.password=senha
```

---

### 2. Dizer ao Spring para importar esse arquivo

No seu `application.properties` principal (ou `application.yml`), você importa o outro:

```properties
spring.config.import=classpath:db.properties
```

> Obs: no Boot moderno (2.4+), essa é a forma oficial de incluir configs adicionais.

---

### 3. Criar uma classe de configuração com `@ConfigurationProperties`

Agora criamos um **bean de configuração tipado** que representa essas propriedades:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "db")
public class DbProperties {

    private String url;
    private String username;
    private String password;

    // getters e setters obrigatórios
    public String getUrl() { return url; }
    public void setUrl(String url) { this.url = url; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
}
```

---

### 4. Injetar em qualquer lugar da sua aplicação

Agora qualquer classe pode receber essa config como um bean normal:

```java
import org.springframework.stereotype.Service;

@Service
public class DatabaseService {

    private final DbProperties dbProperties;

    public DatabaseService(DbProperties dbProperties) {
        this.dbProperties = dbProperties;
    }

    public void connect() {
        System.out.println("Conectando em: " + dbProperties.getUrl());
        System.out.println("Usuário: " + dbProperties.getUsername());
    }
}
```

---

* Você **organizou** as configs em outro arquivo (`db.properties`).
* Importou no `application.properties` principal.
* Criou um bean fortemente tipado (`DbProperties`).
* Injetou onde precisava.

---

**Observação:**
Se preferir YAML, funciona igual:

`db.yml`

```yaml
db:
  url: jdbc:postgresql://localhost:5432/meuBanco
  username: usuario
  password: senha
```

E no `application.yml` principal:

```yaml
spring:
  config:
    import: classpath:db.yml
```

---

### `@PropertySource`

### 1. Criar o arquivo separado

📂 `src/main/resources/db.properties`

```properties
db.url=jdbc:postgresql://localhost:5432/meuBanco
db.username=usuario
db.password=senha
```

---

### 2. Criar uma classe de configuração que importa esse arquivo

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

@Configuration
@PropertySource("classpath:db.properties")
public class DbConfig {
    // essa classe serve apenas para carregar o arquivo no Environment
}
```

---

### 3. Consumir as propriedades com `@Value`

Agora qualquer bean pode injetar os valores usando `@Value("${chave}")`:

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

@Service
public class DatabaseService {

    @Value("${db.url}")
    private String url;

    @Value("${db.username}")
    private String username;

    @Value("${db.password}")
    private String password;

    public void connect() {
        System.out.println("Conectando em: " + url);
        System.out.println("Usuário: " + username);
    }
}
```

---

### Diferença entre `@ConfigurationProperties` e `@PropertySource`

* **`@PropertySource` + `@Value`**

  * Mais simples, direto.
  * Bom para injetar **poucas propriedades isoladas**.
  * Mais verboso (um campo = uma injeção).

* **`@ConfigurationProperties`**

  * Permite **agrupar várias propriedades em um objeto Java**.
  * Mais organizado para configs grandes.
  * Integra melhor com Spring Boot moderno (profiles, validação de propriedades, etc).

---

* Se você precisa apenas de alguns valores soltos → `@PropertySource` com `@Value`.
* Se precisa agrupar configurações inteiras (datasource, jwt, storage, etc.) → `@ConfigurationProperties`.

---






