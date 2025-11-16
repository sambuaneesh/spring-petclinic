```markdown
# Spring PetClinic Dynamic Interaction Flows

## 1. Owner Registration and Management Workflow

### Workflow Description
**Purpose**: Complete lifecycle management of pet owners including creation, search, and profile updates
**Triggers**: User accesses owner management features through web interface
**Communication Patterns**: Synchronous REST calls, form submissions, database transactions

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant OwnerCtrl as OwnerController
    participant OwnerRepo as OwnerRepository
    participant DB as Database
    participant Validator as Bean Validator

    User->>Browser: Navigate to /owners/new
    Browser->>OwnerCtrl: GET /owners/new
    OwnerCtrl->>Browser: Return owner creation form
    Browser->>User: Display owner form
    
    User->>Browser: Fill form & submit
    Browser->>OwnerCtrl: POST /owners/new (form data)
    
    OwnerCtrl->>Validator: Validate owner data
    Validator-->>OwnerCtrl: Validation result
    
    alt Validation Failed
        OwnerCtrl->>Browser: Return form with errors
        Browser->>User: Show validation errors
    else Validation Passed
        OwnerCtrl->>OwnerRepo: save(owner)
        OwnerRepo->>DB: INSERT INTO owners
        DB-->>OwnerRepo: Owner record created
        OwnerRepo-->>OwnerCtrl: Saved owner entity
        OwnerCtrl->>Browser: Redirect to /owners/{ownerId}
        Browser->>OwnerCtrl: GET /owners/{ownerId}
        OwnerCtrl->>OwnerRepo: findById(ownerId)
        OwnerRepo->>DB: SELECT owner + pets + visits
        DB-->>OwnerRepo: Owner data with relations
        OwnerRepo-->>OwnerCtrl: Owner entity
        OwnerCtrl->>Browser: Render owner details page
        Browser->>User: Display owner profile
    end
    
    Note over User,Browser: Owner Search Flow
    User->>Browser: Navigate to /owners/find
    Browser->>OwnerCtrl: GET /owners/find
    OwnerCtrl->>Browser: Return search form
    Browser->>User: Display search form
    
    User->>Browser: Enter last name & search
    Browser->>OwnerCtrl: GET /owners?lastName=Smith
    OwnerCtrl->>OwnerRepo: findByLastNameContaining("Smith")
    OwnerRepo->>DB: SELECT owners WHERE last_name LIKE '%Smith%'
    DB-->>OwnerRepo: Matching owner records
    OwnerRepo-->>OwnerCtrl: List<Owner> results
    OwnerCtrl->>Browser: Render search results
    Browser->>User: Display matching owners
```

## 2. Pet Registration and Management Workflow

### Workflow Description
**Purpose**: Register new pets for existing owners and manage pet information
**Triggers**: Owner wants to add a new pet or update existing pet details
**Communication Patterns**: Form submissions with custom validation, database transactions with cascading operations

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant PetCtrl as PetController
    participant OwnerRepo as OwnerRepository
    participant PetRepo as PetRepository
    participant PetTypeRepo as PetTypeRepository
    participant PetValidator as PetValidator
    participant DB as Database

    User->>Browser: Navigate to owner profile
    Browser->>PetCtrl: GET /owners/{ownerId}
    PetCtrl->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT owner + pets
    DB-->>OwnerRepo: Owner with pet relations
    OwnerRepo-->>PetCtrl: Owner entity
    PetCtrl->>Browser: Render owner details
    Browser->>User: Display owner with pets
    
    User->>Browser: Click "Add New Pet"
    Browser->>PetCtrl: GET /owners/{ownerId}/pets/new
    PetCtrl->>PetTypeRepo: findPetTypes()
    PetTypeRepo->>DB: SELECT * FROM types
    DB-->>PetTypeRepo: All pet types
    PetTypeRepo-->>PetCtrl: List<PetType>
    PetCtrl->>Browser: Return pet creation form
    Browser->>User: Display pet form with type dropdown
    
    User->>Browser: Fill pet form & submit
    Browser->>PetCtrl: POST /owners/{ownerId}/pets/new
    
    PetCtrl->>PetValidator: validate(pet, errors)
    PetValidator->>PetValidator: Check name, birth date, type
    PetValidator-->>PetCtrl: Validation result
    
    alt Validation Failed
        PetCtrl->>PetTypeRepo: findPetTypes()
        PetTypeRepo-->>PetCtrl: List<PetType>
        PetCtrl->>Browser: Return form with errors
        Browser->>User: Show validation errors
    else Validation Passed
        PetCtrl->>OwnerRepo: findById(ownerId)
        OwnerRepo->>DB: SELECT owner
        DB-->>OwnerRepo: Owner record
        OwnerRepo-->>PetCtrl: Owner entity
        
        PetCtrl->>PetRepo: save(pet)
        PetRepo->>DB: INSERT INTO pets
        DB-->>PetRepo: Pet record created
        PetRepo-->>PetCtrl: Saved pet entity
        
        PetCtrl->>Browser: Redirect to /owners/{ownerId}
        Browser->>User: Show updated owner profile
    end
