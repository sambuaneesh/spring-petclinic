
# Spring PetClinic - Dynamic Interaction Flows

## 1. Owner Search and Retrieval Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser as Web Browser
    participant OwnerController as OwnerController
    participant OwnerRepository as OwnerRepository
    participant Database as H2/MySQL/PostgreSQL
    participant Thymeleaf as Thymeleaf Engine

    User->>Browser: Click "Find Owners"
    Browser->>OwnerController: GET /owners/find
    OwnerController->>Thymeleaf: Render findOwners.html
    Thymeleaf->>Browser: Return search form HTML
    Browser->>User: Display search form

    User->>Browser: Enter last name and submit
    Browser->>OwnerController: GET /owners?lastName=Smith
    OwnerController->>OwnerRepository: findByLastNameStartingWith("Smith", Pageable)
    OwnerRepository->>Database: SELECT * FROM owners WHERE last_name LIKE 'Smith%' LIMIT 5
    Database->>OwnerRepository: Return Owner entities
    OwnerRepository->>OwnerController: Return Page<Owner>
    
    Note over OwnerController: Synchronous database operation
    Note over OwnerController: Pagination applied for performance
    
    OwnerController->>Thymeleaf: Render ownerDetails.html with owners list
    Thymeleaf->>Browser: Return HTML with owner data
    Browser->>User: Display search results

    User->>Browser: Click on specific owner
    Browser->>OwnerController: GET /owners/{ownerId}
    OwnerController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ownerId
    OwnerRepository->>Database: SELECT * FROM pets WHERE owner_id = ownerId
    Database->>OwnerRepository: Return Owner with pets
    OwnerRepository->>OwnerController: Return Owner entity
    
    Note over OwnerController: Lazy loading of pets collection
    
    OwnerController->>Thymeleaf: Render ownerDetails.html
    Thymeleaf->>Browser: Return HTML with owner and pet details
    Browser->>User: Display owner details page
```

**Workflow Purpose**: Enables users to search for owners by last name and view their details with associated pets.  
**Communication Patterns**: Synchronous HTTP requests, JPA database queries, server-side rendering with Thymeleaf.

## 2. Owner Creation Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser as Web Browser
    participant OwnerController as OwnerController
    participant OwnerValidator as Validator
    participant OwnerRepository as OwnerRepository
    participant Database as H2/MySQL/PostgreSQL
    participant TransactionManager as Transaction Manager

    User->>Browser: Click "Add Owner"
    Browser->>OwnerController: GET /owners/new
    OwnerController->>Browser: Return ownerForm.html (empty form)
    Browser->>User: Display owner creation form

    User->>Browser: Fill form and submit
    Note over User: Enter: firstName, lastName, address, city, telephone
    Browser->>OwnerController: POST /owners/new (form data)
    
    rect rgb(255, 245, 245)
    Note over OwnerController: Validation Phase
    OwnerController->>OwnerValidator: validate(Owner object)
    OwnerValidator->>OwnerValidator: Check telephone: exactly 10 digits
    OwnerValidator->>OwnerValidator: Check required fields
    alt Validation fails
        OwnerValidator->>OwnerController: BindingResult has errors
        OwnerController->>Browser: Return ownerForm.html with errors
        Browser->>User: Display form with validation messages
    end
    end

    alt Validation passes
        rect rgb(245, 255, 245)
        Note over TransactionManager: Transaction Started
        TransactionManager->>OwnerRepository: Begin transaction
        OwnerController->>OwnerRepository: save(Owner)
        OwnerRepository->>Database: INSERT INTO owners VALUES(...)
        Database->>OwnerRepository: Return generated ID
        OwnerRepository->>OwnerController: Return saved Owner
        TransactionManager->>TransactionManager: Commit transaction
        Note over TransactionManager: Transaction Committed
        end
        
        Note over OwnerController: Post-redirect-get pattern
        OwnerController->>Browser: Redirect: GET /owners/{ownerId}
        Browser->>OwnerController: GET /owners/{ownerId}
        OwnerController->>OwnerRepository: findById(ownerId)
        OwnerRepository->>Database: SELECT * FROM owners WHERE id = ownerId
        Database->>OwnerRepository: Return Owner
        OwnerRepository->>OwnerController: Return Owner
        OwnerController->>Browser: Render ownerDetails.html
        Browser->>User: Display newly created owner details
    end
```

**Workflow Purpose**: Allows users to register new pet owners in the system with proper validation and transaction management.  
**Communication Patterns**: Form submission, server-side validation, database transaction, PRG (Post-Redirect-Get) pattern.

