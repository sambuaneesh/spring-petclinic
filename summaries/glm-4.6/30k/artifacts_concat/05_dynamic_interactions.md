
# Spring PetClinic Dynamic Interaction Flows

## 1. Owner Search and View Workflow

**Purpose:** Users search for owners by last name and view owner details with their pets.  
**Trigger:** User accesses `/owners/find` and submits a search form.  
**Communication Pattern:** Synchronous HTTP request/response with database queries.

```mermaid
sequenceDiagram
    participant User
    participant WebBrowser
    participant OwnerController
    participant OwnerRepository
    participant Database
    participant ThymeleafView

    User->>WebBrowser: GET /owners/find
    WebBrowser->>OwnerController: GET /owners/find
    OwnerController->>ThymeleafView: Return search form view
    ThymeleafView->>WebBrowser: HTML search form
    WebBrowser->>User: Display search form
    
    User->>WebBrowser: Submit search form (last name)
    WebBrowser->>OwnerController: GET /owners?lastName=...
    
    alt Multiple owners found
        OwnerController->>OwnerRepository: findByLastNameStartingWith(lastName, Pageable)
        OwnerRepository->>Database: SELECT * FROM owners WHERE last_name LIKE ? ORDER BY last_name
        Database->>OwnerRepository: Owner entities with pagination
        OwnerRepository->>OwnerController: Page<Owner>
        OwnerController->>ThymeleafView: Pass owners list to view
        ThymeleafView->>WebBrowser: HTML with owners list and pagination
    else Single owner found
        OwnerController->>OwnerRepository: findByLastNameStartingWith(lastName, Pageable)
        OwnerRepository->>Database: SELECT * FROM owners WHERE last_name LIKE ?
        Database->>OwnerRepository: Single Owner entity
        OwnerRepository->>OwnerController: Page<Owner> (single item)
        OwnerController->>OwnerController: Redirect to /owners/{ownerId}
        OwnerController->>WebBrowser: 302 Redirect to /owners/{ownerId}
        WebBrowser->>OwnerController: GET /owners/{ownerId}
        OwnerController->>OwnerRepository: findById(ownerId)
        OwnerRepository->>Database: SELECT * FROM owners WHERE id = ? JOIN pets JOIN visits
        Database->>OwnerRepository: Owner with associated pets and visits
        OwnerRepository->>OwnerController: Owner entity
        OwnerController->>ThymeleafView: Pass owner details to view
        ThymeleafView->>WebBrowser: HTML with owner details, pets, and visits
    end
    
    WebBrowser->>User: Display search results or owner details
```

## 2. Owner Creation Workflow

**Purpose:** Register a new owner in the system with validation.  
**Trigger:** User submits the new owner form.  
**Communication Pattern:** Synchronous HTTP POST with database transaction and validation.

```mermaid
sequenceDiagram
    participant User
    participant WebBrowser
    participant OwnerController
    participant OwnerRepository
    participant Database
    participant Validator
    participant ThymeleafView

    User->>WebBrowser: GET /owners/new
    WebBrowser->>OwnerController: GET /owners/new
    OwnerController->>ThymeleafView: Return new owner form view
    ThymeleafView->>WebBrowser: HTML form for new owner
    WebBrowser->>User: Display form
    
    User->>WebBrowser: Submit owner details
    WebBrowser->>OwnerController: POST /owners/new (form data)
    
    OwnerController->>Validator: Validate owner data
    Validator->>OwnerController: Validation result
    
    alt Validation fails
        OwnerController->>ThymeleafView: Return form with errors
        ThymeleafView->>WebBrowser: HTML form with error messages
        WebBrowser->>User: Display validation errors
    else Validation passes
        OwnerController->>OwnerRepository: save(owner)
        OwnerRepository->>Database: BEGIN TRANSACTION
        OwnerRepository->>Database: INSERT INTO owners (first_name, last_name, address, city, telephone)
        Database->>OwnerRepository: Generated owner ID
        OwnerRepository->>Database: COMMIT
        OwnerRepository->>OwnerController: Saved Owner entity with ID
        OwnerController->>WebBrowser: 302 Redirect to /owners/{ownerId}
        WebBrowser->>OwnerController: GET /owners/{ownerId}
        OwnerController->>OwnerRepository: findById(ownerId)
        OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
        Database->>OwnerRepository: Owner entity
        OwnerRepository->>OwnerController: Owner entity
        OwnerController->>ThymeleafView: Pass owner to view
        ThymeleafView->>WebBrowser: HTML showing created owner
    end
    
    WebBrowser->>User: Display created owner or form with errors
```

## 3. Pet Addition Workflow

