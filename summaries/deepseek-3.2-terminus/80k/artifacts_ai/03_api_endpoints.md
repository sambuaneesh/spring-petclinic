| Component Name | HTTP Method | Endpoint Path | Brief Description |
|----------------|-------------|---------------|-------------------|
| Owner Management | GET | `/owners/new` | Render owner creation form |
| Owner Management | POST | `/owners/new` | Process new owner creation with validation |
| Owner Management | GET | `/owners/find` | Display owner search interface |
| Owner Management | GET | `/owners` | Search owners by last name with pagination |
| Owner Management | GET | `/owners/{ownerId}` | Retrieve and display owner details |
| Owner Management | GET | `/owners/{ownerId}/edit` | Render owner editing form |
| Owner Management | POST | `/owners/{ownerId}/edit` | Process owner updates with validation |
| Pet Management | GET | `/owners/{ownerId}/pets/new` | Render pet creation form for specific owner |
| Pet Management | POST | `/owners/{ownerId}/pets/new` | Create new pet with owner association |
| Pet Management | GET | `/owners/{ownerId}/pets/{petId}/edit` | Render pet editing interface |
| Pet Management | POST | `/owners/{ownerId}/pets/{petId}/edit` | Update pet information |
| Visit Management | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Render visit scheduling form |
| Visit Management | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Create new veterinary visit record |
| Veterinarian Management | GET | `/vets.html` | Display paginated veterinarian list (HTML format) |
| Veterinarian Management | GET | `/vets` | Retrieve veterinarian list (JSON/XML API format) |
| System Management | GET | `/` | Application welcome and landing page |
| System Management | GET | `/oups` | Error simulation and system testing endpoint |