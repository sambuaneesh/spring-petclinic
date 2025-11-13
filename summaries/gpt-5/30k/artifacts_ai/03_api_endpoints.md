| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | / | Welcome home page. |
| CrashController | GET | /oups | Intentionally throws an error to demonstrate error handling. |
| OwnerController | GET | /owners/find | Owners search page. |
| OwnerController | GET | /owners | Paginated owners search (query params: lastName, page); single match redirects to details. |
| OwnerController | GET | /owners/new | Create owner form. |
| OwnerController | POST | /owners/new | Create owner; redirects to /owners/{id}. |
| OwnerController | GET | /owners/{ownerId} | Owner details including pets and visits. |
| OwnerController | GET | /owners/{ownerId}/edit | Edit owner form. |
| OwnerController | POST | /owners/{ownerId}/edit | Update owner; redirects to details. |
| PetController | GET | /owners/{ownerId}/pets/new | Create pet form. |
| PetController | POST | /owners/{ownerId}/pets/new | Create pet with validation; redirects to owner details. |
| PetController | GET | /owners/{ownerId}/pets/{petId}/edit | Edit pet form. |
| PetController | POST | /owners/{ownerId}/pets/{petId}/edit | Update pet with validation; redirects to owner details. |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits/new | Create visit form. |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits/new | Create visit; validates and associates to pet; redirects to owner details. |
| VetController | GET | /vets.html | HTML vets list (page param; page size 5). |
| VetController | GET | /vets | All vets as JSON (or XML via content negotiation). |
| Actuator | GET | /actuator/* | Spring Boot management endpoints (health, info, metrics, etc.). |
| Actuator | GET | /livez | Liveness probe (health). |
| Actuator | GET | /readyz | Readiness probe (health). |
| H2 Console | GET | /h2-console | Embedded H2 database console (when H2 is active). |
| Static Resources | GET | /resources/css/petclinic.css | Compiled CSS asset. |
| Static Resources | GET | /webjars/** | WebJars static assets (e.g., Bootstrap, Font Awesome). |