```

## 3. Veterinary Visit Scheduling Workflow

### Workflow Description
**Purpose**: Schedule and record veterinary visits for pets with automatic date assignment
**Triggers**: Owner schedules a new veterinary appointment for their pet
**Communication Patterns**: Form processing with automatic date population, database transactions with entity relationships

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant VisitCtrl as VisitController
    participant OwnerRepo as OwnerRepository
    participant PetRepo as PetRepository
    participant VisitRepo as VisitRepository
    participant DB as Database
    participant Clock as System Clock

    User->>Browser: Navigate to pet details
    Browser->>VisitCtrl: GET /owners/{ownerId}/pets/{petId}
    VisitCtrl->>PetRepo: findById(petId)
    PetRepo->>DB: SELECT pet + owner + visits
    DB-->>PetRepo: Pet with relations
    PetRepo-->>VisitCtrl: Pet entity
    VisitCtrl->>Browser: Render pet details
    Browser->>User: Display pet with visit history
    
    User->>Browser: Click "Add Visit"
    Browser->>VisitCtrl: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitCtrl->>Browser: Return visit creation form
    Browser->>User: Display visit form
    
    User->>Browser: Enter description & submit
    Browser->>VisitCtrl: POST /owners/{ownerId}/pets/{petId}/visits/new
    
    VisitCtrl->>Clock: getCurrentDate()
    Clock-->>VisitCtrl: Current date
    
    VisitCtrl->>PetRepo: findById(petId)
    PetRepo->>DB: SELECT pet
    DB-->>PetRepo: Pet record
    PetRepo-->>VisitCtrl: Pet entity
    
    VisitCtrl->>VisitRepo: save(visit)
    VisitRepo->>DB: INSERT INTO visits (date, description, pet_id)
    DB-->>VisitRepo: Visit record created
    VisitRepo-->>VisitCtrl: Saved visit entity
    
    VisitCtrl->>Browser: Redirect to /owners/{ownerId}
    Browser->>VisitCtrl: GET /owners/{ownerId}
    VisitCtrl->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT owner + pets + visits
    DB-->>OwnerRepo: Owner with all relations
    OwnerRepo-->>VisitCtrl: Owner entity
    VisitCtrl->>Browser: Render owner details
    Browser->>User: Display owner profile with new visit
```

## 4. Veterinarian Information Retrieval Workflow

