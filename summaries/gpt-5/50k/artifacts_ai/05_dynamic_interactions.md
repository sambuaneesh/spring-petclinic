## Workflow 1: Owner search and navigation (cardinality-based outcomes)

- Purpose/trigger: User searches owners by last name prefix via /owners?lastName=... with pagination (5/page)
- Communication patterns:
  - Synchronous HTTP (Browser -> Spring MVC)
  - In-process controller -> repository calls
  - Synchronous JDBC via JPA/Hibernate (read-only)
  - Server-side HTML rendering (Thymeleaf)
- Notes:
  - OSIV disabled; EAGER fetch for aggregate display
  - Branching outcomes: 0, 1, or >1 results

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant DS as Spring MVC DispatcherServlet
  participant OC as OwnerController
  participant OR as OwnerRepository
  participant EM as JPA/Hibernate (EntityManager)
  participant DB as Relational DB
  participant TV as Thymeleaf View

  B->>DS: GET /owners?lastName=Sm&page=0
  DS->>OC: dispatch handleFindOwners()
  OC->>OR: findByLastNameStartingWith("Sm", Page(0,5))
  OR->>EM: create and execute query (read-only)
  EM->>DB: SELECT owners WHERE last_name LIKE 'Sm%' LIMIT 5
  DB-->>EM: result rows + count
  EM-->>OR: Page<Owner>

  alt 0 results
    note right of OC: add BindingResult error lastName=notFound
    OC-->>DS: return view "owners/findOwners"
    DS->>TV: render owners/findOwners.html
    TV-->>B: 200 OK (form with error)
  else 1 result
    OC-->>DS: 302 Redirect:/owners/{ownerId}
    DS-->>B: 302 Found Location:/owners/7
    B->>DS: GET /owners/7
    DS->>OC: dispatch showOwner()
    OC->>OR: findById(7)
    OR->>EM: SELECT owner + pets + visits (EAGER)
    EM->>DB: SELECT ... JOIN ...
    DB-->>EM: rows
    EM-->>OR: Owner aggregate
    OC-->>DS: view "owners/ownerDetails"
    DS->>TV: render owners/ownerDetails.html
    TV-->>B: 200 OK (owner details)
  else >1 results
    OC-->>DS: view "owners/ownersList" with Page<Owner>
    DS->>TV: render owners/ownersList.html
    TV-->>B: 200 OK (paginated list)
  end
```

---

## Workflow 2: Create Owner (/owners/new) with validation and redirect

- Purpose/trigger: User creates a new owner via form
- Communication patterns:
  - Synchronous HTTP (GET form, POST submit)
  - Bean Validation + WebDataBinder (disallow "id")
  - Synchronous JDBC write via JPA transaction
  - Redirect + Flash attributes
- Error handling:
  - Field errors return form with BindingResult
  - On success, redirect to details

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant DS as DispatcherServlet
  participant OC as OwnerController
  participant OR as OwnerRepository
  participant EM as JPA/Hibernate
  participant DB as Relational DB
  participant TV as Thymeleaf View
  participant FA as FlashAttributes

  B->>DS: GET /owners/new
  DS->>OC: initCreationForm()
  OC-->>DS: view "owners/createOrUpdateOwnerForm"
  DS->>TV: render form
  TV-->>B: 200 OK

  B->>DS: POST /owners/new (form fields)
  DS->>OC: processCreationForm(@Valid Owner)
  OC->>OC: WebDataBinder.setDisallowedFields("id")
  OC->>OC: Bean Validation (telephone 10 digits, names not blank)
  alt validation errors
    OC-->>DS: return view with BindingResult errors
    DS->>TV: render owners/createOrUpdateOwnerForm.html
    TV-->>B: 200 OK (errors shown)
  else valid
    note over EM: TX begin
    OC->>OR: save(owner)
    OR->>EM: INSERT INTO owners(...) VALUES (...)
    EM->>DB: execute insert
    DB-->>EM: generated id
    note over EM: TX commit
    EM-->>OR: Owner{id}
    OC->>FA: add "New Owner Created"
    OC-->>DS: 302 Redirect:/owners/{id}
    DS-->>B: 302 Found
    B->>DS: GET /owners/{id}
    DS->>OC: showOwner()
    OC->>OR: findById(id)
    OR->>EM: SELECT owner + pets + visits (EAGER)
    EM->>DB: SELECT ...
    DB-->>EM: rows
    EM-->>OR: Owner
    OC-->>DS: view "owners/ownerDetails"
    DS->>TV: render ownerDetails.html (with flash)
    TV-->>B: 200 OK
  end
```

---

## Workflow 3: Add Pet to Owner (validation, formatter, cascade save)

