
# Spring PetClinic - Dynamic Interaction Flows and Sequence Diagrams

## 1. Owner Registration Workflow

### Description
This workflow captures the process of registering a new owner in the system. It's triggered when a user accesses the owner creation form and submits the registration details. The flow involves validation, database persistence, and redirects to the owner's detail view.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant OwnerController
    participant OwnerRepository
    participant Database
    participant Validator

    User->>Browser: Navigate to /owners/new
    Browser->>OwnerController: GET /owners/new
    OwnerController->>OwnerController: initCreationForm()
    OwnerController->>Browser: Return owner creation form (HTML)
    Browser->>User: Display registration form
    
    User->>Browser: Fill owner details (name, address, phone)
    User->>Browser: Submit form
    Browser->>OwnerController: POST /owners/new (owner data)
    
    OwnerController->>Validator: validate(owner)
    Validator->>Validator: Check required fields
    Validator->>Validator: Validate telephone format (10 digits)
    alt Validation passes
        Validator-->>OwnerController: Validation successful
        OwnerController->>OwnerRepository: save(owner)
        OwnerRepository->>Database: INSERT INTO owners (...)
        Database-->>OwnerRepository: Owner saved with generated ID
        OwnerRepository-->>OwnerController: Return saved Owner entity
        OwnerController->>OwnerController: "redirect:/owners/{id}"
        OwnerController->>Browser: Redirect to owner details (302)
        Browser->>OwnerController: GET /owners/{id}
        OwnerController->>OwnerRepository: findById(id)
        OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
        Database-->>OwnerRepository: Owner data
        OwnerRepository-->>OwnerController: Return Owner entity
        OwnerController->>Browser: Owner details page (HTML)
        Browser->>User: Display owner details with success message
    else Validation fails
        Validator-->>OwnerController: Validation errors
        OwnerController->>OwnerController: Add error messages to model
        OwnerController->>Browser: Return form with errors (HTML)
        Browser->>User: Display form with validation errors
    end
```

## 2. Pet Management Workflow (Adding a Pet)

### Description
This workflow handles adding a new pet to an existing owner's profile. It demonstrates nested resource management and the parent-child relationship between Owner and Pet entities. The process includes fetching available pet types and validation of pet uniqueness within the owner's collection.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant PetController
    participant OwnerRepository
    participant PetTypeRepository
    participant Cache
    participant Database

    User->>Browser: Navigate to add pet for owner {ownerId}
    Browser->>PetController: GET /owners/{ownerId}/pets/new
    PetController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
    Database-->>OwnerRepository: Owner data
    OwnerRepository-->>PetController: Return Owner entity
    
    PetController->>PetTypeRepository: findPetTypes()
    PetTypeRepository->>Cache: Check cache for pet types
    alt Cache miss
        Cache-->>PetTypeRepository: No cached data
        PetTypeRepository->>Database: SELECT * FROM types
        Database-->>PetTypeRepository: Pet types list
        PetTypeRepository->>Cache: Cache pet types
    else Cache hit
        Cache-->>PetTypeRepository: Return cached pet types
    end
    PetTypeRepository-->>PetController: List of pet types
    
    PetController->>PetController: initCreationForm(owner, petTypes)
    PetController->>Browser: Return pet creation form with owner context
    Browser->>User: Display form with pet type dropdown
    
    User->>Browser: Fill pet details (name, birthDate, type)
    User->>Browser: Submit form
    Browser->>PetController: POST /owners/{ownerId}/pets/new (pet data)
    
    PetController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
    Database-->>OwnerRepository: Owner data
    OwnerRepository-->>PetController: Return Owner entity
    
    PetController->>PetController: Validate pet data
    PetController->>PetController: Check pet name uniqueness for owner
    PetController->>PetController: Validate birth date (not future)
    
    alt Validation passes
        PetController->>PetController: owner.addPet(pet)
        PetController->>OwnerRepository: save(owner)
        OwnerRepository->>Database: BEGIN TRANSACTION
        OwnerRepository->>Database: UPDATE owners ... (if needed)
        OwnerRepository->>Database: INSERT INTO pets (...)
        Database-->>OwnerRepository: Transaction committed
        OwnerRepository-->>PetController: Return updated Owner
        PetController->>PetController: "redirect:/owners/{ownerId}"
        PetController->>Browser: Redirect to owner details (302)
        Browser->>User: Display updated owner with new pet
    else Validation fails
        PetController->>PetTypeRepository: findPetTypes() (for form redisplay)
        PetTypeRepository-->>PetController: Pet types
        PetController->>Browser: Return form with errors
        Browser->>User: Display form with validation messages
    end
```