### Workflow Description
**Purpose**: Display veterinarian lists with specialties in multiple formats with caching
**Triggers**: User requests veterinarian information through web interface or API
**Communication Patterns**: Cached repository calls, multi-format response handling, paginated results

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant API as API Client
    participant VetCtrl as VetController
    participant VetRepo as VetRepository
    participant Cache as Cache Manager
    participant DB as Database

    Note over User,Browser: HTML View with Pagination
    User->>Browser: Navigate to /vets.html?page=0
    Browser->>VetCtrl: GET /vets.html?page=0
    VetCtrl->>VetRepo: findAll(PageRequest.of(0, 5))
    
    VetRepo->>Cache: Check cache for "vets"
    alt Cache Hit
        Cache-->>VetRepo: Cached vet list
    else Cache Miss
        VetRepo->>DB: SELECT vets + specialties (JOIN)
        DB-->>VetRepo: Vet records with specialties
        VetRepo->>Cache: Store in cache "vets"
        Cache-->>VetRepo: Cache updated
    end
    
    VetRepo-->>VetCtrl: Page<Vet> (5 vets)
    VetCtrl->>Browser: Render vets HTML template
    Browser->>User: Display paginated vet list
    
    Note over API,VetCtrl: JSON/XML API Response
    API->>VetCtrl: GET /vets (Accept: application/json)
    VetCtrl->>VetRepo: findAll()
    
    VetRepo->>Cache: Check cache for "vets"
    alt Cache Hit
        Cache-->>VetRepo: Cached vet list
    else Cache Miss
        VetRepo->>DB: SELECT vets + specialties
        DB-->>VetRepo: All vet records
        VetRepo->>Cache: Store in cache "vets"
    end
    
    VetRepo-->>VetCtrl: List<Vet> all vets
    VetCtrl->>VetCtrl: Convert to Vets wrapper
    VetCtrl->>API: Return JSON response
    API->>API: Process veterinarian data
```

## 5. Error Handling and Recovery Workflow

### Workflow Description
**Purpose**: Handle system errors, validation failures, and provide graceful degradation
**Triggers**: System exceptions, validation errors, or simulated errors via /oups endpoint
**Communication Patterns**: Exception handling, error page rendering, validation feedback

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant Controller as Any Controller
    participant Validator as Bean Validator
    participant Repo as Repository
    participant DB as Database
    participant ErrorHandler as Spring Error Handler

    Note over User,Browser: Validation Error Flow
    User->>Browser: Submit invalid form data
    Browser->>Controller: POST with invalid data
    Controller->>Validator: validate(entity)
    Validator-->>Controller: BindingResult with errors
    Controller->>Controller: hasErrors() = true
    Controller->>Browser: Return form with error messages
    Browser->>User: Display form with field errors
    
    Note over Controller,ErrorHandler: Database Error Flow
    User->>Browser: Request data
    Browser->>Controller: GET endpoint
    Controller->>Repo: databaseOperation()
    Repo->>DB: SQL operation
    DB-->>Repo: DatabaseException
    Repo-->>Controller: DataAccessException
    
    Controller->>ErrorHandler: throw DataAccessException
    ErrorHandler->>ErrorHandler: Log exception
    ErrorHandler->>Browser: Return error page (500)
    Browser->>User: Display friendly error message
    
    Note over User,Browser: Simulated Error Flow
    User->>Browser: Navigate to /oups
    Browser->>Controller: GET /oups
    Controller->>Controller: throw RuntimeException
    Controller->>ErrorHandler: Unhandled exception
    ErrorHandler->>ErrorHandler: Log stack trace
    ErrorHandler->>Browser: Return error page
    Browser->>User: Display "Something happened..." message
    
    Note over Controller,DB: Transaction Recovery
    Controller->>Repo: @Transactional operation
    Repo->>DB: BEGIN TRANSACTION
    Repo->>DB: Multiple SQL operations
    DB-->>Repo: ConstraintViolationException
    Repo-->>Controller: Transaction rolled back
    Controller->>Browser: Return error response
    Browser->>User: Display transaction failed message
```

## 6. Data Flow During Owner-Pet-Visit Relationship Management

