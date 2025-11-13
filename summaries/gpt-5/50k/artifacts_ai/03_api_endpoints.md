| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| System | GET | / | Welcome page (Thymeleaf view) |
| System | GET | /oups | Deliberate crash to demonstrate error handling; returns JSON error if Accept=application/json, otherwise HTML error page |
| Owners | GET | /owners/new | Show create Owner form |
| Owners | POST | /owners/new | Create Owner; redirects to /owners/{id} |
| Owners | GET | /owners/find | Show Owner search form |
| Owners | GET | /owners?page={n}&lastName={prefix?} | Search Owners by lastName prefix (5/page); redirects based on result count |
| Owners | GET | /owners/{ownerId} | Owner details (owner, pets, visits) |
| Owners | GET | /owners/{ownerId}/edit | Show edit Owner form |
| Owners | POST | /owners/{ownerId}/edit | Update Owner; validates path vs model id |
| Pets | GET | /owners/{ownerId}/pets/new | Show create Pet form for Owner |
| Pets | POST | /owners/{ownerId}/pets/new | Create Pet for Owner with validation; redirects to Owner details |
| Pets | GET | /owners/{ownerId}/pets/{petId}/edit | Show edit Pet form |
| Pets | POST | /owners/{ownerId}/pets/{petId}/edit | Update Pet with validation; redirects to Owner details |
| Visits | GET | /owners/{ownerId}/pets/{petId}/visits/new | Show create Visit form for a Pet |
| Visits | POST | /owners/{ownerId}/pets/{petId}/visits/new | Create Visit; saves via Owner aggregate; redirects to Owner details |
| Vets | GET | /vets.html?page={n} | HTML list of veterinarians (5/page) |
| Vets | GET | /vets | Vets listing as JSON or XML (cached) |
| Actuator | GET | /actuator | Actuator index |
| Actuator | GET | /actuator/{endpoint} | All Actuator endpoints (fully exposed), e.g., health, info, metrics |
| Actuator | GET | /livez | Liveness probe alias (maps to health liveness) |
| Actuator | GET | /readyz | Readiness probe alias (maps to health readiness) |