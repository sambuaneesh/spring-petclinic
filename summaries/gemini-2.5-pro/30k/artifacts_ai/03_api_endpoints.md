| Component Name | HTTP Method | Endpoint Path | Brief Description |
| :--- | :--- | :--- | :--- |
| Owner Management | `GET` | `/owners/find` | Display owner search form. |
| Owner Management | `GET` | `/owners` | Process owner search by last name (paginated). |
| Owner Management | `GET` | `/owners/new` | Display new owner creation form. |
| Owner Management | `POST` | `/owners/new` | Process creation of a new owner. |
| Owner Management | `GET` | `/owners/{ownerId}` | Display details for a specific owner. |
| Owner Management | `GET` | `/owners/{ownerId}/edit` | Display owner update form. |
| Owner Management | `POST` | `/owners/{ownerId}/edit` | Process owner update. |
| Owner Management | `GET` | `/owners/{ownerId}/pets/new` | Display new pet form for an owner. |
| Owner Management | `POST` | `/owners/{ownerId}/pets/new` | Process creation of a new pet. |
| Owner Management | `GET` | `/owners/{ownerId}/pets/{petId}/edit` | Display pet update form. |
| Owner Management | `POST` | `/owners/{ownerId}/pets/{petId}/edit` | Process pet update. |
| Owner Management | `GET` | `/owners/{ownerId}/pets/{petId}/visits/new` | Display new visit form for a pet. |
| Owner Management | `POST` | `/owners/{ownerId}/pets/{petId}/visits/new` | Process creation of a new visit. |
| Veterinarian Management | `GET` | `/vets.html` | Display a paginated list of all vets (HTML view). |
| Veterinarian Management | `GET` | `/vets` | Return a list of all vets as JSON or XML (data endpoint). |
| System/Cross-Cutting Concerns | `GET` | `/` | Display the application's home/welcome page. |
| System/Cross-Cutting Concerns | `GET` | `/oups` | Intentionally throws an exception to test global error handling. |