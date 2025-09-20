# Variáveis de Ambiente

Variável de ambiente é, no fundo, só uma chave–valor que o sistema operacional guarda na memória e disponibiliza para processos. Você pode pensar como um “post-it invisível” que acompanha sua aplicação na hora que ela roda.

### O que elas são

* São valores definidos fora do código (no SO, no Docker, no Kubernetes, em arquivos `.env`, etc.).
* Normalmente guardam configurações que **não devem estar hardcoded** no código-fonte: credenciais, URLs de banco, portas, chaves de API.

Exemplo no Linux:

```bash
export DB_USER=milena
export DB_PASS=segredo
```

Depois, a aplicação pode ler isso sem precisar que o dado esteja gravado no código.

---

### Por que isso importa em aplicações web com Spring Boot

1. **Configuração externa**

   * O Spring Boot segue a filosofia *“externalized configuration”*. Isso significa que você não precisa recompilar ou alterar código para mudar comportamento: só troca a variável de ambiente.
   * Exemplo: `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`.

2. **Segurança**

   * Nunca é boa ideia subir credenciais no Git. Variáveis de ambiente permitem esconder senhas, tokens e segredos fora do repositório.

3. **Portabilidade**

   * Você pode rodar a mesma aplicação em dev, staging e produção mudando apenas as variáveis de ambiente, sem tocar no JAR.

4. **Integração com containers/orquestradores**

   * Docker e Kubernetes praticamente vivem disso. Configurações são passadas como env vars, o que torna a aplicação facilmente configurável sem alterar imagem.

---

### Como o Spring Boot lida com isso

* Ele possui o **Environment Abstraction**, que unifica diversas fontes de configuração (arquivos `application.properties`, YAML, argumentos de linha de comando, variáveis de ambiente, etc.).
* A prioridade é hierárquica. Se você definir uma variável de ambiente `SPRING_PROFILES_ACTIVE=prod`, isso sobrepõe o que estiver no `application.properties`.

---

## O `PATH`

O **PATH** é uma variável de ambiente bem especial. Ele não é inventado pelo Spring Boot, nem pelo Java — é algo do próprio sistema operacional.

### O que ele guarda

É basicamente uma lista de diretórios onde o sistema operacional vai procurar por programas executáveis quando você digita um comando no terminal.

Por exemplo:

* Quando você digita `java -version`, o sistema não tem um "GPS" para achar o binário do Java.
* Ele varre os diretórios listados no `PATH` e executa o primeiro `java` que encontrar.

No Linux/macOS, o `PATH` costuma ser algo assim:

