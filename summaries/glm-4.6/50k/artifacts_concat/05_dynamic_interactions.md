
# Spring PetClinic Dynamic Interaction Flows

## 1. Owner Management Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant OwnerController
    participant OwnerRepository
    participant Database
    participant Validator
    
    User->>+Browser: Navigate to /owners
    Browser->>+OwnerController: GET /owners
    OwnerController->>+OwnerRepository: findAll()
    OwnerRepository->>+Database: SELECT * FROM owners
    Database-->>-OwnerRepository: Owner list
    OwnerRepository-->>-OwnerController: List<Owner>
    OwnerController-->>-Browser: owners/ownersList.html
    Browser-->>-User: Display owners list
    
    User->>+Browser: Click "Add Owner"
    Browser->>+OwnerController: GET /owners/new
    OwnerController-->>-Browser: owners/createOrUpdateOwnerForm.html
    Browser-->>-User: Show owner creation form
    
    User->>+Browser: Fill and submit form
    Browser->>+OwnerController: POST /owners/new
    OwnerController->>+Validator: validate(ownerForm)
    alt Valid data
        Validator-->>-OwnerController: Valid
        OwnerController->>+OwnerRepository: save(owner)
        OwnerRepository->>+Database: INSERT INTO owners
        Database-->>-OwnerRepository: Saved owner
        OwnerRepository-->>-OwnerController: Owner with ID
        OwnerController->>OwnerController: redirect:/owners/{ownerId}
    else Invalid data
        Validator-->>-OwnerController: Validation errors
        OwnerController-->>-Browser: owners/createOrUpdateOwnerForm.html + errors
    end
    
    User->>+Browser: Search for owner
    Browser->>+OwnerController: GET /owners?lastName=Smith
    OwnerController->>+OwnerRepository: findByLastName("Smith", Pageable)
    OwnerRepository->>+Database: SELECT * FROM owners WHERE last_name LIKE 'Smith%'
    Database-->>-OwnerRepository: Paged result
    OwnerRepository-->>-OwnerController: Page<Owner>
    OwnerController-->>-Browser: owners/ownersList.html
    Browser-->>-User: Display search results
```

## 2. Pet Management Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant PetController
    participant OwnerRepository
    participant PetTypeRepository
    participant PetValidator
    participant Database
    
    User->>+Browser: Navigate to owner details
    Browser->>+PetController: GET /owners/{ownerId}
    PetController->>+OwnerRepository: findById(ownerId)
    OwnerRepository->>+Database: SELECT * FROM owners WHERE id=?
    Database-->>-OwnerRepository: Owner with pets
    OwnerRepository-->>-PetController: Owner
    PetController->>+PetTypeRepository: findAll()
    PetTypeRepository->>+Database: SELECT * FROM pet_types
    Database-->>-PetTypeRepository: List<PetType>
    PetTypeRepository-->>-PetController: Pet types
    PetController-->>-Browser: owners/ownerDetails.html
    Browser-->>-User: Display owner with pets
    
    User->>+Browser: Click "Add New Pet"
    Browser->>+PetController: GET /owners/{ownerId}/pets/new
    PetController->>+OwnerRepository: findById(ownerId)
    PetController->>+PetTypeRepository: findAll()
    PetController-->>-Browser: pets/createOrUpdatePetForm.html
    Browser-->>-User: Show pet creation form
    
    User->>+Browser: Fill pet details and submit
    Browser->>+PetController: POST /owners/{ownerId}/pets/new
    PetController->>+PetValidator: validate(pet, errors)
    PetValidator->>+PetValidator: Check birthDate <= today
    PetValidator->>+PetValidator: Check unique name per owner
    alt Valid pet
        PetValidator-->>-PetController: Valid
        PetController->>PetController: owner.addPet(pet)
        PetController->>+OwnerRepository: save(owner)
        OwnerRepository->>+Database: BEGIN TRANSACTION
        OwnerRepository->>Database: UPDATE owners SET ...
        OwnerRepository->>Database: INSERT INTO pets ...
        Database-->>-OwnerRepository: COMMIT
        OwnerRepository-->>-PetController: Saved owner
        PetController->>PetController: redirect:/owners/{ownerId}
    else Validation errors
        PetValidator-->>-PetController: Errors
        PetController->>+PetTypeRepository: findAll()
        PetController-->>-Browser: pets/createOrUpdatePetForm.html + errors
    end
```

