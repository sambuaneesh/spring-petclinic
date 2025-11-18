
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | `/` | Displays the home page |
| OwnerController | GET | `/owners/new` | Displays the form for creating a new owner |
| OwnerController | POST | `/owners/new` | Processes the form for creating a new owner |
| OwnerController | GET | `/owners/find` | Displays the owner search form |
| OwnerController | POST | `/owners/find` | Processes the owner search |
| OwnerController | GET | `/owners` | Displays a paginated list of owners |
| OwnerController | GET | `/owners/{id}` | Displays details for a specific owner |
| OwnerController | GET | `/owners/{id}/edit` | Displays the form for editing an owner |
| OwnerController | POST | `/owners/{id}/edit` | Processes the form for updating an owner |
| PetController | GET | `/owners/{ownerId}/pets/new` | Displays the form for adding a new pet to an owner |
| PetController | POST | `/owners/{ownerId}/pets/new` | Processes the form for adding a new pet |
| PetController | GET | `/owners/{ownerId}/pets/{petId}/edit` | Displays the form for editing a pet's details |
| PetController | POST | `/owners/{ownerId}/pets/{petId}/edit` | Processes the form for updating a pet's details |
| VisitController | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Displays the form for adding a new visit for a pet |
| VisitController | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Processes the form for creating a new visit |
| VetController | GET | `/vets.html` | Displays a paginated HTML list of veterinarians |
| VetController | GET | `/vets` | Returns a JSON list of all veterinarians |
| CrashController | GET | `/oups` | Triggers and displays an error page for demonstration |