## 3. Visit Scheduling Workflow

### Description
This workflow captures the process of scheduling a veterinary visit for a pet. It demonstrates the three-level hierarchy (Owner -> Pet -> Visit) and includes validation of visit dates against current date.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VisitController
    participant OwnerRepository
    participant PetRepository
    participant Database

    User->>Browser: Navigate to add visit for pet {petId}
    Browser->>VisitController: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
    Database-->>OwnerRepository: Owner data
    OwnerRepository-->>VisitController: Return Owner entity
    
    VisitController->>PetRepository: findById(petId)
    PetRepository->>Database: SELECT * FROM pets WHERE id = ?
    Database-->>PetRepository: Pet data
    PetRepository-->>VisitController: Return Pet entity
    
    VisitController->>VisitController: initNewVisitForm(owner, pet)
    VisitController->>Browser: Return visit creation form
    Browser->>User: Display form with current date pre-filled
    
    User->>Browser: Fill visit details (date, description)
    User->>Browser: Submit form
    Browser->>VisitController: POST /owners/{ownerId}/pets/{petId}/visits/new
    
    VisitController->>PetRepository: findById(petId)
    PetRepository->>Database: SELECT * FROM pets WHERE id = ?
    Database-->>PetRepository: Pet data
    PetRepository-->>VisitController: Return Pet entity
    
    VisitController->>VisitController: Validate visit date (not future)
    VisitController->>VisitController: Validate description
    
    alt Validation passes
        VisitController->>VisitController: pet.addVisit(visit)
        VisitController->>PetRepository: save(pet)
        PetRepository->>Database: BEGIN TRANSACTION
        PetRepository->>Database: INSERT INTO visits (...)
        Database-->>PetRepository: Visit saved with generated ID
        Database-->>PetRepository: Transaction committed
        PetRepository-->>VisitController: Return updated Pet
        VisitController->>VisitController: "redirect:/owners/{ownerId}"
        VisitController->>Browser: Redirect to owner details (302)
        Browser->>User: Display owner with new visit listed
    else Validation fails
        VisitController->>Browser: Return form with errors
        Browser->>User: Display form with validation messages
    end
```

## 4. Vet Listing with Caching Workflow

### Description
This workflow demonstrates retrieving the veterinarian list with pagination and caching. It showcases the caching strategy using JCache with Caffeine provider, showing both cache hit and miss scenarios.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VetController
    participant VetRepository
    participant Cache
    participant JMX
    participant Database

    User->>Browser: Navigate to /vets.html or /vets
    Browser->>VetController: GET /vets.html?page=0
    
    alt First request (cache miss)
        VetController->>VetRepository: findAll(pageable)
        VetRepository->>Cache: Check cache for vets list
        Cache-->>VetRepository: Cache miss
        VetRepository->>Database: SELECT v.*, s.* FROM vets v 
        VetRepository->>Database: LEFT JOIN vet_specialties vs ON v.id = vs.vet_id
        VetRepository->>Database: LEFT JOIN specialties s ON vs.specialty_id = s.id
        VetRepository->>Database: LIMIT 5 OFFSET 0
        Database-->>VetRepository: Vets with specialties data
        VetRepository->>Cache: Store vets list in cache
        VetRepository->>JMX: Update cache statistics
        VetRepository-->>VetController: Return Page<Vet> object
    else Subsequent request (cache hit)
        VetController->>VetRepository: findAll(pageable)
        VetRepository->>Cache: Check cache for vets list
        Cache-->>VetRepository: Return cached vets list
        VetRepository->>JMX: Update cache hit statistics
        VetRepository-->>VetController: Return cached Page<Vet>
    end
    
    VetController->>VetController: Process vets data
    VetController->>Browser: Return HTML/JSON response
    Browser->>User: Display paginated vet list

    Note over VetRepository,JMX: Cache monitoring via JMX beans
    JMX->>JMX: Track cache hit/miss ratio
    JMX->>JMX: Monitor cache size and eviction
```

