### 1. Find and View Owner Details

-   **Workflow Purpose and Triggers**: This workflow allows a user to search for pet owners by their last name. The trigger is the user submitting the "Find Owners" form. The system displays a list of matching owners, or if only one match is found, redirects directly to that owner's detail page.
-   **Communication Patterns**:
    -   **Browser to Server**: Synchronous HTTP GET requests.
    -   **Server to Database**: Synchronous JDBC calls via Spring Data JPA repositories within a database transaction.
    -   **Pattern**: Request-Response, Redirect after GET (for single-result search).

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SpringMVC as Spring DispatcherServlet
    participant OwnerController
    participant OwnerRepository
    participant Database

    User->>Browser: Clicks 'Find Owners' link
    Browser->>SpringMVC: GET /owners/find
    SpringMVC->>OwnerController: initFindForm()
    OwnerController->>SpringMVC: Returns view name "owners/findOwners"
    SpringMVC->>Browser: Renders and returns findOwners.html
    User->>Browser: Enters last name 'Davis' and submits form
    Browser->>SpringMVC: GET /owners?lastName=Davis
    SpringMVC->>OwnerController: processFindForm(owner, result, model)
    OwnerController->>OwnerRepository: findByLastNameStartingWith("Davis", pageable)
    OwnerRepository->>Database: SELECT * FROM owners WHERE last_name LIKE 'Davis%' LIMIT 5 OFFSET 0
    Database-->>OwnerRepository: Returns owner records
    OwnerRepository-->>OwnerController: Returns Page<Owner> with 2 results
    OwnerController->>SpringMVC: addPaginationModel() adds owners to model and returns view "owners/ownersList"
    SpringMVC->>Browser: Renders and returns ownersList.html
    User->>Browser: Clicks on an owner's name (e.g., 'Betty Davis' with id=2)
    Browser->>SpringMVC: GET /owners/2
    SpringMVC->>OwnerController: showOwner(ownerId=2)
    OwnerController->>OwnerRepository: findById(2)
    OwnerRepository->>Database: SELECT * FROM owners/pets/visits WHERE owner_id=2
    Database-->>OwnerRepository: Returns owner, pets, and visits records
    OwnerRepository-->>OwnerController: Returns Owner object with pets
    OwnerController->>SpringMVC: Returns ModelAndView("owners/ownerDetails", owner)
    SpringMVC->>Browser: Renders and returns ownerDetails.html
```

### 2. Add a New Owner

-   **Workflow Purpose and Triggers**: This workflow enables the creation of a new pet owner. It is triggered when a user clicks the "Add Owner" button and submits the new owner form.
-   **Communication Patterns**:
    -   **Browser to Server**: Synchronous HTTP GET (for form) and POST (for submission).
    -   **Server-Side**: Spring MVC validation (`@Valid`, `BindingResult`).
    -   **Server to Database**: Synchronous JDBC write operation via Spring Data JPA within a transaction.
    -   **Pattern**: Request-Response, Redirect-after-Post.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SpringMVC as Spring DispatcherServlet
    participant OwnerController
    participant OwnerRepository
    participant Database

    User->>Browser: Clicks 'Add Owner' button
    Browser->>SpringMVC: GET /owners/new
    SpringMVC->>OwnerController: initCreationForm()
    OwnerController->>SpringMVC: Returns view "owners/createOrUpdateOwnerForm"
    SpringMVC->>Browser: Renders and returns the new owner form

    User->>Browser: Fills form and clicks 'Add Owner'
    Browser->>SpringMVC: POST /owners/new with owner data
    SpringMVC->>OwnerController: processCreationForm(@Valid owner, result)

    alt Form data is valid
        OwnerController->>OwnerRepository: save(owner)
        OwnerRepository->>Database: INSERT INTO owners (...) VALUES (...)
        Database-->>OwnerRepository: Returns new owner with generated ID
        OwnerRepository-->>OwnerController: Returns saved Owner object
        OwnerController->>SpringMVC: Returns "redirect:/owners/{new_id}"
        SpringMVC->>Browser: HTTP 302 Redirect to /owners/{new_id}
        Browser->>SpringMVC: GET /owners/{new_id}
        Note right of Browser: Flow continues as 'View Owner Details'
    else Form data is invalid
        OwnerController->>SpringMVC: Returns view "owners/createOrUpdateOwnerForm" with errors
        SpringMVC->>Browser: Re-renders form with validation error messages
    end
```

