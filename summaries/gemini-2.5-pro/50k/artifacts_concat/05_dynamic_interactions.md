### 1. Workflow: Find and View an Owner

**Description:** This workflow is triggered when a user searches for a pet owner by their last name. The system queries the database and displays a list of matching owners. If only one owner matches the search criteria, the system redirects the user directly to that owner's detailed information page. This interaction is a synchronous, read-only operation.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP GET request.
*   **Internal:** In-process Java method calls (`Controller` -> `Repository`).
*   **Data Access:** Database transaction (read-only `SELECT` query) via Spring Data JPA.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant OwnerController
    participant OwnerRepository
    participant Database
    participant ThymeleafView

    User->>Browser: Enters last name and submits 'Find Owner' form
    Browser->>OwnerController: GET /owners?lastName=...
    OwnerController->>OwnerRepository: findByLastName(lastName)
    OwnerRepository->>Database: SELECT * FROM owners WHERE last_name LIKE '...'
    Database-->>OwnerRepository: Returns Owner records
    OwnerRepository-->>OwnerController: Returns Page<Owner>

    alt More than one owner found
        OwnerController->>ThymeleafView: render("owners/ownersList", model)
        ThymeleafView-->>OwnerController: Returns rendered HTML
        OwnerController-->>Browser: 200 OK (HTML Page)
        Browser-->>User: Displays list of owners
    else Exactly one owner found
        Note over OwnerController: Extracts owner ID from result
        OwnerController->>Browser: 302 Redirect to /owners/{ownerId}
        Browser->>OwnerController: GET /owners/{ownerId}
        Note over OwnerController: This initiates the "Show Owner Details" flow
        OwnerController->>OwnerRepository: findById(ownerId)
        OwnerRepository->>Database: SELECT * FROM owners WHERE id=...
        Database-->>OwnerRepository: Returns Owner record with Pets and Visits
        OwnerRepository-->>OwnerController: Returns Owner object
        OwnerController->>ThymeleafView: render("owners/ownerDetails", model)
        ThymeleafView-->>OwnerController: Returns rendered HTML
        OwnerController-->>Browser: 200 OK (HTML Page)
        Browser-->>User: Displays owner details
    else No owners found
        OwnerController->>ThymeleafView: render("owners/findOwners", model with error)
        ThymeleafView-->>OwnerController: Returns rendered HTML
        OwnerController-->>Browser: 200 OK (HTML Page)
        Browser-->>User: Displays search form with error message
    end

```

### 2. Workflow: Add a New Owner

**Description:** This workflow allows a user to create a new pet owner. The user submits a form with the owner's details. The system validates the input and, if successful, persists the new owner to the database. Upon success, it uses the Post-Redirect-Get (PRG) pattern to prevent duplicate form submissions.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP POST request.
*   **Internal:** In-process method calls with data binding and validation.
*   **Data Access:** Database transaction (`INSERT` statement).
*   **Pattern:** Post-Redirect-Get (PRG).

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant OwnerController
    participant SpringValidator
    participant OwnerRepository
    participant Database

    User->>Browser: Submits 'Add Owner' form
    Browser->>OwnerController: POST /owners/new (with form data)
    Note over OwnerController: Spring binds form data to Owner object
    OwnerController->>SpringValidator: validate(owner)

    alt Input data is invalid
        SpringValidator-->>OwnerController: Returns validation errors
        OwnerController-->>Browser: 200 OK (HTML with errors)
        Note over Browser: Re-renders the 'Add Owner' form with error messages
        Browser-->>User: Displays form with errors
    else Input data is valid
        SpringValidator-->>OwnerController: Returns success
        OwnerController->>OwnerRepository: save(owner)
        Note over OwnerRepository: A new DB transaction begins
        OwnerRepository->>Database: INSERT INTO owners (...) VALUES (...)
        Database-->>OwnerRepository: Returns new owner ID
        Note over OwnerRepository: Transaction commits
        OwnerRepository-->>OwnerController: Returns saved Owner object
        OwnerController->>Browser: 302 Redirect to /owners/{newId}
        Browser->>User: Browser follows redirect, initiating "View Owner" flow
    end
```

### 3. Workflow: Add a New Pet to an Owner

