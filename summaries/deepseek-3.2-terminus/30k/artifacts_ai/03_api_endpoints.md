| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---------------|-------------|---------------|-------------------|
| WelcomeController | GET | `/` | Welcome page |
| CrashController | GET | `/oups` | Error simulation endpoint for testing |
| OwnerController | GET | `/owners/new` | Show owner creation form |
| OwnerController | POST | `/owners/new` | Create new owner |
| OwnerController | GET | `/owners/find` | Show owner search form |
| OwnerController | GET | `/owners` | Search owners by last name (paginated, 5 per page) |
| OwnerController | GET | `/owners/{ownerId}/edit` | Show owner edit form |
| OwnerController | POST | `/owners/{ownerId}/edit` | Update owner |
| OwnerController | GET | `/owners/{ownerId}` | Show owner details |
| PetController | GET | `/owners/{ownerId}/pets/new` | Show pet creation form |
| PetController | POST | `/owners/{ownerId}/pets/new` | Create new pet |
| PetController | GET | `/owners/{ownerId}/pets/{petId}/edit` | Show pet edit form |
| PetController | POST | `/owners/{ownerId}/pets/{petId}/edit` | Update pet |
| VisitController | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Show visit creation form |
| VisitController | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Create new visit |
| VetController | GET | `/vets.html` | HTML vet list with pagination (5 per page) |
| VetController | GET | `/vets` | JSON/XML vet list (all vets) |