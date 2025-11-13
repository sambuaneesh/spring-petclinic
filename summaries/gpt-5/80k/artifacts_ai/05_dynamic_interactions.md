# Dynamic Interaction Flows and Sequence Diagrams for Spring PetClinic (Monolith)

## Veterinarians listing (HTML, paginated, with cache)

Purpose and trigger:
- User navigates to list veterinarians page with pagination.
- Trigger: GET /vets.html?page={n}

Communication patterns:
- HTTP GET (synchronous)
- Spring MVC → Repository (in-process, synchronous)
- JCache (Caffeine) read-through cache on VetRepository.findAll(Pageable)
- JPA read-only transaction; SQL SELECT with paging
- Thymeleaf server-side rendering

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant I18N as LocaleChangeInterceptor
  participant VC as VetController
  participant R as VetRepository
  participant Cache as JCache (Caffeine)
  participant JPA as EntityManager/JPA
  participant DB as RDBMS
  participant V as Thymeleaf View

  B->>D: GET /vets.html?page=2
  D->>I18N: preHandle(?lang)
  I18N-->>D: proceed
  D->>VC: listVets(page=2)
  VC->>R: findAll(Pageable(size=5,page=2))
  R->>Cache: get(key=Page:2)
  alt Cache miss
    Cache-->>R: miss
    R->>JPA: createQuery for vets + specialties (paged)
    JPA->>DB: SELECT vets LEFT JOIN vet_specialties... LIMIT/OFFSET
    DB-->>JPA: result set
    JPA-->>R: Page<Vet>
    R->>Cache: put(key=Page:2, value=Page<Vet>)
  else Cache hit
    Cache-->>R: Page<Vet>
  end
  R-->>VC: Page<Vet>
  VC-->>D: ModelAndView(vetList.html, model=page)
  D->>V: render vetList.html
  V-->>B: 200 OK (HTML)
```

---

## Veterinarians listing (JSON, cached)

Purpose and trigger:
- API consumer retrieves vets as JSON.
- Trigger: GET /vets

Communication patterns:
- HTTP GET (synchronous)
- Spring MVC → Repository (in-process)
- JCache (Caffeine) read-through cache on VetRepository.findAll()
- JPA read-only transaction; SQL SELECT (no paging)
- Jackson serialization to JSON

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser/API Client
  participant D as DispatcherServlet
  participant VC as VetController
  participant R as VetRepository
  participant Cache as JCache (Caffeine)
  participant JPA as EntityManager/JPA
  participant DB as RDBMS

  B->>D: GET /vets (Accept: application/json)
  D->>VC: showResourcesVetList()
  VC->>R: findAll()
  R->>Cache: get(key=ALL)
  alt Cache miss
    Cache-->>R: miss
    R->>JPA: createQuery for vets + specialties
    JPA->>DB: SELECT vets JOIN specialties
    DB-->>JPA: rows
    JPA-->>R: Collection<Vet>
    R->>Cache: put(key=ALL, value=vets)
  else Cache hit
    Cache-->>R: Collection<Vet>
  end
  R-->>VC: Collection<Vet>
  VC-->>D: Vets wrapper (vetList)
  D-->>B: 200 OK (application/json)
```

---

## Find Owners (search + pagination + redirect/empty cases)

Purpose and trigger:
- User searches owners by last name prefix; paginated listing.
- Trigger: GET /owners?page={n}&lastName={prefix}

