### Workflow: Find Owner by Last Name

*   **Description**: This workflow is triggered when a user searches for a pet owner using their last name via the "Find owners" form. The system handles three scenarios: no owners found, a single owner found, or multiple owners found.
*   **Communication Patterns**:
    *   **User Interaction**: HTTP Form Submission (GET request).
    *   **Internal Communication**: Synchronous Java method calls within the monolithic application.
    *   **Data Access**: Spring Data JPA repository method call (`findByLastName`).
    *   **Database**: A single SQL SELECT query is executed within an implicit read-only transaction.
    *   **Response**: Server-side rendered HTML view or an HTTP redirect.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Browser
    participant DispatcherServlet as Spring MVC
    participant OwnerController
    participant OwnerRepository
    participant Database

    User->>Browser: Enters last name (e.g., "Davis") and clicks "Find Owner"
    Browser->>DispatcherServlet: GET /owners?lastName=Davis
    DispatcherServlet->>OwnerController: processFindForm(lastName="Davis", model)
    OwnerController->>OwnerRepository: findByLastName("Davis")
    OwnerRepository->>Database: SELECT * FROM owners WHERE last_name = 'Davis'
    Database-->>OwnerRepository: Returns result set
    OwnerRepository-->>OwnerController: Returns Collection<Owner>

    alt Single Owner Found
        OwnerController->>DispatcherServlet: Returns redirect view: "redirect:/owners/{id}"
        DispatcherServlet->>Browser: HTTP 302 Redirect to /owners/12
        Browser->>DispatcherServlet: GET /owners/12
        note right of Browser: This triggers the "View Owner Details" workflow
    else Multiple Owners Found
        OwnerController->>DispatcherServlet: Adds owner list to model, returns view name: "owners/ownersList"
        DispatcherServlet->>Browser: Renders and returns HTML page with list of owners
        Browser->>User: Displays list of owners matching "Davis"
    else No Owner Found
        OwnerController->>DispatcherServlet: Adds error message to model, returns view name: "owners/findOwners"
        DispatcherServlet->>Browser: Renders and returns HTML page with "has not been found" message
        Browser->>User: Displays the find owner form again with an error message
    end
```

### Workflow: Add a New Owner

*   **Description**: This workflow starts when a user decides to add a new owner to the system. It involves displaying a creation form, processing the submitted data, validating it, and persisting the new owner to the database.
*   **Communication Patterns**:
    *   **User Interaction**: HTTP GET to display the form, HTTP POST to submit the new owner data.
    *   **Internal Communication**: Synchronous method calls. The controller interacts directly with the repository.
    *   **Data Access**: Spring Data JPA repository method call (`save`).
    *   **Database**: An `INSERT` statement is executed within a database transaction.
    *   **Validation**: Bean Validation (JSR 303) is applied automatically by Spring MVC before the controller method is executed.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Browser
    participant DispatcherServlet as Spring MVC
    participant OwnerController
    participant OwnerRepository
    participant Database

    User->>Browser: Clicks "Add Owner" link
    Browser->>DispatcherServlet: GET /owners/new
    DispatcherServlet->>OwnerController: initCreationForm(model)
    OwnerController->>DispatcherServlet: Returns view name: "owners/createOrUpdateOwnerForm"
    DispatcherServlet->>Browser: Renders and returns empty owner form as HTML

    User->>Browser: Fills out form and clicks "Add Owner"
    Browser->>DispatcherServlet: POST /owners/new (with owner data in body)
    DispatcherServlet->>OwnerController: processCreationForm(owner, result)
    note over OwnerController: Spring automatically performs data binding and validation.

    alt Validation Fails
        OwnerController->>DispatcherServlet: Returns view name: "owners/createOrUpdateOwnerForm"
        DispatcherServlet->>Browser: Renders and returns the form with validation error messages
        Browser->>User: Displays form with errors
    else Validation Succeeds
        OwnerController->>OwnerRepository: save(owner)
        note right of OwnerController: A database transaction begins here.
        OwnerRepository->>Database: INSERT INTO owners (...) VALUES (...)
        Database-->>OwnerRepository: Returns new owner ID
        OwnerRepository-->>OwnerController: Returns saved Owner object with ID
        note right of OwnerController: The transaction is committed.
        OwnerController->>DispatcherServlet: Returns redirect view: "redirect:/owners/{newId}"
        DispatcherServlet->>Browser: HTTP 302 Redirect to the new owner's page
        Browser->>User: Browser navigates to the new owner's detail page
    end
```

### Workflow: View Veterinarians List (with Caching)