```bash
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

No Windows, é parecido, só que com `;` como separador:

```
C:\Program Files\Java\jdk-17\bin;C:\Windows\System32;...
```

---

### Por que isso importa no contexto do Java / Spring Boot

* **Executar comandos globalmente**: quando você instala o JDK, precisa colocar o diretório `bin` do Java no `PATH` para poder chamar `java` ou `javac` de qualquer pasta.
* **Executar o Maven ou Gradle**: se você usa ferramentas de build, o `PATH` garante que o terminal saiba onde achar `mvn` ou `gradle`.
* **Rodar sua aplicação**: quando você empacota seu projeto Spring Boot em um `jar`, você chama `java -jar app.jar`. Esse `java` só é encontrado porque o binário do JDK está no `PATH`.

---

Ou seja: o **PATH** não é uma variável de ambiente para configurar sua aplicação, mas sim uma variável de ambiente que garante que o próprio sistema consiga achar os programas necessários (incluindo o Java que roda sua aplicação).

---

## Criando variáveis de ambiente

No macOS dá pra fazer isso de algumas formas, depende se você quer a variável só para a sessão atual do terminal ou sempre disponível no sistema.

---

### 1. Temporária (só vale até fechar o terminal)

```bash
export MINHA_VARIAVEL="algum valor"
echo $MINHA_VARIAVEL
```

Depois que você fechar o terminal, sumiu. Útil para testes rápidos.

---

### 2. Permanente para o seu usuário

Você coloca o `export` dentro de um arquivo de configuração do shell.
No macOS moderno o padrão é o **zsh**, então edite o arquivo `~/.zshrc`:

```bash
nano ~/.zshrc
```

Adicione no final:

```bash
export DB_USER="milena"
export DB_PASS="segredo"
```

Salve e recarregue:

```bash
source ~/.zshrc
```

Agora, toda vez que abrir o terminal, essas variáveis vão estar disponíveis.

Aspas duplas não são obrigatórias!

Quando você escreve:

```bash
export DB_USER=milena
```

isso funciona normalmente.

As aspas só viram **necessárias** em casos em que o valor contém:

* espaços
* caracteres especiais (`$`, `!`, `*`, etc.) que o shell poderia interpretar de outra forma.

Exemplo:

```bash
export MENSAGEM="Olá mundo bonito"
```

Se não usar aspas aqui, o shell ia tentar exportar `MENSAGEM=Olá` e depois reclamar do resto.

### Atualizar

1. Abra:

   ```bash
   nano ~/.zshrc
   ```
2. Procure a linha com o `export` que você quer mudar:

   ```bash
   export DB_USER=milena
   ```
3. Troque o valor:

   ```bash
   export DB_USER=admin
   ```
4. Salve (`CTRL+O`, depois `ENTER`) e saia (`CTRL+X`).
5. Recarregue:

   ```bash
   source ~/.zshrc
   ```

### Apagar

1. Mesmo processo: abra o `~/.zshrc`.
2. Delete a linha com a variável que você não quer mais.
3. Salve e feche.
4. Recarregue com `source ~/.zshrc`.

---

* O `source ~/.zshrc` só recarrega no terminal atual.
* Se você abrir novos terminais, eles já vão vir com a configuração atualizada (sem a variável removida ou com o valor novo).
* Se tiver aberto uma IDE ou outro app que herdou a variável antiga, talvez precise fechar e abrir de novo pra ele “esquecer”.

---

### 3. Para aplicações gráficas (fora do terminal)

Às vezes você quer que apps rodando no Finder ou IntelliJ também vejam suas variáveis. Nesse caso o jeito é colocar no arquivo `~/.zprofile` (carregado no login) ou definir via `launchctl`:

```bash
launchctl setenv MINHA_VARIAVEL "valor"
```

Isso deixa disponível para programas rodando sob o ambiente gráfico do macOS.

---

### 4. Em projetos específicos (Spring Boot, por exemplo)

Muita gente prefere usar um arquivo `.env` e um gerenciador (como o **direnv** ou até Docker Compose). Assim, você não “polui” seu ambiente global e mantém as variáveis só naquele projeto.

---

## Variáveis de Ambiente em Aplicações

No Spring Boot, o **truque** é que o `application.properties` (ou `application.yml`) entende placeholders `${…}` que se conectam com variáveis de ambiente.

Imagina que você tem esse `application.properties`:

```properties
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
spring.jpa.hibernate.ddl-auto=update
```

Aqui, o `DB_URL`, `DB_USER` e `DB_PASS` **não estão definidos no arquivo**. O Spring vai procurar primeiro no ambiente (variáveis de ambiente do sistema, Docker, Kubernetes, etc.) e só depois em outras fontes.

---

### No macOS

Você pode criar as variáveis no terminal (ou no `~/.zshrc`):

```bash
export DB_URL=jdbc:postgresql://localhost:5432/minhabase
export DB_USER=milena
export DB_PASS=segredo123
```

Depois, quando rodar sua aplicação:

```bash
./mvnw spring-boot:run
```

ou

```bash
java -jar target/app.jar
```

o Spring Boot vai ler essas variáveis e substituir automaticamente nos pontos `${…}` do `application.properties`.

---

### Com valores padrão (fallback)

Você ainda pode deixar um valor de “reserva” no `application.properties`:

```properties
spring.datasource.url=${DB_URL:jdbc:h2:mem:testdb}
spring.datasource.username=${DB_USER:default_user}
spring.datasource.password=${DB_PASS:default_pass}
```

Se a variável de ambiente **não existir**, ele usa o valor depois dos `:`.

---

Você pode retornar o valor guardado de uma variável de ambiente usando:

```bash
printenv | grep DB_USER
```
* Retorna o valor armazenado por `DB_USER`

Ou simplesmente:

```bash
printenv
```
* Retorna o valor de todas as variáveis de ambientes armazenadas no sistema




