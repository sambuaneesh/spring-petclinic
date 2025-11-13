| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | / | Home/welcome page. |
| CrashController | GET | /oups | Throws a RuntimeException to test error handling (renders error page). |
| VetController | GET | /vets.html | HTML veterinarians list (paginated; page query param is 1-based). |
| VetController | GET | /vets | JSON list of vets with specialties (cached). |
| OwnerController | GET | /owners/new | Owner creation form. |
| OwnerController | POST | /owners/new | Create owner; validates and redirects to /owners/{id}. |
| OwnerController | GET | /owners/find | Owner search form. |
| OwnerController | GET | /owners | Process owner search; supports lastName prefix and page (1-based); renders list or redirects to single match. |
| OwnerController | GET | /owners/{ownerId}/edit | Edit owner form. |
| OwnerController | POST | /owners/{ownerId}/edit | Update owner; validates ID match; redirects to details. |
| OwnerController | GET | /owners/{ownerId} | Owner details view. |
| PetController | GET | /owners/{ownerId}/pets/new | Create pet form (new pet attached to owner). |
| PetController | POST | /owners/{ownerId}/pets/new | Create pet; validates (name/type/birthDate, unique name per owner); redirects to owner details. |
| PetController | GET | /owners/{ownerId}/pets/{petId}/edit | Edit pet form. |
| PetController | POST | /owners/{ownerId}/pets/{petId}/edit | Update pet; validates and enforces unique name; redirects to owner details. |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits/new | Create visit form for a pet. |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits/new | Create visit; description required; redirects to owner details. |
| Spring Boot Actuator | GET | /actuator | Actuator index; all web endpoints are exposed. |
| Spring Boot Actuator | GET | /actuator/health | Health endpoint. |
| Spring Boot Actuator | GET | /actuator/info | Application info. |
| Spring Boot Actuator | GET | /actuator/metrics | Metrics index. |
| Spring Boot Actuator | GET | /actuator/caches | Cache statistics (JCache/Caffeine). |
| Spring Boot Actuator | GET | /actuator/beans | Beans endpoint. |
| Spring Boot Actuator | GET | /actuator/conditions | Auto-configuration conditions. |
| Spring Boot Actuator | GET | /actuator/configprops | Configuration properties. |
| Spring Boot Actuator | GET | /actuator/env | Environment properties. |
| Spring Boot Actuator | GET | /actuator/loggers | Loggers endpoint. |
| Spring Boot Actuator | GET | /actuator/mappings | Request mappings. |
| Spring Boot Actuator | GET | /actuator/threaddump | Thread dump. |
| Spring Boot Actuator | GET | /actuator/heapdump | Heap dump. |
| Spring Boot Actuator | GET | /livez | Liveness probe alias (enabled when probes.add-additional-paths=true). |
| Spring Boot Actuator | GET | /readyz | Readiness probe alias (enabled when probes.add-additional-paths=true). |
| H2 Console (Dev) | GET | /h2-console | Embedded H2 database web console (development profile). |