## 3. Pet Creation Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser as Web Browser
    participant PetController as PetController
    participant PetValidator as Validator
    participant PetTypeFormatter as Formatter
    participant OwnerRepository as OwnerRepository
    participant PetRepository as PetRepository
    participant PetTypeRepository as PetTypeRepository
    participant Database as H2/MySQL/PostgreSQL
    participant TransactionManager as Transaction Manager

    User->>Browser: Navigate to owner details
    Browser->>PetController: GET /owners/{ownerId}/pets/new
    PetController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ownerId
    Database->>OwnerRepository: Return Owner
    OwnerRepository->>PetController: Return Owner
    
    PetController->>PetTypeRepository: findPetTypes()
    PetTypeRepository->>Database: SELECT * FROM types ORDER BY name
    Database->>PetTypeRepository: Return List<PetType>
    PetTypeRepository->>PetController: Return pet types
    
    PetController->>Browser: Render createOrUpdatePetForm.html
    Browser->>User: Display pet creation form with type dropdown

    User->>Browser: Fill pet details and submit
    Note over User: Enter: name, birthDate, type
    Browser->>PetController: POST /owners/{ownerId}/pets/new
    
    rect rgb(255, 245, 245)
    Note over PetController: Data Conversion and Validation
    PetController->>PetTypeFormatter: parse(String petTypeName)
    PetTypeFormatter->>PetTypeRepository: findByName(petTypeName)
    PetTypeRepository->>Database: SELECT * FROM types WHERE name = ?
    Database->>PetTypeRepository: Return PetType
    PetTypeRepository->>PetTypeFormatter: Return PetType
    PetTypeFormatter->>PetController: Return PetType object
    
    PetController->>PetValidator: validate(Pet, BindingResult)
    PetValidator->>PetValidator: Check birthDate not in future
    PetValidator->>PetValidator: Check name uniqueness within owner's pets
    alt Validation fails
        PetValidator->>PetController: BindingResult has errors
        PetController->>Browser: Return form with errors
        Browser->>User: Display validation messages
    end
    end

    alt Validation passes
        rect rgb(245, 255, 245)
        Note over TransactionManager: Transaction Started
        TransactionManager->>PetRepository: Begin transaction
        PetController->>PetRepository: save(Pet)
        PetRepository->>Database: INSERT INTO pets VALUES(...)
        Database->>PetRepository: Return generated ID
        PetRepository->>PetController: Return saved Pet
        TransactionManager->>TransactionManager: Commit transaction
        Note over TransactionManager: Transaction Committed
        end
        
        PetController->>Browser: Redirect: GET /owners/{ownerId}
        Browser->>PetController: GET /owners/{ownerId}
        PetController->>OwnerRepository: findById(ownerId)
        OwnerRepository->>Database: SELECT with JOIN to get pets
        Database->>OwnerRepository: Return Owner with new pet
        OwnerRepository->>PetController: Return Owner
        PetController->>Browser: Render ownerDetails.html
        Browser->>User: Display updated owner with new pet
    end
```

**Workflow Purpose**: Enables adding new pets to existing owners with type selection, validation, and data consistency checks.  
**Communication Patterns**: Data formatting, custom validation, transaction management, foreign key relationships.

## 4. Visit Scheduling Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser as Web Browser
    participant VisitController as VisitController
    participant OwnerRepository as OwnerRepository
    participant PetRepository as PetRepository
    participant VisitRepository as VisitRepository
    participant Database as H2/MySQL/PostgreSQL
    participant TransactionManager as Transaction Manager

    User->>Browser: Navigate to pet details
    Browser->>VisitController: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id = ownerId
    Database->>OwnerRepository: Return Owner
    OwnerRepository->>VisitController: Return Owner
    
    VisitController->>PetRepository: findById(petId)
    PetRepository->>Database: SELECT * FROM pets WHERE id = petId
    Database->>PetRepository: Return Pet
    PetRepository->>VisitController: Return Pet
    
    VisitController->>Browser: Render createOrUpdateVisitForm.html
    Browser->>User: Display visit form (date defaults to today)

    User->>Browser: Fill visit details and submit
    Note over User: Enter: visitDate, description
    Browser->>VisitController: POST /owners/{ownerId}/pets/{petId}/visits/new
    
    rect rgb(255, 245, 245)
    Note over VisitController: Visit Validation
    VisitController->>VisitController: Validate visitDate not null
    VisitController->>VisitController: Set visitDate to today if empty
    VisitController->>VisitController: Validate description not empty
    alt Validation fails
        VisitController->>Browser: Return form with errors
        Browser->>User: Display validation messages
    end
    end

    alt Validation passes
        rect rgb(245, 255, 245)
        Note over TransactionManager: Transaction Started
        TransactionManager->>VisitRepository: Begin transaction
        VisitController->>VisitRepository: save(Visit)
        Note over VisitController: Link visit to pet via petId
        VisitRepository->>Database: INSERT INTO visits VALUES(...)
        Database->>VisitRepository: Return generated ID
        VisitRepository->>VisitController: Return saved Visit
        TransactionManager->>TransactionManager: Commit transaction
        Note over TransactionManager: Transaction Committed
        end
        
        Note over VisitController: Redirect to pet details
        VisitController->>Browser: Redirect: GET /owners/{ownerId}
        Browser->>VisitController: GET /owners/{ownerId}
        VisitController->>OwnerRepository: findById(ownerId)
        OwnerRepository->>Database: SELECT owners JOIN pets JOIN visits
        Database->>OwnerRepository: Return Owner with pets and visits
        OwnerRepository->>VisitController: Return Owner
        VisitController->>Browser: Render ownerDetails.html
        Browser->>User: Display updated owner with new visit
    end
```

