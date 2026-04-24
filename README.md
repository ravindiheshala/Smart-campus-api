Smart Campus REST API
Module: 5COSC022W – Client-Server Architectures  
Student: S.M.D.R. Heshala Senarathne  
UOW ID: W2120505  

Project Overview
This project is a RESTful API developed using JAX-RS (Jersey) for managing a Smart Campus system.  
It handles Rooms, Sensors, and Sensor Readings.

The API follows REST principles including:
- Resource-based design
- Proper HTTP methods
- Meaningful status codes
- JSON data format
- Error handling with Exception Mappers

Technologies Used
- Java
- JAX-RS (Jersey)
- Maven
- Embedded Server (Grizzly/Tomcat)
- Postman (for testing)
  
  Prerequisites
Make sure you have installed:
- Java JDK (version 8 or above)
- Maven
- NetBeans / IntelliJ (optional)
- Postman (for testing)
  
Part 1: Service Architecture & Setup
1.	Project & Application Configuration
Question: In your report, explain the default lifecycle of a JAX-RS Resource class. Is a new instance instantiated for every incoming request, or does the runtime treat it as a singleton? Elaborate on how this architectural decision impacts the way you manage and synchronize your in-memory data structures (maps/lists) to prevent data loss or race con-ditions.
Out of the box, JAX-RS resource classes are requested scoped (so a new instance is created per every incoming HTTP request). Since Resource class itself does not maintain state across requests, all in-memory data structures (HashMap or ArrayList) will need to be persisted at service layer as shared method variable or have Singleton annotation. Race Condition or Data Loss If concurrent threads are accessing collections, these must be using thread-safe implementations (ConcurrentHashMap or synchronized blocks).

2.	The Discovery Endpoint
Question: Why is the provision of ”Hypermedia” (links and navigation within responses) considered a hallmark of advanced RESTful design (HATEOAS)? 
How does this approach benefit client developers compared to static documentation?
Hypermedia as the Engine of Application State (HATEOAS) gives response links to related resources, making the API self-discoverable. This is advantageous for developers since they can dynamic navigate the API in accordance with the response returned by server, instead of hard coded URLs and static documentation that goes out of date quickly as the API changes.

Part 2: Room Management
1.	RoomResource Implementation
Question: When returning a list of rooms, what are the implications of returning only IDs versus returning the full room objects? 
Consider network bandwidth and client-side processing.
They return only ids of rooms, which saves network bandwidth and speeds up responses at the expense of having more API calls from clients in order to get all details. Returning full room objects streamlines the processing on the client side as all data comes back in one response but leads to increased payload sizes and network usage.

2.	RoomDeletion & Safety Logic
Question: Is the DELETE operation idempotent in your implementation? Provide detailed justification by describing what happens if a client mistakenly sends the exact same DELETE request for a room multiple time.
Yes, in this implementation the DELETE operation is idempotent. Here's an example to illustrate the scenario, when a client sends DELETE for Room, only the first request will succeed in removing that room from the system. It will always get the same result if we ask for the DELETE again on the same room (since it is deleted). For Example, the API will return a 404 Not Found or similar after first deletion but not change any system state. Thus, executing the same DELETE request does not lead to any side effects and proves that this operation is idempotent.

Part 3: Sensor Operations & Linking
1.	Sensor Resource & Integrity
Question: We explicitly use the @Consumes (MediaType.APPLICATION_JSON) annotation on the POST method. Explain the technical consequences if a client attempts to send data in a different format, such as text/plain or application/xml. How does JAX-RS handle this mismatch?
When @Consumes(MediaType.APPLICATION_JSON) is used on a POST method, the endpoint only accepts requests where the Content-Type is application/Json. If a client sends data in a different format such as text/plain or application/xml, JAX-RS will not match the request to that method and will reject it before the method body is executed. In this case, the framework typically returns an HTTP 415 Unsupported Media Type response.
JAX-RS uses body readers messages to convert data into Java objects, and if there are no available readers for the format of our request, it will not be able to process it. Thus, the mismatch is automatically managed by the framework which secures type safety and avoids wrong data from entering into your application.

