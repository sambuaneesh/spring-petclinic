| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| System | GET | `/` | Display the welcome page. |
| System | GET | `/oups` | Intentionally throws an exception for error handling demonstration. |
| Owner | GET | `/owners/find` | Display the owner search form. |
| Owner | GET | `/owners` | Process owner search. Finds by last name or lists all if name is empty. Supports pagination. |
| Owner | GET | `/owners/new` | Display the form to create a new owner. |
| Owner | POST | `/owners/new` | Process the creation of a new owner. |
| Owner | GET | `/owners/{ownerId}` | Display detailed information for a specific owner. |
| Owner | GET | `/owners/{ownerId}/edit` | Display the form to update an owner's details. |
| Owner | POST | `/owners/{ownerId}/edit` | Process the update of an owner's details. |
| Owner | GET | `/owners/{ownerId}/pets/new` | Display the form to add a new pet to an owner. |
| Owner | POST | `/owners/{ownerId}/pets/new` | Process the addition of a new pet. |
| Owner | GET | `/owners/{ownerId}/pets/{petId}/edit` | Display the form to update a pet's details. |
| Owner | POST | `/owners/{ownerId}/pets/{petId}/edit` | Process the update of a pet's details. |
| Owner | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Display the form to add a new visit for a pet. |
| Owner | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Process the addition of a new visit. |
| Vet | GET | `/vets.html` | Display a paginated list of veterinarians (HTML view). |
| Vet | GET | `/vets` | Return a list of all veterinarians (JSON format). |