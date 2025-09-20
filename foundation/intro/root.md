# Roteiro de Estudos

---

## **Fase 1 – Fundamentos Spring**

| Tópico              | Detalhes / Sub-tópicos                                                                                                                            | Livro principal |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | --------------- |
| IoC e DI            | Beans, `@Component`, `@Service`, `@Repository`, `@Autowired`, ciclo de vida (`@PostConstruct`, `@PreDestroy`), escopos (`singleton`, `prototype`) | Weissmann       |
| Configuração Spring | XML, Java Config, Annotation Config, SpEL                                                                                                         | Weissmann       |
| AOP                 | Aspectos, pointcuts, advices, logging, transações                                                                                                 | Weissmann       |
| MVC clássico        | `@Controller`, `@RequestMapping`, ModelAndView, views JSP/Thymeleaf, submissão de formulários, validação                                          | Weissmann       |
| Transações          | `@Transactional`, propagation, rollback, commit                                                                                                   | Weissmann       |

---

## **Fase 2 – Persistência e Banco de Dados**

| Tópico                 | Detalhes / Sub-tópicos                                                                                     | Livro principal |
| ---------------------- | ---------------------------------------------------------------------------------------------------------- | --------------- |
| JPA/Hibernate clássico | `@Entity`, `@Id`, `@GeneratedValue`, relacionamentos, Lazy/Eager, DAO pattern, CRUD manual                 | Weissmann       |
| Spring Data JPA        | `JpaRepository`, queries automáticas (`findBy...`), queries customizadas (`@Query`), paginação e ordenação | Boaglio         |
| Transações em Boot     | `@Transactional` aplicado em services, gerenciamento automático de commits e rollbacks                     | Boaglio         |

---

## **Fase 3 – APIs REST e MVC Moderno**

| Tópico                         | Detalhes / Sub-tópicos                                                                                                              | Livro principal     |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| REST Controllers               | `@RestController`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@RequestParam`, `@PathVariable`, `@RequestBody` | Boaglio             |
| DTOs e validação               | `@Valid`, `BindingResult`, mapeamento de DTOs                                                                                       | Boaglio             |
| HATEOAS                        | Construção de links e boas práticas REST                                                                                            | Boaglio             |
| Tratamento de exceções         | `@ControllerAdvice`, `@ExceptionHandler`                                                                                            | Boaglio             |
| Comparação MVC clássico x Boot | ModelAndView vs JSON automático, abstrações Boot                                                                                    | Weissmann + Boaglio |

---

## **Fase 4 – Segurança e Aspectos Transversais**

| Tópico               | Detalhes / Sub-tópicos                                                             | Livro principal     |
| -------------------- | ---------------------------------------------------------------------------------- | ------------------- |
| Spring Security      | Autenticação, autorização, JWT, roles/authorities (`@PreAuthorize`, `@Secured`)    | Boaglio             |
| AOP aplicado em Boot | Logging de requisições/respostas, transações automáticas, interceptação de métodos | Weissmann + Boaglio |

---

## **Fase 5 – Projetos e Ferramentas Modernas**

| Tópico                   | Detalhes / Sub-tópicos                                                        | Livro principal |
| ------------------------ | ----------------------------------------------------------------------------- | --------------- |
| Docker e containerização | Dockerfile, Docker Compose, separação de serviços, volumes                    | Boaglio         |
| Actuator e monitoramento | Health, metrics, logging                                                      | Boaglio         |
| Profiles e deploy        | `application-dev.yml`, `application-prod.yml`, deploy em AWS, Railway, Heroku | Boaglio         |
| Testes                   | Unitários e integração com JUnit 5, Mockito, `@SpringBootTest`                | Boaglio         |

---

## **Fase 6 – Projetos integrados (Capstone)**

| Tópico                      | Detalhes / Sub-tópicos                                                     | Livro principal |
| --------------------------- | -------------------------------------------------------------------------- | --------------- |
| Projeto full-stack Boot     | REST API + JPA/Hibernate, Security JWT, Docker, profiles, Actuator, testes | Boaglio         |
| Extras avançados (opcional) | Filas (RabbitMQ/Kafka), Spring Reativo, cache (`@Cacheable`, Redis)        | Boaglio         |

---

## **Weissmann**

* Entender **conceitos por trás do Spring Framework**: IoC, DI, AOP, beans, ciclo de vida, transações, MVC clássico, JPA/Hibernate básico.
* Ou seja, você quer **domínio teórico** e de “como as coisas funcionam por baixo do panos”.

---

## **Boaglio**

* Ensina **Spring Boot moderno**, aplicações práticas, REST, Thymeleaf, Docker, Actuator, testes, segurança, reatividade, deploy em nuvem.
* Boaglio **não aprofunda fundamentos teóricos** de DI, AOP, ciclo de vida dos beans, MVC clássico — ele assume que você já entende.

---

## **Sincronizando Conteúdos**

Analisando o sumário do Weissmann:

| Capítulo | Conteúdo                                                                        | Observação                                                                                                                                   |
| -------- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 a 5    | Conceitos de alto acoplamento, IoC, DI, Container, AOP, ciclo de vida dos beans | Essencial ler até aqui, pois fornece **fundamentos teóricos sólidos**                                                                        |
| 6        | Colocando a mão na massa                                                        | Opcional ler, pode pular para Boaglio se quiser praticar Boot diretamente                                                                    |
| 7        | Acesso a dados (JDBC, Hibernate, JPA clássico)                                  | Leia ao menos **7.1 a 7.7** para entender DAO e integração com JPA/Hibernate. Não precisa se aprofundar em JDBC puro, pois Boot abstrai isso |
| 8        | Desenvolva aplicações web com Spring MVC                                        | Leia **8.1 a 8.5** para entender o fluxo DispatcherServlet, controlador e ModelAndView; depois pode migrar, porque Boot simplifica MVC       |
| 9        | Ações recorrentes com Spring MVC                                                | Pode pular parcialmente, pois Boot tem suas abstrações REST modernas                                                                         |
| 10       | Gerenciando transações                                                          | Leia **10.1 a 10.4** para entender como Boot gerencia transações; detalhes de programáticas podem ser consultados se necessário              |
| 11       | Spring Security                                                                 | Pode ser usado como referência teórica; Boaglio vai mostrar implementação prática com Boot e JWT                                             |

**Conclusão prática:**

> **Pare de estudar Weissmann no capítulo 8 (8.5) ou no máximo no 10.4**.
> A partir daí, você pode migrar para Boaglio, porque:
>
> * Você já tem entendimento do container, DI, AOP, ciclo de vida, MVC clássico, JPA/Hibernate básico, transações.
> * Boaglio vai te ensinar **como aplicar isso em Boot moderno**: REST, Thymeleaf, Docker, Actuator, segurança, testes, deploy.


