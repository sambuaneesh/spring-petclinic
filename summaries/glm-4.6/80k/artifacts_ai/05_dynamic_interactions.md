
# Spring PetClinic Dynamic Interaction Flows

## 1. Owner Registration Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant OwnerController
    participant OwnerService
    participant OwnerRepository
    participant Database
    participant CacheManager

    User->>Browser: Navigates to /owners/new
    Browser->>OwnerController: GET /owners/new
    OwnerController->>Browser: Returns owner creation form
    Browser-->>User: Displays form fields

    User->>Browser: Fills owner details and submits
    Browser->>OwnerController: POST /owners/new (OwnerDTO)
    
    OwnerController->>OwnerService: processNewOwnerForm(OwnerDTO)
    OwnerService->>OwnerService: validateOwnerData()
    OwnerService->>OwnerRepository: save(Owner)
    
    OwnerRepository->>Database: BEGIN TRANSACTION
    Database-->>OwnerRepository: Transaction started
    OwnerRepository->>Database: INSERT INTO owners (...)
    Database-->>OwnerRepository: Owner created with ID
    OwnerRepository->>Database: COMMIT
    Database-->>OwnerRepository: Transaction committed
    
    OwnerRepository-->>OwnerService: Owner entity with ID
    OwnerService-->>OwnerController: Saved Owner
    OwnerController->>OwnerController: redirect:/owners/{id}
    
    Note over OwnerController,Database: Synchronous REST call with<br/>database transaction
    OwnerController->>Browser: 302 Redirect to /owners/{id}
    Browser->>OwnerController: GET /owners/{id}
    OwnerController->>OwnerRepository: findById(id)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id=?
    Database-->>OwnerRepository: Owner data
    OwnerRepository-->>OwnerController: Owner entity
    OwnerController->>Browser: Owner detail view
    Browser-->>User: Shows newly created owner profile
```

**Description**: User registers a new owner in the system. Triggered by accessing the owner creation form and submitting valid owner data. Uses synchronous REST calls with database transactions ensuring ACID properties.

## 2. Pet Addition Under Owner Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant OwnerController
    participant PetService
    participant PetRepository
    participant PetTypeRepository
    participant Database
    participant CacheManager

    User->>Browser: Clicks "Add Pet" for owner {ownerId}
    Browser->>OwnerController: GET /owners/{ownerId}/pets/new
    OwnerController->>PetTypeRepository: findPetTypes()
    
    alt Cache Hit
        CacheManager->>PetTypeRepository: @Cacheable("petTypes")
        PetTypeRepository-->>CacheManager: Returns cached pet types
    else Cache Miss
        PetTypeRepository->>Database: SELECT * FROM types
        Database-->>PetTypeRepository: All pet types
        PetTypeRepository->>CacheManager: Cache result
    end
    
    PetTypeRepository-->>OwnerController: List<PetType>
    OwnerController->>Browser: Pet creation form with pet types
    Browser-->>User: Displays form

    User->>Browser: Fills pet details and submits
    Browser->>OwnerController: POST /owners/{ownerId}/pets/new (PetDTO)
    
    OwnerController->>PetService: processNewPetForm(PetDTO, ownerId)
    PetService->>PetService: validatePetData()
    PetService->>OwnerRepository: findById(ownerId)
    OwnerRepository->>Database: SELECT * FROM owners WHERE id=?
    Database-->>OwnerRepository: Owner entity
    OwnerRepository-->>PetService: Owner
    
    PetService->>PetRepository: save(Pet)
    PetRepository->>Database: BEGIN TRANSACTION
    Database-->>PetRepository: Transaction started
    PetRepository->>Database: INSERT INTO pets (owner_id, type_id, ...)
    Database-->>PetRepository: Pet created with ID
    PetRepository->>Database: INSERT INTO visits (...)
    Database-->>PetRepository: Initial visit created
    PetRepository->>Database: COMMIT
    Database-->>PetRepository: Transaction committed
    
    PetRepository-->>PetService: Pet entity
    PetService-->>OwnerController: Saved Pet
    OwnerController->>Browser: 302 Redirect to /owners/{ownerId}
    
    Note over OwnerController,Database: Cascading save operation<br/>with transactional integrity
```

**Description**: User adds a new pet to an existing owner. Triggered from owner detail view. Involves cached pet types lookup and cascading database transaction for pet and initial visit creation.