2.	Filtered Retrieval & Search
Question: You implemented this filtering using @QueryParam. Contrast this with an alterna -tive design where the type is part of the URL path (e.g., /api/vl/sensors/type/CO2). Why is the queryparameterapproachgenerallyconsideredsuperiorforfilteringandsearching collections?
The @QueryParam approach is generally considered better for filtering because it keeps the base resource URL clean and more flexible. For example, using GET /sensors? Type=CO2 clearly treats “type” as an optional filter on the sensor collection, and it allows multiple filters to be added easily in the future without changing the URL structure.
On the other side, a path parameter like /sensors/type/CO2 is not flexible enough because it hard codes the filter into the resource path and does not perform well when extending with another filtering mechanism. Using query parameters is also the default REST approach for searching and filtering collections, while path parameters are more appropriate to identifying resources rather than applying dynamic conditions.

Part 4: Deep Nesting with Sub- Resources
1.	The Sub-Resource Locator Pattern
Question: Discuss the architectural benefits of the Sub-Resource Locator pattern. How does delegating logic to separate classes help manage complexity in large APIs compared to defining every nested path (e.g., sensors/{id}/readings/{rid}) in one massive con troller class?
The Sub-Resource Locator pattern improves API architecture by delegating responsibility for nested resources to separate classes instead of handling all endpoints in one large controller. This significantly reduces complexity because each class focuses on a specific part of the system, such as sensor readings in SensorReadingResource, rather than managing all sensor-related operations in a single file. This separation of concerns makes the code easier to read, maintain, and debug. It also improves scalability, since new nested resources can be added without modifying a large central controller. Compared to defining all paths like sensors/{id}/readings/{rid} in one class, the sub-resource approach avoids clutter, reduces duplication, and follows a cleaner modular design, which is more suitable for large and evolving REST APIs.

Part 5: Advanced Error Handling, Exception Mapping & Logging
2.	Dependency Validation (422 Unprocessable Entity)
Question: WhyisHTTP422oftenconsideredmoresemanticallyaccurate than a standard 404 whenthe issue is a missing reference inside a valid JSON payload?
HTTP 422 (Unprocessable Entity) is considered more semantically accurate than 404 in this case because the request itself is valid and correctly formatted, but the problem lies in the business logic or data integrity. For example, when a sensor is created with a valid JSON body but references a roomId that does not exist, the endpoint exists and the request is properly structured, so 404 (Not Found) would be misleading since nothing is wrong with the URL or resource endpoint. Instead, 422 clearly indicates that the server understands the request but cannot process it due to a logical validation error in the provided data. This makes the error response more meaningful and helps clients understand that the issue is with the input data rather than a missing API endpoint.

4. The Global Safety Net (500)
Question: From a cybersecurity standpoint, explain the risks associated with exposing internal Java stack traces to external API consumers. What specific information could an attacker gather from such a trace?
If those internal Java stack traces are being sent out to external API consumers, there's a significant security risk because it exposes information about the system's implementation that's not meant for public viewing. In terms of cybersecurity, the stack traces can provide attackers with information about internal class names, package structures, framework versions (for example JAX-RS or Jersey), application containers — and even server details. Posted files, database queries or configuration details may be affected in some cases as well. This knowledge provides the exact structure of your system and tells the attacker where they can target attacks (like injection or path traversal) or define exploits specific to a framework, etc.
Therefore, APIs should always return generic error messages (like HTTP 500 responses) instead of detailed stack traces to prevent information leakage and improve overall system security.

5. API Request & Response Logging Filters
Question: Why is it advantageous to use JAX-RS filters for cross-cutting concerns like logging, rather than manually inserting Logger.info() statements inside every single re-source method?
Logging is a cross-cutting concern, i.e., it is not specific to one resource rather applies to all the endpoints – JAX-RS filters provide an opportunity for us. Using ContainerRequestFilter and ContainerResponseFilter, logging is centralised therefore not repeating Logger for every request and response. info() in every method. This improves code maintainability and eliminates duplication of classes to keep resource classes clean with a focus only on business logic, All endpoints will be logged in one format and this ensure consistency across the API as well. On the other hand, manual logging inside each method leads to lost logs, inconsistent messages and harder maintenance as the API grows.