**Workflow Purpose**: Records medical visits for pets with proper date handling and maintains medical history.  
**Communication Patterns**: Default value handling, cross-entity relationships, join queries for complete data retrieval.

## 5. Veterinarian List Retrieval with Caching

```mermaid
sequenceDiagram
    participant User
    participant Browser as Web Browser
    participant VetController as VetController
    participant VetRepository as VetRepository
    participant CacheManager as JCache (Caffeine)
    participant Database as H2/MySQL/PostgreSQL
    participant Thymeleaf as Thymeleaf Engine

    Note over User: First Request - Cache Miss
    User->>Browser: Request vets page
    Browser->>VetController: GET /vets.html
    VetController->>VetRepository: findAll(Pageable)
    
    rect rgb(255, 245, 245)
    Note over VetRepository: Cache Check - MISS
    VetRepository->>CacheManager: get("vets")
    CacheManager->>VetRepository: null (cache miss)
    end
    
    VetRepository->>Database: SELECT v.* FROM vets v<br/>LEFT JOIN vet_specialties vs ON v.id = vs.vet_id<br/>LEFT JOIN specialties s ON vs.specialty_id = s.id
    Database->>VetRepository: Return List<Vet> with specialties
    
    rect rgb(245, 255, 245)
    Note over VetRepository: Cache Population
    VetRepository->>CacheManager: put("vets", List<Vet>)
    CacheManager->>CacheManager: Store in Caffeine cache
    end
    
    VetRepository->>VetController: Return Page<Vet>
    VetController->>Thymeleaf: Render vetList.html
    Thymeleaf->>Browser: Return HTML with vets data
    Browser->>User: Display paginated vet list

    Note over User: Second Request - Cache Hit
    User->>Browser: Refresh page
    Browser->>VetController: GET /vets.html
    VetController->>VetRepository: findAll(Pageable)
    
    rect rgb(245, 255, 245)
    Note over VetRepository: Cache Check - HIT
    VetRepository->>CacheManager: get("vets")
    CacheManager->>VetRepository: Return cached List<Vet>
    Note over VetRepository: Database query skipped
    end
    
    VetRepository->>VetController: Return Page<Vet>
    VetController->>Thymeleaf: Render vetList.html
    Thymeleaf->>Browser: Return HTML
    Browser->>User: Display vet list (faster response)

    Note over User: API Request for JSON
    User->>Browser: API client request
    Browser->>VetController: GET /vets (Accept: application/json)
    VetController->>VetRepository: findAll()
    
    rect rgb(245, 255, 245)
    Note over VetRepository: Cache Hit
    VetRepository->>CacheManager: get("vets")
    CacheManager->>VetRepository: Return cached List<Vet>
    end
    
    VetRepository->>VetController: Return List<Vet>
    VetController->>Browser: Return JSON response
    Browser->>User: Display JSON data
```

**Workflow Purpose**: Provides veterinarian listings with caching optimization for frequently accessed data.  
**Communication Patterns**: Caching layer with JCache/Caffeine, content negotiation (HTML/JSON), pagination, many-to-many relationship handling.

## 6. Error Handling and Recovery Flow