## 3. Visit Scheduling Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VisitController
    participant VisitService
    participant VisitRepository
    participant PetRepository
    participant VetRepository
    participant Database

    User->>Browser: Clicks "Add Visit" for pet {petId}
    Browser->>VisitController: GET /owners/{ownerId}/pets/{petId}/visits/new
    VisitController->>PetRepository: findById(petId)
    PetRepository->>Database: SELECT * FROM pets WHERE id=?
    Database-->>PetRepository: Pet entity
    PetRepository-->>VisitController: Pet
    VisitController->>Browser: Visit creation form
    Browser-->>User: Displays form

    User->>Browser: Fills visit details and submits
    Browser->>VisitController: POST /owners/{ownerId}/pets/{petId}/visits/new (VisitDTO)
    
    VisitController->>VisitService: processNewVisitForm(VisitDTO, petId)
    VisitService->>VisitService: validateVisitData()
    VisitService->>PetRepository: findById(petId)
    PetRepository->>Database: SELECT * FROM pets WHERE id=?
    Database-->>PetRepository: Pet entity
    PetRepository-->>VisitService: Pet
    
    VisitService->>VisitRepository: save(Visit)
    VisitRepository->>Database: BEGIN TRANSACTION
    Database-->>VisitRepository: Transaction started
    VisitRepository->>Database: INSERT INTO visits (pet_id, visit_date, ...)
    Database-->>VisitRepository: Visit created with ID
    VisitRepository->>Database: COMMIT
    Database-->>VisitRepository: Transaction committed
    
    VisitRepository-->>VisitService: Visit entity
    VisitService-->>VisitController: Saved Visit
    VisitController->>Browser: 302 Redirect to /owners/{ownerId}
    
    Note over VisitController,Database: Database transaction ensures<br/>visit integrity without cascade
```

**Description**: User schedules a new visit for a specific pet. Triggered from pet detail view under owner context. Uses direct database transaction without cascade operations for visit creation.

## 4. Vet Directory with Caching Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant VetController
    participant VetRepository
    participant SpecialtyRepository
    participant Database
    participant CacheManager

    User->>Browser: Requests vet directory
    Browser->>VetController: GET /vets.html
    
    VetController->>VetRepository: findAll()
    
    alt Cache Hit for Vets
        CacheManager->>VetRepository: @Cacheable("vets")
        VetRepository-->>CacheManager: Returns cached vets
    else Cache Miss for Vets
        VetRepository->>Database: SELECT v.*, s.name as specialty FROM vets v
        Database-->>VetRepository: Vets with specialties
        VetRepository->>CacheManager: Cache vets result
    end
    
    VetRepository->>SpecialtyRepository: findAll()
    SpecialtyRepository->>Database: SELECT * FROM specialties
    Database-->>SpecialtyRepository: All specialties
    SpecialtyRepository-->>VetRepository: Specialty mapping
    VetRepository-->>VetController: List<Vet> with specialties
    
    VetController->>VetController: processVetList()
    VetController->>Browser: HTML response with vet list
    Browser-->>User: Displays vet directory

    Note over VetController,CacheManager: Multi-level caching strategy<br/>for performance optimization
```

**Description**: User views the veterinarian directory. Triggered by accessing vets page. Implements multi-level caching with repository-level @Cacheable annotation for performance optimization.

## 5. Owner Search with Pagination Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant OwnerController
    participant OwnerRepository
    participant Database

    User->>Browser: Navigates to /owners/find
    Browser->>OwnerController: GET /owners/find
    OwnerController->>Browser: Search form
    Browser-->>User: Displays search interface

    User->>Browser: Enters last name and searches
    Browser->>OwnerController: POST /owners/find (lastName)
    OwnerController->>OwnerRepository: findByLastNameStartingWith(lastName, Pageable)
    
    OwnerRepository->>Database: SELECT * FROM owners 
    Database->>Database: WHERE last_name LIKE 'Smith%'
    Database->>Database: ORDER BY last_name ASC
    Database->>Database: LIMIT 5 OFFSET 0
    Database-->>OwnerRepository: Page<Owner> with metadata
    
    OwnerRepository-->>OwnerController: Paginated results
    OwnerController->>Browser: Owner list view
    Browser-->>User: Shows search results

    User->>Browser: Clicks next page
    Browser->>OwnerController: GET /owners?page=1
    OwnerController->>OwnerRepository: findByLastNameStartingWith(lastName, Pageable(page=1))
    
    OwnerRepository->>Database: SELECT * FROM owners 
    Database->>Database: WHERE last_name LIKE 'Smith%'
    Database->>Database: ORDER BY last_name ASC
    Database->>Database: LIMIT 5 OFFSET 5
    Database-->>OwnerRepository: Page<Owner> with metadata
    
    OwnerRepository-->>OwnerController: Next page results
    OwnerController->>Browser: Updated owner list
    Browser-->>User: Shows page 2 results

    Note over OwnerController,Database: Database pagination with<br/>indexed last_name column