Communication patterns:
- HTTP GET (synchronous)
- Spring MVC → Repository (in-process)
- JPA read-only transaction; SQL SELECT + COUNT for pagination
- Thymeleaf SSR

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant OC as OwnerController
  participant R as OwnerRepository
  participant JPA as EntityManager/JPA
  participant DB as RDBMS
  participant V as Thymeleaf View

  B->>D: GET /owners?page=1&lastName=Smi
  D->>OC: processFindForm(prefix="Smi", page=1)
  OC->>R: findByLastNameStartingWith("Smi", Pageable(size=5, page=0))
  R->>JPA: build query + count
  JPA->>DB: SELECT owners WHERE last_name LIKE 'Smi%' LIMIT/OFFSET
  DB-->>JPA: rows + count
  JPA-->>R: Page<Owner>
  R-->>OC: Page<Owner>
  alt Exactly one match
    OC-->>D: redirect:/owners/{id}
    D-->>B: 302 Found
    B->>D: GET /owners/{id}
    D->>OC: getOwner(id)
    OC->>R: findById(id)
    R->>DB: SELECT owner + pets (EAGER) + visits (EAGER)
    DB-->>R: owner graph
    R-->>OC: Owner
    D->>V: render ownerDetails.html
    V-->>B: 200 OK (HTML)
  else Zero matches
    OC-->>D: return findOwners.html with BindingResult error on lastName
    D->>V: render findOwners.html
    V-->>B: 200 OK (HTML with error)
  else Multiple matches
    OC-->>D: return ownersList.html (Page<Owner>)
    D->>V: render ownersList.html
    V-->>B: 200 OK (HTML)
  end
```

---

## Create Owner (validation + save + redirect)

Purpose and trigger:
- Create a new owner record.
- Triggers:
  - GET /owners/new for form
  - POST /owners/new to submit

Communication patterns:
- HTTP GET/POST (synchronous)
- Bean Validation (JSR 380)
- Spring MVC → Repository (in-process)
- JPA write transaction; SQL INSERT
- Redirect + Flash attribute; SSR

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant OC as OwnerController
  participant BV as Bean Validation
  participant R as OwnerRepository
  participant JPA as EntityManager/JPA
  participant DB as RDBMS
  participant V as Thymeleaf View

  B->>D: GET /owners/new
  D->>OC: initCreationForm()
  OC-->>D: createOrUpdateOwnerForm.html
  D->>V: render form
  V-->>B: 200 OK (HTML)

  B->>D: POST /owners/new (form fields)
  D->>BV: validate Owner (required names, address, city, 10-digit phone)
  alt Validation errors
    D->>V: render createOrUpdateOwnerForm.html with errors
    V-->>B: 200 OK
  else Valid
    D->>OC: processCreationForm(owner)
    OC->>R: save(owner)
    R->>JPA: persist Owner
    JPA->>DB: INSERT INTO owners...
    DB-->>JPA: generated id
    JPA-->>R: Owner{id}
    R-->>OC: Owner{id}
    OC-->>D: redirect:/owners/{id} (+flash)
    D-->>B: 302 Found
    B->>D: GET /owners/{id}
    D->>OC: getOwner(id)
    OC->>R: findById(id)
    R->>DB: SELECT owner + pets (EAGER) + visits (EAGER)
    DB-->>R: owner graph
    R-->>OC: Owner
    D->>V: render ownerDetails.html
    V-->>B: 200 OK (HTML)
  end
```

---

## Create Pet for an Owner (aggregate cascade + custom validation)

Purpose and trigger:
- Add a pet to an owner with duplicate-name and date rules.
- Triggers:
  - GET /owners/{ownerId}/pets/new
  - POST /owners/{ownerId}/pets/new

