
# Spring Pet Clinic - Dynamic Interaction Flows

## 1. Owner Registration and Search Workflow

```mermaid
sequenceDiagram
    participant User
    participant OwnerController as OC
    participant OwnerRepository as OR
    participant Database as DB
    participant Validator as Val

    User->>+OC: Search owner by last name
    OC->>+OR: findByLastName(lastName)
    OR->>+DB: Execute query with wildcard search
    DB-->>-OR: Return owner results
    OR-->>-OC: List<Owner> results

    alt No results found
        OC-->>User: Redirect to search form with "not found" message
    else Single result found
        OC-->>User: Redirect to owner details page
    else Multiple results found
        OC-->>User: Display paginated list of owners
    end

    User->>+OC: Initiate new owner registration
    OC-->>User: Display owner creation form

    User->>+OC: Submit owner registration form
    OC->>Val: validate(owner)
    Val-->>OC: Validation result

    alt Validation passes
        OC->>+OR: save(owner)
        OR->>+DB: INSERT into owners table
        DB-->>-OR: Persisted owner with ID
        OR-->>-OC: Owner entity with generated ID
        OC-->>User: Redirect to owner details page
    else Validation fails
        OC-->>User: Return to form with error messages
    end
```

**Description**: This workflow handles owner search functionality with intelligent routing (single result redirects to details, multiple results show list, no results show search form) and new owner registration with validation and persistence. Communication is synchronous REST-style with database transactions.

## 2. Pet Management Workflow

```mermaid
sequenceDiagram
    participant User
    participant PetController as PC
    participant OwnerRepository as OR
    participant PetValidator as PV
    participant PetTypeFormatter as PTF
    participant Database as DB

    User->>+PC: Request to add new pet (ownerId)
    PC->>+OR: findById(ownerId)
    OR->>+DB: SELECT owner with pets
    DB-->>-OR: Owner entity
    OR-->>-PC: Owner with pet collection
    PC->>PTF: getAvailablePetTypes()
    PTF-->>PC: List<PetType>
    PC-->>User: Display pet creation form with owner context

    User->>+PC: Submit pet creation form
    PC->>PV: validate(pet, errors)
    PV->>PV: checkPetName(owner, pet)
    PV->>PV: validateBirthDate(pet)
    PV-->>PC: Validation result

    alt Validation passes
        PC->>+OR: save(owner)
        OR->>+DB: INSERT into pets table
        DB-->>-OR: Updated owner with new pet
        OR-->>-PC: Persisted owner-pet relationship
        PC-->>User: Redirect to owner details page
    else Validation fails
        PC-->>User: Return to form with validation errors
    end

    User->>+PC: Request to edit pet (petId)
    PC->>+OR: findPetById(petId)
    OR->>+DB: SELECT pet with owner
    DB-->>-OR: Pet entity
    OR-->>-PC: Pet with owner reference
    PC-->>User: Display pet edit form
```

**Description**: Handles pet creation and editing within owner context, including duplicate name validation, birth date validation, and type formatting. Uses synchronous controller-service-repository pattern with transactional database operations.

## 3. Visit Scheduling Workflow

```mermaid
sequenceDiagram
    participant User
    participant VisitController as VC
    participant OwnerRepository as OR
    participant Database as DB
    participant VisitValidator as VV

    User->>+VC: Initiate new visit (petId, ownerId)
    VC->>+OR: findById(ownerId)
    OR->>+DB: SELECT owner with pets and visits
    DB-->>-OR: Owner with complete relationships
    OR-->>-VC: Owner entity with pet collection
    VC-->>User: Display visit creation form with pet context

    User->>+VC: Submit visit form (date, description)
    VC->>VV: validate(visit, errors)
    VV-->>VC: Validation result

    alt Validation passes
        VC->>VC: createVisit(pet, date, description)
        VC->>+OR: save(owner)
        OR->>+DB: INSERT into visits table
        DB-->>-OR: Updated owner with new visit
        OR-->>-VC: Persisted visit record
        VC-->>User: Redirect to owner details with updated history
    else Validation fails
        VC-->>User: Return to form with error messages
    end
```

**Description**: Manages appointment scheduling for pets, creating chronological medical history. Includes form validation, date validation, and updates the pet-owner relationship with new visit records through synchronous database transactions.

## 4. Veterinarian Directory Access with Caching