### Workflow Description
**Purpose**: Demonstrate complex data flow when managing interrelated owner, pet, and visit entities
**Triggers**: Complete lifecycle operations involving multiple entity types
**Communication Patterns**: Eager loading of relationships, cascading operations, complex database queries

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant OwnerCtrl as OwnerController
    participant OwnerRepo as OwnerRepository
    participant PetRepo as PetRepository
    participant VisitRepo as VisitRepository
    participant DB as Database

    User->>Browser: View owner details with pets and visits
    Browser->>OwnerCtrl: GET /owners/{ownerId}
    
    OwnerCtrl->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT owners WHERE id=?
    DB-->>OwnerRepo: Owner record
    OwnerRepo->>DB: SELECT pets WHERE owner_id=? (EAGER fetch)
    DB-->>OwnerRepo: Pet records for owner
    OwnerRepo->>DB: SELECT visits WHERE pet_id IN (?) (EAGER fetch)
    DB-->>OwnerRepo: Visit records for pets
    OwnerRepo-->>OwnerCtrl: Owner with pets and visits
    
    OwnerCtrl->>Browser: Render owner details template
    Browser->>User: Display complete owner profile
    
    Note over User,DB: Adding New Visit with Full Context
    User->>Browser: Add visit for specific pet
    Browser->>OwnerCtrl: GET /owners/{ownerId}/pets/{petId}/visits/new
    OwnerCtrl->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: SELECT owner
    DB-->>OwnerRepo: Owner record
    OwnerRepo-->>OwnerCtrl: Owner entity
    
    OwnerCtrl->>PetRepo: findById(petId)
    PetRepo->>DB: SELECT pet
    DB-->>PetRepo: Pet record
    PetRepo-->>OwnerCtrl: Pet entity
    
    OwnerCtrl->>Browser: Return visit form with context
    Browser->>User: Display visit form with owner/pet info
    
    User->>Browser: Submit visit form
    Browser->>OwnerCtrl: POST /owners/{ownerId}/pets/{petId}/visits/new
    
    OwnerCtrl->>VisitRepo: save(visit)
    VisitRepo->>DB: INSERT INTO visits (pet_id, date, description)
    DB-->>VisitRepo: Visit record created
    VisitRepo-->>OwnerCtrl: Saved visit
    
    OwnerCtrl->>Browser: Redirect to owner details
    Browser->>OwnerCtrl: GET /owners/{ownerId}
    
    OwnerCtrl->>OwnerRepo: findById(ownerId)
    OwnerRepo->>DB: Complex JOIN: owners + pets + visits
    DB-->>OwnerRepo: Complete owner hierarchy
    OwnerRepo-->>OwnerCtrl: Owner with updated visits
    OwnerCtrl->>Browser: Render updated profile
    Browser->>User: Display owner with new visit
```

## 7. Internationalization and Localization Flow

### Workflow Description
**Purpose**: Handle multi-language support with session-based locale resolution
**Triggers**: User changes language preference or accesses application with locale headers
**Communication Patterns**: Session management, message bundle resolution, template localization

### Sequence Diagram
```mermaid
sequenceDiagram
    participant User as Web User
    participant Browser as Web Browser
    participant WebConfig as WebConfiguration
    participant LocaleResolver as SessionLocaleResolver
    participant MessageSource as ResourceBundleMessageSource
    participant Template as Thymeleaf Template
    participant Controller as Any Controller

    User->>Browser: Access application first time
    Browser->>Controller: GET / (no locale)
    Controller->>LocaleResolver: resolveLocale(request)
    LocaleResolver-->>Controller: Default locale (en)
    Controller->>MessageSource: getMessage(key, null, locale)
    MessageSource-->>Controller: English message
    Controller->>Template: Render with English messages
    Template->>Browser: HTML with English content
    Browser->>User: Display English interface
    
    User->>Browser: Change language to German (?lang=de)
    Browser->>Controller: GET /?lang=de
    Controller->>LocaleResolver: setLocale(request, response, "de")
    LocaleResolver->>LocaleResolver: Store "de" in session
    LocaleResolver-->>Controller: German locale
    
    Controller->>MessageSource: getMessage(key, null, "de")
    MessageSource->>MessageSource: Load messages_de.properties
    MessageSource-->>Controller: German message
    Controller->>Template: Render with German messages
    Template->>Browser: HTML with German content
    Browser->>User: Display German interface
    
    Note over User,Controller: Subsequent Requests with Session
    User->>Browser: Navigate to another page
    Browser->>Controller: GET /owners (with session cookie)
    Controller->>LocaleResolver: resolveLocale(request)
    LocaleResolver->>LocaleResolver: Retrieve "de" from session
    LocaleResolver-->>Controller: German locale
    Controller->>MessageSource: getMessage(key, null, "de")
    MessageSource-->>Controller: German message
    Controller->>Template: Render with German messages
    Template->>Browser: HTML with German content
    Browser->>User: Consistent German interface
```