- Purpose/trigger: Owner adds a new pet via /owners/{ownerId}/pets/new
- Communication patterns:
  - Synchronous HTTP
  - PetTypeFormatter.parse() during data binding
  - PetValidator (required fields, future date, duplicate name per owner)
  - Cascade persist via OwnerRepository.save(owner)
- Notes:
  - Pet types loaded via PetTypeRepository
  - Duplicate name check uses owner's in-memory pets collection

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant DS as DispatcherServlet
  participant PC as PetController
  participant OR as OwnerRepository
  participant PTR as PetTypeRepository
  participant F as PetTypeFormatter
  participant PV as PetValidator
  participant EM as JPA/Hibernate
  participant DB as Relational DB
  participant TV as Thymeleaf View
  participant FA as FlashAttributes

  B->>DS: GET /owners/7/pets/new
  DS->>PC: initCreationForm(ownerId=7)
  PC->>OR: findById(7)
  OR->>EM: SELECT owner + pets (EAGER)
  EM->>DB: SELECT...
  DB-->>EM: rows
  EM-->>OR: Owner
  PC->>PC: owner.addPet(new Pet) (placeholder)
  PC->>PTR: findPetTypes() (sorted)
  PTR->>EM: SELECT types ORDER BY name
  EM->>DB: SELECT...
  DB-->>EM: rows
  EM-->>PTR: Collection<PetType>
  PC-->>DS: view "pets/createOrUpdatePetForm" (owner, pet, types)
  DS->>TV: render form
  TV-->>B: 200 OK

  B->>DS: POST /owners/7/pets/new (name, birthDate, type="dog")
  DS->>PC: processCreationForm(ownerId=7, @ModelAttribute Pet)
  PC->>OR: findById(7) (load aggregate)
  OR->>EM: SELECT owner + pets
  EM->>DB: SELECT...
  DB-->>EM: rows
  EM-->>OR: Owner
  PC->>F: parse("dog", locale)
  F->>PTR: findPetTypes()
  PTR->>EM: SELECT types ORDER BY name (may be cached by JPA 2nd-level if enabled)
  EM->>DB: SELECT...
  DB-->>EM: rows
  EM-->>PTR: types
  F-->>PC: PetType{DOG}
  PC->>PV: validate(pet, owner.pets)
  alt validation errors (missing fields, future date, duplicate name)
    PC-->>DS: return view with BindingResult errors
    DS->>TV: render pets/createOrUpdatePetForm.html
    TV-->>B: 200 OK (errors displayed)
  else valid
    note over EM: TX begin
    PC->>PC: owner.addPet(pet) (associate)
    PC->>OR: save(owner)
    OR->>EM: INSERT INTO pets ... (cascade from owner)
    EM->>DB: execute insert(s)
    DB-->>EM: OK
    note over EM: TX commit
    PC->>FA: add "New Pet has been Added"
    PC-->>DS: 302 Redirect:/owners/7
    DS-->>B: 302 Found
    B->>DS: GET /owners/7
    DS->>...: (Owner details as in Workflow 1)
  end
```

---

## Workflow 4: Book a Visit for a Pet (aggregate save via Owner)

- Purpose/trigger: User books a visit for a specific pet via /owners/{ownerId}/pets/{petId}/visits/new
- Communication patterns:
  - Synchronous HTTP
  - Bean Validation on Visit (description required)
  - Aggregate write via owner.addVisit(...) and save(owner)
  - Redirect + Flash
- Notes:
  - Visit date defaults to now; EAGER visits fetch ensures view-ready aggregate

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant DS as DispatcherServlet
  participant VC as VisitController
  participant OR as OwnerRepository
  participant EM as JPA/Hibernate
  participant DB as Relational DB
  participant TV as Thymeleaf View
  participant FA as FlashAttributes

  B->>DS: GET /owners/7/pets/13/visits/new
  DS->>VC: initNewVisitForm(ownerId=7, petId=13)
  VC->>OR: findById(7)
  OR->>EM: SELECT owner + pets + visits (EAGER)
  EM->>DB: SELECT...
  DB-->>EM: rows
  EM-->>OR: Owner
  VC->>VC: create Visit(petId=13), associate to model
  VC-->>DS: view "pets/createOrUpdateVisitForm"
  DS->>TV: render form
  TV-->>B: 200 OK

  B->>DS: POST /owners/7/pets/13/visits/new (description)
  DS->>VC: processNewVisitForm(@Valid Visit)
  VC->>OR: findById(7) (load aggregate)
  OR->>EM: SELECT owner + pets + visits
  EM->>DB: SELECT...
  DB-->>EM: rows
  EM-->>OR: Owner
  alt validation errors (missing description)
    VC-->>DS: return view with BindingResult errors
    DS->>TV: render pets/createOrUpdateVisitForm.html
    TV-->>B: 200 OK
  else valid
    VC->>VC: owner.addVisit(petId=13, visit)
    note over EM: TX begin
    VC->>OR: save(owner)
    OR->>EM: INSERT INTO visits ... (cascade)
    EM->>DB: execute insert
    DB-->>EM: OK
    note over EM: TX commit
    VC->>FA: add "Your visit has been booked"
    VC-->>DS: 302 Redirect:/owners/7
    DS-->>B: 302 Found
    B->>DS: GET /owners/7
    DS->>...: (render owner details)
  end
```

