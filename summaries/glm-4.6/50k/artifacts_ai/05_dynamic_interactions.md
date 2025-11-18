
# Owner Management Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant OwnerController
    participant OwnerRepository
    participant Database
    participant JPA/Hibernate

    Note over User,Database: Owner Search Workflow
    User->>Browser: Access /owners page
    Browser->>OwnerController: GET /owners
    OwnerController->>OwnerRepository: findAll()
    OwnerRepository->>JPA/Hibernate: Query all owners
    JPA/Hibernate->>Database: SELECT * FROM owners
    Database-->>JPA/Hibernate: Owner entities
    JPA/Hibernate-->>OwnerRepository: Owner list
    OwnerRepository-->>OwnerController: List<Owner>
    OwnerController-->>Browser: owners/ownersList.html
    Browser-->>User: Display owners page

    Note over User,Database: Owner Search with Pagination
    User->>Browser: Search by last name
    Browser->>OwnerController: GET /owners?lastName=Smith
    OwnerController->>OwnerRepository: findByLastName("Smith", Pageable)
    OwnerRepository->>JPA/Hibernate: Query with pagination
    JPA/Hibernate->>Database: SELECT * FROM owners WHERE last_name LIKE 'Smith%' LIMIT 20 OFFSET 0
    Database-->>JPA/Hibernate: Paginated results
    JPA/Hibernate-->>OwnerRepository: Page<Owner>
    OwnerRepository-->>OwnerController: Paginated owners
    OwnerController-->>Browser: owners/ownersList.html with pagination
    Browser-->>User: Display filtered owners

    Note over User,Database: Create New Owner
    User->>Browser: Fill owner form & submit
    Browser->>OwnerController: POST /owners/new
    OwnerController->>OwnerController: validate owner (JSR-303)
    OwnerController->>OwnerRepository: save(owner)
    OwnerRepository->>JPA/Hibernate: persist owner
    JPA/Hibernate->>Database: INSERT INTO owners...
    Database-->>JPA/Hibernate: Generated ID
    JPA/Hibernate-->>OwnerRepository: Saved Owner with ID
    OwnerRepository-->>OwnerController: Owner entity
    OwnerController-->>Browser: redirect:/owners/{ownerId}
    Browser->>OwnerController: GET /owners/{ownerId}
    OwnerController->>Browser: owners/ownerDetails.html
    Browser-->>User: Display owner details
```

# Pet Management Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant PetController
    participant OwnerRepository
    participant PetTypeRepository
    participant PetValidator
    participant Database
    participant JPA/Hibernate

    Note over User,Database: Add New Pet to Owner
    User->>Browser: Navigate to owner's pet list
    Browser->>PetController: GET /owners/{ownerId}/pets/new
    PetController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>JPA/Hibernate: SELECT owner
    JPA/Hibernate->>Database: Query owner by ID
    Database-->>JPA/Hibernate: Owner entity
    JPA/Hibernate-->>OwnerRepository: Owner
    OwnerRepository-->>PetController: Owner with pets
    PetController->>PetTypeRepository: findPetTypes()
    PetTypeRepository->>JPA/Hibernate: SELECT pet_types
    JPA/Hibernate->>Database: Query all pet types
    Database-->>JPA/Hibernate: PetType list
    JPA/Hibernate-->>PetTypeRepository: List<PetType>
    PetTypeRepository-->>PetController: Available pet types
    PetController-->>Browser: pets/createOrUpdatePetForm.html
    Browser-->>User: Display pet creation form

    Note over User,Database: Submit Pet Form
    User->>Browser: Fill pet details & submit
    Browser->>PetController: POST /owners/{ownerId}/pets/new
    PetController->>PetValidator: validate(pet, errors)
    PetValidator->>PetValidator: Check unique name per owner
    PetValidator->>PetValidator: Validate birth date ≤ today
    PetValidator-->>PetController: Validation result
    alt Validation fails
        PetController-->>Browser: Return form with errors
    else Validation passes
        PetController->>OwnerRepository: save(owner with new pet)
        OwnerRepository->>JPA/Hibernate: persist/cascade save
        JPA/Hibernate->>Database: INSERT INTO pets...
        Database-->>JPA/Hibernate: Success confirmation
        JPA/Hibernate-->>OwnerRepository: Updated Owner
        OwnerRepository-->>PetController: Saved Owner
        PetController-->>Browser: redirect:/owners/{ownerId}
        Browser-->>User: Display updated owner details
    end
```