*   **Description**: This workflow is triggered when a user requests the list of veterinarians. It demonstrates a read-through caching pattern. The system first checks a cache for the vet list; if the data isn't in the cache (a "cache miss"), it queries the database and populates the cache for subsequent requests.
*   **Communication Patterns**:
    *   **User Interaction**: HTTP GET request.
    *   **Internal Communication**: Synchronous method calls.
    *   **Caching**: JCache (JSR-107) with Spring's cache abstraction. An aspect intercepts the repository call to manage cache interactions.
    *   **Data Access**: Spring Data JPA repository method call (`findAll`).
    *   **Response**: Server-side rendered HTML. A separate JSON endpoint also exists (`/vets`).

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Browser
    participant DispatcherServlet as Spring MVC
    participant VetController
    participant CacheAspect as Spring Cache Aspect
    participant VetRepository
    participant Database

    User->>Browser: Clicks "Veterinarians" navigation link
    Browser->>DispatcherServlet: GET /vets.html
    DispatcherServlet->>VetController: showVetList(model)
    VetController->>CacheAspect: VetRepository.findAll()
    note right of VetController: Controller calls repository, but Spring's CacheAspect intercepts it.

    CacheAspect->>CacheAspect: Check 'vets' cache for data

    alt Cache Hit
        CacheAspect-->>VetController: Returns cached Collection<Vet>
        note right of CacheAspect: Database is not contacted.
    else Cache Miss
        CacheAspect->>VetRepository: findAll()
        VetRepository->>Database: SELECT * FROM vets
        Database-->>VetRepository: Returns result set
        VetRepository-->>CacheAspect: Returns Collection<Vet>
        CacheAspect->>CacheAspect: Store result in 'vets' cache
        CacheAspect-->>VetController: Returns Collection<Vet>
    end

    VetController->>DispatcherServlet: Adds vet list to model, returns view name: "vets/vetList"
    DispatcherServlet->>Browser: Renders and returns HTML page with list of vets
    Browser->>User: Displays list of veterinarians
```

### Workflow: Add a New Visit for a Pet

*   **Description**: This workflow allows a user to record a new visit for a specific pet. It highlights the strong data coupling within the monolith, as adding a visit requires loading the parent `Owner` entity first and then saving it to trigger cascading persistence for the new `Visit`.
*   **Communication Patterns**:
    *   **User Interaction**: HTTP POST with form data.
    *   **Internal Communication**: Synchronous method calls.
    *   **Data Access**: Spring Data JPA with `@Transactional` behavior. The `save` call on the `OwnerRepository` cascades to the `Visit` entity.
    *   **Database**: An `INSERT` into the `visits` table is executed as part of the transaction that updates the owner-pet-visit aggregate.

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Browser
    participant DispatcherServlet as Spring MVC
    participant VisitController
    participant OwnerRepository
    participant Database

    User->>Browser: Fills out "New Visit" form and clicks "Add Visit"
    Browser->>DispatcherServlet: POST /owners/{ownerId}/pets/{petId}/visits/new (with visit data)
    DispatcherServlet->>VisitController: processNewVisitForm(ownerId, petId, visit, result)

    VisitController->>OwnerRepository: findById(ownerId)
    note right of VisitController: Controller must load the aggregate root (Owner) first.
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
    Database-->>OwnerRepository: Returns Owner data
    OwnerRepository-->>VisitController: Returns Owner object (with its pets already fetched eagerly)
    
    note over VisitController: Controller finds the correct Pet in the Owner's pet list and adds the new Visit to it. Spring performs data binding and validation.

    alt Validation Fails
         VisitController->>DispatcherServlet: Returns view name: "pets/createOrUpdateVisitForm"
         DispatcherServlet->>Browser: Renders and returns form with validation errors
         Browser->>User: Displays form with errors
    else Validation Succeeds
        VisitController->>OwnerRepository: save(owner)
        note right of VisitController: A transaction begins. Saving the Owner cascades the save operation to the new Visit.
        OwnerRepository->>Database: INSERT INTO visits (pet_id, visit_date, description) VALUES (...)
        Database-->>OwnerRepository: Confirms insert
        note right of OwnerRepository: Transaction is committed.
        OwnerRepository-->>VisitController: Returns saved Owner object
        VisitController->>DispatcherServlet: Returns redirect view: "redirect:/owners/{ownerId}"
        DispatcherServlet->>Browser: HTTP 302 Redirect to the owner's detail page
        Browser->>User: Browser navigates to owner page, which now shows the new visit
    end
```

### Workflow: System Error Handling

*   **Description**: This sequence demonstrates the application's generic error handling mechanism. When a controller method throws an unhandled exception, Spring Boot's error handling infrastructure catches it and renders a user-friendly error page. The `/oups` endpoint is designed specifically to trigger this flow.
*   **Communication Patterns**:
    *   **User Interaction**: HTTP GET request to a problematic endpoint.
    *   **Internal Communication**: Synchronous method call leading to a `RuntimeException`.
    *   **Error Handling**: The exception propagates up to the Spring MVC framework, which then invokes its configured error handling logic (e.g., `BasicErrorController`).
    *   **Response**: Server-side rendered HTML error page with an appropriate HTTP status code (e.g., 500 Internal Server Error).

```mermaid
sequenceDiagram
    autonumber
    actor User
    participant Browser
    participant DispatcherServlet as Spring MVC
    participant CrashController
    participant ErrorHandler as Spring Boot Error Handling

    User->>Browser: Navigates to the /oups URL
    Browser->>DispatcherServlet: GET /oups
    DispatcherServlet->>CrashController: triggerException()

    CrashController-->>DispatcherServlet: throws RuntimeException("Expected!")
    note right of CrashController: The controller method throws an unhandled exception.

    DispatcherServlet->>ErrorHandler: Forwards exception for processing
    note over ErrorHandler: Catches the exception, logs it, and determines the appropriate error view and HTTP status code.

    ErrorHandler->>DispatcherServlet: Forwards to error view (e.g., "/error")
    DispatcherServlet->>Browser: Renders and returns the 'error.html' template with HTTP 500 status
    Browser->>User: Displays the generic application error page
```