```

**Description**: User searches for owners by last name with pagination support. Triggered from search form. Uses database-level pagination with indexed queries for efficient result retrieval.

## 6. Global Error Handling Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant Controller
    participant GlobalExceptionHandler
    participant Logger
    participant ErrorRepository as Error Store

    User->>Browser: Makes request to /oups
    Browser->>Controller: GET /oups
    Controller->>Controller: triggerException()
    Controller-->>Controller: throws RuntimeException

    Controller->>GlobalExceptionHandler: handleException(ex)
    GlobalExceptionHandler->>Logger: logError("Exception occurred", ex)
    Logger-->>GlobalExceptionHandler: Logged
    
    GlobalExceptionHandler->>ErrorRepository: storeError(errorDetails)
    ErrorRepository-->>GlobalExceptionHandler: Stored
    
    alt REST API Error
        GlobalExceptionHandler->>Browser: JSON error response
        Browser-->>User: Error message displayed
    else Web UI Error
        GlobalExceptionHandler->>Browser: Redirect to /error.html
        Browser-->>User: Error page displayed
    end

    Note over GlobalExceptionHandler,ErrorRepository: Centralized error handling<br/>with logging and persistence
```

**Description**: System handles unexpected exceptions globally. Triggered by any unhandled exception. Provides centralized error handling with logging, error storage, and appropriate user feedback.

## 7. Language Switching Workflow

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant LocaleChangeInterceptor
    participant SessionLocaleResolver
    participant MessageSource
    participant ResourceBundle

    User->>Browser: Adds ?lang=de to URL
    Browser->>Controller: GET /owners?lang=de
    
    Note over Controller,Browser: Request processing pipeline
    Controller->>LocaleChangeInterceptor: preHandle(request)
    LocaleChangeInterceptor->>SessionLocaleResolver: setLocale(request, Locale.GERMAN)
    SessionLocaleResolver-->>LocaleChangeInterceptor: Locale set in session
    LocaleChangeInterceptor-->>Controller: Continue processing

    Controller->>MessageSource: getMessage("welcome.title", null, Locale.GERMAN)
    MessageSource->>ResourceBundle: messages_de.properties
    ResourceBundle-->>MessageSource: German message
    MessageSource-->>Controller: "Willkommen bei der Tierklinik"
    
    Controller->>Browser: HTML response with German text
    Browser-->>User: Shows German interface

    Note over SessionLocaleResolver,ResourceBundle: Session-based locale<br/>with resource bundle resolution
```

**Description**: User switches interface language. Triggered by adding lang parameter to URL. Uses interceptor pattern with session storage and resource bundle message resolution.

## 8. Microservice Orchestration - Owner Detail with Pets and Visits

```mermaid
sequenceDiagram
    participant User
    participant APIGateway
    participant OwnerService
    participant PetService
    participant VisitService
    participant EventBus as Kafka
    participant OwnerDB
    participant PetDB
    participant VisitDB

    User->>APIGateway: GET /api/owners/{id}
    APIGateway->>OwnerService: GET /owners/{id}
    
    OwnerService->>OwnerDB: SELECT * FROM owners WHERE id=?
    OwnerDB-->>OwnerService: Owner data
    OwnerService-->>APIGateway: Owner DTO
    
    APIGateway->>PetService: GET /pets/owner/{id}
    PetService->>PetDB: SELECT * FROM pets WHERE owner_id=?
    PetDB-->>PetService: List<Pet>
    PetService-->>APIGateway: Pet DTOs list
    
    loop For each pet
        APIGateway->>VisitService: GET /visits/pet/{petId}
        VisitService->>VisitDB: SELECT * FROM visits WHERE pet_id=?
        VisitDB-->>VisitService: List<Visit>
        VisitService-->>APIGateway: Visit DTOs
    end
    
    APIGateway->>APIGateway: aggregateResponse(Owner, Pets[], Visits[][])
    APIGateway-->>User: Complete owner detail with pets and visits

    Note over APIGateway:EventBus: Event-driven updates
    OwnerService->>EventBus: publish(OwnerUpdated)
    EventBus->>PetService: consume(OwnerUpdated)
    EventBus->>VisitService: consume(OwnerUpdated)
    
    Note over APIGateway: Synchronous request<br/>with asynchronous event propagation
