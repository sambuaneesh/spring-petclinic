
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| OwnerController | GET | /owners | List owners with search and pagination |
| OwnerController | POST | /owners | Create a new owner |
| PetController | GET | /owners/{ownerId}/pets | List pets for a specific owner |
| PetController | POST | /owners/{ownerId}/pets | Add a new pet to an owner |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits | List visits for a specific pet |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits | Schedule a new visit for a pet |
| VetController | GET | /vets.html | Display vet listing page (HTML) |
| VetController | GET | /vets | Get vet listing (JSON) |
| WelcomeController | GET | / | Homepage |
| CrashController | GET | /oups | Error testing endpoint |
| Actuator | GET | /livez | Liveness probe |
| Actuator | GET | /readyz | Readiness probe |