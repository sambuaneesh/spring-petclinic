
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| Owner Management | GET | /owners/new | Show owner registration form |
| Owner Management | POST | /owners/new | Process owner registration |
| Owner Management | GET | /owners/find | Show owner search form |
| Owner Management | POST | /owners/find | Process owner search |
| Owner Management | GET | /owners | Paginated owner listing |
| Owner Management | GET | /owners/{id} | Owner detail view |
| Owner Management | GET | /owners/{id}/edit | Show owner edit form |
| Owner Management | POST | /owners/{id}/edit | Process owner update |
| Pet Management | GET | /owners/{ownerId}/pets/new | Show pet addition form |
| Pet Management | POST | /owners/{ownerId}/pets/new | Process pet addition |
| Pet Management | GET | /owners/{ownerId}/pets/{petId}/edit | Show pet edit form |
| Pet Management | POST | /owners/{ownerId}/pets/{petId}/edit | Process pet update |
| Visit Management | GET | /owners/{ownerId}/pets/{petId}/visits/new | Show visit scheduling form |
| Visit Management | POST | /owners/{ownerId}/pets/{petId}/visits/new | Process visit scheduling |
| Veterinary Management | GET | /vets.html | Paginated vet listing (HTML) |
| Veterinary Management | GET | /vets | Vet list API endpoint (JSON) |
| System | GET | / | Application home page |
| System | GET | /oups | Error demonstration endpoint |
| Health Checks | GET | /livez | Liveness probe endpoint |
| Health Checks | GET | /readyz | Readiness probe endpoint |
| Management | GET | /actuator/health | Service health status |
| Management | GET | /actuator/metrics | Application metrics |
| Management | GET | /actuator/prometheus | Prometheus metrics endpoint |
| Management | GET | /actuator/loggers | Logger management |
| Management | GET | /actuator/info | Service information |