```

**Description**: Retrieves complete owner detail including all pets and visits in microservice architecture. Uses synchronous request aggregation with asynchronous event propagation for data consistency.

## 9. Saga Pattern for Visit Scheduling (Microservice)

```mermaid
sequenceDiagram
    participant User
    participant APIGateway
    participant VisitSagaOrchestrator
    participant TimeSlotService
    participant VisitService
    participant NotificationService
    participant CompensationService
    participant EventBus as Kafka

    User->>APIGateway: POST /api/visits/schedule
    APIGateway->>VisitSagaOrchestrator: initiateSaga(ScheduleVisitCommand)
    
    VisitSagaOrchestrator->>TimeSlotService: reserveSlot(visitRequest)
    TimeSlotService->>TimeSlotService: checkAvailability()
    TimeSlotService->>EventBus: publish(TimeSlotReserved)
    
    EventBus->>VisitSagaOrchestrator: consume(TimeSlotReserved)
    VisitSagaOrchestrator->>VisitService: createVisit(visitRequest)
    VisitService->>EventBus: publish(VisitCreated)
    
    EventBus->>VisitSagaOrchestrator: consume(VisitCreated)
    VisitSagaOrchestrator->>NotificationService: sendOwnerNotification(visit)
    NotificationService->>EventBus: publish(NotificationSent)
    
    EventBus->>VisitSagaOrchestrator: consume(NotificationSent)
    VisitSagaOrchestrator->>EventBus: publish(SagaCompleted)
    
    alt Failure during visit creation
        TimeSlotService->>CompensationService: releaseSlot(slotId)
        CompensationService->>EventBus: publish(SagaCompensated)
    end
    
    VisitSagaOrchestrator-->>APIGateway: SagaResult
    APIGateway-->>User: Visit scheduling confirmation

    Note over VisitSagaOrchestrator,EventBus: Distributed transaction<br/>with compensation
```

**Description**: Coordinates visit scheduling across multiple microservices using Saga pattern. Ensures eventual consistency with compensation for failed operations.

## 10. Circuit Breaker Pattern for Service Resilience

```mermaid
sequenceDiagram
    participant Client
    participant ServiceA
    participant CircuitBreaker
    participant ServiceB
    participant FallbackHandler
    participant MetricsCollector

    Client->>ServiceA: processRequest()
    ServiceA->>CircuitBreaker: callServiceB()
    
    alt Service B is healthy
        CircuitBreaker->>ServiceB: remoteCall()
        ServiceB-->>CircuitBreaker: successful response
        CircuitBreaker->>MetricsCollector: recordSuccess()
        CircuitBreaker-->>ServiceA: response
        ServiceA-->>Client: processed response
    else Service B is failing (Circuit OPEN)
        CircuitBreaker->>MetricsCollector: recordFailure()
        CircuitBreaker->>FallbackHandler: executeFallback()
        FallbackHandler-->>CircuitBreaker: fallback response
        CircuitBreaker-->>ServiceA: cached/fallback response
        ServiceA-->>Client: degraded response
        
        Note over CircuitBreaker: Circuit transitions:<br/>CLOSED -> OPEN -> HALF_OPEN -> CLOSED
    else Service B timeout
        CircuitBreaker->>MetricsCollector: recordTimeout()
        CircuitBreaker->>FallbackHandler: executeFallback()
        FallbackHandler-->>CircuitBreaker: timeout fallback
        CircuitBreaker-->>ServiceA: timeout fallback
        ServiceA-->>Client: timeout response
    end

    Note over CircuitBreaker,MetricsCollector: Resilience pattern with<br/>health monitoring