Communication patterns:
- HTTP GET/POST (synchronous)
- @ModelAttribute preloading: Owner, Pet, Types
- Custom validator (PetValidator) + duplicate-name check within owner aggregate
- Spring MVC → OwnerRepository.save(owner) with cascade to Pet
- JPA write transaction; SQL INSERT into pets
- Redirect + SSR

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant PC as PetController
  participant R as OwnerRepository
  participant PTR as PetTypeRepository
  participant PV as PetValidator + DuplicateNameCheck
  participant JPA as EntityManager/JPA
  participant DB as RDBMS
  participant V as Thymeleaf View

  B->>D: GET /owners/7/pets/new
  D->>PC: initCreationForm(ownerId=7)
  PC->>R: findById(7)
  R->>DB: SELECT owner + pets (EAGER) + visits (EAGER)
  DB-->>R: owner graph
  R-->>PC: Owner
  PC->>PTR: findPetTypes()
  PTR->>DB: SELECT types ORDER BY name
  DB-->>PTR: types
  PC->>PC: create new Pet; owner.addPet(pet)
  D->>V: render createOrUpdatePetForm.html (owner, types, pet)
  V-->>B: 200 OK (HTML)

  B->>D: POST /owners/7/pets/new (form fields)
  D->>PC: processCreationForm(ownerId=7, pet form)
  PC->>PV: validate (name, type required on create, birthDate not future, duplicate name per owner)
  alt Validation errors
    D->>V: render createOrUpdatePetForm.html with errors
    V-->>B: 200 OK
  else Valid
    PC->>R: save(owner)  // cascade persists new Pet
    R->>JPA: persist Pet (via Owner aggregate)
    JPA->>DB: INSERT INTO pets(...)
    DB-->>JPA: generated id
    JPA-->>R: Owner with Pet{id}
    R-->>PC: Owner
    PC-->>D: redirect:/owners/7 (+flash)
    D-->>B: 302 Found
    B->>D: GET /owners/7
    D->>R: findById(7)
    R->>DB: SELECT owner + pets + visits (EAGER)
    DB-->>R: graph
    D->>V: render ownerDetails.html
    V-->>B: 200 OK (HTML)
  end
```

---

## Edit Pet (update in-aggregate + uniqueness rule)

Purpose and trigger:
- Update a pet’s fields ensuring unique name within the same owner.
- Triggers:
  - GET /owners/{ownerId}/pets/{petId}/edit
  - POST /owners/{ownerId}/pets/{petId}/edit

Communication patterns:
- HTTP GET/POST (synchronous)
- @ModelAttribute preloading: Owner, Pet, Types
- Custom validation and intra-aggregate uniqueness check excluding the current pet
- Spring MVC → OwnerRepository.save(owner) with cascade to Pet
- JPA write transaction; SQL UPDATE to pets
- Redirect + SSR

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant PC as PetController
  participant R as OwnerRepository
  participant PTR as PetTypeRepository
  participant PV as PetValidator + UniqueNameCheck
  participant JPA as EntityManager/JPA
  participant DB as RDBMS
  participant V as Thymeleaf View

  B->>D: GET /owners/7/pets/12/edit
  D->>PC: initUpdateForm(ownerId=7, petId=12)
  PC->>R: findById(7)
  R->>DB: SELECT owner + pets + visits (EAGER)
  DB-->>R: owner graph
  R-->>PC: Owner(with Pet#12)
  PC->>PTR: findPetTypes()
  PTR->>DB: SELECT types ORDER BY name
  DB-->>PTR: types
  D->>V: render createOrUpdatePetForm.html (existing pet)
  V-->>B: 200 OK (HTML)

  B->>D: POST /owners/7/pets/12/edit (form fields)
  D->>PC: processUpdateForm(ownerId=7, petId=12)
  PC->>PV: validate (name required, birthDate not future, unique name vs other pets of owner)
  alt Validation errors
    D->>V: render createOrUpdatePetForm.html with errors
    V-->>B: 200 OK
  else Valid
    PC->>PC: update fields of Pet#12; ensure kept in aggregate
    PC->>R: save(owner)  // cascade updates Pet
    R->>JPA: merge Owner/Pet
    JPA->>DB: UPDATE pets SET ...
    DB-->>JPA: OK
    R-->>PC: Owner
    PC-->>D: redirect:/owners/7 (+flash)
    D-->>B: 302 Found
  end
```

---

## Create Visit for a Pet (cascade via Owner aggregate)

Purpose and trigger:
- Add a visit for a pet; description required, date defaults to today.
- Triggers:
  - GET /owners/{ownerId}/pets/{petId}/visits/new
  - POST /owners/{ownerId}/pets/{petId}/visits/new

