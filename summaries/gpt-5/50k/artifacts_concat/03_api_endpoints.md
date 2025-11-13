| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | / | Home page (welcome view) |
| CrashController | GET | /oups | Intentional error to demonstrate error handling (HTML or JSON by Accept) |
| OwnerController | GET | /owners/new | Show create owner form |
| OwnerController | POST | /owners/new | Create owner; redirects to /owners/{ownerId} |
| OwnerController | GET | /owners/find | Show owner search form |
| OwnerController | GET | /owners | Process owner search; supports ?lastName=&page=; returns list or redirects if single result |
| OwnerController | GET | /owners/{ownerId} | Show owner details |
| OwnerController | GET | /owners/{ownerId}/edit | Show edit owner form |
| OwnerController | POST | /owners/{ownerId}/edit | Update owner; redirects to /owners/{ownerId} |
| PetController | GET | /owners/{ownerId}/pets/new | Show create pet form (under owner) |
| PetController | POST | /owners/{ownerId}/pets/new | Create pet for owner; redirects to /owners/{ownerId} |
| PetController | GET | /owners/{ownerId}/pets/{petId}/edit | Show edit pet form |
| PetController | POST | /owners/{ownerId}/pets/{petId}/edit | Update pet; redirects to /owners/{ownerId} |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits/new | Show create visit form |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits/new | Create visit for pet; redirects to /owners/{ownerId} |
| VetController | GET | /vets.html | Vets list page (HTML); supports ?page= for pagination |
| VetController | GET | /vets | Vets list (JSON/XML) |
| Actuator | GET | /actuator/* | All Spring Boot Actuator endpoints exposed |
| Actuator | GET | /livez | Liveness probe alias (maps to actuator health) |
| Actuator | GET | /readyz | Readiness probe alias (maps to actuator health) |
| Static Resources | GET | /resources/css/petclinic.css | Compiled CSS asset |
| Static Resources | GET | /webjars/bootstrap/dist/js/bootstrap.bundle.min.js | Bootstrap JS via WebJars |