---

## Workflow 5: Edit Owner with path/model ID tampering protection

- Purpose/trigger: User submits edit form for an owner; controller protects against ID mismatch between path and form
- Communication patterns:
  - Synchronous HTTP
  - WebDataBinder disallows "id" binding; explicit path vs model ID check
  - Redirect with flash on mismatch
- Error handling:
  - On mismatch: redirect back to edit with flash error; no write executed

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant DS as DispatcherServlet
  participant OC as OwnerController
  participant OR as OwnerRepository
  participant EM as JPA/Hibernate
  participant FA as FlashAttributes
  participant TV as Thymeleaf View

  B->>DS: POST /owners/7/edit (form contains id=99)
  DS->>OC: processUpdateOwner(ownerId=7, @ModelAttribute Owner)
  OC->>OC: WebDataBinder.setDisallowedFields("id")
  OC->>OC: validate form fields
  alt path/model mismatch detected
    OC->>FA: add error "Owner ID mismatch"
    OC-->>DS: 302 Redirect:/owners/7/edit
    DS-->>B: 302 Found
    B->>DS: GET /owners/7/edit
    DS->>OC: initUpdateForm()
    OC->>OR: findById(7)
    OR->>EM: SELECT owner
    EM->>DB: SELECT...
    DB-->>EM: row
    EM-->>OR: Owner
    OC-->>DS: view "owners/createOrUpdateOwnerForm" (flash error)
    DS->>TV: render form
    TV-->>B: 200 OK
  else IDs aligned
    note over EM: TX begin
    OC->>OR: save(owner)
    OR->>EM: UPDATE owners SET ... WHERE id=7
    EM->>DB: execute update
    DB-->>EM: OK
    note over EM: TX commit
    OC->>FA: add "Owner Values Updated"
    OC-->>DS: 302 Redirect:/owners/7
    DS-->>B: 302 Found
  end
```

---

## Workflow 6: Vets list (HTML) with JCache/Caffeine look-aside caching

- Purpose/trigger: User views vets page /vets.html?page={n}
- Communication patterns:
  - Synchronous HTTP
  - @Cacheable("vets") look-aside cache (JCache -> Caffeine)
  - Synchronous JDBC on cache miss
  - Server-side HTML rendering
- Notes:
  - Cache key includes Pageable; cache stats available via provider/Actuator

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant DS as DispatcherServlet
  participant VC as VetController
  participant VR as VetRepository (@Cacheable "vets")
  participant Cache as JCache (Caffeine)
  participant EM as JPA/Hibernate
  participant DB as Relational DB
  participant TV as Thymeleaf View

  B->>DS: GET /vets.html?page=0
  DS->>VC: showVetList(Page(0,5))
  VC->>VR: findAll(Page(0,5))
  VR->>Cache: get(key=Page(0,5))
  alt cache hit
    Cache-->>VR: Page<Vet>
  else cache miss
    Cache--X VR: miss
    VR->>EM: SELECT vets + specialties (JOIN, read-only)
    EM->>DB: SELECT...
    DB-->>EM: rows
    EM-->>VR: Page<Vet>
    VR->>Cache: put(key, Page<Vet>)
  end
  VC-->>DS: view "vets/vetList" (model attr listVets)
  DS->>TV: render vets/vetList.html
  TV-->>B: 200 OK
```

---

## Workflow 7: Vets API (JSON/XML) with content negotiation and caching

- Purpose/trigger: Client fetches machine-readable vets at /vets with Accept JSON or XML
- Communication patterns:
  - Synchronous HTTP
  - Content negotiation (Jackson JSON, JAXB XML)
  - @Cacheable("vets") look-aside cache
  - No view rendering; direct serialization
- Notes:
  - Vets DTO wraps List<Vet> as property vetList

