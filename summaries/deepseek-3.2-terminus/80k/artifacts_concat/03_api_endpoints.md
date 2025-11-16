| Component Name | HTTP Method | Endpoint Path | Brief Description |
|----------------|-------------|---------------|-------------------|
| Owner Management | GET | `/owners/new` | Owner creation form |
| Owner Management | POST | `/owners/new` | Create new owner |
| Owner Management | GET | `/owners/find` | Owner search form |
| Owner Management | GET | `/owners` | Search owners by last name with pagination |
| Owner Management | GET | `/owners/{ownerId}` | Owner details |
| Owner Management | GET | `/owners/{ownerId}/edit` | Owner edit form |
| Owner Management | POST | `/owners/{ownerId}/edit` | Update owner |
| Pet Management | GET | `/owners/{ownerId}/pets/new` | Pet creation form |
| Pet Management | POST | `/owners/{ownerId}/pets/new` | Create new pet |
| Pet Management | GET | `/owners/{ownerId}/pets/{petId}/edit` | Pet edit form |
| Pet Management | POST | `/owners/{ownerId}/pets/{petId}/edit` | Update pet |
| Visit Management | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Visit creation form |
| Visit Management | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Create new visit |
| Veterinarian Management | GET | `/vets.html` | Vet list (HTML format) |
| Veterinarian Management | GET | `/vets` | Vet list (JSON/XML format) |
| System Configuration | GET | `/` | Welcome page |
| System Configuration | GET | `/oups` | Error simulation endpoint |