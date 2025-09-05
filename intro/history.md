# Conhecendo o Spring Framework pt.01

## **Contexto pré-Spring (início dos anos 2000):**

Na época, o desenvolvimento de aplicações corporativas em Java girava em torno da Java EE (antiga J2EE), que trazia o modelo EJB (Enterprise JavaBeans) como a “receita oficial” para lidar com lógica de negócio em aplicações grandes e distribuídas.

* Só que os **EJBs eram muito complexos**: exigiam configuração pesada em XML, tinham ciclo de vida complicado, precisavam de servidores de aplicação enormes e caros, e acabavam impondo uma curva de aprendizado alta. Isso fazia sentido para aplicações distribuídas gigantes, mas não era prático para casos mais simples, que representavam a maior parte dos sistemas corporativos.

(*Rod Johnson* - 2002)

O livro Expert One-to-One J2EE Design and Development criticando justamente essa “visão única” da Sun:

* EJBs não eram sempre necessários.

* Muitas vezes havia overengineering.

* Java EE impunha um modelo engessado, que nem sempre atendia bem os projetos reais.

## Nascimento do Spring (2003–2004):

Em 2003, começou-se a desenvolver o Spring Framework.

Em 2004, ele foi lançado oficialmente junto com o livro Expert One-to-One J2EE Development Without EJB.

A ideia central do Spring era:

* Reduzir complexidade (menos configuração, mais produtividade).

* Dar flexibilidade (você só usa o que precisa, nada de imposição).

* Facilitar testes (injeção de dependência e POJOs em vez de componentes pesados).

* Tornar o Java EE mais leve, aproveitando o que era bom e descartando o que era burocrático.

**Limitações da plataforma Java (na época):**

1. Complexidade excessiva (especialmente com EJBs).

2. Alto acoplamento do código à infraestrutura do servidor.

3. Curva de aprendizado muito íngreme para novos desenvolvedores.

4. Baixa produtividade devido à necessidade de muito código “boilerplate” (repetitivo).

5. Pouca testabilidade – era difícil testar unidades de código sem rodar a aplicação inteira.

O Spring nasceu como uma resposta pragmática à rigidez e complexidade da Java EE, trazendo um modelo mais leve e flexível, que permitia ao desenvolvedor focar no negócio em vez de lutar contra a infraestrutura.