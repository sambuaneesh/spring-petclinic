| Component Name | Language | Frameworks | Database | Communication | Patterns |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Owner Management** | Java | Spring Boot, Spring MVC, Spring Data JPA, Jakarta Validation | Relational (via Data Persistence) | HTTP, Direct Method Calls | MVC, Repository, Dependency Injection |
| **Veterinarian Management** | Java | Spring Boot, Spring MVC, Spring Data, Spring Cache (JCache/Caffeine) | Relational (via Data Persistence) | HTTP, Direct Method Calls | MVC, Repository, Caching, Dependency Injection |
| **System** | Java | Spring Boot, Spring MVC, Spring Cache | N/A | HTTP, AOP/Interceptors | Configuration, Controller, Interceptor |
| **User Interface** | HTML, CSS/SCSS | Thymeleaf, Bootstrap, WebJars | N/A | HTTP | Server-Side Rendering (SSR), Template View |
| **Shared Kernel** | Java | Jakarta Persistence (JPA) | N/A | In-memory object sharing | Domain Model, Entity, Mapped Superclass |
| **Data Persistence** | Java, SQL | Spring Data JPA, Hibernate | H2, MySQL, PostgreSQL | JDBC | Repository, Data Access Object (DAO) |