## 5. Owner Search Workflow with Pagination

### Description
This workflow handles searching for owners by last name with pagination support. It demonstrates the searchable repository pattern and page navigation functionality.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant OwnerController
    participant OwnerRepository
    participant Database

    User->>Browser: Navigate to /owners/find
    Browser->>OwnerController: GET /owners/find
    OwnerController->>OwnerController: initFindForm()
    OwnerController->>Browser: Return search form
    Browser->>User: Display search form
    
    User->>Browser: Enter last name (e.g., "Dav")
    User->>Browser: Submit search
    Browser->>OwnerController: POST /owners/find (lastName: "Dav")
    
    OwnerController->>OwnerRepository: findByLastNameStartingWith("Dav", pageable)
    OwnerRepository->>Database: SELECT * FROM owners 
    OwnerRepository->>Database: WHERE last_name LIKE 'Dav%'
    OwnerRepository->>Database: ORDER BY last_name, first_name
    OwnerRepository->>Database: LIMIT 5 OFFSET 0
    Database-->>OwnerRepository: Page of owners matching search
    OwnerRepository-->>OwnerController: Return Page<Owner>
    
    alt Multiple owners found
        OwnerController->>Browser: Return owners list page
        Browser->>User: Display paginated search results
        User->>Browser: Click next page (page 1)
        Browser->>OwnerController: GET /owners?page=1
        OwnerController->>OwnerRepository: findByLastNameStartingWith("Dav", pageable(1))
        OwnerRepository->>Database: SELECT * FROM owners 
        OwnerRepository->>Database: WHERE last_name LIKE 'Dav%'
        OwnerRepository->>Database: LIMIT 5 OFFSET 5
        Database-->>OwnerRepository: Page 2 of results
        OwnerRepository-->>OwnerController: Return Page<Owner>
        OwnerController->>Browser: Return second page
    else Single owner found
        OwnerController->>OwnerController: "redirect:/owners/{id}"
        OwnerController->>Browser: Redirect to owner details
    else No owners found
        OwnerController->>Browser: Return "not found" message
    end
```

## 6. Error Handling and Recovery Workflow

### Description
This workflow demonstrates the application's error handling mechanisms, including the crash controller for intentional error demonstration and global exception handling patterns.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Controller
    participant ErrorController
    participant Logger
    participant Monitoring

    User->>Browser: Navigate to /oups (error demo)
    Browser->>ErrorController: GET /oups
    ErrorController->>ErrorController: triggerException()
    ErrorController->>ErrorController: throw new RuntimeException("Expected: controller used to showcase what happens when an exception is thrown")
    
    ErrorController->>ErrorController: Exception propagates up
    ErrorController->>Logger: Log error details
    Logger->>Logger: Write stack trace to logs
    Logger->>Monitoring: Report error metrics
    Monitoring->>Monitoring: Increment error counter
    
    ErrorController->>Browser: Return error page (HTML)
    Browser->>User: Display "Something happened..." error page
    
    Note over ErrorController,Monitoring: Production error handling
    alt Real application error
        User->>Browser: Invalid request (e.g., /owners/999)
        Browser->>Controller: GET /owners/999
        Controller->>Controller: Business logic processing
        Controller->>Controller: Entity not found
        Controller->>Logger: Log warning
        Controller->>Browser: Return 404 or custom error page
        Browser->>User: Display friendly error message
    end
    
    Note over ErrorController,Monitoring: Health check endpoints
    User->>Browser: GET /livez
    Browser->>Controller: Health check
    Controller->>Controller: Check application status
    Controller->>Browser: Return 200 OK
    Browser->>User: Service is alive
    
    User->>Browser: GET /readyz
    Browser->>Controller: Readiness check
    Controller->>Database: Test database connection
    Database-->>Controller: Connection OK
    Controller->>Cache: Test cache access
    Cache-->>Controller: Cache OK
    Controller->>Browser: Return 200 OK
    Browser->>User: Service is ready
```

