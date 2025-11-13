| Component Name | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| WelcomeController | GET | / | Render application welcome/home page |
| CrashController | GET | /oups | Deliberately trigger a RuntimeException to demonstrate error handling |
| VetController | GET | /vets.html | Render veterinarians list page (supports pagination via page param) |
| VetController | GET | /vets | Return list of veterinarians as JSON/XML (Vets resource) |
| OwnerController | GET | /owners/new | Show owner creation form |
| OwnerController | POST | /owners/new | Create a new owner and redirect to owner details |
| OwnerController | GET | /owners/find | Show owner search form |
| OwnerController | GET | /owners | Search owners by last name; render list or redirect if single match (supports page param) |
| OwnerController | GET | /owners/{ownerId}/edit | Show owner update form |
| OwnerController | POST | /owners/{ownerId}/edit | Update owner and redirect to owner details |
| OwnerController | GET | /owners/{ownerId} | Show owner details (including pets and visits) |
| PetController | GET | /owners/{ownerId}/pets/new | Show pet creation form for an owner |
| PetController | POST | /owners/{ownerId}/pets/new | Create a new pet for an owner and redirect to owner details |
| PetController | GET | /owners/{ownerId}/pets/{petId}/edit | Show pet update form |
| PetController | POST | /owners/{ownerId}/pets/{petId}/edit | Update pet and redirect to owner details |
| VisitController | GET | /owners/{ownerId}/pets/{petId}/visits/new | Show visit creation form for a pet |
| VisitController | POST | /owners/{ownerId}/pets/{petId}/visits/new | Create a new visit for a pet and redirect to owner details |