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

1. Get API Discovery
curl -X GET http://localhost:8080/

2. Get all rooms
curl -X GET http://localhost:8080/rooms

3. Create a new room
curl -X POST http://localhost:8080/rooms -H "Content-Type: application/json" -d "{\"id\":\"CS-101\",\"name\":\"Computer Science Lab\",\"capacity\":40}"

4. Create a new sensor
curl -X POST http://localhost:8080/sensors -H "Content-Type: application/json" -d "{\"id\":\"TEMP-002\",\"type\":\"Temperature\",\"status\":\"ACTIVE\",\"currentValue\":25.0,\"roomId\":\"LAB-101\"}"

5. Add a sensor reading
curl -X POST http://localhost:8080/sensors/TEMP-001/readings -H "Content-Type: application/json" -d "{\"value\":26.5}"

6. Filter sensors by type
curl -X GET http://localhost:8080/sensors?type=CO2

7. Delete a room
curl -X DELETE http://localhost:8080/rooms/CS-101

Report Questions and Answers

Part 1 - Q1: JAX-RS Resource Class Lifecycle
By default, JAX-RS instantiates one object of each resource class for each HTTP
request that comes into the application. This is known as per-request lifecycle,
which means that there is no persistence of data between invocations since any
data that is persisted would get lost on the next invocation of that method.
However, to avoid such an issue, we can utilize a singleton class called DataStore
that uses ConcurrentHashMap.

Part 1 - Q2: HATEOAS
“HATEOAS,” which stands for “Hypermedia as the Engine of Application State,”
involves embedding the navigation links within the API’s response messages. 
For instance, if an API returns a room, it will include links pointing to the 
sensors associated with that particular room. This is helpful for client developers 
since they don’t have to remember or hard-code any URLs.

Part 2 - Q1: Returning IDs vs Full Objects
It is faster to return IDs as it requires less bandwidth, but additional requests 
have to be made by the client for each of the rooms. It consumes more bandwidth but
fewer calls have to be made for large objects. Since our collection is relatively
smaller such as the rooms on campus,returning objects would be a better approach.

Part 2 - Q2: DELETE Idempotency
Yes, DELETE operation is an idempotent method here. On receiving the first request,
the room will be deleted and the response status will be 200 OK. In case the same
DELETE request is sent again, it will result in a 404 Not Found error because the room
is no longer available.

Part 3 - Q1: @Consumes Annotation
In the event that the content type provided by the user is something other than
application/json such as application/xml or text/plain, then the framework responds
with a 415 error, "Unsupported Media Type". The framework verifies the content type
specified in the content-type header against the @Consumes annotation before even
calling the method.

Part 3 - Q2: @QueryParam vs Path Parameter
The use of @QueryParam when it comes to filtering using a URL like /sensors?type=CO2 
is more appropriate because query parameters are optional. This means that an endpoint 
can either have filtered information or unfiltered one depending on the inclusion of the 
query parameter. A URL like /sensors/type/CO2 implies that there’s a certain hierarchical 
structure, but this doesn’t make sense in the case of filters.

Part 4 - Q1: Sub-Resource Locator Pattern
The Sub-Resource Locator pattern relies on different classes for resolving the
nested URL paths. Rather than declaring all the endpoints in one huge class,
the resources have their own logic declared in them. This helps in better
management, separation of responsibilities, and testability of the API. When
dealing with nested URLs, this design pattern makes sure that resources do not
become unmanageable.

Part 5 - Q1: HTTP 422 vs 404
An HTTP 404 indicates that the resource or URL could not be located. An HTTP 422
indicates that the request was correct in syntax but incorrect in semantics.
When a client passes in a valid JSON body with a non-existent roomId, it can
be considered that the URL is present, and the JSON body is also valid; however,
it references an object that cannot be located.

Part 5 - Q2: Stack Trace Security Risk
By revealing stack trace details to outside clients, internal information such as
class names, the package structure that will help hackers understand your code base,
the library version that may lead to vulnerabilities and file path of the server can
be revealed. GlobalExceptionMapper solves the problem by intercepting any errors that
occur and only giving the 500 error message back

Part 5 - Q3: JAX-RS Filters vs Manual Logging
Using JAX-RS filters for cross-cutting concerns like logging is better because it
follows the DRY principle - the logging logic is written once and applied automatically
to every request and response. Manually inserting Logger.info() in every resource
method is repetitive, error-prone, and easy to forget. Filters also keep resource
classes clean and focused on business logic only, improving readability and maintainability.