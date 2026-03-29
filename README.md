# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1
##### **1. In this tutorial, we used `RwLock<>` to synchronise the use of `Vec` of `Notifications`. Explain why it is necessary for this case, and explain why we do not use `Mutex<>` instead?**

**I explained why `RwLock<Vec<Notification>>` used in this tutorial. Here is my reasons:**

- Concurrent read-heavy access
- Notification collection is shared by REST handlers (list, receive, etc.).
- Many operations (e.g., listing notifications) are reads and can safely run in parallel.
`RwLock` allows:
- multiple simultaneous readers (read()), and
- one writer (write()), blocking readers while writing.

**Even so, why some writes still exist?**

- Adding/removing subscriptions or messages needs exclusive access.
- `RwLock` handles this with writer lock semantics, while allowing reads to be unhindered when no writer is active.

**Why not just `Mutex<Vec<Notification>>`?**

- Necause `Mutex` is simpler: always exclusive lock, even for reads.
That means: every list call blocks other list calls, lower throughput and more latency under concurrent clients than `RwLock`.
- For read-mostly workflows (notifications observed frequently, updates less frequent), `RwLock` is a better fit.

##### **2. In this tutorial, we used lazy_static external library to define Vec and DashMap as a “static” variable. Compared to Java where we can mutate the content of a static variable via a static function, why did not Rust allow us to do so?**

I explain why Rust uses `lazy_static` and stricter static mutability than Java. The short answer is Rust’s rules around static are intentionally strict to enforce safety at compile-time.

**Rust static vs Java static**
- Java:
1. `static` variables can be mutable by default.
2. You can write static `List<Foo>` list and mutate it in any static method.
3. Thread safety is not enforced by compiler; developers must synchronize manually.
- Rust:
1. `static` with mutable content is unsafe (`static mut`) because unsynchronized mutation can cause data races.
2. Rust forbids easy global mutations to ensure memory safety and data-race freedom.
 
#### Reflection Subscriber-2

**1. Have you explored things outside of the steps in the tutorial, for example: `src/lib.rs`? If not, explain why you did not do so. If yes, explain things that you have learned from those other parts of code.**

Yes, I explored parts of the codebase outside the tutorial steps, which is `lib.rs`. Here's what I learned from these components:

##### **From `lib.rs`:**
- **Global State Management**: The file uses `lazy_static` to define two global static variables (`REQWEST_CLIENT` and `APP_CONFIG`). This is necessary because Rust's standard static variables can't be mutated safely without unsafe code, so `lazy_static` provides a safe way to initialize shared state once at runtime.
- **Configuration Handling**: The `AppConfig` struct demonstrates how to load configuration from environment variables using Rocket's Figment library. It merges default values with prefixed environment variables (e.g., `APP_INSTANCE_NAME`), showing a pattern for runtime configuration in web apps.
- **Error Handling Patterns**: Custom error types (`Error` and `ErrorResponse`) are defined with Rocket's Custom response type, allowing structured JSON error responses with status codes. The `compose_error_response` function provides a reusable way to create these errors.
- **Type Aliases**: The `Result<T, E>` type alias simplifies error handling throughout the codebase by using the custom Error type as the default error variant.

**2. Since you have completed the tutorial by now and have tried to test your notification system by spawning multiple instances of Receiver, explain how Observer pattern eases you to plug in more subscribers. How about spawning more than one instance of Main app, will it still be easy enough to add to the system?**
Ok, let's answer this question.
The Observer pattern is a behavioral design pattern that establishes a one-to-many relationship between a subject (publisher) and its observers (subscribers). In your bambangshop-receiver Rust project, this pattern is implemented through HTTP-based communication, where the Receiver acts as an observer that subscribes to notifications from a central publisher (likely a separate application running on port 8000 by default). Let me explain how this pattern facilitates adding more subscribers and address the implications of spawning multiple instances of the "Main app" (which I interpret as the publisher based on your code and tutorial context).

##### **How the Observer Pattern Eases Adding More Subscribers**
In the Observer pattern, the subject (publisher) maintains a list of observers and notifies them of state changes without needing to know the specifics of each observer's implementation. This decoupling makes it straightforward to add new subscribers dynamically. Here's how it applies to your notification system:

##### **Loose Coupling Between Publisher and Subscribers:**

Subscribers (like your Receiver instances) implement a simple interface: they provide a URL endpoint (/receive) where they can accept notifications via HTTP POST. The publisher doesn't need to know anything about the subscriber's internal logic (e.g., how notifications are stored in the NotificationRepository or displayed).
Registration is handled via a simple API call: Subscribers send a SubscriberRequest (containing their url and name) to the publisher's `/notification/subscribe/{product_type}` endpoint. The publisher adds the subscriber to its internal list (likely a map or vector keyed by `product_type`).
When a notification event occurs (e.g., a product is created or deleted), the publisher iterates through its subscriber list and sends HTTP POST requests to each registered URL with the Notification payload.
**Ease of Adding More Subscribers:**