```mermaid
sequenceDiagram
  autonumber
  participant C as API Client
  participant DS as DispatcherServlet
  participant VC as VetController
  participant VR as VetRepository (@Cacheable "vets")
  participant Cache as JCache (Caffeine)
  participant EM as JPA/Hibernate
  participant DB as Relational DB
  participant Ser as HttpMessageConverters (Jackson/JAXB)

  C->>DS: GET /vets (Accept: application/json or application/xml)
  DS->>VC: showResourcesVetList()
  VC->>VR: findAll()  // unpaged
  VR->>Cache: get(key=UNPAGED)
  alt cache hit
    Cache-->>VR: Collection<Vet>
  else cache miss
    Cache--X VR: miss
    VR->>EM: SELECT vets + specialties
    EM->>DB: SELECT...
    DB-->>EM: rows
    EM-->>VR: Collection<Vet>
    VR->>Cache: put(key, Collection<Vet>)
  end
  VC-->>DS: body: new Vets(vetList)
  alt Accept: application/json
    DS->>Ser: serialize to JSON
    Ser-->>C: 200 OK (application/json)
  else Accept: application/xml
    DS->>Ser: marshal to XML
    Ser-->>C: 200 OK (application/xml)
  end
```

---

## Workflow 8: Error handling and content negotiation (/oups crash demo)

- Purpose/trigger: Demonstrate error handling path via CrashController GET /oups throwing RuntimeException
- Communication patterns:
  - Synchronous HTTP
  - Exception propagation to Spring Boot error handling
  - Content negotiation between JSON and HTML error representations

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser/Client
  participant DS as DispatcherServlet
  participant CC as CrashController
  participant ERR as Error Handling (BasicErrorController)
  participant TV as Thymeleaf Error View

  B->>DS: GET /oups (Accept: application/json or text/html)
  DS->>CC: handleOups()
  CC--X DS: throws RuntimeException
  DS->>ERR: resolveException()
  alt Accept: application/json
    ERR-->>B: 500 JSON {timestamp,status,error,message,path}
  else Accept: text/html
    ERR->>TV: render custom error page
    TV-->>B: 500 HTML error page
  end
```

---

## Workflow 9: Internationalization (i18n) locale switch

- Purpose/trigger: User requests a different locale via ?lang=xx; views render using message bundles
- Communication patterns:
  - Synchronous HTTP
  - LocaleChangeInterceptor updates SessionLocaleResolver
  - Message resolution during view rendering (messages/*.properties)

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant DS as DispatcherServlet
  participant LCI as LocaleChangeInterceptor (param "lang")
  participant SLR as SessionLocaleResolver
  participant Ctrl as Any Controller
  participant TV as Thymeleaf View
  participant MS as MessageSource (messages/*)

  B->>DS: GET /owners/find?lang=de
  DS->>LCI: preHandle()
  LCI->>SLR: setLocale(request, response, "de")
  DS->>Ctrl: handle request (controller logic)
  Ctrl-->>DS: return view name + model
  DS->>TV: render view with locale=de
  TV->>MS: resolve labels #{...} for "de"
  MS-->>TV: localized strings
  TV-->>B: 200 OK (German content)
```

---

## Workflow 10: Static web assets with caching headers

- Purpose/trigger: Browser fetches Bootstrap JS or compiled CSS; server serves static resources with Cache-Control
- Communication patterns:
  - Synchronous HTTP GET for static files
  - No controller; ResourceHttpRequestHandler serves from classpath
  - Cache-Control: max-age=12h

```mermaid
sequenceDiagram
  autonumber
  participant B as Browser
  participant RH as Static Resource Handler
  participant FS as Classpath Resources

  B->>RH: GET /webjars/bootstrap/dist/js/bootstrap.bundle.min.js
  RH->>FS: read resource
  FS-->>RH: bytes
  RH-->>B: 200 OK (+ Cache-Control: max-age=43200)

  B->>RH: GET /resources/css/petclinic.css
  RH->>FS: read resource
  FS-->>RH: bytes
  RH-->>B: 200 OK (+ Cache-Control: max-age=43200)
```

---

Global interaction summary

- Inter-component communication:
  - Controllers -> Repositories: in-process synchronous method calls
  - Repositories -> Database: synchronous JDBC via JPA/Hibernate; transaction boundaries around save/update
  - Caching: look-aside via JCache (Caffeine) for vets; synchronous cache get/put on repository calls
  - Views: Thymeleaf templates rendered server-side for HTML; HttpMessageConverters for JSON/XML
- Event-driven interactions:
  - None; no messaging or domain events emitted in runtime paths shown
- Error handling and recovery:
  - Validation/binding errors return forms with BindingResult
  - ID tampering results in redirect with flash error, no DB write
  - Global errors handled by Spring Boot error controller with content negotiation
  - OSIV disabled; eager associations ensure data for view is loaded within controller/repository scope