```mermaid
sequenceDiagram
    participant User
    participant Browser as Web Browser
    participant Controller as Spring Controller
    participant ExceptionHandler as GlobalExceptionHandler
    participant ErrorRepository as ErrorLogging
    participant Thymeleaf as Thymeleaf Engine

    Note over User: Controlled Exception Scenario
    User->>Browser: Access /oups endpoint
    Browser->>Controller: GET /oups
    Controller->>Controller: throw new RuntimeException("Expected")
    
    rect rgb(255, 245, 245)
    Note over Controller: Exception Propagation
    Controller->>ExceptionHandler: handleException(RuntimeException)
    ExceptionHandler->>ErrorRepository: logError(exception)
    ErrorRepository->>ErrorRepository: Write to application log
    end
    
    ExceptionHandler->>Thymeleaf: Render error.html
    Thymeleaf->>Browser: Return error page HTML
    Browser->>User: Display controlled error page

    Note over User: Validation Error Scenario
    User->>Browser: Submit invalid form data
    Browser->>Controller: POST with invalid data
    Controller->>Controller: @Valid annotation triggers validation
    
    rect rgb(255, 245, 245)
    Note over Controller: Validation Failure
    Controller->>Controller: BindingResult has errors
    Controller->>Thymeleaf: Return form with error messages
    Thymeleaf->>Browser: Return form with validation errors
    Browser->>User: Display form with inline error messages
    end

    Note over User: Database Error Scenario
    User->>Browser: Trigger database operation
    Browser->>Controller: POST request requiring DB access
    Controller->>Controller: Repository operation
    
    rect rgb(255, 200, 200)
    Note over Controller: Database Exception
    Controller->>Controller: DataAccessException thrown
    Controller->>ExceptionHandler: handleDataAccessException
    ExceptionHandler->>ErrorRepository: logDatabaseError(exception)
    end
    
    ExceptionHandler->>Thymeleaf: Render error.html with DB error
    Thymeleaf->>Browser: Return user-friendly error page
    Browser->>User: Display "Database temporarily unavailable" message

    Note over User: Resource Not Found Scenario
    User->>Browser: Access non-existent resource
    Browser->>Controller: GET /invalid/path
    
    rect rgb(255, 245, 245)
    Note over Controller: 404 Handling
    Controller->>ExceptionHandler: handleNoHandlerFoundException
    ExceptionHandler->>Thymeleaf: Render error.html
    Thymeleaf->>Browser: Return 404 error page
    Browser->>User: Display "Page not found" message
    end
```

**Workflow Purpose**: Manages application errors gracefully with proper logging, user-friendly messages, and recovery mechanisms.  
**Communication Patterns**: Exception handling pipeline, error logging, user feedback without exposing internal details.

## 7. Internationalization Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser as Web Browser
    participant LocaleResolver as SessionLocaleResolver
    participant LocaleChangeInterceptor as Interceptor
    participant MessageSource as ResourceBundleMessageSource
    participant Controller as Spring Controller
    participant Thymeleaf as Thymeleaf Engine

    Note over User: Initial Request (Default Locale)
    User->>Browser: Access homepage
    Browser->>Controller: GET / (Accept-Language: en-US)
    
    rect rgb(255, 245, 245)
    Note over LocaleResolver: Locale Resolution
    LocaleResolver->>LocaleResolver: Check session for locale
    LocaleResolver->>LocaleResolver: Default to system locale
    end
    
    Controller->>Thymeleaf: Render welcome.html
    Thymeleaf->>MessageSource: getMessage("welcome.title", null, locale)
    MessageSource->>MessageSource: Load messages.properties
    MessageSource->>Thymeleaf: Return "Welcome to PetClinic"
    Thymeleaf->>Browser: Return HTML with English text
    Browser->>User: Display English page

    Note over User: Language Change Request
    User->>Browser: Click German language link
    Browser->>Controller: GET /?lang=de
    
    rect rgb(245, 255, 245)
    Note over Interceptor: Locale Change Interception
    LocaleChangeInterceptor->>LocaleResolver: setLocale(Locale.GERMAN)
    LocaleResolver->>LocaleResolver: Store in HTTP session
    end
    
    Controller->>Thymeleaf: Render welcome.html
    Thymeleaf->>MessageSource: getMessage("welcome.title", null, Locale.GERMAN)
    MessageSource->>MessageSource: Load messages_de.properties
    MessageSource->>Thymeleaf: Return "Willkommen in der PetClinic"
    Thymeleaf->>Browser: Return HTML with German text
    Browser->>User: Display German page

    Note over User: Subsequent Requests (Persisted Locale)
    User->>Browser: Navigate to other page
    Browser->>Controller: GET /vets.html
    
    rect rgb(245, 255, 245)
    Note over LocaleResolver: Locale Retrieved from Session
    LocaleResolver->>LocaleResolver: Get locale from session
    LocaleResolver->>Controller: Return Locale.GERMAN
    end
    
    Controller->>Thymeleaf: Render vets page
    Thymeleaf->>MessageSource: Multiple getMessage calls
    MessageSource->>MessageSource: Load messages_de.properties
    MessageSource->>Thymeleaf: Return German translations
    Thymeleaf->>Browser: Return HTML with German text
    Browser->>User: Display German vets page

    Note over User: Missing Translation Handling
    User->>Browser: Access page with missing translation
    Browser->>Controller: GET /somepage
    Controller->>Thymeleaf: Render template
    
    rect rgb(255, 245, 245)
    Note over MessageSource: Fallback Mechanism
    Thymeleaf->>MessageSource: getMessage("missing.key", null, Locale.GERMAN)
    MessageSource->>MessageSource: Check messages_de.properties (not found)
    MessageSource->>MessageSource: Fallback to messages.properties
    MessageSource->>Thymeleaf: Return English default
    end
    
    Thymeleaf->>Browser: Return HTML with fallback text
    Browser->>User: Display page with some English text
