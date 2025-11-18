
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | `/` | Home page |
| OwnerController | GET | `/owners/find` | Search form |
| OwnerController | GET | `/owners` | Find owners with pagination |
| OwnerController | GET | `/owners/{ownerId}` | Owner details |
| OwnerController | GET | `/owners/new` | Create owner form |
| OwnerController | POST | `/owners/new` | Create owner |
| OwnerController | GET | `/owners/{ownerId}/edit` | Update owner form |
| OwnerController | POST | `/owners/{ownerId}/edit` | Update owner |
| PetController | GET | `/owners/{ownerId}/pets/new` | Create pet form |
| PetController | POST | `/owners/{ownerId}/pets/new` | Create pet |
| PetController | GET | `/owners/{ownerId}/pets/{petId}/edit` | Update pet form |
| PetController | POST | `/owners/{ownerId}/pets/{petId}/edit` | Update pet |
| VisitController | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Create visit form |
| VisitController | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Create visit |
| VetController | GET | `/vets.html` | Paginated vet listing (HTML) |
| VetController | GET | `/vets` | Full vet list (JSON API) |
| CrashController | GET | `/oups` | Triggers an exception for error handling demonstration |