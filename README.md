
[![Build Status](https://travis-ci.org/imrenagi/microservice-skeleton.svg?branch=master)](https://travis-ci.org/imrenagi/microservice-skeleton)  [![codecov](https://codecov.io/gh/imrenagi/microservice-skeleton/branch/master/graph/badge.svg)](https://codecov.io/gh/imrenagi/microservice-skeleton) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) 

# BHYM: Mission Management System Based on a Service Oriented Architecture

BHYM is a mission management system that follows the Service Oriented Architecture (SOA) integration pattern using technologies like Mule ESB, Spring Boot, Spring Web Services, and Spring for GraphQL among many others. The system is intended to manage missions, mission requests, and mission reimbursement requests in a modular, decoupled, and efficient fashion.

The system is currently comprised of 9 services, each of which can be tested, built, and deployed independently. The system supports two kinds of communications between the existing services; synchronous via Mule's ESB, and asynchronous via Rabbit's message broker.

![Infrastructure plan](https://i.ibb.co/7vPFrWG/Copy-of-Mission-Management-System-Page-1-drawio-1.png)

## Services

### Mission Request Service
Exposes a number of endpoints for handling mission requests and their respective lifecycles

| Method | Path         | Description                          |   Privilege               | Query Parameters         |
|--------|--------------|--------------------------------------|--------------------------|--------------------------|
| GET    | /mission-requests | Retrieves all mission requests   |  SUPERVISOR |  |
| GET    | /mission-requests | Retrieves all mission requests made by a specific professor |  SUPERVISOR, PROFESSOR(ID) | ID |
| POST    | /mission-requests  | Creates a new mission request user  | PROFESSOR  |   
| GET    | /mission-requests/{id}  | Retrieves a mission request based on ID  | SUPERVISOR  | 
| PATCH    | /mission-requests/{id}/approve  | Approves a mission request by ID  | SUPERVISOR  | 
| PATCH    | /mission-requests/{id}/cancel  | Cancels a mission request by ID  | PROFESSOR  | 
| PATCH    | /mission-requests/{id}/reject  | Rejects a mission request by ID  | SUPERVISOR  | 

### Mission Service
Contains a number of SOAP operations for handling mission creation and retrieval

| Operation | Request Message                  | Response Message                 | Description                                     |
|-----------|----------------------------------|----------------------------------|-------------------------------------------------|
| `getMission`   | `GetMissionRequest` | `GetMissionResponse` | Retrieves details of a mission based on the provided mission ID.                        |
| `createMission` | `CreateMissionRequest` | `CreateMissionResponse` | Creates a new mission based on the information provided in the request.                |
| `getAllMissions` | `GetAllMissionsRequest` | `GetAllMissionsResponse` | Retrieves a list of all missions available in the system.                              |

##### Operations:

1. **`getMission`:**
   - **Request Message:** `GetMissionRequest`
     - *Attributes:*
       - `id` (type: `xs:long`, required): The unique identifier of the mission to retrieve.
   - **Response Message:** `GetMissionResponse`
     - *Attributes:*
       - `mission` (type: `tns:mission`): Details of the retrieved mission.
   - **Description:** Retrieves details of a mission identified by the provided mission ID.

2. **`createMission`:**
   - **Request Message:** `CreateMissionRequest`
     - *Attributes:*
       - `professorId` (type: `xs:long`, required): The unique identifier of the professor associated with the mission.
       - `title` (type: `xs:string`, required): The title of the mission.
       - `description` (type: `xs:string`, required): The description of the mission.
   - **Response Message:** `CreateMissionResponse`
     - *Attributes:*
       - `mission` (type: `tns:mission`): Details of the created mission.
   - **Description:** Creates a new mission based on the information provided in the request.

   **How to Create a Mission:**
   To create a new mission, send a SOAP request with the `CreateMissionRequest` message to the service endpoint. Include the required attributes `professorId`, `title`, and `description` with the relevant information. The service will respond with a `CreateMissionResponse` containing details of the newly created mission, including its unique identifier (`id`), title, and description.

3. **`getAllMissions`:**
   - **Request Message:** `GetAllMissionsRequest`
   - **Response Message:** `GetAllMissionsResponse`
     - *Attributes:*
       - `mission` (type: `tns:mission`, minOccurs: 0, maxOccurs: unbounded): List of missions available in the system.
   - **Description:** Retrieves a list of all missions available in the system.


### Professor Service
A GraphQL service for managing professors

| Field Name        | Description                                      | Example Query                                              | Example Response                                             |
|-------------------|--------------------------------------------------|------------------------------------------------------------|--------------------------------------------------------------|
| `professorById`   | Retrieves details of a professor based on ID.     | `query { professorById(id: "3") { id, fullName } }`        | `{ "data": { "professorById": { "id": "3", "fullName": "John Smith" } } }` |
| `allProfessors`   | Retrieves a list of all professors.              | `query { allProfessors { id, fullName } }`                | `{ "data": { "allProfessors": [ { "id": "1", "fullName": "Professor 1" }, { "id": "2", "fullName": "Professor 2" }, ... ] } }`       |



### Reimbursement Service
Exposes a number of endpoints for handling reimbursement requests and their respective lifecycles

| Method | Path                                      | Description                                              | Privilege                   |
|--------|-------------------------------------------|----------------------------------------------------------|-----------------------------|
| GET    | /mission-reimbursement-requests           | Retrieves all mission reimbursement requests             | SUPERVISOR                  |
| POST   | /mission-reimbursement-requests           | Creates a new mission reimbursement request user          | PROFESSOR                  |
| GET    | /mission-reimbursement-requests/{id}      | Retrieves a mission reimbursement request based on ID     | SUPERVISOR                  |
| PATCH | /mission-reimbursement-requests/{id}/approve | Approves a mission reimbursement request by ID           | SUPERVISOR                  |
| PATCH | /mission-reimbursement-requests/{id}/reject  | Rejects a mission reimbursement request by ID           | SUPERVISOR                  |


### Mission Cost Calculation Service
Manages mission calculation costs and lifecycles. Listens for the "mission reimbursement request approved" event, calculates the mission cost in response, and publishes an event upon verification by the manager.

| Method | Path                               | Description                                        | Privilege                   |
|--------|------------------------------------|----------------------------------------------------|-----------------------------|
| GET    | /mission-cost-calculations         | Retrieves all mission cost calculations            | MANAGER                      |
| GET    | /mission-cost-calculations/{id}    | Retrieves a mission cost calculation based on ID    | MANAGER, PROFESSOR(ID)                     |
| PATCH  | /mission-cost-calculations/{id}/verify | Verifies a mission cost calculation by ID       | MANAGER                        |
| PATCH  | /mission-cost-calculations/{id}/refute | Refutes a mission cost calculation by ID       | MANAGER                        |


### Mission Order Service Listener

- Listens to the "mission transport treated" event.
- Reacts by:
     1.  Retrieving mission and professor information from Mule ESB;
     2.  Generating a mission order document from the retrieved resources and saving it to the database. 
- Generates a mission order document using a printer service .
-  Exposes a number of endpoints for customized retrieval of mission orders:

| Method | Path         | Description                          |   Privilege               | Query Parameters         |
|--------|--------------|--------------------------------------|--------------------------|--------------------------|
| GET    | /mission-orders | Retrieves all mission orders  |  SUPERVISOR |  |
| GET    | /mission-orders | Retrieves all mission orders of a given professor |  SUPERVISOR, PROFESSOR(professorId) | professorId |
| GET    | /mission-orders | Retrieves mission order for a given mission request|  SUPERVISOR, PROFESSOR(ID) | missionRequestId |


### Mission Transport Treatment Service Listener

- Listens to the "mission request approved" event.
- Reacts by:
     1.  Retrieving mission and professor information from Mule ESB;
     2.  Treating the transport of the retrieved mission and persisting the resulted treatment in the database.
- Exposes a number of endpoints for customized retrieval of mission transport treatments:

| Method | Path         | Description                          |   Privilege               | Query Parameters         |
|--------|--------------|--------------------------------------|--------------------------|--------------------------|
| GET    | /mission-transport-treatments | Retrieves all mission transport treatments  |  SUPERVISOR |  |
| GET    | /mission-transport-treatments | Retrieves all mission transport treatments of a given professor |  SUPERVISOR, PROFESSOR(professorId) | professorId |
| GET    | /mission-transport-treatments | Retrieves mission transport treatments for a given mission request|  SUPERVISOR, PROFESSOR(ID) | missionRequestId |


