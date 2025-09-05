# Tipos de Arquitetura de Aplicações

---

## **Aplicação Monolítica**

Uma **aplicação monolítica** é um tipo de arquitetura de software em que **todas as funcionalidades do sistema estão integradas em um único bloco de código ou projeto**.

* **“Monolítico”** vem de “monólito”, ou seja, algo grande e indivisível.
* Isso significa que **a interface, a lógica de negócio e o acesso a dados** estão, geralmente, juntos em uma única aplicação.

---

### **Como ela funciona na prática?**

Imagine que você tem um sistema de e-commerce:

* Cadastro de usuários
* Catálogo de produtos
* Carrinho de compras
* Pagamentos
* Relatórios

Em uma aplicação monolítica:

1. Todos esses módulos **compartilham o mesmo código e banco de dados**.
2. Há **uma única aplicação** que você precisa compilar, empacotar e rodar.
3. Quando você faz uma mudança em um módulo (por exemplo, adiciona um novo tipo de pagamento), normalmente você precisa **recompilar e redeployar toda a aplicação**.
4. Tudo é executado em **um mesmo processo ou servidor**.

---

### **Vantagens:**

* Simples de desenvolver no início.
* Fácil de testar em pequenos projetos.
* Não precisa lidar com comunicação entre serviços.

### **Desvantagens:**

* Dificuldade de escalar individualmente (não dá para escalar apenas o módulo de pagamentos, por exemplo).
* Deploy lento e arriscado — um erro em um módulo pode derrubar toda a aplicação.
* Com o crescimento do sistema, o código pode ficar **difícil de manter e entender**.

---

## **Aplicação em Microsserviços**

### **O que são microserviços?**

**Microserviços** são uma abordagem arquitetural em que um sistema é dividido em **vários serviços menores, independentes e especializados**, cada um responsável por **uma funcionalidade ou domínio específico**.

* Cada serviço **funciona de forma isolada**, podendo ter seu próprio banco de dados, linguagem de programação e ciclo de deploy.
* Esses serviços se comunicam entre si geralmente via **HTTP/REST, gRPC ou mensageria**.

Voltando ao exemplo do e-commerce:

* Cadastro de usuários → serviço de usuários
* Catálogo de produtos → serviço de produtos
* Pagamentos → serviço de pagamentos
* Relatórios → serviço de relatórios

Cada serviço pode ser **desenvolvido, testado e implantado independentemente**.

---

### **Problemas que os microserviços resolvem em relação ao monólito**

| Problema no monólito                       | Como microserviços ajudam                                                                                                        |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| Escalabilidade limitada                    | Cada serviço pode ser escalado independentemente, conforme a demanda. Ex: apenas o serviço de pagamentos recebe mais instâncias. |
| Deploy arriscado e lento                   | Como os serviços são independentes, você pode atualizar apenas o serviço necessário sem afetar o restante do sistema.            |
| Código grande e difícil de manter          | O código fica modular, focado em uma única responsabilidade por serviço.                                                         |
| Falha em um módulo derruba tudo            | Falhas podem ser isoladas; um serviço pode parar sem afetar todos os outros.                                                     |
| Dificuldade de usar tecnologias diferentes | Cada serviço pode usar a tecnologia mais adequada ao seu domínio.                                                                |

---

### **Vantagens adicionais**

* Permite **times menores e especializados** trabalhando em serviços específicos.
* Facilita a adoção de **novas tecnologias ou frameworks** sem reescrever tudo.
* Melhora a **resiliência**: falhas são isoladas, e o sistema continua funcionando.

---

### **Desvantagens**

* Maior complexidade de **orquestração e comunicação entre serviços**.
* Necessidade de lidar com **mensageria, balanceamento de carga e observabilidade**.
* Mais difícil de testar e depurar do que um monólito simples.

---

## **O que é uma API REST?**

