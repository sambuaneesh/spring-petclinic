| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | / | Render welcome (home) page (HTML). |
| CrashController | GET | /oups | Deliberately throws an error to exercise error handling. |
| VetController | GET | /vets.html | Vets list view (HTML), paginated. |
| VetController | GET | /vets | JSON list of veterinarians (Vets wrapper). |
| OwnerController | GET | /owners/new | Owner creation form (HTML). |
| OwnerController | POST | /owners/new | Create owner; validate; redirect to owner details on success. |
| OwnerController | GET | /owners/find | Owner search form (HTML). |
| OwnerController | GET | /owners | Process owner search; supports lastName filter; paginated; redirects to details if a single match. |
| OwnerController | GET | /owners/{ownerId} | Owner details view (HTML). |
| OwnerController | GET | /owners/{ownerId}/edit | Edit owner form (HTML). |
| OwnerController | POST | /owners/{ownerId}/edit | Update owner; validate path/body ID consistency; redirect to details. |
| PetController | GET | /owners/{ownerId}/pets/new | New pet form for an owner (HTML). |
| PetController | POST | /owners/{ownerId}/pets/new | Create pet; validate duplicate name and birthDate; save via owner. |
| PetController | GET | /owners/{ownerId}/pets/{petId}/edit | Edit pet form (HTML). |
| PetController | POST | /owners/{ownerId}/pets/{petId}/edit | Update pet; validate duplicate name and birthDate; save via owner. |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits/new | New visit form for a pet (HTML). |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits/new | Create visit; validate; add via owner and save. |
| Actuator | GET | /actuator | Actuator endpoint index. |
| Actuator | GET | /actuator/health | Health status. |
| Actuator | GET | /actuator/info | Application info. |
| Actuator | GET | /livez | Liveness probe (Actuator health alias). |
| Actuator | GET | /readyz | Readiness probe (Actuator health alias). |
| H2 Console | GET | /h2-console | Embedded H2 database web console (when H2 profile is active). |