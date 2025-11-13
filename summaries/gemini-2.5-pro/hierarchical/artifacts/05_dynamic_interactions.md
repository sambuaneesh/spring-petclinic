### Workflow 1: Find and View a Pet Owner

-   **Workflow Purpose and Trigger**: This workflow is triggered when a clinic staff member needs to find an existing pet owner's record. They initiate a search using the owner's last name. The system either displays a list of matching owners, directly shows the owner's detailed profile if only one is found, or indicates that no owner was found.
-   **Communication Patterns**: The workflow primarily uses synchronous **HTTP GET/POST** requests for user interaction. The backend uses synchronous method calls to the data access layer, which in turn executes **SQL SELECT queries** within a database transaction managed by Spring Data JPA.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant OwnerController
    participant OwnerRepository
    participant Database

    User->>Browser: Navigates to owner search page
    Browser->>OwnerController: GET /owners/find
    OwnerController->>Browser: Renders search form view

    User->>Browser: Enters last name and submits form
    Browser->>OwnerController: POST /owners (lastName="Davis")
    OwnerController->>OwnerRepository: findByLastName("Davis")
    activate OwnerRepository
    OwnerRepository->>Database: SELECT * FROM owners WHERE last_name = 'Davis'
    Database-->>OwnerRepository: Returns List<Owner>
    OwnerRepository-->>OwnerController: Returns results (e.g., a list with one owner)
    deactivate OwnerRepository

    alt Single Owner Found
        OwnerController->>Browser: HTTP 302 Redirect to /owners/{id}
        Browser->>OwnerController: GET /owners/123
        OwnerController->>OwnerRepository: findById(123)
        activate OwnerRepository
        Note right of OwnerRepository: Fetches Owner with associated Pets and Visits
        OwnerRepository->>Database: SELECT * FROM owners, pets, visits WHERE owner_id = 123
        Database-->>OwnerRepository: Returns Owner object graph
        OwnerRepository-->>OwnerController: Returns Owner("George Franklin")
        deactivate OwnerRepository
        OwnerController->>Browser: Renders owner details view with pets and visit history
    else Multiple Owners Found
        OwnerController->>Browser: Renders owners list view with multiple results
    else No Owners Found
        OwnerController->>Browser: Renders search form view with "not found" message
    end

```

### Workflow 2: Add a New Pet to an Existing Owner

-   **Workflow Purpose and Trigger**: This workflow starts when a clinic staff member needs to register a new pet for an existing owner. The user navigates from the owner's detail page to the "Add Pet" form.
-   **Communication Patterns**: This flow involves synchronous **HTTP GET/POST** requests. It includes multiple database reads (**SQL SELECT**) to prepare the form (fetching the owner and pet types) and a transactional database write (**SQL INSERT**) to save the new pet. It also demonstrates server-side validation.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant PetController
    participant OwnerRepository
    participant PetTypeRepository
    participant PetValidator
    participant Database

    User->>Browser: Clicks "Add New Pet" on an owner's detail page
    Browser->>PetController: GET /owners/{ownerId}/pets/new
    
    PetController->>OwnerRepository: findById(ownerId)
    activate OwnerRepository
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = {ownerId}
    Database-->>OwnerRepository: Returns Owner object
    OwnerRepository-->>PetController: Returns Owner object
    deactivate OwnerRepository

    PetController->>PetTypeRepository: findPetTypes()
    activate PetTypeRepository
    PetTypeRepository->>Database: SELECT * FROM types ORDER BY name
    Database-->>PetTypeRepository: Returns List<PetType>
    PetTypeRepository-->>PetController: Returns pet types (e.g., Dog, Cat)
    deactivate PetTypeRepository
    
    PetController->>Browser: Renders "createPet" form with Owner and PetType data

    User->>Browser: Fills out pet details and submits form
    Browser->>PetController: POST /owners/{ownerId}/pets/new (Pet data)

    PetController->>PetValidator: validate(pet)
    activate PetValidator
    
    alt Validation Fails (e.g., name is blank)
        PetValidator-->>PetController: Returns validation errors
        PetController->>OwnerRepository: findById(ownerId)
        PetController->>PetTypeRepository: findPetTypes()
        Note right of PetController: Repopulates form data for re-display
        PetController->>Browser: Renders "createPet" form again with error messages
    else Validation Succeeds
        PetValidator-->>PetController: No errors
        deactivate PetValidator
        
        PetController->>OwnerRepository: findById(ownerId)
        activate OwnerRepository
        OwnerRepository->>Database: SELECT * FROM owners WHERE id = {ownerId}
        Database-->>OwnerRepository: Returns Owner object
        OwnerRepository-->>PetController: Owner object
        deactivate OwnerRepository

        Note over PetController: owner.addPet(newPet)
        
        PetController->>OwnerRepository: save(owner)
        activate OwnerRepository
        OwnerRepository->>Database: BEGIN TRANSACTION
        OwnerRepository->>Database: INSERT INTO pets (...) VALUES (...)
        OwnerRepository->>Database: COMMIT
        Database-->>OwnerRepository: Success
        OwnerRepository-->>PetController: Saved Owner
        deactivate OwnerRepository
        
        PetController->>Browser: HTTP 302 Redirect to /owners/{ownerId}
    end
```

### Workflow 3: Add a Medical Visit for a Pet