**Purpose:** Add a new pet to an existing owner with type and birth date validation.  
**Trigger:** User accesses pet creation form from owner details page.  
**Communication Pattern:** Synchronous operations with custom validation and type conversion.

```mermaid
sequenceDiagram
    participant User
    participant WebBrowser
    participant PetController
    participant OwnerRepository
    participant PetRepository
    participant PetValidator
    participant PetTypeFormatter
    participant Database
    participant ThymeleafView

    User->>WebBrowser: GET /owners/{ownerId}/pets/new
    WebBrowser->>PetController: GET /owners/{ownerId}/pets/new
    PetController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
    Database->>OwnerRepository: Owner entity
    OwnerRepository->>PetController: Owner entity
    PetController->>PetTypeFormatter: Convert PetTypes to strings
    PetController->>ThymeleafView: Pass owner and pet types to view
    ThymeleafView->>WebBrowser: HTML pet form with owner and type options
    WebBrowser->>User: Display pet creation form
    
    User->>WebBrowser: Submit pet details
    WebBrowser->>PetController: POST /owners/{ownerId}/pets/new
    
    PetController->>PetTypeFormatter: Convert string to PetType
    PetTypeFormatter->>PetController: PetType entity
    
    PetController->>PetValidator: Validate pet data
    PetValidator->>PetController: Validation result
    
    alt Validation fails
        PetController->>ThymeleafView: Return form with errors
        ThymeleafView->>WebBrowser: HTML form with validation errors
        WebBrowser->>User: Display form with errors
    else Validation passes
        PetController->>OwnerRepository: findById(ownerId)
        OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
        Database->>OwnerRepository: Owner entity
        OwnerRepository->>PetController: Owner entity
        PetController->>Pet: Create new Pet with owner
        PetController->>PetRepository: save(pet)
        PetRepository->>Database: BEGIN TRANSACTION
        PetRepository->>Database: INSERT INTO pets (name, birth_date, type_id, owner_id)
        Database->>PetRepository: Generated pet ID
        PetRepository->>Database: COMMIT
        PetRepository->>PetController: Saved Pet entity
        PetController->>WebBrowser: 302 Redirect to /owners/{ownerId}
    end
    
    WebBrowser->>User: Display updated owner page or form with errors
```

## 4. Visit Scheduling Workflow

**Purpose:** Create a new veterinary visit for a pet with date and description.  
**Trigger:** User schedules a visit from pet details page.  
**Communication Pattern:** Synchronous with default date handling and database persistence.

```mermaid
sequenceDiagram
    participant User
    participant WebBrowser
    participant VisitController
    participant OwnerRepository
    participant PetRepository
    participant VisitRepository
    participant Database
    participant ThymeleafView

    User->>WebBrowser: GET /owners/{ownerId}/pets/{petId}/visits/new
    WebBrowser->>VisitController: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ?
    Database->>OwnerRepository: Owner entity
    OwnerRepository->>VisitController: Owner entity
    VisitController->>PetRepository: findById(petId)
    PetRepository->>Database: SELECT * FROM pets WHERE id = ?
    Database->>PetRepository: Pet entity
    PetRepository->>VisitController: Pet entity
    VisitController->>ThymeleafView: Pass owner and pet to view
    ThymeleafView->>WebBrowser: HTML visit form with pet info
    WebBrowser->>User: Display visit creation form
    
    User->>WebBrowser: Submit visit details (date, description)
    WebBrowser->>VisitController: POST /owners/{ownerId}/pets/{petId}/visits/new
    
    alt Visit date not provided
        VisitController->>VisitController: Set date to current date
    end
    
    VisitController->>Visit: Create new Visit with pet
    VisitController->>VisitRepository: save(visit)
    VisitRepository->>Database: BEGIN TRANSACTION
    VisitRepository->>Database: INSERT INTO visits (pet_id, visit_date, description)
    Database->>VisitRepository: Generated visit ID
    VisitRepository->>Database: COMMIT
    VisitRepository->>VisitController: Saved Visit entity
    VisitController->>WebBrowser: 302 Redirect to /owners/{ownerId}
    
    WebBrowser->>VisitController: GET /owners/{ownerId}
    VisitController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ? JOIN pets JOIN visits
    Database->>OwnerRepository: Owner with updated pets and visits
    OwnerRepository->>VisitController: Owner entity
    VisitController->>ThymeleafView: Pass owner to view
    ThymeleafView->>WebBrowser: HTML with new visit visible
    WebBrowser->>User: Display updated owner page with new visit
```

## 5. Veterinarian Listing Workflow with Caching

**Purpose:** Display list of veterinarians with their specialties, using cache for performance.  
**Trigger:** User accesses vets listing page.  
**Communication Pattern:** Synchronous with cache layer for performance optimization.