- **No Code Changes Required on the Publisher Side**: The publisher's notification logic remains unchanged. It simply appends new subscribers to its list when they register. In your case, spawning additional Receiver instances (e.g., on different ports like 8002, 8003) and having them subscribe to the same `product_type` is trivial—the publisher automatically includes them in future broadcasts.
- **Subscriber Independence**: Each Receiver instance is self-contained. You can run multiple instances in parallel (e.g., via `cargo run` in separate terminals or Docker containers), each with its own configuration (e.g., via environment variables like `APP_INSTANCE_ROOT_URL` and `APP_INSTANCE_NAME`). They store notifications independently in their own `RwLock<Vec<Notification>>` repository, allowing for horizontal scaling without conflicts.
- **Dynamic Registration/Unregistration**: Subscribers can join or leave at runtime via `/subscribe` and `/unsubscribe` endpoints. This is ideal for scenarios like load balancing or fault tolerance—if one Receiver instance goes down, others continue receiving notifications.
- **Practical Example from Your Code**: In `notification.rs`, the subscribe method sends a request to the publisher with the Receiver's URL (e.g., http://localhost:8001/receive). The publisher responds with a confirmation, and from then on, it will POST notifications to that URL. Adding a second Receiver on port 8002 just requires updating its config and subscribing—it plugs in seamlessly without modifying the publisher or other subscribers. This ease of extension is a core strength of the Observer pattern, making it perfect for event-driven systems like yours, where the number of subscribers can vary dynamically.

Spawning More Than One Instance of the Main App (Publisher)
The "Main app" likely refers to the publisher application (running on port 8000), which acts as the subject in the Observer pattern. Spawning multiple instances of the publisher introduces complexity and is not as straightforward as adding subscribers, for several reasons rooted in the pattern's design and your system's architecture:

Challenges with Multiple Publishers:

- **Observer Pattern Assumption**: The classic Observer pattern assumes a single subject that broadcasts to multiple observers. With multiple publishers, you'd need a way to coordinate notifications across them (e.g., ensuring a product creation event is broadcasted by all publishers or deduplicated). Without this, subscribers might receive duplicate notifications or miss events if they only subscribe to one publisher.
- **Subscription Management**: In your current setup, subscribers register with a single publisher URL (configured via APP_PUBLISHER_ROOT_URL). If there are multiple publishers, subscribers would need to know about and subscribe to each one individually, leading to manual configuration overhead. For example, a Receiver would have to call /subscribe on Publisher A, Publisher B, etc., which violates the pattern's simplicity.
- **State Synchronization**: Publishers likely maintain their own state (e.g., the list of subscribers per `product_type`). If multiple instances exist, they'd need shared state (e.g., via a database or distributed cache) to avoid inconsistencies. In your code, the publisher's logic isn't visible here, but if it's storing subscribers in memory (like your Receiver does with notifications), multiple instances would have isolated lists, breaking the system.
- **Notification Routing**: When an event occurs, which publisher handles the broadcast? If publishers are independent, you would face risk incomplete notifications. A common solution is a Mediator pattern or a message broker (e.g., RabbitMQ or Azure Service Bus, which might be the migration target in your tutorial), but that adds significant complexity.

##### **Is It Still "Easy Enough"?:**

- **Not as Easy as Adding Subscribers**: Unlike observers, which are plug-and-play, multiple subjects require architectural changes. You'd need to implement discovery (e.g., a registry service for publishers), load balancing, or event aggregation. For instance, you could introduce a central "notification hub" that publishers register with, and subscribers subscribe to the hub instead. However, this shifts away from the pure Observer pattern and increases maintenance.
Potential Workarounds: If publishers are stateless and events are external (e.g., triggered by a shared database), you could run multiple instances behind a load balancer. But based on your HTTP-based implementation, this would require refactoring the subscription mechanism to support multiple endpoints.
- **Practical Impact**: In testing, spawning multiple publishers might lead to issues like subscribers missing notifications or receiving them multiple times. It's feasible for development (e.g., via Docker Compose with multiple containers), but production would need careful design to avoid race conditions or data inconsistencies.
In summary, the Observer pattern excels at scaling observers (subscribers) because of its loose coupling—adding more Receivers is as simple as running extra instances and subscribing. However, scaling the subject (publisher) to multiple instances requires additional patterns or infrastructure, making it less "easy" and potentially introducing reliability challenges. If your tutorial involves migrating to a cloud-native setup (e.g., Azure), tools like Azure Event Grid or Service Bus could simplify multi-publisher scenarios by acting as a centralized mediator. 

**3. Have you tried to make your own Tests, or enhance documentation on your Postman collection? If you have tried those features, tell us whether it is useful for your work (it can be your tutorial work or your Group Project).**

Yes — I’ve used both tests and documentation enhancements in API collections before, and they’re extremely useful for team projects and tutorials.

Postman tests made it easy to verify API behavior after changes (status codes, response schema, auth flow).
Rich docs in collection helped teammates understand what each endpoint does, required inputs, and expected outputs.
This saved time during integration and reduced misunderstandings in group projects.

##### **Why It Helps**

**For tutorials**: inline request descriptions and examples are a teaching accelerator.
**For group work**: test scripts in Postman catch regressions early and provide automated sanity checks.
**For handoff**: generated docs are a lightweight API contract for developers and quality assurance (QA).