Communication patterns:
- HTTP GET/POST (synchronous)
- @ModelAttribute: load Owner + Pet; create Visit and add to Pet
- Bean Validation (description)
- Spring MVC → OwnerRepository.save(owner) cascade to Visit through Pet
- JPA write transaction; SQL INSERT into visits
- Redirect + SSR

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant VC as VisitController
  participant R as OwnerRepository
  participant BV as Bean Validation
  participant JPA as EntityManager/JPA
  participant DB as RDBMS
  participant V as Thymeleaf View

  B->>D: GET /owners/7/pets/12/visits/new
  D->>VC: initNewVisitForm(ownerId=7, petId=12)
  VC->>R: findById(7)
  R->>DB: SELECT owner + pets + visits (EAGER)
  DB-->>R: owner graph
  R-->>VC: Owner(with Pet#12)
  VC->>VC: Visit v = new Visit(); owner.addVisit(12, v)
  D->>V: render createOrUpdateVisitForm.html
  V-->>B: 200 OK (HTML)

  B->>D: POST /owners/7/pets/12/visits/new (form fields)
  D->>VC: processNewVisitForm(ownerId=7, petId=12)
  VC->>BV: validate Visit (description required; date defaults to now if blank)
  alt Validation errors
    D->>V: render createOrUpdateVisitForm.html with errors
    V-->>B: 200 OK
  else Valid
    VC->>R: save(owner)  // cascade persists Visit
    R->>JPA: persist Visit via Pet aggregate
    JPA->>DB: INSERT INTO visits(...)
    DB-->>JPA: generated id
    R-->>VC: Owner
    VC-->>D: redirect:/owners/7 (+flash)
    D-->>B: 302 Found
  end
```

---

## View Owner Details (EAGER graph + error path)

Purpose and trigger:
- Display owner details including pets and visits via eager loading.
- Trigger: GET /owners/{ownerId}

Communication patterns:
- HTTP GET (synchronous)
- @ModelAttribute getOwner throws IllegalArgumentException if not found
- Spring MVC → Repository (in-process)
- JPA read-only transaction; EAGER fetch for pets and visits
- Thymeleaf SSR
- Error handling via Spring Boot error page

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant OC as OwnerController
  participant R as OwnerRepository
  participant JPA as EntityManager/JPA
  participant DB as RDBMS
  participant V as Thymeleaf View
  participant E as BasicErrorController

  B->>D: GET /owners/9999
  D->>OC: getOwner(ownerId=9999)
  OC->>R: findById(9999)
  R->>DB: SELECT owner + pets + visits (EAGER)
  alt Found
    DB-->>R: owner graph
    R-->>OC: Owner
    D->>V: render ownerDetails.html
    V-->>B: 200 OK (HTML)
  else Not found
    DB-->>R: empty
    R-->>OC: Optional.empty
    OC->>OC: throw IllegalArgumentException("Owner not found")
    D->>E: handleError(exception)
    E->>V: render error.html (status 500)
    V-->>B: 500 Internal Server Error (HTML)
  end
  note over D,JPA: spring.jpa.open-in-view=false, EAGER ensures data is loaded before view rendering
```

---

## Locale switch via interceptor

Purpose and trigger:
- Switch UI language for the session using ?lang=xx.
- Trigger: any GET with ?lang=es (etc.)

Communication patterns:
- HTTP GET (synchronous)
- LocaleChangeInterceptor + SessionLocaleResolver
- Thymeleaf SSR with MessageSource

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant I18N as LocaleChangeInterceptor
  participant V as Thymeleaf View

  B->>D: GET /owners/find?lang=es
  D->>I18N: preHandle(lang=es)
  I18N->>I18N: SessionLocaleResolver.setLocale(es)
  I18N-->>D: proceed
  D->>V: render findOwners.html with locale=es
  V-->>B: 200 OK (HTML, messages_es.properties)
```

---

## Crash/error workflow (/oups)

Purpose and trigger:
- Demonstrate error handling view.
- Trigger: GET /oups

Communication patterns:
- HTTP GET (synchronous)
- Controller throws RuntimeException
- Error handling by BasicErrorController + Thymeleaf error template

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant D as DispatcherServlet
  participant CC as CrashController
  participant E as BasicErrorController
  participant V as Thymeleaf View

  B->>D: GET /oups
  D->>CC: triggerException()
  CC->>CC: throw RuntimeException("Expected: controller used to showcase...")

  D->>E: handleError(exception)
  E->>V: render error.html (status 500)
  V-->>B: 500 Internal Server Error (HTML)
```

---

## Actuator readiness/liveness probes (with DB health)

Purpose and trigger:
- Kubernetes liveness/readiness checks via /livez and /readyz when enabled.
- Trigger: GET /readyz (or /livez)

Communication patterns:
- HTTP GET (synchronous)
- Actuator HealthEndpoint
- DataSourceHealthIndicator performs DB ping
- JSON response

```mermaid
sequenceDiagram
  autonumber
  participant K8s as K8s Probe
  participant D as DispatcherServlet (Actuator)
  participant HE as HealthEndpoint
  participant HI as DataSourceHealthIndicator
  participant DB as RDBMS

  K8s->>D: GET /readyz
  D->>HE: health()
  HE->>HI: health()
  HI->>DB: validation query (e.g., SELECT 1)
  DB-->>HI: OK
  HI-->>HE: Status.UP (db: up, details)
  HE-->>D: health JSON
  D-->>K8s: 200 OK (application/json)
```

---

## Vets cache lifecycle (hit/miss and eviction)

Purpose and trigger:
- Illustrate synchronous read-through cache and eventual eviction for vets data.
- Trigger: multiple GET /vets or /vets.html requests

Communication patterns:
- Cache lookup → DB on miss → Cache put; later hit
- Eviction is internal to Caffeine (time/size), no app-level event handling

```mermaid
sequenceDiagram
  autonumber
  participant Client as Client (HTML or JSON)
  participant R as VetRepository
  participant Cache as JCache (Caffeine)
  participant DB as RDBMS

  Client->>R: findAll()/findAll(Pageable)
  R->>Cache: get(key)
  alt Miss
    Cache-->>R: miss
    R->>DB: SELECT vets (+specialties)
    DB-->>R: rows
    R->>Cache: put(key, value)
    R-->>Client: vets
  else Hit
    Cache-->>R: vets
    R-->>Client: vets
  end

  note over Cache: Eviction occurs based on Caffeine policy (size/TTL)\nNo application event handlers are registered
```

---

## Startup database initialization (profiles and service binding)

Purpose and trigger:
- Initialize schema and data on startup based on active profile/database.
- Triggers: Application boot, Spring SQL init

Communication patterns:
- Spring Boot SQL init executes schema.sql and data.sql
- Reads database configuration from properties or service binding files
- Single RDBMS connection

```mermaid
sequenceDiagram
  autonumber
  participant SB as Spring Boot
  participant CFG as Config/Profiles & Service Binding
  participant DB as RDBMS

  SB->>CFG: Resolve datasource via profile (H2/MySQL/Postgres) or SERVICE_BINDING_ROOT
  CFG-->>SB: JDBC URL, user, pass, init locations
  SB->>DB: Connect (JDBC)
  SB->>DB: Execute db/${database}/schema.sql
  DB-->>SB: OK
  SB->>DB: Execute db/${database}/data.sql
  DB-->>SB: OK
  SB-->>SB: ApplicationContext ready
```

---

# Communication Patterns Summary

- Synchronous HTTP request/response for all user interactions.
- In-process synchronous calls from controllers to Spring Data repositories.
- Single relational database; JPA manages transactions (writes are transactional; reads typically read-only).
- Caching: JCache (Caffeine) read-through cache for vets; no explicit invalidation from application code.
- Validation: Bean Validation (JSR 380) and custom PetValidator with domain rules.
- i18n: LocaleChangeInterceptor + SessionLocaleResolver; views use message bundles.
- Error handling: Exceptions bubble to BasicErrorController; error.html rendered with appropriate status.