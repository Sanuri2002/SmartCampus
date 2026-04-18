Smart Campus - Sensor & Room Management API

Overview
A RESTful API built with JAX-RS (Jersey) and Grizzly embedded server for managing
campus rooms and sensors. Built for the 5COSC022W Client-Server Architectures coursework.

Technology Stack
- Java 21
- JAX-RS (Jersey 3.1.3)
- Grizzly HTTP Server
- Jackson (JSON)
- Maven

Project Structure
src/main/java/com/smartcampus/
├── Main.java                    
├── SmartCampusApp.java          
├── model/
│   ├── Room.java
│   ├── Sensor.java
│   └── SensorReading.java
├── resource/
│   ├── DiscoveryResource.java
│   ├── RoomResource.java
│   ├── SensorResource.java
│   └── SensorReadingResource.java
├── exception/
│   ├── RoomNotEmptyException.java
│   ├── RoomNotEmptyExceptionMapper.java
│   ├── LinkedResourceNotFoundException.java
│   ├── LinkedResourceNotFoundExceptionMapper.java
│   ├── SensorUnavailableException.java
│   ├── SensorUnavailableExceptionMapper.java
│   └── GlobalExceptionMapper.java
├── filter/
│   └── LoggingFilter.java
└── store/
    └── DataStore.java

How to Build and Run

Prerequisites
- Java JDK 21
- Apache Maven 3.9+
- Apache NetBeans 29

Steps
1. Clone the repository:
git clone https://github.com/Sanuri2002/SmartCampus.git

2. Open NetBeans → File → Open Project → select the smart-campus folder

3. Right-click project → Run

4. Server starts at: http://localhost:8080/

API Endpoints

| Method | URL | Description |
|--------|-----|-------------|
| GET | / | Discovery endpoint |
| GET | /rooms | Get all rooms |
| POST | /rooms | Create a room |
| GET | /rooms/{id} | Get room by ID |
| DELETE | /rooms/{id} | Delete a room |
| GET | /sensors | Get all sensors |
| GET | /sensors?type=CO2 | Filter sensors by type |
| POST | /sensors | Create a sensor |
| GET | /sensors/{id} | Get sensor by ID |
| GET | /sensors/{id}/readings | Get sensor readings |
| POST | /sensors/{id}/readings | Add a reading |

Sample curl Commands

### 1. Get API Discovery
curl -X GET http://localhost:8080/

### 2. Get all rooms
curl -X GET http://localhost:8080/rooms

### 3. Create a new room
curl -X POST http://localhost:8080/rooms -H "Content-Type: application/json" -d "{\"id\":\"CS-101\",\"name\":\"Computer Science Lab\",\"capacity\":40}"

### 4. Create a new sensor
curl -X POST http://localhost:8080/sensors -H "Content-Type: application/json" -d "{\"id\":\"TEMP-002\",\"type\":\"Temperature\",\"status\":\"ACTIVE\",\"currentValue\":25.0,\"roomId\":\"LAB-101\"}"

### 5. Add a sensor reading
curl -X POST http://localhost:8080/sensors/TEMP-001/readings -H "Content-Type: application/json" -d "{\"value\":26.5}"

### 6. Filter sensors by type
curl -X GET http://localhost:8080/sensors?type=CO2

### 7. Delete a room
curl -X DELETE http://localhost:8080/rooms/CS-101

## Report Questions and Answers

Part 1 - Q1: JAX-RS Resource Class Lifecycle
By default, JAX-RS creates a new instance of each resource class for every incoming
HTTP request. This is called per-request lifecycle. This means instance variables
are not shared between requests, which could cause data loss if we stored data in
instance variables. To solve this, we use a Singleton DataStore class with
ConcurrentHashMap, which is shared across all instances. ConcurrentHashMap ensures
thread safety, preventing race conditions when multiple requests modify data simultaneously.

Part 1 - Q2: HATEOAS
HATEOAS (Hypermedia as the Engine of Application State) means including navigation
links inside API responses. For example, when returning a room, you also return links
to its sensors. This benefits client developers because they do not need to memorise
or hardcode URLs - they can discover them dynamically from responses, making the API
self-documenting and reducing coupling between client and server.

Part 2 - Q1: Returning IDs vs Full Objects
Returning only IDs uses less network bandwidth and is faster, but forces the client
to make additional requests to fetch details for each room. Returning full objects
uses more bandwidth but reduces the number of API calls needed. For small collections
like campus rooms, returning full objects is preferable as it reduces round trips and
simplifies client-side processing.

Part 2 - Q2: DELETE Idempotency
Yes, DELETE is idempotent in this implementation. The first DELETE request removes
the room and returns 200 OK. Any subsequent DELETE requests for the same room ID
will return 404 Not Found because the room no longer exists. The server state remains
the same after the first successful deletion, satisfying the idempotency requirement.

Part 3 - Q1: @Consumes Annotation
If a client sends data in a format other than application/json, such as text/plain
or application/xml, JAX-RS automatically returns a 415 Unsupported Media Type error.
The framework checks the Content-Type header of the request against the @Consumes
annotation before even invoking the method. This protects the API from receiving
malformed or unexpected data formats.

Part 3 - Q2: @QueryParam vs Path Parameter
Using @QueryParam for filtering such as /sensors?type=CO2 is superior because query
parameters are optional by nature, so the same endpoint works for both filtered and
unfiltered requests. Path parameters such as /sensors/type/CO2 suggest a fixed
resource hierarchy, which is semantically incorrect for filtering. Query parameters
are the REST standard for search and filter operations on collections.

Part 4 - Q1: Sub-Resource Locator Pattern
The Sub-Resource Locator pattern delegates handling of nested paths to separate
classes. Instead of defining all endpoints in one massive class, each resource
manages its own logic. This improves maintainability, separation of concerns, and
testability. For large APIs with many nested resources, this pattern prevents
resource classes from becoming unmanageable and makes the codebase easier to navigate.

Part 5 - Q1: HTTP 422 vs 404
HTTP 404 means the requested URL or resource was not found. HTTP 422 means the
request was syntactically correct but semantically invalid. When a client sends a
valid JSON payload containing a roomId that does not exist, the URL is valid and
the JSON is well-formed, but the referenced resource is missing. Therefore 422 is
more accurate as it points to a data integrity issue inside the payload rather than
a missing endpoint.

Part 5 - Q2: Stack Trace Security Risk
Exposing Java stack traces to external clients reveals sensitive internal information
such as class names and package structure which helps attackers map the codebase,
library versions which helps identify known vulnerabilities, file paths on the server,
and the exact line of code that failed which helps craft targeted attacks. The
GlobalExceptionMapper prevents this by catching all unexpected errors and returning
only a generic 500 message with no internal details.

Part 5 - Q3: JAX-RS Filters vs Manual Logging
Using JAX-RS filters for cross-cutting concerns like logging is better because it
follows the DRY principle - the logging logic is written once and applied automatically
to every request and response. Manually inserting Logger.info() in every resource
method is repetitive, error-prone, and easy to forget. Filters also keep resource
classes clean and focused on business logic only, improving readability and maintainability.