```mermaid
graph TB
  B[Browser] --> |HTTP GET/POST| C[Controllers<br/>- OwnerController<br/>- PetController<br/>- VisitController<br/>- VetController<br/>- Welcome/Crash]

  subgraph Monolith[Spring PetClinic Monolith (Spring Boot)]
    direction TB

    subgraph Web[Web/UI Layer]
      C --> |render HTML views| T[Thymeleaf Views/Templates]
      C --> |validate/format| VF[Validator/Formatter<br/>- PetValidator<br/>- PetTypeFormatter]
      C --> |serves static| SR[Static Resources Handler<br/>(WebJars, CSS, images)]
      WF[Web Config & i18n<br/>- LocaleChangeInterceptor<br/>- SessionLocaleResolver]
      WF --> |locale selection (?lang)| T
    end

    subgraph Persistence[Domain & Persistence]
      R1[OwnerRepository]
      R2[PetTypeRepository]
      R3[VetRepository (@Cacheable "vets")]
      M[Domain Model (JPA Entities)<br/>Owner, Pet, PetType, Visit, Vet, Specialty]
      JPA[Spring Data JPA / Hibernate]
      Cache[JCache (Caffeine)<br/>cache: "vets"]

      C --> |owners CRUD| R1
      C --> |pet types| R2
      C --> |vets list (HTML/JSON)| R3

      R1 --- M
      R2 --- M
      R3 --- M

      R1 --> JPA
      R2 --> JPA
      R3 --> JPA
      R3 <--> |@Cacheable| Cache
    end

    subgraph Ops[Operations]
      Act[Actuator (/actuator, /livez, /readyz)]
    end
  end

  JPA --> DB[(Relational Database<br/>H2/MySQL/Postgres)]
  T --> |i18n messages| I18N[(Message Bundles<br/>messages/*)]
  SR --> WebJ[(Packaged WebJars<br/>bootstrap, font-awesome)]
  B --> |Ops/Health| Act
```

Rationale:
The monolith is structured by packages into a Web/UI layer (controllers, Thymeleaf, validation/formatting, i18n) and a persistence layer (Spring Data repositories operating on JPA entities via Hibernate), with VetRepository reads accelerated by a local JCache/Caffeine cache. Controllers synchronously invoke repositories in-process and render server-side HTML; VetController additionally supports JSON via content negotiation, while all data ultimately persists to a single relational database selected by profile. Actuator exposes operational endpoints separately, and static assets and message bundles are served from the application classpath.