# Visit Scheduling Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VisitController
    participant OwnerRepository
    participant Database
    participant JPA/Hibernate
    participant VisitValidator

    Note over User,Database: Schedule New Visit
    User->>Browser: Click "Add Visit" for pet
    Browser->>VisitController: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>OwnerRepository: findById(ownerId)
    OwnerRepository->>JPA/Hibernate: SELECT owner with pets and visits
    JPA/Hibernate->>Database: Query with JOIN FETCH
    Database-->>JPA/Hibernate: Owner with lazy-loaded collections
    JPA/Hibernate-->>OwnerRepository: Owner entity
    OwnerRepository-->>VisitController: Owner with specific pet
    VisitController->>VisitController: Extract pet from owner
    VisitController-->>Browser: visits/createOrUpdateVisitForm.html
    Browser-->>User: Display visit scheduling form

    Note over User,Database: Submit Visit Form
    User->>Browser: Fill visit details & submit
    Browser->>VisitController: POST /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>VisitValidator: validate(visit, errors)
    VisitValidator->>VisitValidator: Check date ≤ today
    VisitValidator->>VisitValidator: Check description not blank
    VisitValidator-->>VisitController: Validation result
    alt Validation fails
        VisitController-->>Browser: Return form with errors
    else Validation passes
        VisitController->>OwnerRepository: save(owner with new visit)
        OwnerRepository->>JPA/Hibernate: cascade persist visit
        JPA/Hibernate->>Database: INSERT INTO visits...
        Database-->>JPA/Hibernate: Success with visit ID
        JPA/Hibernate-->>OwnerRepository: Updated Owner
        OwnerRepository-->>VisitController: Saved Owner
        VisitController-->>Browser: redirect:/owners/{ownerId}
        Browser-->>User: Display updated owner with new visit
    end
```

# Vet Listing Workflow with Caching

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VetController
    participant VetRepository
    participant CacheManager
    participant Database
    participant JPA/Hibernate

    Note over User,Database: First Request (Cache Miss)
    User->>Browser: Access /vets.html
    Browser->>VetController: GET /vets.html
    VetController->>VetRepository: findAll()
    VetRepository->>CacheManager: check cache("vets")
    CacheManager-->>VetRepository: Cache miss
    VetRepository->>JPA/Hibernate: Query vets with specialties
    JPA/Hibernate->>Database: SELECT v.*, s.* FROM vets v LEFT JOIN vet_specialties vs...
    Database-->>JPA/Hibernate: Vet entities with specialties
    JPA/Hibernate-->>VetRepository: List<Vet>
    VetRepository->>CacheManager: put("vets", vetList)
    CacheManager-->>VetRepository: Stored in cache
    VetRepository-->>VetController: Vet list
    VetController-->>Browser: vets/vetList.html with pagination
    Browser-->>User: Display vet listing

    Note over User,Database: Subsequent Request (Cache Hit)
    User->>Browser: Refresh /vets.html
    Browser->>VetController: GET /vets.html
    VetController->>VetRepository: findAll()
    VetRepository->>CacheManager: check cache("vets")
    CacheManager-->>VetRepository: Cache hit - vetList
    VetRepository-->>VetController: Vet list (from cache)
    VetController-->>Browser: vets/vetList.html
    Browser-->>User: Display vet listing (faster)

    Note over User,Database: JSON API Request
    User->>Browser: API call /vets
    Browser->>VetController: GET /vets
    VetController->>VetRepository: findAll()
    VetRepository->>CacheManager: check cache("vets")
    CacheManager-->>VetRepository: Cached vet list
    VetRepository-->>VetController: Vet list
    VetController-->>Browser: JSON response
    Browser-->>User: Process vet data
```

