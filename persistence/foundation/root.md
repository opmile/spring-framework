## **Nível 1 – CRUD básico com Spring Data JPA**

1. Crie uma entidade Java chamada Produto com os campos id, nome e preco, usando anotações JPA. Explique cada anotação que você usou (como @Entity, @Id, @GeneratedValue, @Column).
2. Crie um repository para a entidade Produto estendendo JpaRepository. Liste os métodos CRUD já disponíveis por padrão (findAll, findById, save, deleteById) e explique como o Spring os implementa automaticamente.
3. Crie um service ProdutoService e injete o ProdutoRepository. Implemente métodos que chamam os métodos CRUD do repository e explique o papel do service na arquitetura.
4. Crie um controller ProdutoController que exponha endpoints REST para listar, criar, atualizar e deletar produtos. Faça exemplos de requisições GET, POST, PUT e DELETE.

---

## **Nível 2 – Consultas simples e derivadas**

1. Crie métodos no repository como findByNome, findByPrecoGreaterThan e findByNomeContaining. Explique como o Spring converte os nomes dos métodos em queries SQL.
2. Crie um método que busque produtos por faixa de preço usando a anotação @Query e linguagem JPQL. Explique a diferença entre JPQL e SQL nativo.
3. Implemente métodos que retornem produtos ordenados por nome ou preço. Adicione paginação usando Pageable. Explique como funciona internamente.

---

## **Nível 3 – Relacionamentos e Modelagem**

1. Crie uma entidade Categoria e estabeleça relacionamento OneToMany com Produto. Configure mapeamento bidirecional com ManyToOne no Produto. Explique como o Hibernate gerencia essas associações.
2. Crie uma entidade Tag e relacione produtos e tags com ManyToMany. Explique a tabela de junção criada pelo Hibernate e como controlar fetch e cascade.
3. Crie um método no repository para buscar produtos de uma determinada categoria ou com uma tag específica usando JPQL. Explique como o join funciona no JPA.

---

## **Nível 4 – Transações, exceções e performance**

1. Configure um método de service com @Transactional que atualize múltiplos produtos em uma operação. Explique como o Spring gerencia commit e rollback.
2. Mostre como capturar exceções como DataIntegrityViolationException ou EntityNotFoundException e retornar mensagens apropriadas para o cliente REST.
3. Explique a diferença entre fetch eager e lazy. Crie exemplos práticos mostrando como consultas podem gerar N+1 queries e como resolver.

---

## **Nível 5 – Avançado: Criteria API e Native Queries**

1. Crie uma busca dinâmica de produtos usando Criteria API, permitindo filtrar por nome, preço e categoria. Explique como a API cria consultas tipadas.
2. Crie métodos que executem queries SQL nativas no PostgreSQL usando @Query(nativeQuery=true). Explique quando é melhor usar nativo em vez de JPQL.
3. Mostre como criar consultas que retornam DTOs customizados ao invés da entidade completa. Explique a vantagem em performance e desacoplamento.

---