-   **Workflow Purpose and Trigger**: A clinic staff member records a new medical visit for a specific pet. This is triggered by clicking an "Add Visit" button on the owner's detail page next to the relevant pet.
-   **Communication Patterns**: The workflow uses synchronous **HTTP GET/POST** calls. Data is fetched using **SQL SELECT** to prepare the form, and a new visit is recorded via a transactional **SQL INSERT**. Server-side validation ensures the visit description is not empty.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant VisitController
    participant OwnerRepository
    participant Database
    
    User->>Browser: Clicks "Add Visit" for a specific pet
    Browser->>VisitController: GET /owners/{ownerId}/pets/{petId}/visits/new
    
    VisitController->>OwnerRepository: findById(ownerId)
    activate OwnerRepository
    Note right of OwnerRepository: Loads Owner and its associated Pets
    OwnerRepository->>Database: SELECT * FROM owners, pets WHERE owner_id = {ownerId}
    Database-->>OwnerRepository: Returns Owner object with Pet list
    OwnerRepository-->>VisitController: Returns Owner
    deactivate OwnerRepository
    
    Note over VisitController: Finds Pet with id={petId} from Owner's pet list
    VisitController->>Browser: Renders "createVisit" form with Pet and Owner info

    User->>Browser: Enters visit date and description, then submits
    Browser->>VisitController: POST /owners/{ownerId}/pets/{petId}/visits/new (Visit data)

    alt Validation Fails (e.g., description is empty)
        VisitController->>OwnerRepository: findById(ownerId)
        Note right of VisitController: Repopulates form data for re-display
        VisitController->>Browser: Renders "createVisit" form with validation errors
    else Validation Succeeds
        VisitController->>OwnerRepository: findById(ownerId)
        activate OwnerRepository
        OwnerRepository->>Database: SELECT * FROM owners, pets WHERE owner_id = {ownerId}
        Database-->>OwnerRepository: Returns Owner with Pet list
        OwnerRepository-->>VisitController: Returns Owner
        deactivate OwnerRepository
        
        Note over VisitController: Finds Pet, then pet.addVisit(newVisit)
        
        VisitController->>OwnerRepository: save(owner)
        activate OwnerRepository
        OwnerRepository->>Database: BEGIN TRANSACTION
        Note right of Database: The new Visit is persisted via<br/>@Cascade from Owner -> Pet -> Visit
        OwnerRepository->>Database: INSERT INTO visits (pet_id, visit_date, description) VALUES (...)
        OwnerRepository->>Database: COMMIT
        Database-->>OwnerRepository: Success
        OwnerRepository-->>VisitController: Saved Owner object
        deactivate OwnerRepository
        
        VisitController->>Browser: HTTP 302 Redirect to /owners/{ownerId}
    end
```

### Workflow 4: View Veterinarian List with Caching

-   **Workflow Purpose and Trigger**: This workflow is initiated when a user wants to see the list of all veterinarians available at the clinic. It is designed for high performance by caching the vet list.
-   **Communication Patterns**: This demonstrates a synchronous **HTTP GET** request enhanced with a **Cache-Aside** pattern. The initial request triggers a **SQL SELECT** query, but subsequent requests are served directly from an in-memory cache, avoiding database access.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant VetController
    participant VetRepository
    participant SpringCacheManager
    participant Database

    User->>Browser: Navigates to Veterinarians page
    Browser->>VetController: GET /vets
    
    VetController->>VetRepository: findAll()
    activate VetRepository
    
    Note over VetRepository: Method is annotated with @Cacheable("vets")
    VetRepository->>SpringCacheManager: Check "vets" cache for result
    
    alt Cache Miss (First Request)
        SpringCacheManager-->>VetRepository: Cache entry not found
        VetRepository->>Database: SELECT * FROM vets JOIN vet_specialties ...
        Database-->>VetRepository: Returns List<Vet>
        
        VetRepository->>SpringCacheManager: Put result into "vets" cache
        SpringCacheManager-->>VetRepository: Acknowledged
        
        VetRepository-->>VetController: Returns List<Vet> from database
    else Cache Hit (Subsequent Requests)
        SpringCacheManager-->>VetRepository: Returns cached List<Vet>
        VetRepository-->>VetController: Returns List<Vet> from cache
    end
    deactivate VetRepository
    
    VetController->>Browser: Renders view with list of veterinarians
```

### Workflow 5: System-wide Error Handling

-   **Workflow Purpose and Trigger**: This flow illustrates how the system gracefully handles unexpected runtime exceptions that can occur during any operation. It's triggered when an unhandled error propagates up from a lower layer.
-   **Communication Patterns**: This is an example of an **AOP (Aspect-Oriented Programming)** cross-cutting concern. A global exception handler (`@ControllerAdvice`) intercepts exceptions from the synchronous request-response flow. The system then responds differently based on the client's `Accept` header (e.g., JSON for API clients, HTML for web browsers).

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SomeController
    participant SomeService_Or_Repository
    participant GlobalExceptionHandler

    User->>Browser: Submits a request that will cause an error
    Browser->>SomeController: POST /some/operation
    
    SomeController->>SomeService_Or_Repository: performCriticalOperation()
    activate SomeService_Or_Repository
    
    Note right of SomeService_Or_Repository: An unexpected error occurs!
    SomeService_Or_Repository-->>SomeController: throws RuntimeException
    deactivate SomeService_Or_Repository
    
    Note over SomeController: Controller does not handle the exception, it propagates up
    
    SomeController->>GlobalExceptionHandler: Spring MVC routes unhandled exception
    activate GlobalExceptionHandler
    
    Note over GlobalExceptionHandler: Intercepts RuntimeException via @ExceptionHandler
    
    alt API Request (Accept: application/json)
        GlobalExceptionHandler-->>Browser: HTTP 500 with JSON error body {"error": "Internal Server Error", ...}
    else Web Browser Request (Accept: text/html)
        GlobalExceptionHandler-->>Browser: Renders a custom "error.html" view
    end
    deactivate GlobalExceptionHandler
```