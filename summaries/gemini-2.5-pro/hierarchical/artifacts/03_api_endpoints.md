| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Owner Management | GET | `/owners/new` | Displays the form to create a new owner. |
| Owner Management | POST | `/owners/new` | Processes the creation of a new owner record. |
| Owner Management | GET | `/owners/find` | Displays the owner search form. |
| Owner Management | GET | `/owners` | Finds owners by last name and displays a list of results. |
| Owner Management | GET | `/owners/{ownerId}` | Displays detailed information for a specific owner, including their pets. |
| Owner Management | GET | `/owners/{ownerId}/edit` | Displays the form to update an existing owner's details. |
| Owner Management | POST | `/owners/{ownerId}/edit` | Processes the update of an owner's details. |
| Pet Management | GET | `/owners/{ownerId}/pets/new` | Displays the form to add a new pet to an owner. |
| Pet Management | POST | `/owners/{ownerId}/pets/new` | Processes the addition of a new pet to an owner's record. |
| Pet Management | GET | `/owners/{ownerId}/pets/{petId}/edit` | Displays the form to update an existing pet's details. |
| Pet Management | POST | `/owners/{ownerId}/pets/{petId}/edit` | Processes the update of a pet's details. |
| Visit Management | GET | `/owners/{ownerId}/pets/{petId}/visits/new` | Displays the form to add a new medical visit for a pet. |
| Visit Management | POST | `/owners/{ownerId}/pets/{petId}/visits/new` | Processes the addition of a new medical visit to a pet's record. |
| Veterinarian Management | GET | `/vets` or `/vets.html` | Displays a paginated list of all veterinarians (HTML view). |
| Veterinarian Management | GET | `/vets.json` | Retrieves a list of all veterinarians and their specialties (JSON API). |
| Veterinarian Management | GET | `/vets.xml` | Retrieves a list of all veterinarians and their specialties (XML API). |
| System | GET | `/` | Displays the application's welcome page. |
| System | GET | `/oups` | Intentionally triggers a server error to test global error handling. |