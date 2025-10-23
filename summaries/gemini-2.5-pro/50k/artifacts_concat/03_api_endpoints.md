| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| System | GET | `/` | Displays the application welcome page. |
| System | GET | `/oups` | Triggers a runtime exception to demonstrate error handling. |
| Owner Management | GET | `/owners/find` | Displays the form to search for owners. |
| Owner Management | GET | `/owners` | Processes the owner search and displays a list of results or a single owner. |
| Owner Management | GET | `/owners/new` | Displays the form to create a new owner. |
| Owner Management | POST | `/owners/new` | Processes the creation of a new owner. |
| Owner Management | GET | `/owners/{ownerId}` | Displays detailed information for a specific owner. |
| Owner Management | GET | `/owners/{ownerId}/edit` | Displays the form to edit an existing owner's details. |
| Owner Management | POST | `/owners/{ownerId}/edit` | Processes the update of an owner's details. |
| Pet Management | GET | `/owners/{ownerId}/pets/new` | Displays the form to add a new pet for an owner. |
| Pet Management | POST | `/owners/{ownerId}/pets/new` | Processes the creation of a new pet for an owner. |
| Pet Management | GET | `/owners/{ownerId}/pets/{petId}/edit` | Displays the form to edit an existing pet's details. |
| Pet Management | POST | `/owners/{ownerId}/pets/{petId}/edit` | Processes the update of a pet's details. |
| Visit Management | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Displays the form to add a new visit for a pet. |
| Visit Management | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Processes the creation of a new visit for a pet. |
| Veterinarian | GET | `/vets.html` | Displays a paginated HTML list of all veterinarians. |
| Veterinarian | GET | `/vets` | Returns a JSON or XML list of all veterinarians (Data API). |