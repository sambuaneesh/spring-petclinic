| Component Name | Language | Frameworks | Database | Communication | Patterns |
|---|---|---|---|---|---|
| Application Bootstrap & Configuration | Java 17+ | Spring Boot 3.x, Spring AOT (GraalVM native hints) | N/A | In-process | Configuration-as-code, Convention-over-configuration |
| Web Presentation (HTML MVC) | Java 17+ | Spring MVC, Spring Boot Web, Thymeleaf, WebJars | Relational (H2 default; MySQL/PostgreSQL via profiles) | HTTP (HTML), in-process to repositories | MVC, Controller, Server-side rendering, Pagination |
| JSON API (/vets) | Java 17+ | Spring MVC, Jackson | Relational (H2 default; MySQL/PostgreSQL via profiles) | HTTP (JSON) | REST endpoint, DTO wrapper (Vets), Caching at repository |
| Domain: Owners/Pets/Visits | Java 17+ | JPA/Hibernate, Spring Boot, Jakarta Bean Validation (JSR 380) | Relational (H2 default; MySQL/PostgreSQL via profiles) | In-process domain operations | DDD Entities, Aggregate Roots (Owner→Pet, Pet→Visit), Cascading writes, Validation |
| Domain: Vets/Specialties | Java 17+ | JPA/Hibernate, Spring Boot | Relational (H2 default; MySQL/PostgreSQL via profiles) | In-process domain operations | Entities, Many-to-many, Read-mostly |
| Persistence & Repositories | Java 17+ | Spring Data JPA, Hibernate | Relational (H2 default; MySQL/PostgreSQL via profiles) | In-process; JDBC via ORM | Repository pattern, Query methods, Pagination |
| Validation & Formatting | Java 17+ | Jakarta Bean Validation (JSR 380), Spring Validator, Spring Formatter | N/A | In-process | Validator (PetValidator), Formatter (PetTypeFormatter) |
| Caching | Java 17+ | Spring Cache, JCache (javax.cache), Caffeine | N/A | In-process cache | Read-through cache (@Cacheable), Cache-per-query (vets) |
| Internationalization (i18n) | Java 17+ | Spring MVC i18n (LocaleResolver, LocaleChangeInterceptor), Message bundles | N/A | HTTP (lang param), Session | Locale per session, Message bundle resolution |
| Observability & Actuator | Java 17+ | Spring Boot Actuator | N/A | HTTP management endpoints (/actuator, /livez, /readyz) | Health checks, Metrics, Probes integration |