* **API** = *Application Programming Interface* → é uma forma de **permitir que sistemas diferentes “conversem” entre si**.
* **REST** = *Representational State Transfer* → é um estilo arquitetural criado por Roy Fielding na sua tese de doutorado em 2000 que define **regras e boas práticas para comunicação entre sistemas via HTTP**.

Na época, os sistemas distribuídos (aplicações que precisavam se comunicar via rede) sofriam com protocolos complexos e pesados como CORBA, SOAP e RPC.

Uma **API REST** permite que diferentes sistemas, independentemente da linguagem ou plataforma, **acessam recursos de um serviço de forma padronizada**.

---

### **Por que surgiu REST?**

Antes do REST, existiam principalmente:

* **RPC (Remote Procedure Call)** → chamava funções remotamente, mas era **fortemente acoplado** e dependente de tecnologias específicas.
* **SOAP (Simple Object Access Protocol)** → baseado em XML e muito verboso, complexo de implementar.

Essas soluções tinham alguns problemas sérios:

1. **Alto acoplamento**: os clientes precisavam conhecer detalhes internos do servidor (como métodos e contratos específicos).

2. **Complexidade**: protocolos como SOAP usavam XML extenso, envelopes e metadados complicados. Isso aumentava o custo de implementação, manutenção e até o consumo de rede.

3. **Baixa escalabilidade**: muitos desses modelos exigiam manter estado no servidor (session state), o que dificultava distribuir carga em vários servidores.

4. **Pouca aderência à Web**: a Web já usava HTTP de forma massiva, mas esses protocolos tentavam reinventar a roda, criando camadas extras ao invés de aproveitar HTTP “como ele é”.

REST surgiu para oferecer:

1. **Simplicidade** → usa padrões já existentes da web (HTTP, URIs, headers e status code).
2. **Independência de plataforma** → qualquer linguagem que entenda HTTP pode consumir a API.
3. **Escalabilidade** → cada recurso é independente e pode ser cacheado ou replicado.
4. **Stateless** → cada requisição contém todas as informações necessárias, sem depender de sessão do servidor, esse que, portanto, não guarda contexto entre requisições.
4. **Menor Acoplamento** → o cliente só precisa conhecer os recursos expostos (ex: `/usuarios`, `/produtos`) e não a implementação interna.

---

### **Princípios principais do REST**

1. **Recursos bem definidos** → tudo é um recurso identificado por uma URI (`/usuarios`, `/produtos/123`).
2. **Operações padronizadas** → uso de métodos HTTP:

   * `GET` → buscar
   * `POST` → criar
   * `PUT` → atualizar
   * `DELETE` → remover
3. **Stateless** → o servidor não mantém estado entre requisições.
4. **Representação dos recursos** → geralmente JSON (mais leve que XML).
5. **Client-Server** → cliente e servidor são separados, permitindo evolução independente.

---

* **Stateless**: sem estado

Isso significa que cada requisição HTTP enviada pelo cliente deve conter todas as informações necessárias para o servidor processá-la, sem depender de informações de requisições anteriores.

* O servidor não guarda o contexto entre as chamadas.

Ou seja, ele não “lembra” quem você é ou o que você fez antes, a menos que você mande esses dados novamente.

Ex) Imagine que você está fazendo compras online:

* **Com estado (stateful):**

Você entra no site, faz login e o servidor guarda sua sessão em memória.

Cada requisição que você fizer depois, o servidor “lembra” quem você é porque mantém um registro.

Isso cria dependência: se o servidor cair, ou se sua requisição for para outro servidor no cluster, sua sessão pode se perder.

* **Sem estado (stateless / REST):**

Você faz login, e o servidor devolve um token (por exemplo, JWT).

Em cada requisição subsequente, você envia o token no header (ex: `Authorization: Bearer <token>`).

O servidor processa a requisição apenas com base nos dados que você enviou naquele momento, sem depender de memória local.

---

REST é como um conjunto de **regras para enviar e receber cartas entre sistemas**: você sabe o endereço, escreve a carta de um jeito padronizado, e o servidor responde de forma previsível.

---