# Error Handling and Recovery Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Controller
    participant GlobalExceptionHandler
    participant ErrorTemplate
    participant Actuator
    participant LoggingService

    Note over User,LoggingService: Validation Error
    User->>Browser: Submit invalid form data
    Browser->>Controller: POST with invalid data
    Controller->>Controller: validate input (JSR-303)
    Controller-->>Browser: Return form with error messages
    Browser-->>User: Display validation errors

    Note over User,LoggingService: Business Logic Error
    User->>Browser: Perform invalid operation
    Browser->>Controller: Request triggering error
    Controller->>Controller: Business validation fails
    Controller->>LoggingService: logError()
    LoggingService-->>Controller: Logged
    Controller-->>Browser: Redirect with error message
    Browser-->>User: Display error feedback

    Note over User,LoggingService: System Error
    User->>Browser: Request causing exception
    Browser->>Controller: Request
    Controller->>Controller: Exception thrown
    Controller->>GlobalExceptionHandler: handleException()
    GlobalExceptionHandler->>LoggingService: logException()
    LoggingService-->>GlobalExceptionHandler: Logged
    GlobalExceptionHandler->>ErrorTemplate: Render error page
    ErrorTemplate-->>GlobalExceptionHandler: error.html
    GlobalExceptionHandler-->>Browser: Error page with details
    Browser-->>User: Display error page

    Note over User,LoggingService: Crash Endpoint Testing
    User->>Browser: Access /oups
    Browser->>Controller: GET /oups
    Controller->>Controller: Simulate RuntimeException
    Controller->>GlobalExceptionHandler: handleException()
    GlobalExceptionHandler->>Actuator: Log to health metrics
    Actuator-->>GlobalExceptionHandler: Metrics updated
    GlobalExceptionHandler-->>Browser: Custom error page
    Browser-->>User: Display crash error page
```

# Database Configuration and Initialization Workflow

```mermaid
sequenceDiagram
    participant SpringBoot
    participant ProfileManager
    participant ConfigService
    participant DataSource
    participant Database
    participant SchemaLoader
    participant DataLoader

    Note over SpringBoot,DataLoader: Application Startup
    SpringBoot->>ProfileManager: Read active profiles
    ProfileManager-->>SpringBoot: [default|mysql|postgres]

    Note over SpringBoot,DataLoader: Default H2 Configuration
    alt No profile specified
        SpringBoot->>ConfigService: Load application.properties
        ConfigService-->>SpringBoot: H2 in-memory config
        SpringBoot->>DataSource: Create H2 DataSource
        DataSource->>Database: Initialize in-memory H2
        Database-->>DataSource: Connection ready
    end

    Note over SpringBoot,DataLoader: MySQL Configuration
    alt mysql profile active
        SpringBoot->>ConfigService: Load application-mysql.properties
        ConfigService-->>SpringBoot: MySQL config with @ServiceConnection
        SpringBoot->>DataSource: Create MySQL DataSource
        DataSource->>Database: Connect to MySQL container
        Database-->>DataSource: Connection established
    end

    Note over SpringBoot,DataLoader: PostgreSQL Configuration
    alt postgres profile active
        SpringBoot->>ConfigService: Load application-postgres.properties
        ConfigService-->>SpringBoot: PostgreSQL config with @ServiceConnection
        SpringBoot->>DataSource: Create PostgreSQL DataSource
        DataSource->>Database: Connect to PostgreSQL container
        Database-->>DataSource: Connection established
    end

    Note over SpringBoot,DataLoader: Schema Initialization
    SpringBoot->>SchemaLoader: Execute schema.sql
    SchemaLoader->>DataSource: Execute DDL statements
    DataSource->>Database: CREATE TABLE statements
    Database-->>DataSource: Schema created
    DataSource-->>SchemaLoader: Success
    SchemaLoader-->>SpringBoot: Schema initialized

    Note over SpringBoot,DataLoader: Data Loading
    SpringBoot->>DataLoader: Execute data.sql
    DataLoader->>DataSource: Execute DML statements
    DataSource->>Database: INSERT sample data
    Database-->>DataSource: Data loaded
    DataSource-->>DataLoader: Success
    DataLoader-->>SpringBoot: Data initialized

    Note over SpringBoot,DataLoader: Application Ready
    SpringBoot->>SpringBoot: Start web server
    SpringBoot-->>SpringBoot: Application ready for requests