## 3. Visit Scheduling Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VisitController
    participant OwnerRepository
    participant VisitValidator
    participant Database
    
    User->>+Browser: Navigate to pet details
    Browser->>+VisitController: GET /owners/{ownerId}/pets/{petId}
    VisitController->>+OwnerRepository: findById(ownerId)
    OwnerRepository->>+Database: SELECT * FROM owners WHERE id=?
    Database-->>-OwnerRepository: Owner with pets and visits
    OwnerRepository-->>-VisitController: Owner
    VisitController-->>-Browser: pets/petDetails.html
    Browser-->>-User: Display pet with visit history
    
    User->>+Browser: Click "Add Visit"
    Browser->>+VisitController: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>+OwnerRepository: findById(ownerId)
    VisitController-->>-Browser: visits/createOrUpdateVisitForm.html
    Browser-->>-User: Show visit scheduling form
    
    User->>+Browser: Fill visit details and submit
    Browser->>+VisitController: POST /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>+VisitValidator: validate(visit, errors)
    VisitValidator->>+VisitValidator: Check date <= today
    VisitValidator->>+VisitValidator: Check description not blank
    alt Valid visit
        VisitValidator-->>-VisitController: Valid
        VisitController->>VisitController: pet.addVisit(visit)
        VisitController->>+OwnerRepository: save(owner)
        OwnerRepository->>+Database: BEGIN TRANSACTION
        OwnerRepository->>Database: UPDATE owners
        OwnerRepository->>Database: INSERT INTO visits ...
        Database-->>-OwnerRepository: COMMIT
        OwnerRepository-->>-VisitController: Saved
        VisitController->>VisitController: redirect:/owners/{ownerId}
    else Validation errors
        VisitValidator-->>-VisitController: Errors
        VisitController-->>-Browser: visits/createOrUpdateVisitForm.html + errors
    end
```

## 4. Vet Management Workflow with Caching

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VetController
    participant VetRepository
    participant CacheManager
    participant Database
    
    User->>+Browser: Navigate to /vets.html
    Browser->>+VetController: GET /vets.html
    VetController->>+VetRepository: findAll()
    VetRepository->>+CacheManager: get("vets")
    alt Cache miss
        CacheManager-->>-VetRepository: null
        VetRepository->>+Database: SELECT v.*, s.* FROM vets v LEFT JOIN vet_specialties vs ON v.id = vs.vet_id LEFT JOIN specialties s ON vs.specialty_id = s.id
        Database-->>-VetRepository: List<Vet> with specialties
        VetRepository->>+CacheManager: put("vets", vetList)
        CacheManager-->>-VetRepository: Cached
    else Cache hit
        CacheManager-->>-VetRepository: cached Vet list
    end
    VetRepository-->>-VetController: List<Vet>
    VetController-->>-Browser: vets/vetList.html
    Browser-->>-User: Display vet list with specialties
    
    User->>+Browser: Request JSON vet list
    Browser->>+VetController: GET /vets
    VetController->>+VetRepository: findAll()
    VetRepository->>+CacheManager: get("vets")
    CacheManager-->>-VetRepository: cached Vet list
    VetRepository-->>-VetController: List<Vet>
    VetController-->>-Browser: JSON response
    Browser-->>-User: Process vet data
```

## 5. Error Handling and Validation Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Controller
    participant Validator
    participant ErrorHandler
    participant ErrorView
    
    User->>+Browser: Submit invalid form
    Browser->>+Controller: POST with invalid data
    Controller->>+Validator: validate(form)
    Validator->>Validator: Check @NotBlank constraints
    Validator->>Validator: Check phone format regex
    Validator->>Validator: Check date rules
    Validator-->>-Controller: Field errors
    Controller->>Controller: BindingResult.hasErrors()
    Controller->>Controller: populate form data
    Controller-->>-Browser: Return form view + errors
    Browser-->>-User: Show validation errors
    
    User->>+Browser: Access invalid URL
    Browser->>+Controller: GET /invalid/path
    Controller->>+ErrorHandler: handle404()
    ErrorHandler-->>-Controller: Error response
    Controller-->>-Browser: /error template
    Browser-->>-User: Display 404 page
    
    User->>+Browser: Trigger system error
    Browser->>+Controller: GET /oups
    Controller->>Controller: throw new RuntimeException()
    Controller->>+ErrorHandler: handleException(exception)
    ErrorHandler->>+ErrorView: error.html
    ErrorView-->>-ErrorHandler: Rendered error page
    ErrorHandler-->>-Browser: Error response
    Browser-->>-User: Display error page
    
    Note over Browser,ErrorHandler: Global error handling via<br/>BasicErrorController