### 3. Add a New Pet to an Existing Owner

-   **Workflow Purpose and Triggers**: This workflow allows a user to add a new pet for an existing owner. It is triggered when a user, on an owner's detail page, clicks "Add New Pet".
-   **Communication Patterns**:
    -   **Browser to Server**: Synchronous HTTP GET (for form) and POST (for submission).
    -   **Server-Side**: Custom validation (`PetValidator`).
    -   **Server to Database**: Synchronous read (`findById`) and write (`save`) operations. Adding a pet is cascaded from saving the owner object.
    -   **Pattern**: Request-Response, Redirect-after-Post.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SpringMVC as Spring DispatcherServlet
    participant PetController
    participant OwnerRepository
    participant PetTypeRepository
    participant Database

    User->>Browser: On owner details, clicks 'Add New Pet'
    Browser->>SpringMVC: GET /owners/1/pets/new
    SpringMVC->>PetController: initCreationForm(owner, model)
    PetController->>OwnerRepository: findById(1)
    OwnerRepository-->>PetController: Returns Owner object
    PetController->>PetTypeRepository: findPetTypes()
    PetTypeRepository-->>PetController: Returns List<PetType>
    PetController->>SpringMVC: Returns view "pets/createOrUpdatePetForm" with pet and types in model
    SpringMVC->>Browser: Renders form to add a new pet

    User->>Browser: Fills pet details and submits form
    Browser->>SpringMVC: POST /owners/1/pets/new with pet data
    SpringMVC->>PetController: processCreationForm(owner, @Valid pet, result)
    PetController->>PetController: Performs validation (e.g., duplicate name check)

    alt Pet data is valid
        PetController->>OwnerRepository: save(owner with new pet)
        OwnerRepository->>Database: BEGIN TRANSACTION
        Database->>Database: INSERT INTO pets (...) VALUES (...)
        Database->>Database: COMMIT
        Database-->>OwnerRepository: Success
        OwnerRepository-->>PetController: Returns updated Owner object
        PetController->>SpringMVC: Returns "redirect:/owners/1"
        SpringMVC->>Browser: HTTP 302 Redirect to /owners/1
        Browser->>SpringMVC: GET /owners/1
        Note right of Browser: Flow continues to show owner details with new pet
    else Pet data is invalid
        PetController->>SpringMVC: Returns view "pets/createOrUpdatePetForm" with errors
        SpringMVC->>Browser: Re-renders form with validation errors
    end
```

### 4. Add a Visit for a Pet

-   **Workflow Purpose and Triggers**: This workflow is for scheduling a new visit for a pet. A user triggers it by clicking "Add Visit" for a specific pet on the owner's detail page.
-   **Communication Patterns**:
    -   **Browser to Server**: Synchronous HTTP GET and POST.
    -   **Server-Side**: Form backing object creation using `@ModelAttribute`.
    -   **Server to Database**: Read (`findById`) and write (`save`) operations. The new visit is persisted by cascading from the owner entity.
    -   **Pattern**: Request-Response, Redirect-after-Post.

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SpringMVC as Spring DispatcherServlet
    participant VisitController
    participant OwnerRepository
    participant Database

    User->>Browser: On owner details, clicks 'Add Visit' for a pet
    Browser->>SpringMVC: GET /owners/1/pets/7/visits/new
    SpringMVC->>VisitController: @ModelAttribute loadPetWithVisit(ownerId, petId, model)
    VisitController->>OwnerRepository: findById(1)
    OwnerRepository-->>VisitController: Returns Owner object with its pets
    Note over VisitController: Finds pet with id=7, creates a new Visit, adds to model
    SpringMVC->>VisitController: initNewVisitForm()
    VisitController->>SpringMVC: Returns view "pets/createOrUpdateVisitForm"
    SpringMVC->>Browser: Renders form to add a new visit

    User->>Browser: Enters visit date and description, then submits
    Browser->>SpringMVC: POST /owners/1/pets/7/visits/new with visit data
    SpringMVC->>VisitController: processNewVisitForm(owner, petId, @Valid visit, result)

    alt Visit data is valid
        VisitController->>OwnerRepository: save(owner with new visit)
        OwnerRepository->>Database: BEGIN TRANSACTION
        Database->>Database: INSERT INTO visits (...) VALUES (...)
        Database->>Database: COMMIT
        Database-->>OwnerRepository: Success
        OwnerRepository-->>VisitController: Returns updated Owner object
        VisitController->>SpringMVC: Returns "redirect:/owners/1"
        SpringMVC->>Browser: HTTP 302 Redirect to /owners/1
    else Visit data is invalid
        VisitController->>SpringMVC: Returns view "pets/createOrUpdateVisitForm" with errors
        SpringMVC->>Browser: Re-renders form with validation errors
    end
```

