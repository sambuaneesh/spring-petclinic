| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | / | Landing page; renders the “welcome” view to verify the web stack and provide an entry point. |
| WelcomeController | GET | /welcome | Alternative landing page route; renders the “welcome” view. |
| CrashController | GET | /oups | Deliberately triggers a RuntimeException to exercise global error handling and diagnostics. |
| OwnerController | GET | /owners/find | Initialize the owner search form. |
| OwnerController | GET | /owners | Process owner search by lastName (with pagination); shows list or redirects to a single owner when exactly one match. |
| OwnerController | GET | /owners/{ownerId} | Display details for a specific owner, including pets (eagerly loaded). |
| OwnerController | GET | /owners/new | Initialize the create-owner form. |
| OwnerController | POST | /owners/new | Create a new owner with validation; redirects to the owner details on success. |
| OwnerController | GET | /owners/{ownerId}/edit | Initialize the update-owner form for the given owner. |
| OwnerController | POST | /owners/{ownerId}/edit | Update an existing owner using path-based ID; validation and redirect on success. |
| PetController | GET | /owners/{ownerId}/pets/new | Initialize the create-pet form within an owner context (provides pet types). |
| PetController | POST | /owners/{ownerId}/pets/new | Create a new pet for the owner; validates required fields and unique name per owner. |
| PetController | GET | /owners/{ownerId}/pets/{petId}/edit | Initialize the update-pet form for the specified pet. |
| PetController | POST | /owners/{ownerId}/pets/{petId}/edit | Update an existing pet; validates input and redirects to the owner details. |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits/new | Initialize the create-visit form for a pet (defaults date to today). |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits/new | Create a new visit for the pet; validates and persists via the Owner aggregate. |
| VetController | GET | /vets.html | Render the paginated veterinarians directory as an HTML view. |
| VetController | GET | /vets | Return all veterinarians as a Vets wrapper (JSON/XML via content negotiation). |