```

## 6. Database Transaction and Consistency Workflow

```mermaid
sequenceDiagram
    participant Client
    participant Service
    participant Repository
    participant TransactionManager
    participant Database
    
    Client->>+Service: updateOwnerWithPets(owner, pets)
    Service->>+TransactionManager: begin()
    TransactionManager->>TransactionManager: Open connection
    TransactionManager->>TransactionManager: BEGIN TRANSACTION
    
    Service->>+Repository: save(owner)
    Repository->>+Database: UPDATE owners SET ...
    Database-->>-Repository: Rows affected
    
    loop For each pet
        Service->>+Repository: save(pet)
        Repository->>+Database: INSERT/UPDATE pets ...
        Database-->>-Repository: Rows affected
    end
    
    loop For each visit
        Service->>+Repository: save(visit)
        Repository->>+Database: INSERT/UPDATE visits ...
        Database-->>-Repository: Rows affected
    end
    
    alt All operations successful
        Service->>+TransactionManager: commit()
        TransactionManager->>+Database: COMMIT
        Database-->>-TransactionManager: Success
        TransactionManager-->>-Service: Committed
        Service-->>-Client: Success response
    else Any operation fails
        Service->>+TransactionManager: rollback()
        TransactionManager->>+Database: ROLLBACK
        Database-->>-TransactionManager: Rolled back
        TransactionManager-->>-Service: Rolled back
        Service-->>-Client: Error response
    end
    
    Note over Service,Repository: @Transactional at repository layer<br/>ensures ACID properties
```

## 7. Multi-Database Configuration Workflow

```mermaid
sequenceDiagram
    participant Application
    participant ConfigLoader
    participant DataSource
    participant JPA
    participant Database
    
    Application->>+ConfigLoader: startup()
    ConfigLoader->>ConfigLoader: Read spring.profiles.active
    
    alt Profile = mysql
        ConfigLoader->>+DataSource: create MySQL datasource
        DataSource->>DataSource: URL: jdbc:mysql://localhost:3306/petclinic
        DataSource->>DataSource: Driver: MySQL Connector
        DataSource->>+Database: Test connection
        Database-->>-DataSource: Connected
        DataSource-->>-ConfigLoader: MySQL DataSource
    else Profile = postgres
        ConfigLoader->>+DataSource: create PostgreSQL datasource
        DataSource->>DataSource: URL: jdbc:postgresql://localhost:5432/petclinic
        DataSource->>DataSource: Driver: PostgreSQL Driver
        DataSource->>+Database: Test connection
        Database-->>-DataSource: Connected
        DataSource-->>-ConfigLoader: PostgreSQL DataSource
    else Profile = default (h2)
        ConfigLoader->>+DataSource: create H2 datasource
        DataSource->>DataSource: URL: jdbc:h2:mem:petclinic
        DataSource->>DataSource: Driver: H2 Driver
        DataSource->>+Database: Create in-memory DB
        Database-->>-DataSource: Ready
        DataSource-->>-ConfigLoader: H2 DataSource
    end
    
    ConfigLoader->>+JPA: configureEntityManager(dataSource)
    JPA->>JPA: Set Hibernate dialect
    JPA->>JPA: Configure schema generation
    JPA->>Database: Create/validate schema
    Database-->>-JPA: Schema ready
    JPA-->>-ConfigLoader: EntityManager configured
    ConfigLoader-->>-Application: Application ready
```

## 8. Performance Monitoring and Health Check Workflow

```mermaid
sequenceDiagram
    participant K8s/Monitor
    participant Actuator
    participant HealthChecker
    participant DataSource
    participant CacheManager
    participant Database
    
    K8s/Monitor->>+Actuator: GET /livez
    Actuator->>Actuator: Check application running
    Actuator-->>-K8s/Monitor: HTTP 200 (Live)
    
    K8s/Monitor->>+Actuator: GET /readyz
    Actuator->>+HealthChecker: health()
    HealthChecker->>+DataSource: checkConnection()
    DataSource->>+Database: SELECT 1
    Database-->>-DataSource: Success
    DataSource-->>-HealthChecker: DB healthy
    
    HealthChecker->>+CacheManager: checkCaches()
    CacheManager->>CacheManager: Verify cache status
    CacheManager-->>-HealthChecker: Caches active
    
    HealthChecker-->>-Actuator: UP status
    Actuator-->>-K8s/Monitor: HTTP 200 (Ready)
    
    K8s/Monitor->>+Actuator: GET /actuator/health
    Actuator->>+HealthChecker: detailedHealth()
    HealthChecker->>+DataSource: getConnectionInfo()
    DataSource-->>-HealthChecker: Connection details
    HealthChecker->>+CacheManager: getCacheStats()
    CacheManager-->>-HealthChecker: Hit ratios, sizes
    HealthChecker-->>-Actuator: Detailed health info
    Actuator-->>-K8s/Monitor: JSON health report
    
    Note over Actuator,HealthChecker: Actuator exposes comprehensive<br/>health and metrics endpoints
```