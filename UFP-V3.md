# Universal Functional Pseudocode (UFP) v3.0
## The Essential Specification

---

## 1. Core Philosophy

UFP expresses software intent with mathematical clarity. It is a specification language, not executable code.

**Three principles:**
- Declare *what*, not *how*
- Immutable data, pure functions
- Explicit effects, explicit errors

---

## 2. Types

```
// Primitives
int  float  decimal  string  bool  void  bytes

// Composites
[T]                 // Array
map<K,V>            // Dictionary  
{T, U}              // Tuple
Option<T>           // Some(T) | None
Result<T,E>         // Success(T) | Error(E)

// Refined
type Email = string where { contains("@") }
type Age = int where { value >= 0 && value <= 150 }
type Port = int where { value >= 1 && value <= 65535 }

// Durations
100ms  5s  1m  24h  7d
```

---

## 3. Data

```
// Structs
struct User = {
  id: string,
  name: string,
  email: Email,
  createdAt: timestamp,
  @version version: int
}

// Enums  
enum Status = Draft | Active | Archived

// Invariants
invariant User {
  length(name) > 0
  version >= 1
}
```

---

## 4. Functions

```
// Pure function (no effects, no errors)
@pure
function add(a: int, b: int) -> int {
  a + b
}

// With effects (declared, not implemented)
function getUser(id: string) -> Result<User, Error> {
  let user = fetchUser(id)?     // effect: database
  log("User fetched", Info)     // effect: logging
  Success(user)
}

// Effect declaration
effect fetchUser(id: string) -> User
@implementation {
  target: "database",
  table: "users",
  timeout: 500ms,
  retry: 3
}

effect log(message: string, level: string) -> void
```

---

## 5. Expressions

```
// Control flow (all return values)
let status = if age >= 18 then "adult" else "minor"

let result = case user.role of
  "admin" -> fullAccess()
  "user"  -> limitedAccess()
  _       -> Error("Unknown role")

// Error propagation
let user = getUser(id)?              // Returns error early
let user = getUser(id)? :> AppError  // Maps error type

// Composition
let result = 
  users
  |> filter(u -> u.active)
  |> map(u -> u.email)
  |> take(10)

// Blocks (last expression is value)
let value = {
  let a = computeX()
  let b = computeY()
  a + b
}

// Resources (auto cleanup)
let data = using (conn = connect(url)) {
  conn.query("SELECT * FROM users")
}
```

---

## 6. Concurrency

```
// Parallel execution
parallel {
  let user = fetchUser(id)
  let permissions = getPermissions(id)
} -> (user, permissions)

// Async operations
let result = async {
  let data = await fetchData()
  processData(data)
}

// Race (first result wins)
let result = race {
  let r1 = fetchPrimary()
  let r2 = fetchCache()
} -> r1
```

---

## 7. State Machines

```
state DocumentState {
  Draft -> Published when { user.isAuthor }
  Draft -> Archived
  Published -> Archived when { user.isAdmin }
  Published -> Draft
}

// Transition function
function transition(state: DocumentState, action: string, user: User) -> DocumentState {
  // Validates against state rules above
}
```

---

## 8. Database

```
@table("users")
struct User = {
  @primaryKey id: string,
  @index email: Email,
  @createdAt createdAt: timestamp,
  @updatedAt updatedAt: timestamp
}

@repository(User) {
  findById(id: string) -> Option<User>
  findByEmail(email: Email) -> Option<User>
  save(user: User) -> Result<User, Error>
  delete(id: string) -> Result<bool, Error>
}

@transaction(isolation: "serializable")
function transfer(from: Account, to: Account, amount: decimal) -> Result<Success, Error> {
  // ...
}
```

---

## 9. Interfaces

```
// Unified interface definition
@interface(type: REST, method: GET, path: "/users/:id")
@interface(type: REST, method: POST, path: "/users")
@interface(type: WebSocket, event: "document.sync")
@interface(type: GraphQL, kind: Query)
@interface(type: gRPC, service: "UserService")

// With auth and rate limiting
@interface(type: REST, method: GET, path: "/users/:id") {
  auth: required,
  rateLimit: "1000/min",
  cache: { ttl: 5m }
}
function getUser(id: string) -> Result<User, Error>

// Events
@event UserCreated {
  userId: string,
  timestamp: timestamp
}

@subscribe(UserCreated)
function onUserCreated(event: UserCreated) -> void {
  sendWelcomeEmail(event.userId)
}
```

---

## 10. Testing

```
@Test("User creation succeeds with valid data")
function testCreateUser() -> Result<bool, string> {
  let user = createUser("john@email.com", "John")
  user.name == "John"
}

@Test("Email validation rejects invalid", kind: Property, samples: 100)
function propertyEmailValidation(email: string) -> bool {
  validateEmail(email) == contains(email, "@")
}

@Test("Checkout completes in 500ms", kind: Benchmark, iterations: 1000)
function benchmarkCheckout() -> Order {
  processCheckout(testCart)
}
```

---

## 11. Observability

```
// Logging
@log(level: INFO, message: "User {userId} created")
function createUser(data: UserData) -> Result<User, Error>

// Metrics
@metrics {
  counter: "errors" { labels: ["type"] },
  gauge: "active_users",
  histogram: "latency" { buckets: [10ms, 50ms, 100ms, 500ms] }
}
function handleRequest(req: Request) -> Response

// Tracing
@trace(includeArgs: true)
function processOrder(order: Order) -> Result<Success, Error>

// Health
@health(name: "database", timeout: 1s) {
  using (conn = getConnection()) { conn.ping() }
}
```