```mermaid
sequenceDiagram
    participant User
    participant VetController as VC
    participant CacheManager as CM
    participant VetRepository as VR
    participant Database as DB
    participant VetCache as Cache

    User->>+VC: Request veterinarian directory (/vets.html)
    VC->>CM: getCache("vets")
    CM-->>VC: VetCache instance

    alt Cache miss or first request
        VC->>+VR: findAll()
        VR->>+DB: SELECT vets with specialties
        DB-->>-VR: List<Vet> with specialties
        VR-->>-VC: Complete vet list
        VC->>Cache: put("vets", vetList)
        Cache-->>VC: Cached successfully
    else Cache hit
        VC->>Cache: get("vets")
        Cache-->>VC: Cached vet list
    end

    User->>+VC: Request vet API (/vets)
    VC->>Cache: get("vets") or VR.findAll()
    Cache-->>VC: Vet list
    VC->>VC: createVetsDTO(vetList)
    VC-->>User: Return JSON/XML response
```

**Description**: Demonstrates cached data retrieval for veterinarian directory, showing both web UI and REST API access patterns. Uses JCache for performance optimization, reducing database load for frequently accessed, infrequently changing data.

## 5. System Error Handling Flow

```mermaid
sequenceDiagram
    participant User
    participant Controller as C
    participant CrashController as CC
    participant GlobalExceptionHandler as GEH
    participant ErrorLogger as EL
    participant ErrorView as EV

    User->>+C: Normal request processing
    C->>C: Business logic execution

    alt Unexpected error occurs
        C-->>CC: Propagate RuntimeException
        CC->>+GEH: handleException(ex)
        GEH->>EL: logError(exception, request)
        EL-->>GEH: Error logged
        GEH->>EV: prepareErrorModel(exception)
        EV-->>GEH: Error model prepared
        GEH-->>User: Return error page with details
    else Controlled error test
        User->>+CC: Access /oups endpoint
        CC->>CC: throw new RuntimeException("Expected")
        CC-->>+GEH: Exception propagated
        GEH->>EL: logControlledError(exception)
        EL-->>GEH: Logged successfully
        GEH-->>User: Return error view
    end
```

**Description**: Illustrates the application's error handling mechanism, including both unexpected errors and controlled testing scenarios. Shows global exception handling, logging, and graceful user feedback through error views.

## 6. Complete Owner-Pet-Visit Management Workflow

```mermaid
sequenceDiagram
    participant Owner
    participant OwnerController as OC
    participant PetController as PC
    participant VisitController as VC
    participant ServiceLayer as SL
    participant RepositoryLayer as RL
    participant Database as DB

    Owner->>+OC: Register new owner
    OC->>+SL: saveOwner(owner)
    SL->>+RL: ownerRepository.save(owner)
    RL->>+DB: INSERT owner
    DB-->>-RL: Owner with ID
    RL-->>-SL: Persisted owner
    SL-->>-OC: Success
    OC-->>Owner: Owner created

    Owner->>+PC: Add pet to owner
    PC->>+SL: addPetToOwner(owner, pet)
    SL->>+RL: ownerRepository.save(owner)
    RL->>+DB: INSERT pet with owner_id
    DB-->>-RL: Pet created
    RL-->>-SL: Updated owner
    SL-->>-PC: Pet added
    PC-->>Owner: Pet added to profile

    Owner->>+VC: Schedule visit for pet
    VC->>+SL: createVisit(pet, visit)
    SL->>+RL: ownerRepository.save(owner)
    RL->>+DB: INSERT visit with pet_id
    DB-->>-RL: Visit created
    RL-->>-SL: Visit added to pet history
    SL-->>-VC: Visit scheduled
    VC-->>Owner: Visit confirmed

    Owner->>+OC: View complete profile
    OC->>+SL: getOwnerWithDetails(ownerId)
    SL->>+RL: ownerRepository.findById(ownerId)
    RL->>+DB: SELECT owner JOIN pets JOIN visits
    DB-->>-RL: Complete owner profile
    RL-->>-SL: Owner with pets and visits
    SL-->>-OC: Full profile data
    OC-->>Owner: Display complete medical history
```

**Description**: Comprehensive workflow showing the complete lifecycle from owner registration through pet management to visit scheduling and profile viewing. Demonstrates transactional data persistence and the relationship management between entities in the veterinary clinic system.