**Description:** This workflow allows a user to add a new pet to an existing owner's record. The system validates the pet's details (e.g., name, birth date) and ensures the name is unique for that specific owner. The `Owner` entity acts as the aggregate root, and saving the `Owner` cascades the persistence to the new `Pet`.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP POST request.
*   **Internal:** In-process method calls, custom validation logic (`PetValidator`).
*   **Data Access:** Database transaction (`INSERT` into `pets` table, potentially `UPDATE` on `owners`). The operation is cascaded from the `Owner` aggregate.
*   **Pattern:** Post-Redirect-Get (PRG).

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant PetController
    participant OwnerRepository
    participant PetValidator
    participant Database

    User->>Browser: Submits 'Add Pet' form for a specific owner
    Browser->>PetController: POST /owners/{ownerId}/pets/new (with form data)
    
    PetController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = {ownerId}
    Database-->>OwnerRepository: Returns Owner record
    OwnerRepository-->>PetController: Returns Owner object

    Note over PetController: Spring binds form data to new Pet object
    PetController->>PetValidator: validate(pet)
    
    alt Pet data is invalid
        PetValidator-->>PetController: Returns validation errors
        PetController-->>Browser: 200 OK (HTML with Pet form and errors)
        Browser-->>User: Displays form with errors
    else Pet data is valid
        PetValidator-->>PetController: Returns success
        PetController->>OwnerRepository: owner.addPet(pet); save(owner)
        Note over OwnerRepository: A new DB transaction begins
        OwnerRepository->>Database: INSERT INTO pets (name, birth_date, type_id, owner_id) VALUES (...)
        Note over Database: Hibernate cascades the save from the Owner entity
        Database-->>OwnerRepository: Success
        Note over OwnerRepository: Transaction commits
        OwnerRepository-->>PetController: Returns saved Owner
        PetController->>Browser: 302 Redirect to /owners/{ownerId}
        Browser->>User: Browser follows redirect, displaying updated Owner details
    end
```

### 4. Workflow: View Veterinarian List (with Caching)

**Description:** A user or API client requests the list of veterinarians. The system uses a cache to avoid repeated database queries for this relatively static data. The first request fetches data from the database and populates the cache. Subsequent requests are served directly from the cache, improving performance and reducing database load.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP GET request.
*   **Internal:** In-process method calls, AOP-based caching via JCache.
*   **Data Access:** Conditional database transaction (`SELECT` query) only on cache miss.
*   **API:** Supports content negotiation for HTML (web UI) and JSON/XML (data API).

```mermaid
sequenceDiagram
    actor Client
    participant SpringWeb
    participant VetController
    participant CachingProxy
    participant VetRepository
    participant JCacheManager
    participant Database

    Client->>SpringWeb: GET /vets.html (or /vets with Accept: application/json)
    SpringWeb->>VetController: showVetList() or showResourcesVetList()
    VetController->>CachingProxy: VetRepository.findAll()
    
    opt First Request (Cache Miss)
        CachingProxy->>JCacheManager: Check cache "vets"
        JCacheManager-->>CachingProxy: Cache miss
        CachingProxy->>VetRepository: findAll()
        VetRepository->>Database: SELECT * FROM vets JOIN vet_specialties ...
        Database-->>VetRepository: Returns Vet records
        VetRepository-->>CachingProxy: Returns List<Vet>
        CachingProxy->>JCacheManager: Put result into "vets" cache
        JCacheManager-->>CachingProxy: OK
    end

    opt Subsequent Requests (Cache Hit)
        CachingProxy->>JCacheManager: Check cache "vets"
        JCacheManager-->>CachingProxy: Cache hit, returns cached List<Vet>
        Note right of CachingProxy: Database is not contacted!
    end
    
    CachingProxy-->>VetController: Returns List<Vet>
    
    alt Request for HTML view
        Note over VetController: Renders HTML using Thymeleaf
        VetController-->>SpringWeb: HTML content
    else Request for JSON/XML API
        Note over VetController: Serializes Vet list to JSON/XML
        VetController-->>SpringWeb: JSON/XML content
    end

    SpringWeb-->>Client: 200 OK (HTML or JSON/XML response)
```

### 5. Workflow: General Error Handling

**Description:** This workflow demonstrates how the system handles unexpected runtime exceptions. A dedicated controller endpoint is used to trigger an error intentionally. Spring Boot's centralized error handling mechanism catches the exception and renders a user-friendly error page instead of a raw stack trace.

**Communication Patterns:**
*   **Client-Server:** Synchronous HTTP GET request.
*   **Internal:** Exception propagation up the call stack.
*   **Framework:** Handled by Spring Boot's `BasicErrorController` or a custom `@ControllerAdvice`.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant DispatcherServlet
    participant CrashController
    participant SpringErrorHandler

    User->>Browser: Navigates to the error-triggering URL
    Browser->>DispatcherServlet: GET /oups
    DispatcherServlet->>CrashController: triggerException()
    CrashController-->>DispatcherServlet: throws RuntimeException("Oups!")
    
    Note over DispatcherServlet: Exception propagates up
    
    DispatcherServlet->>SpringErrorHandler: Handles uncaught exception
    Note over SpringErrorHandler: Determines error view and status code (500)
    
    SpringErrorHandler->>DispatcherServlet: Forward request to render error view (e.g., /error)
    DispatcherServlet-->>SpringErrorHandler: Renders generic error.html template
    SpringErrorHandler-->>DispatcherServlet: Returns rendered HTML
    DispatcherServlet-->>Browser: 500 Internal Server Error (HTML Page)
    Browser-->>User: Displays user-friendly error page
```