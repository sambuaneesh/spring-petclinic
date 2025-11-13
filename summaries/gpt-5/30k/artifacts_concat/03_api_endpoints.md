| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | / | Welcome/home page view |
| VetController | GET | /vets.html | Vets HTML list with pagination (query param: page) |
| VetController | GET | /vets | Return all veterinarians as JSON (XML also supported) |
| OwnerController | GET | /owners/new | Display create owner form |
| OwnerController | POST | /owners/new | Create owner; redirect to /owners/{id} |
| OwnerController | GET | /owners/find | Display owner search form |
| OwnerController | GET | /owners | Search owners by lastName with pagination; returns list or redirects if single match |
| OwnerController | GET | /owners/{ownerId}/edit | Display edit owner form |
| OwnerController | POST | /owners/{ownerId}/edit | Update owner; redirect to /owners/{ownerId} |
| OwnerController | GET | /owners/{ownerId} | Owner details view (includes pets and visits) |
| PetController | GET | /owners/{ownerId}/pets/new | Display create pet form |
| PetController | POST | /owners/{ownerId}/pets/new | Validate and create pet; redirect to owner details |
| PetController | GET | /owners/{ownerId}/pets/{petId}/edit | Display edit pet form |
| PetController | POST | /owners/{ownerId}/pets/{petId}/edit | Validate and update pet; redirect to owner details |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits/new | Display create visit form |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits/new | Validate and add visit; redirect to owner details |
| CrashController | GET | /oups | Deliberately throws RuntimeException to test error handling |
| Actuator | GET | /livez | Liveness probe endpoint |
| Actuator | GET | /readyz | Readiness probe endpoint |
| Actuator | GET | /actuator/* | Operational endpoints (health, info, metrics, etc.; all actuator endpoints exposed) |
| H2 Console (dev) | GET | /h2-console | H2 database web console (dev-only) |