```

**Description**: Implements circuit breaker pattern for inter-service communication resilience. Provides fallback mechanisms and automatic service health monitoring with circuit state transitions.

## 11. Event-Driven Owner Data Synchronization

```mermaid
sequenceDiagram
    participant OwnerService
    participant EventBus as Kafka
    participant PetService
    participant VisitService
    participant SearchService
    participant AuditService
    participant OwnerDB
    participant PetDB
    participant VisitDB
    participant SearchIndex

    OwnerService->>OwnerDB: UPDATE owners SET ...
    OwnerDB-->>OwnerService: Update successful
    
    OwnerService->>EventBus: publish(OwnerUpdated{ownerId, timestamp})
    
    par Parallel event processing
        EventBus->>PetService: consume(OwnerUpdated)
        PetService->>PetDB: UPDATE pets SET owner_info=... WHERE owner_id=?
        PetDB-->>PetService: Updated
        PetService->>EventBus: publish(PetsUpdated)
        
    and
        EventBus->>VisitService: consume(OwnerUpdated)
        VisitService->>VisitDB: UPDATE visits SET owner_details=... WHERE pet_id IN (SELECT id FROM pets WHERE owner_id=?)
        VisitDB-->>VisitService: Updated
        VisitService->>EventBus: publish(VisitsUpdated)
        
    and
        EventBus->>SearchService: consume(OwnerUpdated)
        SearchService->>SearchIndex: UPDATE owner_document_{ownerId}
        SearchIndex-->>SearchService: Index updated
        
    and
        EventBus->>AuditService: consume(OwnerUpdated)
        AuditService->>AuditService: logOwnerChange(event)
    end

    Note over EventBus: At-least-once delivery<br/>with idempotent processing
```

**Description**: Synchronizes owner data changes across multiple services using event-driven architecture. Ensures eventual consistency with parallel processing and audit logging.

## 12. Distributed Transaction with Outbox Pattern

```mermaid
sequenceDiagram
    participant Client
    participant OrderService
    participant OutboxTable
    participant Database
    participant PollingService
    participant EventBus as Kafka
    participant InventoryService
    participant PaymentService

    Client->>OrderService: placeOrder(OrderRequest)
    
    OrderService->>Database: BEGIN TRANSACTION
    OrderService->>Database: INSERT INTO orders (...)
    OrderService->>Database: INSERT INTO outbox (event_type, payload, status='PENDING')
    OrderService->>Database: COMMIT
    Database-->>OrderService: Transaction committed
    
    OrderService-->>Client: Order accepted (async processing)
    
    loop Polling for outbox events
        PollingService->>OutboxTable: SELECT * FROM outbox WHERE status='PENDING' LIMIT 100
        OutboxTable-->>PollingService: Pending events
        
        PollingService->>OutboxTable: UPDATE SET status='PROCESSING' WHERE id IN (...)
        PollingService->>EventBus: publish(OrderPlaced{orderId, items})
        
        alt Successfully published
            EventBus-->>PollingService: ack
            PollingService->>OutboxTable: UPDATE SET status='COMPLETED' WHERE id IN (...)
        else Failed to publish
            PollingService->>OutboxTable: UPDATE SET status='FAILED' WHERE id IN (...)
        end
    end
    
    par Event consumption
        EventBus->>InventoryService: consume(OrderPlaced)
        InventoryService->>InventoryService: reserveInventory(items)
        
    and
        EventBus->>PaymentService: consume(OrderPlaced)
        PaymentService->>PaymentService: processPayment(orderDetails)
    end

    Note over OutboxTable,EventBus: Reliable event delivery<br/>without 2PC
