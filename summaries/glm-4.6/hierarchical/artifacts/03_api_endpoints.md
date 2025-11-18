
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | `/` | Display the application's welcome page. |
| CrashController | GET | `/oups` | Intentionally trigger a runtime exception for testing error handling. |
| OwnerController | GET | `/owners/find` | Search for owners with pagination support. |
| OwnerController | POST | `/owners/new` | Process the form to create a new owner. |
| OwnerController | GET | `/owners/new` | Show the form for creating a new owner. |
| OwnerController | GET | `/owners/{ownerId}` | Display the details of a specific owner and their pets. |
| OwnerController | POST | `/owners/{ownerId}/edit` | Process the form to update an existing owner. |
| OwnerController | GET | `/owners/{ownerId}/edit` | Show the form for editing an existing owner. |
| PetController | POST | `/owners/{ownerId}/pets/new` | Process the form to create a new pet for an owner. |
| PetController | GET | `/owners/{ownerId}/pets/new` | Show the form for adding a new pet to an owner. |
| PetController | POST | `/owners/{ownerId}/pets/{petId}/edit` | Process the form to update an existing pet. |
| PetController | GET | `/owners/{ownerId}/pets/{petId}/edit` | Show the form for editing an existing pet's details. |
| VisitController | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Process the form to create a new visit record for a pet. |
| VisitController | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Show the form for scheduling a new visit for a pet. |
| VetController | GET | `/vets` | Retrieve a list of all veterinarians as JSON/XML (REST API). |
| VetController | GET | `/vets.html` | Display a paginated list of veterinarians in a web page. |