
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| OwnerController | GET/POST | /owners | CRUD + search/pagination |
| PetController | GET/POST | /owners/{ownerId}/pets | Pet management under owner |
| VisitController | GET/POST | /owners/{ownerId}/pets/{petId}/visits | Visit scheduling |
| VetController | GET | /vets.html | Vet listing (paginated) |
| VetController | GET | /vets | JSON vet list |
| WelcomeController | GET | / | Homepage |
| CrashController | GET | /oups | Error testing endpoint |