---

## 12. Configuration

```
@environment(dev) {
  databaseUrl: "postgres://localhost/dev",
  logLevel: "DEBUG",
  retryCount: 1
}

@environment(prod) {
  databaseUrl: "postgres://prod:5432/prod",
  logLevel: "WARN",
  retryCount: 5,
  replicas: 3
}

// Feature flags
@feature("new_ui", default: false)
@feature("ai_search", default: false, experiments: ["beta"])
```

---

## 13. Constraints

```
constraints {
  // Performance
  p95Latency: 200ms,
  throughput: 1000/sec,
  memoryLimit: 2GB,
  
  // Security
  encryption: "transit+rest",
  authentication: "jwt",
  rateLimit: "100/sec",
  auditLog: true,
  
  // Reliability  
  uptime: "99.95%",
  retryCount: 5,
  circuitBreaker: { threshold: 5, timeout: 30s }
}
```

---

## 14. Modules

```
module ecommerce/checkout {
  import common/database
  import common/auth
  
  export processCheckout
  export CheckoutResult
  
  struct CheckoutResult = {
    success: bool,
    order: Option<Order>,
    error: Option<string>
  }
  
  function processCheckout(cart: Cart, payment: Payment) -> CheckoutResult {
    let user = authenticateUser(payment.token)?
    let order = createOrder(user, cart)
    let result = chargePayment(payment, order.total)?
    saveOrder(order)
    Success(CheckoutResult { success: true, order: Some(order), error: None })
  }
}
```

---

## 15. Migration

```
@version(1) struct Document = {
  id: string,
  content: string
}

@version(2) struct Document = {
  id: string,
  title: string,      // Added
  content: string,
  tags: [string]       // Added
}

@migration(from: 1, to: 2) {
  title = split(content, "\n")[0],
  tags = []
}
```

---

## 16. Complete Example: Todo API

```
// todo/api.ufp

module todo/api {
  import common/database
  import common/auth
  
  // Types
  struct Todo = {
    @primaryKey id: string,
    title: string where { length > 0 && length <= 200 },
    completed: bool,
    userId: string,
    @createdAt createdAt: timestamp,
    @updatedAt updatedAt: timestamp
  }
  
  enum Error = {
    NotFound,
    Unauthorized,
    ValidationError(string)
  }
  
  // Database
  @table("todos")
  @index("userId")
  @repository(Todo) {
    findByUser(userId: string) -> [Todo]
    create(todo: Todo) -> Result<Todo, Error>
    update(todo: Todo) -> Result<Todo, Error>
    delete(id: string) -> Result<bool, Error>
  }
  
  // API
  @interface(type: REST, method: GET, path: "/todos") {
    auth: required
  }
  @log(level: INFO, message: "Fetching todos for {userId}")
  @trace
  function getTodos(userId: string) -> Result<[Todo], Error> {
    Todo.findByUser(userId)
  }
  
  @interface(type: REST, method: POST, path: "/todos") {
    auth: required,
    rateLimit: "100/min"
  }
  @metrics { counter: "todos_created" }
  function createTodo(userId: string, data: TodoInput) -> Result<Todo, Error> {
    if length(data.title) == 0 then
      Error(ValidationError("Title required"))
    
    let todo = Todo {
      id: generateId(),
      title: data.title,
      completed: false,
      userId: userId,
      createdAt: now(),
      updatedAt: now()
    }
    
    Todo.create(todo)
  }
  
  @interface(type: REST, method: PUT, path: "/todos/:id") {
    auth: required
  }
  function updateTodo(userId: string, id: string, data: TodoInput) -> Result<Todo, Error> {
    let todo = Todo.findById(id)?
    if todo.userId != userId then Error(Unauthorized)
    
    let updated = { todo | title: data.title, completed: data.completed, updatedAt: now() }
    Todo.update(updated)
  }
  
  @interface(type: REST, method: DELETE, path: "/todos/:id") {
    auth: required,
    rateLimit: "50/min"
  }
  function deleteTodo(userId: string, id: string) -> Result<bool, Error> {
    let todo = Todo.findById(id)?
    if todo.userId != userId then Error(Unauthorized)
    Todo.delete(id)
  }
  
  @Test("User can create and fetch todos")
  function testTodoFlow() -> Result<bool, string> {
    let todo = createTodo("user1", { title: "Test", completed: false })?
    let todos = getTodos("user1")?
    contains(todos, todo)
  }
  
  constraints {
    p95Latency: 200ms,
    maxTodoItems: 500
  }
}
```

---

## Implementation Mapping

| UFP | Go | Rust | TypeScript |
|-----|-----|------|------------|
| `struct` | `type struct` | `struct` | `interface` |
| `Result<T,E>` | `(T, error)` | `Result<T,E>` | `T \| Error` |
| `Option<T>` | `*T` | `Option<T>` | `T \| null` |
| `[T]` | `[]T` | `Vec<T>` | `T[]` |
| `map<K,V>` | `map[K]V` | `HashMap<K,V>` | `Map<K,V>` |
| `\|>` | method chaining | `.iter()` | method chaining |
| `using` | `defer` | `Drop` | `using`/`try-finally` |
| `parallel` | goroutines | `rayon` | `Promise.all` |
| `async` | goroutines | `async` | `async/await` |
| `?` operator | `if err != nil` | `?` | `throw`/`return` |

