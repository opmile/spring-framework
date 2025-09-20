# `Environment`?

O `Environment` é um **bean do próprio Spring** que abstrai o conjunto de **propriedades** disponíveis no contexto da aplicação.
Ele é capaz de ler:

1. **Variáveis de ambiente do sistema operacional**

   * ex.: `PATH`, `JAVA_HOME`, `DB_HOST`.

2. **Propriedades de sistema (System properties do JVM)**

   * ex.: `-Dspring.profiles.active=dev`.

3. **Arquivos de configuração do Spring**

   * ex.: `application.properties` ou `application.yml`.

4. **Perfis ativos (profiles)**

   * ex.: saber se está em `dev`, `test`, `prod`.

#### O que são variáveis de ambiente?

Variáveis de ambiente são valores nomeados, definidos fora do código de uma aplicação, que afetam o comportamento de processos em um sistema.

Elas armazenam informações dinâmicas, como configurações, caminhos para executáveis e chaves de API, e permitem que aplicações sejam configuradas para diferentes ambientes (desenvolvimento, produção) sem a necessidade de alterar o código-fonte.

* Diferenciação de ambiente, caso tratado com o `Environment`: permitem que uma aplicação tenha comportamentos distintos em diferentes ambientes, como desenvolvimento, testes e produção, sem que o código precise ser modificado para cada um. 

* Armazenamento de informações sensíveis: usadas para guardar dados confidenciais, como chaves de API e credenciais de banco de dados, que devem ser mantidas fora do código-fonte.

* Configuração flexível: permitem a alteração de valores de configuração sem a necessidade de recompilar ou redistribuir a aplicação.

* Caminhos do sistema: armazenam informações sobre o sistema operacional, como diretórios e caminhos para executáveis, facilitando a localização de arquivos e programas.

**Funcionamento**:

1. **Definição**: as variáveis de ambiente são criadas e configuradas no sistema operacional do ambiente em que a aplicação está sendo executada.

2. **Acesso**: a aplicação pode ler e acessar o valor dessas variáveis usando a API do sistema operacional ou bibliotecas específicas da linguagem. (`Environment`)

---

## Por que isso é relevante?

### 1. **Centralização da configuração**

Ao invés de você **hardcodar** valores no código (ex.: URL de banco, credenciais, portas), você injeta esses valores dinamicamente a partir do `Environment`.

* Isso torna sua aplicação **configurável sem recompilar**.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class ConfigPrinter {

    @Autowired
    private Environment env;

    public void printConfig() {
        String url = env.getProperty("spring.datasource.url");
        System.out.println("URL do banco: " + url);
    }
}
```

---

### 2. **Diferença entre ambientes (profiles)**

O `Environment` é a base para o Spring decidir qual **profile** está ativo (`dev`, `test`, `prod`).
Isso é **fundamental** porque em projetos sérios você nunca usa as mesmas configs em desenvolvimento e produção.

```java
String[] activeProfiles = env.getActiveProfiles();
for (String profile : activeProfiles) {
    System.out.println("Profile ativo: " + profile);
}
```

---

### 3. **Flexibilidade em runtime**

Você pode fazer **feature toggling** (ativar/desativar recursos com base em config externa).

Isso dá à sua aplicação um comportamento **dinâmico**, sem mexer no código.

```java
boolean featureX = Boolean.parseBoolean(env.getProperty("feature.x.enabled", "false"));
if (featureX) {
    // Executa algo novo
} else {
    // Comportamento antigo
}
```

---

### 4. **Integração com nuvem e containers**

Em ambientes modernos (Docker, Kubernetes, AWS, Azure), quase tudo é configurado por **variáveis de ambiente**.
O `Environment` permite que seu código Spring leia essas variáveis sem precisar se preocupar se vêm do `application.properties` ou do container.

Isso é o que faz o Spring Boot ser tão **cloud-native friendly**.

---

## Comparando com `@Value`

Você pode pensar: *"Mas eu poderia só usar `@Value("${chave}")`"*.

* Sim, para propriedades simples o `@Value` é mais prático.
* Mas o `Environment` te dá acesso **programático**: você pode iterar, verificar se existe, escolher valores default, etc.

```java
String value = env.getProperty("minha.config", "valorDefault");
```

---

O `Environment` é relevante porque:

1. Centraliza e abstrai todas as fontes de configuração.
2. Permite rodar a mesma aplicação em múltiplos ambientes sem recompilar.
3. Dá suporte a perfis (`dev`, `test`, `prod`) e cloud-native configs.
4. Te dá flexibilidade em runtime para ler configs dinamicamente.

Em resumo: ele é o **"ponto único de verdade"** para configs da sua aplicação Spring.

---

## O que significa "profiles ativos"?

Perfis (`profiles`) no Spring são uma forma de dizer:

> “Esse pedaço de configuração/bean só vale em tal ambiente (dev, test, prod, etc.)”.

Assim você consegue rodar a **mesma aplicação** com **configs diferentes** sem alterar código, só mudando o profile ativo.

Exemplo clássico:

* Em **desenvolvimento** (`dev`) → Banco de dados em memória H2.
* Em **produção** (`prod`) → Banco PostgreSQL real.

---

## Como configurar profiles no Spring Boot

### 1. No `application.properties`

Por padrão, o Spring Boot lê `application.properties` ou `application.yml`.
Você pode criar **arquivos específicos para cada profile**:

* `application-dev.properties`
* `application-test.properties`
* `application-prod.properties`

O arquivo `application.properties` (sem sufixo) é o **default** e vale para todos.

```properties
# application-dev.properties
spring.datasource.url=jdbc:h2:mem:meuBanco
spring.jpa.hibernate.ddl-auto=create-drop
```

```properties
# application-prod.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/meuBanco
spring.datasource.username=usuario
spring.datasource.password=senha
```

---

### 2. Definindo qual profile está ativo

Você pode ativar o profile de várias formas:

#### a) Dentro do `application.properties`

```properties
spring.profiles.active=dev
```

#### b) Via linha de comando

```bash
java -jar meuApp.jar --spring.profiles.active=prod
```

#### c) Via variável de ambiente (muito comum em Docker/Kubernetes)

```bash
export SPRING_PROFILES_ACTIVE=test
```

---

### 3. Ativando beans diferentes por profile

Você também pode marcar classes ou métodos com `@Profile` para que só carreguem em certo ambiente.

```java
import org.springframework.context.annotation.Profile;
import org.springframework.stereotype.Repository;

@Repository
@Profile("dev")
public class FakeRepository implements MyRepository {
    // Fake data só em DEV
}
```

```java
@Repository
@Profile("prod")
public class RealRepository implements MyRepository {
    // Conecta no banco real em PROD
}
```

* Se `spring.profiles.active=dev`, o `FakeRepository` será usado.

* Se `spring.profiles.active=prod`, o `RealRepository` será usado.

---

* **Profiles ativos** servem para mudar comportamento/config da aplicação conforme o ambiente.
* Você configura criando `application-{profile}.properties` e ativando com `spring.profiles.active`.
* Também pode usar `@Profile` em beans para alternar implementações.

---