```

# Multi-Database Support and Testcontainers Integration Workflow

```mermaid
sequenceDiagram
    participant TestRunner
    participant SpringBootTest
    participant Testcontainers
    participant DatabaseContainer
    participant DataSource
    participant JPA/Hibernate
    participant Repository

    Note over TestRunner,Repository: MySQL Integration Test
    TestRunner->>SpringBootTest: @SpringBootTest with @ActiveProfiles("mysql")
    SpringBootTest->>Testcontainers: Start MySQL container
    Testcontainers->>DatabaseContainer: docker run mysql:9.2
    DatabaseContainer-->>Testcontainers: Container running
    Testcontainers->>DatabaseContainer: Wait for database readiness
    DatabaseContainer-->>Testcontainers: Database accepting connections
    Testcontainers-->>SpringBootTest: Container ready, port mapped
    SpringBootTest->>DataSource: Create MySQL DataSource with mapped port
    DataSource->>DatabaseContainer: Establish connection
    DatabaseContainer-->>DataSource: Connection successful
    DataSource-->>SpringBootTest: DataSource initialized
    SpringBootTest->>JPA/Hibernate: Configure with MySQL dialect
    JPA/Hibernate->>Repository: Initialize repositories
    Repository-->>SpringBootTest: Ready for testing

    Note over TestRunner,Repository: PostgreSQL Integration Test
    TestRunner->>SpringBootTest: @SpringBootTest with @ActiveProfiles("postgres")
    SpringBootTest->>Testcontainers: Start PostgreSQL container
    Testcontainers->>DatabaseContainer: docker run postgres:17.5
    DatabaseContainer-->>Testcontainers: Container running
    Testcontainers->>DatabaseContainer: Wait for database readiness
    DatabaseContainer-->>Testcontainers: Database accepting connections
    Testcontainers-->>SpringBootTest: Container ready
    SpringBootTest->>DataSource: Create PostgreSQL DataSource
    DataSource->>DatabaseContainer: Establish connection
    DatabaseContainer-->>DataSource: Connection successful
    DataSource-->>SpringBootTest: DataSource initialized
    SpringBootTest->>JPA/Hibernate: Configure with PostgreSQL dialect
    JPA/Hibernate->>Repository: Initialize repositories
    Repository-->>SpringBootTest: Ready for testing

    Note over TestRunner,Repository: Repository Testing
    TestRunner->>Repository: Execute test operations
    Repository->>JPA/Hibernate: save/find/update/delete
    JPA/Hibernate->>DataSource: Execute SQL
    DataSource->>DatabaseContainer: Database operations
    DatabaseContainer-->>DataSource: Results
    DataSource-->>JPA/Hibernate: Query results
    JPA/Hibernate-->>Repository: Entity mappings
    Repository-->>TestRunner: Test results

    Note over TestRunner,Repository: Cleanup
    TestRunner->>SpringBootTest: Test cleanup
    SpringBootTest->>Testcontainers: Stop containers
    Testcontainers->>DatabaseContainer: docker stop
    DatabaseContainer-->>Testcontainers: Container stopped
    Testcontainers-->>SpringBootTest: Cleanup complete
```