```

**Workflow Purpose**: Provides multi-language support with session-based locale persistence and fallback mechanisms.  
**Communication Patterns**: Locale resolution, message resource loading, session storage, fallback translation handling.

## 8. Database Transaction Management Flow

```mermaid
sequenceDiagram
    participant Client as Client Code
    participant Service as @Service Layer
    participant TransactionManager as PlatformTransactionManager
    participant Repository as JpaRepository
    participant Database as H2/MySQL/PostgreSQL
    participant TransactionSynchronization as TransactionSynchronizationManager

    Client->>Service: callTransactionalMethod()

    rect rgb(245, 255, 245)
    Note over TransactionManager: Transaction Begin
    TransactionManager->>TransactionSynchronization: bindTransaction
    TransactionSynchronization->>TransactionSynchronization: Store transaction context
    TransactionManager->>Database: BEGIN TRANSACTION
    Database->>TransactionManager: Transaction started
    end

    Service->>Repository: save(entity1)
    Repository->>Database: INSERT INTO table1 VALUES(...)
    Database->>Repository: Success
    Repository->>Service: Return entity1

    Service->>Repository: save(entity2)
    Repository->>Database: INSERT INTO table2 VALUES(...)
    Database->>Repository: Success
    Repository->>Service: Return entity2

    Note over Service: Business logic processing
    
    Service->>Repository: save(entity3)
    Repository->>Database: INSERT INTO table3 VALUES(...)
    
    rect rgb(255, 200, 200)
    Note over Database: Simulated Constraint Violation
    Database->>Repository: Throw DataIntegrityViolationException
    Repository->>Service: Propagate exception
    end

    rect rgb(255, 200, 200)
    Note over TransactionManager: Transaction Rollback
    Service->>TransactionManager: markTransactionForRollback()
    TransactionManager->>Database: ROLLBACK
    Database->>TransactionManager: All changes reverted
    TransactionManager->>TransactionSynchronization: unbindTransaction
    TransactionSynchronization->>TransactionSynchronization: Clear transaction context
    end

    Service->>Client: Throw exception to caller

    Note over Client: Retry Scenario
    Client->>Service: callTransactionalMethod() with valid data

    rect rgb(245, 255, 245)
    Note over TransactionManager: New Transaction
    TransactionManager->>Database: BEGIN TRANSACTION
    end

    Service->>Repository: save(entity4)
    Repository->>Database: INSERT INTO table4 VALUES(...)
    Database->>Repository: Success

    Service->>Repository: save(entity5)
    Repository->>Database: INSERT INTO table5 VALUES(...)
    Database->>Repository: Success

    rect rgb(245, 255, 245)
    Note over TransactionManager: Transaction Commit
    Service->>TransactionManager: commit()
    TransactionManager->>Database: COMMIT
    Database->>TransactionManager: All changes persisted
    TransactionManager->>TransactionSynchronization: unbindTransaction
    end

    Service->>Client: Return success
```

**Workflow Purpose**: Demonstrates Spring's declarative transaction management with proper rollback and commit scenarios.  
**Communication Patterns**: Transaction boundaries, rollback on exceptions, commit on successful completion, context management.