```mermaid
sequenceDiagram
    participant User
    participant WebBrowser
    participant VetController
    participant VetRepository
    participant Cache("vets cache")
    participant Database
    participant ThymeleafView

    User->>WebBrowser: GET /vets.html
    WebBrowser->>VetController: GET /vets.html
    
    VetController->>VetRepository: findAll()
    
    alt Cache hit
        VetRepository->>Cache: get("vets")
        Cache->>VetRepository: Cached Vet list
    else Cache miss
        VetRepository->>Database: SELECT v.*, s.name as specialty FROM vets v LEFT JOIN vet_specialties vs ON v.id = vs.vet_id LEFT JOIN specialties s ON vs.specialty_id = s.id ORDER BY v.last_name
        Database->>VetRepository: Vet entities with specialties
        VetRepository->>Cache: put("vets", vetList)
    end
    
    VetRepository->>VetController: List<Vet>
    VetController->>VetController: Apply pagination (5 per page)
    VetController->>ThymeleafView: Pass paged vets to view
    ThymeleafView->>WebBrowser: HTML with vet list and pagination
    WebBrowser->>User: Display paginated veterinarian listing
    
    Note over VetController,Cache: Cache invalidation not implemented (cache expires via policy)
```

## 6. Error Handling Workflow

**Purpose:** Handle exceptions and display appropriate error pages.  
**Trigger:** Application throws an exception or error occurs.  
**Communication Pattern:** Exception propagation with global error handling.

```mermaid
sequenceDiagram
    participant User
    participant WebBrowser
    participant Controller
    participant Service/Repository
    participant Database
    participant GlobalExceptionHandler
    participant ThymeleafView

    User->>WebBrowser: Request endpoint
    WebBrowser->>Controller: HTTP request
    
    Controller->>Service/Repository: Business operation
    Service/Repository->>Database: Data access
    
    alt Database error
        Database->>Service/Repository: DataAccessException
        Service/Repository->>Controller: Runtime Exception
        Controller->>GlobalExceptionHandler: Unhandled exception
        GlobalExceptionHandler->>ThymeleafView: error.html with exception details
        ThymeleafView->>WebBrowser: 500 Internal Server Error page
    else Validation error
        Controller->>Controller: BindingResult has errors
        Controller->>ThymeleafView: Return to form with error messages
        ThymeleafView->>WebBrowser: 400 Bad Request with form
    else Resource not found
        Controller->>Service/Repository: findById(nonExistentId)
        Service/Repository->>GlobalExceptionHandler: ObjectRetrievalFailureException
        GlobalExceptionHandler->>ThymeleafView: error.html
        ThymeleafView->>WebBrowser: 404 Not Found page
    else Forced crash (/oups)
        Controller->>Controller: Throw new RuntimeException("Expected: controller used to showcase...")
        Controller->>GlobalExceptionHandler: Exception
        GlobalExceptionHandler->>ThymeleafView: error.html with custom message
        ThymeleafView->>WebBrowser: 500 Error page
    end
    
    WebBrowser->>User: Display error page or form with errors
```

## 7. Language Switching Workflow

**Purpose:** Change application language for internationalization support.  
**Trigger:** User clicks language link with ?lang= parameter.  
**Communication Pattern:** Synchronous with session-based locale storage.

```mermaid
sequenceDiagram
    participant User
    participant WebBrowser
    participant LocaleChangeInterceptor
    participant Controller
    participant Session
    participant MessageSource
    participant ThymeleafView

    User->>WebBrowser: Click language link ?lang=de
    WebBrowser->>LocaleChangeInterceptor: Intercept request with lang param
    LocaleChangeInterceptor->>Session: setLocale(new Locale("de"))
    Session->>LocaleChangeInterceptor: Locale stored
    LocaleChangeInterceptor->>Controller: Continue with new locale
    Controller->>ThymeleafView: Process view with German locale
    
    Note over ThymeleafView,MessageSource: Thymeleaf uses #{} expressions to resolve messages
    
    ThymeleafView->>MessageSource: getMessage("key", null, Locale.GERMAN)
    MessageSource->>ThymeleafView: German message text
    ThymeleafView->>WebBrowser: HTML with German text
    WebBrowser->>User: Display localized page
    
    User->>WebBrowser: Navigate to another page
    WebBrowser->>LocaleChangeInterceptor: Check session locale
    LocaleChangeInterceptor->>Session: getLocale()
    Session->>LocaleChangeInterceptor: Locale.GERMAN
    LocaleChangeInterceptor->>Controller: Continue with German locale
    Controller->>ThymeleafView: Process with German locale
    ThymeleafView->>WebBrowser: HTML with German text
    WebBrowser->>User: Display page in German
    
    Note over Session: Locale persists for the entire session
```