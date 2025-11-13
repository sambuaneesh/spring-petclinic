| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| `WelcomeController` | GET | `/` | Displays the welcome page. |
| `OwnerController` | GET | `/owners/find` | Displays the owner search form. |
| `OwnerController` | GET | `/owners` | Processes owner search and displays a list or redirects to a single owner's page. |
| `OwnerController` | GET | `/owners/new` | Displays the form to create a new owner. |
| `OwnerController` | POST | `/owners/new` | Processes the creation of a new owner. |
| `OwnerController` | GET | `/owners/{ownerId}` | Displays detailed information for a specific owner. |
| `OwnerController` | GET | `/owners/{ownerId}/edit` | Displays the form to edit an owner's details. |
| `OwnerController` | POST | `/owners/{ownerId}/edit` | Processes the update of an owner's details. |
| `PetController` | GET | `/owners/{ownerId}/pets/new` | Displays the form to add a new pet for an owner. |
| `PetController` | POST | `/owners/{ownerId}/pets/new` | Processes the creation of a new pet. |
| `PetController` | GET | `/owners/{ownerId}/pets/{petId}/edit` | Displays the form to edit a pet's details. |
| `PetController` | POST | `/owners/{ownerId}/pets/{petId}/edit` | Processes the update of a pet's details. |
| `VisitController` | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Displays the form to add a new visit for a pet. |
| `VisitController` | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Processes the creation of a new visit record. |
| `VetController` | GET | `/vets.html` | Displays a paginated HTML list of all veterinarians. |
| `VetController` | GET | `/vets` | Returns a JSON or XML list of all veterinarians via content negotiation. |
| `CrashController` | GET | `/oups` | Triggers a runtime exception to demonstrate and test error handling. |