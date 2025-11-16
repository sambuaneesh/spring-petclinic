| Component Name | HTTP Method | Endpoint Path | Brief Description |
|----------------|-------------|---------------|-------------------|
| WelcomeController | GET | `/` | Welcome page |
| OwnerController | GET | `/owners/new` | Owner creation form |
| OwnerController | POST | `/owners/new` | Create owner |
| OwnerController | GET | `/owners/find` | Owner search form |
| OwnerController | GET | `/owners` | Owner search results with pagination and lastName filter |
| OwnerController | GET | `/owners/{id}` | Owner details |
| OwnerController | GET | `/owners/{id}/edit` | Owner edit form |
| OwnerController | POST | `/owners/{id}/edit` | Update owner |
| PetController | GET | `/owners/{ownerId}/pets/new` | Pet creation form |
| PetController | POST | `/owners/{ownerId}/pets/new` | Create pet |
| PetController | GET | `/owners/{ownerId}/pets/{petId}/edit` | Pet edit form |
| PetController | POST | `/owners/{ownerId}/pets/{petId}/edit` | Update pet |
| VisitController | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Visit creation form |
| VisitController | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Create visit |
| VetController | GET | `/vets.html` | Veterinarian list (HTML) |
| VetController | GET | `/vets` | Veterinarian list (JSON/XML) |
| CrashController | GET | `/oups` | Error simulation for testing |