```

**Description**: Implements distributed transaction using outbox pattern for reliable event publishing. Ensures exactly-once semantics between database operations and event publishing.

## 13. API Request Flow with Distributed Tracing

```mermaid
sequenceDiagram
    participant Client
    participant Gateway
    participant AuthService
    participant RateLimiter
    participant ServiceA
    participant ServiceB
    participant JaegerCollector
    participant ZipkinCollector

    Client->>Gateway: GET /api/resource (with JWT)
    
    Gateway->>AuthService: validateToken(jwt)
    AuthService-->>Gateway: token valid
    
    Gateway->>RateLimiter: checkRateLimit(clientId)
    RateLimiter-->>Gateway: allowed
    
    Gateway->>Gateway: createSpan("gateway-request")
    Gateway->>ServiceA: GET /internal/resource (headers: X-Trace-ID, X-Span-ID)
    
    ServiceA->>ServiceA: createChildSpan("service-a-process")
    ServiceA->>ServiceB: GET /service-b/data (headers: X-Trace-ID, X-Span-ID, X-Parent-Span-ID)
    
    ServiceB->>ServiceB: createChildSpan("service-b-query")
    ServiceB-->>ServiceA: response (with trace headers)
    ServiceA-->>Gateway: response (with trace headers)
    
    Note over Gateway,JaegerCollector: Trace propagation
    Gateway->>JaegerCollector: submitSpan(span_data)
    ServiceA->>JaegerCollector: submitSpan(span_data)
    ServiceB->>JaegerCollector: submitSpan(span_data)
    
    Gateway-->>Client: final response

    Note over JaegerCollector,ZipkinCollector: Distributed tracing<br/>for observability
```

**Description**: Demonstrates complete request flow with distributed tracing across services. Shows span creation, propagation, and collection for end-to-end observability.

## 14. Cache Invalidation and Update Workflow

```mermaid
sequenceDiagram
    participant Admin
    participant VetController
    participant VetRepository
    participant CacheManager
    participant Database
    participant MessageBroker as Redis Pub/Sub
    participant OtherInstances

    Admin->>VetController: PUT /vets/{id}
    VetController->>VetService: updateVet(vetDTO)
    VetService->>VetRepository: save(vet)
    
    VetRepository->>Database: BEGIN TRANSACTION
    VetRepository->>Database: UPDATE vets SET ...
    VetRepository->>Database: COMMIT
    Database-->>VetRepository: Update successful
    
    VetRepository->>CacheManager: evict("vets")
    CacheManager->>CacheManager: clearCacheEntry("vets")
    
    VetRepository->>MessageBroker: publish("cache-invalidation", {key: "vets"})
    
    par Broadcast to other instances
        MessageBroker->>OtherInstances: cache-invalidation message
        loop Each instance
            OtherInstances->>CacheManager: evict("vets")
        end
    end
    
    VetRepository-->>VetService: Updated vet
    VetService-->>VetController: Success
    VetController-->>Admin: Update confirmation

    Note over MessageBroker,OtherInstances: Distributed cache<br/>invalidation strategy
```

**Description**: Handles cache invalidation across multiple application instances when data changes. Uses distributed messaging to ensure cache consistency in cluster environment.

## 15. Database Migration and Schema Evolution

```mermaid
sequenceDiagram
    participant DevOps
    participant MigrationTool as Flyway
    participant Database
    participant SchemaRegistry
    participant CompatibilityChecker
    participant Application

    DevOps->>MigrationTool: flyway migrate
    
    MigrationTool->>Database: Check schema_version table
    Database-->>MigrationTool: Current version: v1.2
    
    MigrationTool->>MigrationTool: Load pending migrations
    Note over MigrationTool: V1.3__add_pet_metadata.sql<br/>V1.4__create_visit_notes.sql
    
    MigrationTool->>SchemaRegistry: validateSchema(V1.4)
    SchemaRegistry->>CompatibilityChecker: checkBackwardCompatibility()
    CompatibilityChecker-->>SchemaRegistry: Compatible
    SchemaRegistry-->>MigrationTool: Validation passed
    
    MigrationTool->>Database: BEGIN TRANSACTION
    MigrationTool->>Database: ALTER TABLE pets ADD COLUMN metadata JSON
    Database-->>MigrationTool: Column added
    MigrationTool->>Database: CREATE TABLE visit_notes ...
    Database-->>MigrationTool: Table created
    MigrationTool->>Database: UPDATE schema_version SET version='V1.4'
    Database-->>MigrationTool: Version updated
    MigrationTool->>Database: COMMIT
    Database-->>MigrationTool: Migration complete
    
    MigrationTool->>Application: triggerReload()
    Application->>Application: Refresh entity mappings
    Application-->>MigrationTool: Ready for V1.4
    MigrationTool-->>DevOps: Migration successful

    Note over MigrationTool,Database: Schema evolution with<br/>rollback capability
```

**Description**: Manages database schema evolution using migration tools. Includes validation, compatibility checking, and safe deployment of schema changes with rollback capability.