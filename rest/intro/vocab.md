
## Glossário

### Servidor de aplicações

Um servidor de aplicações (ou application server) é uma plataforma de software que fornece um ambiente para executar e gerir aplicações, atuando como intermediário entre os clientes e os sistemas de back-end. 

Ele hospeda, executa e entrega software aos utilizadores através da rede, sendo responsável por tarefas como segurança, balanceamento de carga e tratamento de exceções, permitindo que os desenvolvedores se concentrem na lógica de negócios da aplicação em vez da infraestrutura subjacente. 

* O Spring surgiu justamente como alternativa “mais leve” aos servidores de aplicações Java EE. Em vez de depender de um servidor gigante que provê tudo, o Spring te dá só o que você precisa (injeção de dependências, MVC, transações, etc.).

* Ou seja: o Spring “emula” várias funções de um servidor de aplicações dentro o próprio framework, o container de IoC. 

* Uma aplicação Spring é, então, executada usando somente a JVM, e, caso queira rodar a aplicação na rede, pode-se usufuir de um Servlet "simples", como Tomcat.

* Esse argumento corrobora para afirmar que o Spring lida com containers lighweight, ou seja, minimamente intrusivos, já que, ao contrário dos EJBs, o container IoC Spring gerencia seus beans sem alterar sua natureza (o bean funciona e existe com ou sem container).

### Servidor Web

Um servidor web é um sistema que armazena os ficheiros de um website e os entrega aos navegadores dos utilizadores através da internet, usando o protocolo HTTP.

Funciona recebendo pedidos de páginas web através de uma URL, processando-os (muitas vezes consultando um servidor de base de dados) e enviando de volta o conteúdo solicitado.

Enquanto um servidor de aplicações vai além, fornecendo infraestrutura para executar código mais complexo e interagir com bases de dados e outros sistemas, com suporte a diversos protocolos além do HTTP, um servidor web (web server) é um servidor de aplicações, mas mais simples, porque sua única função é entender HTTP, responder as requisições e entregar a resposta em formato de páginas HTML, arquivos estáticos, imagens, JSON, etc.

* Ele não tem por padrão suporte a componentes Java corporativos, apenas lida com requisições HTTP

O termo "servidor web" refere-se tanto ao hardware (o computador físico que armazena os dados) quanto ao software (o programa que executa as funções de servir as páginas). 

* Quando você usa Spring MVC ou Spring Boot, você está rodando seu app dentro de um servidor web Java (por exemplo, Tomcat ou Jetty). O Spring cuida da sua lógica de negócio, mas quem realmente lida com a requisição HTTP e a transforma em algo que o Spring entende é o servidor web.

### Servlet

Um servlet é um objeto Java que roda num servidor web para criar páginas dinâmicas, processando requisições HTTP e gerando respostas ao cliente.

Funciona como um componente Java que é executado e gerido por um container de servlets (como o Tomcat) e que processa requisições HTTP. É como a unidade fundamental para responder requisições web em Java.

* O Spring MVC é contruído em cima da API de servlets. Ele registra um `DispatcherServlet`, que funciona como um front controller. Esse servlet intercepta todas as requisições, faz o roteamento para seus controllers, chama os serviços e retorna a resposta.

* Então quando um cliente faz uma requisição web, o container de servlets intercepta esse requisição, injeta o servlet para processando e por fim gerar uma resposta.

### Apache Tomcat

Apache Tomcat é um container de servlets (e também um servidor web Java). Ele entende HTTP e sabe executar servlets/JSP.

* Fornece um ambiente de execução para aplicações web Java, servindo páginas com código dinâmico (injeta o servlet adequado para a requisição).

---

## Link Geral:

* Web Server: responde requisições HTTP.

* Servlet: unidade de processamento dessas requisições dentro do web server.

* Tomcat: implementação de servidor web + container de servlets mais usado com Spring.

* Servidor de aplicações: "mais pesado", mas o Spring substitui boa parte dessa função, rodando em servidores leves, como o Tomcat.

O Spring Boot embute um Tomcat (servidor web/container de servlets). Esse Tomcat executa o DispatcherServlet do Spring MVC, que é o ponto de entrada das requisições. O Spring por sua vez organiza e injeta as dependências da sua aplicação, simulando várias funções que antes só um servidor de aplicações Java EE fazia.
