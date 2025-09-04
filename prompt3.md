| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| **System** | GET | `/` | Displays the application's welcome page. |
| **System** | GET | `/oups` | Triggers a runtime exception to demonstrate error handling. |
| **Owner Management** | GET | `/owners/find` | Displays the form to find owners by last name. |
| **Owner Management** | GET | `/owners` | Processes the owner search and displays a list of results or a single owner's details. |
| **Owner Management** | GET | `/owners/new` | Displays the form for creating a new owner. |
| **Owner Management** | POST | `/owners/new` | Processes the submission of the new owner form. |
| **Owner Management** | GET | `/owners/{ownerId}` | Displays detailed information for a specific owner. |
| **Owner Management** | GET | `/owners/{ownerId}/edit` | Displays the form for updating an existing owner's details. |
| **Owner Management** | POST | `/owners/{ownerId}/edit` | Processes the submission of the owner update form. |
| **Owner Management** | GET | `/owners/{ownerId}/pets/new` | Displays the form for adding a new pet to an owner. |
| **Owner Management** | POST | `/owners/{ownerId}/pets/new` | Processes the submission of the new pet form. |
| **Owner Management** | GET | `/owners/{ownerId}/pets/{petId}/edit` | Displays the form for updating an existing pet's details. |
| **Owner Management** | POST | `/owners/{ownerId}/pets/{petId}/edit` | Processes the submission of the pet update form. |
| **Owner Management** | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Displays the form for adding a new visit for a pet. |
| **Owner Management** | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Processes the submission of the new visit form. |
| **Veterinarian Management** | GET | `/vets.html` | Displays a paginated list of all veterinarians as an HTML page. |
| **Veterinarian Management** | GET | `/vets` | Returns a list of all veterinarians in JSON format. |