### 5. View Veterinarians List (with Caching)

-   **Workflow Purpose and Triggers**: This workflow displays a list of all veterinarians in the clinic. It is triggered by clicking the "Veterinarians" link. This interaction demonstrates the use of caching to improve performance.
-   **Communication Patterns**:
    -   **Browser to Server**: Synchronous HTTP GET.
    -   **Server-Side**: Caching (`@Cacheable`) is applied at the repository level.
    -   **Server to Database**: A synchronous database query is executed only if the data is not in the cache.
    -   **Pattern**: Request-Response, Cache-Aside pattern (managed by Spring).

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SpringMVC as Spring DispatcherServlet
    participant VetController
    participant VetRepository
    participant Cache as "JCache (vets)"
    participant Database

    User->>Browser: Clicks 'Veterinarians' link
    Browser->>SpringMVC: GET /vets.html
    SpringMVC->>VetController: showVetList(page=1, model)
    VetController->>VetRepository: findAll(pageable)

    alt Cache Miss (First Request)
        VetRepository->>Cache: Check for 'vets' page 1 data
        Cache-->>VetRepository: Data not found
        VetRepository->>Database: SELECT * FROM vets LIMIT 5 OFFSET 0
        Database-->>VetRepository: Returns Vet records
        VetRepository->>Cache: Store result in cache
        Cache-->>VetRepository: Acknowledge storage
        VetRepository-->>VetController: Returns Page<Vet>
        VetController->>SpringMVC: Adds vets to model, returns "vets/vetList"
        SpringMVC->>Browser: Renders and returns vetList.html
    end

    User->>Browser: Navigates away and back to 'Veterinarians'
    Browser->>SpringMVC: GET /vets.html
    SpringMVC->>VetController: showVetList(page=1, model)
    VetController->>VetRepository: findAll(pageable)

    alt Cache Hit (Second Request)
        VetRepository->>Cache: Check for 'vets' page 1 data
        Cache-->>VetRepository: Returns cached Page<Vet> object
        deactivate VetRepository
        VetRepository-->>VetController: Returns Page<Vet> from cache
        VetController->>SpringMVC: Adds vets to model, returns "vets/vetList"
        SpringMVC->>Browser: Renders and returns vetList.html
    end
```

### 6. System Error Handling

-   **Workflow Purpose and Triggers**: This demonstrates the application's global exception handling mechanism. It is triggered when a user accesses a specific URL (`/oups`) designed to throw an unhandled exception.
-   **Communication Patterns**:
    -   **Browser to Server**: Synchronous HTTP GET.
    -   **Server-Side**: Spring Boot's default error handling mechanism intercepts the `RuntimeException`.
    -   **Pattern**: Exception Handling, renders a custom error view instead of the default "Whitelabel Error Page".

```mermaid
sequenceDiagram
    actor User
    participant Browser
    participant SpringMVC as Spring DispatcherServlet
    participant CrashController
    participant ErrorHandler as "Spring Boot Error Handler"
    participant ViewResolver

    User->>Browser: Navigates to the /oups URL
    Browser->>SpringMVC: GET /oups
    SpringMVC->>CrashController: triggerException()
    CrashController-->>SpringMVC: throws RuntimeException
    SpringMVC->>ErrorHandler: Forwards the exception

    ErrorHandler->>ErrorHandler: Processes exception, sets status to 500
    ErrorHandler->>ViewResolver: Request to resolve "error" view
    ViewResolver->>ErrorHandler: Returns error.html template
    ErrorHandler->>Browser: Renders error.html with exception message
```