## 7. Internationalization Workflow

### Description
This workflow demonstrates the internationalization (i18n) support in the application, showing how language switching works and how message bundles are resolved based on session locale.

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Controller
    participant LocaleResolver
    participant MessageSource
    participant ResourceBundle

    User->>Browser: Initial request GET /
    Browser->>Controller: GET / (default locale: en)
    Controller->>LocaleResolver: resolveLocale(request)
    LocaleResolver->>LocaleResolver: Check session for locale
    LocaleResolver->>LocaleResolver: Check Accept-Language header
    LocaleResolver-->>Controller: Return Locale (e.g., en_US)
    
    Controller->>MessageSource: getMessage("welcome", null, locale)
    MessageSource->>ResourceBundle: Check messages_en.properties
    ResourceBundle-->>MessageSource: Return welcome message
    MessageSource-->>Controller: Return localized message
    Controller->>Browser: Return HTML with localized content
    Browser->>User: Display page in English
    
    User->>Browser: Click language link ?lang=de
    Browser->>Controller: GET /?lang=de
    Controller->>LocaleResolver: setLocale(request, response, Locale.GERMAN)
    LocaleResolver->>LocaleResolver: Store locale in session
    Controller->>MessageSource: getMessage("welcome", null, Locale.GERMAN)
    MessageSource->>ResourceBundle: Check messages_de.properties
    ResourceBundle-->>MessageSource: Return German welcome message
    MessageSource-->>Controller: Return localized message
    Controller->>Browser: Return HTML with German content
    Browser->>User: Display page in German
    
    Note over MessageSource,ResourceBundle: Message resolution fallback
    alt Message not found in requested locale
        MessageSource->>ResourceBundle: Check messages_es.properties
        ResourceBundle-->>MessageSource: Message not found
        MessageSource->>ResourceBundle: Fall back to messages.properties (default)
        ResourceBundle-->>MessageSource: Return default message
    end
```

## 8. Database Transaction Management Workflow

### Description
This workflow illustrates transaction boundaries during complex operations involving multiple entity modifications, particularly when updating an owner with multiple pets.

```mermaid
sequenceDiagram
    participant Controller
    participant Service
    participant Repository
    participant TransactionManager
    participant Database

    Controller->>Service: updateOwnerWithPets(ownerDto)
    Service->>TransactionManager: beginTransaction()
    TransactionManager->>Database: BEGIN TRANSACTION
    TransactionManager-->>Service: Transaction started
    
    Service->>Repository: findById(ownerId)
    Repository->>Database: SELECT * FROM owners WHERE id = ?
    Database-->>Repository: Owner data
    Repository-->>Service: Owner entity
    
    Service->>Service: Update owner properties
    Service->>Repository: save(owner)
    Repository->>Database: UPDATE owners SET ...
    Database-->>Repository: Owner updated
    
    loop For each pet to update
        Service->>Repository: findById(petId)
        Repository->>Database: SELECT * FROM pets WHERE id = ?
        Database-->>Repository: Pet data
        Repository-->>Service: Pet entity
        
        Service->>Service: Update pet properties
        Service->>Repository: save(pet)
        Repository->>Database: UPDATE pets SET ...
        Database-->>Repository: Pet updated
    end
    
    alt All operations successful
        Service->>TransactionManager: commitTransaction()
        TransactionManager->>Database: COMMIT
        Database-->>TransactionManager: Transaction committed
        TransactionManager-->>Service: Success
        Service-->>Controller: Update successful
    else Exception occurs
        Service->>Service: Handle exception
        Service->>TransactionManager: rollbackTransaction()
        TransactionManager->>Database: ROLLBACK
        Database-->>TransactionManager: Transaction rolled back
        TransactionManager-->>Service: Rollback complete
        Service-->>Controller: Propagate exception
    end
    
    Note over TransactionManager,Database: Connection pooling
    TransactionManager->>TransactionManager: Borrow connection from pool
    TransactionManager->>TransactionManager: Return connection to pool
```