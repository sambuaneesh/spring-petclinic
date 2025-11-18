
| Component Name | HTTP Method | Endpoint Path | Brief Description |
|----------------|-------------|---------------|-------------------|
| Owner Management | GET | `/owners/find` | Search form |
| Owner Management | GET | `/owners` | Find owners with pagination |
| Owner Management | GET | `/owners/{ownerId}` | Owner details view |
| Owner Management | GET | `/owners/new` | Create owner form |
| Owner Management | POST | `/owners/new` | Create owner |
| Owner Management | GET | `/owners/{ownerId}/edit` | Update owner form |
| Owner Management | POST | `/owners/{ownerId}/edit` | Update owner |
| Pet Management | GET | `/owners/{ownerId}/pets/new` | Create pet form |
| Pet Management | POST | `/owners/{ownerId}/pets/new` | Create pet |
| Pet Management | GET | `/owners/{ownerId}/pets/{petId}/edit` | Update pet form |
| Pet Management | POST | `/owners/{ownerId}/pets/{petId}/edit` | Update pet |
| Visit Management | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Create visit form |
| Visit Management | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Create visit |
| Veterinary Management | GET | `/` | Home page |
| Veterinary Management | GET | `/vets.html` | Paginated vet listing (5/page) |
| Veterinary Management | GET | `/vets` | Full vet list (API) |
| Error Handling | GET | `/oups` | Controlled exception endpoint |