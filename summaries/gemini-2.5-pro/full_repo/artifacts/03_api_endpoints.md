| Component | HTTP Method | Endpoint Path | Brief Description |
|---|---|---|---|
| System | GET | `/` | Display the welcome page. |
| System | GET | `/oups` | Trigger a runtime exception for error handling demonstration. |
| Owner | GET | `/owners/find` | Display the form to find owners by last name. |
| Owner | GET | `/owners` | Process the find owners form and display a list of results. |
| Owner | GET | `/owners/new` | Display the form to create a new owner. |
| Owner | POST | `/owners/new` | Process the creation of a new owner. |
| Owner | GET | `/owners/{ownerId}` | Display details for a specific owner. |
| Owner | GET | `/owners/{ownerId}/edit` | Display the form to update an existing owner. |
| Owner | POST | `/owners/{ownerId}/edit` | Process the update of an existing owner. |
| Pet | GET | `/owners/{ownerId}/pets/new` | Display the form to add a new pet to an owner. |
| Pet | POST | `/owners/{ownerId}/pets/new` | Process the creation of a new pet for an owner. |
| Pet | GET | `/owners/{ownerId}/pets/{petId}/edit` | Display the form to update an existing pet. |
| Pet | POST | `/owners/{ownerId}/pets/{petId}/edit` | Process the update of an existing pet. |
| Visit | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Display the form to add a new visit for a pet. |
| Visit | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Process the creation of a new visit for a pet. |
| Vet | GET | `/vets.html` | Display a paginated list of veterinarians (HTML view). |
| Vet | GET | `/vets` | Get a list of all veterinarians (JSON/XML format). |