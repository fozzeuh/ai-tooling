# Clean Code Practices

> Parts of this guide are based on concepts from the **"Clean Code" course summary** by [Academind GmbH / Maximilian Schwarzmuller](https://academind.com) (c) 2020.

## What is Clean Code?

Code that is:
- **Easy to understand** - reveals intent clearly
- **Easy to change** - modifications are localized
- **Easy to test** - dependencies are injectable
- **Simple** - no unnecessary complexity

## The Human-Centered Approach

Code has THREE consumers:
1. **Users** - get their needs met
2. **Customers** - make or save money
3. **Developers** - must maintain it

Design for all three, but remember: **developers read code 10x more than they write it.**

> Compatibility note (.NET): C# properties and records are legitimate language constructs. Treat numeric thresholds in this guide as heuristics, and favor code that is clear in your project context.

## Naming Principles

### 1. Consistency & Uniqueness (HIGHEST PRIORITY)
Same concept = same name everywhere. One name per concept.

```typescript
// BAD: Inconsistent names for same concept
getUserById(id)
fetchCustomerById(id)
retrieveClientById(id)

// GOOD: Consistent
getUser(id)
getOrder(id)
getProduct(id)
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD
User GetUserById(UserId id) => ...;
User FetchCustomerById(UserId id) => ...;
User RetrieveClientById(UserId id) => ...;

// GOOD
User GetUser(UserId id) => ...;
Order GetOrder(OrderId id) => ...;
Product GetProduct(ProductId id) => ...;
```

### 2. Understandability
Use domain language, not technical jargon.

```typescript
// BAD: Technical
const arr = users.filter(u => u.isActive);

// GOOD: Domain language
const activeCustomers = users.filter(user => user.isActive);
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Technical
var arr = users.Where(u => u.IsActive);

// GOOD: Domain language
var activeCustomers = users.Where(user => user.IsActive);
```

### 3. Specificity
Avoid vague names: `data`, `info`, `manager`, `handler`, `processor`, `utils`

```typescript
// BAD: Vague
class DataManager { }
function processInfo(data) { }

// GOOD: Specific
class OrderRepository { }
function validatePayment(payment) { }
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Vague
public sealed class DataManager { }
void ProcessInfo(object data) { }

// GOOD: Specific
public sealed class OrderRepository { }
void ValidatePayment(Payment payment) { }
```

### 4. Brevity (but not at cost of clarity)
Short names are good only if meaning is preserved.

```typescript
// BAD: Too cryptic
const usrLst = getUsrs();

// BAD: Unnecessarily long
const listOfAllActiveUsersInTheSystem = getActiveUsers();

// GOOD: Brief but clear
const activeUsers = getActiveUsers();
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Too cryptic
var usrLst = GetUsrs();

// BAD: Unnecessarily long
var listOfAllActiveUsersInTheSystem = GetActiveUsers();

// GOOD: Brief but clear
var activeUsers = GetActiveUsers();
```

### 5. Searchability
Names should be unique enough to grep/search.

```typescript
// BAD: Common word, hard to search
const data = fetch();

// GOOD: Unique, searchable
const orderSummary = fetchOrderSummary();
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Common word, hard to search
var data = Fetch();

// GOOD: Unique, searchable
var orderSummary = FetchOrderSummary();
```

### 6. Pronounceability
You should be able to say it in conversation.

```typescript
// BAD
const genymdhms = generateYearMonthDayHourMinuteSecond();

// GOOD
const timestamp = generateTimestamp();
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD
var genymdhms = GenerateYearMonthDayHourMinuteSecond();

// GOOD
var timestamp = GenerateTimestamp();
```

### 7. Austerity
Avoid unnecessary filler words.

```typescript
// BAD: Redundant
const userData = user; // 'Data' adds nothing
class UserClass { }    // 'Class' adds nothing

// GOOD
const user = user;
class User { }
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Redundant
var userData = user; // 'Data' adds nothing
public sealed class UserClass { } // 'Class' adds nothing

// GOOD
var currentUser = user;
public sealed class User { }
```

---

## Functions & Methods

Functions are the building blocks of any codebase. Clean functions are **small, focused, and readable**.

### Minimize Parameters

The fewer parameters, the easier to read and call.

```typescript
// BEST: No parameters - clear intent
createSession();
user.save();

// GOOD: One parameter - straightforward
isValid(email);
file.write(data);

// OKAY: Two parameters - depends on context
login('test@test.com', 'testers');
createProduct('Carpet', 12.99);

// BAD: Three+ parameters - hard to read and call
createUser('Max', 'Max', 'test@test.com', 'testers', 31, ['Sports', 'Cooking']);
```

**C#/.NET (illustrative counterpart):**
```csharp
// BEST: No parameters - clear intent
CreateSession();
user.Save();

// GOOD: One parameter - straightforward
IsValid(email);
file.Write(data);

// OKAY: Two parameters - depends on context
Login("test@test.com", "testers");
CreateProduct("Carpet", 12.99m);

// BAD: Three+ parameters - hard to read and call
CreateUser("Max", "Max", "test@test.com", "testers", 31, ["Sports", "Cooking"]);
```

**Reduce parameters by grouping into objects:**

```typescript
// BAD: Many positional parameters
createRectangle(10, 9, 30, 12);

// GOOD: Parameter object
createRectangle({ x: 10, y: 9, width: 30, height: 12 });
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD
CreateRectangle(10, 9, 30, 12);

// GOOD
CreateRectangle(new RectangleDimensions(X: 10, Y: 9, Width: 30, Height: 12));

public readonly record struct RectangleDimensions(int X, int Y, int Width, int Height);
```

### Keep Functions Small

Small functions are easier to read and force you to extract well-named helper functions.

```typescript
// BAD: Long function with mixed concerns
function login(email, password) {
  if (!email.includes('@') || password.length < 7) {
    throw new Error('Invalid input!');
  }
  const existingUser = database.find('users', 'email', '==', email);
  if (!existingUser) {
    throw new Error('Could not find a user for the provided email.');
  }
  if (existingUser.password === password) {
    // create a session
  } else {
    throw new Error('Invalid credentials!');
  }
}

// GOOD: Small, composed functions
function login(email, password) {
  validateUserInput(email, password);
  const existingUser = findUserByEmail(email);
  existingUser.validatePassword(password);
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public void Login(string email, string password)
{
    ValidateUserInput(email, password);
    var existingUser = FindUserByEmail(email);
    existingUser.ValidatePassword(password);
}
```

### Do One Thing (Levels of Abstraction)

Functions should do **one thing** at **one level of abstraction**.

All operations in the function body should be on the **same level** of abstraction, exactly **one level below** the function name.

```typescript
// BAD: Mixed levels of abstraction
function printDocument(documentPath) {
  const fsConfig = { mode: 'read', onError: 'retry' };
  const document = fileSystem.readFile(documentPath, fsConfig);
  const printer = new Printer('pdf');
  printer.print(document);
}

// GOOD: Consistent abstraction level
function printDocument(documentPath) {
  const document = readFromFile(documentPath);
  const printer = new Printer('pdf');
  printer.print(document);
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD
public void PrintDocument(string documentPath)
{
    var fsConfig = new FileReadConfiguration("read", "retry");
    var document = _fileSystem.ReadFile(documentPath, fsConfig);
    var printer = new Printer("pdf");
    printer.Print(document);
}

// GOOD
public void PrintDocument(string documentPath)
{
    var document = ReadFromFile(documentPath);
    var printer = new Printer("pdf");
    printer.Print(document);
}
```

### Split Functions Reasonably

Don't extract just for the sake of extraction. **Three warning signs** of bad splits:
1. You're just **renaming the operation** (e.g., `buildUser()` that just calls `new User()`)
2. You need to **scroll more** to follow a simple function
3. You **can't find a good name** because the original name is already taken

```typescript
// OVER-SPLIT: Too many trivial extractions
function createUser(email, password) {
  validateInput(email, password);
  saveUser(email, password);
}
function saveUser(email, password) {
  const user = buildUser(email, password);
  user.save();
}
function buildUser(email, password) {
  return new User(email, password);  // just renaming!
}

// BETTER: Meaningful splits only
function createUser(email, password) {
  validateInput(email, password);
  const user = new User(email, password);
  user.save();
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// OVER-SPLIT
public void CreateUser(string email, string password)
{
    ValidateInput(email, password);
    SaveUser(email, password);
}

private void SaveUser(string email, string password)
{
    var user = BuildUser(email, password);
    user.Save();
}

private static User BuildUser(string email, string password) => new(email, password);

// BETTER
public void CreateUserBetter(string email, string password)
{
    ValidateInput(email, password);
    var user = new User(email, password);
    user.Save();
}
```

**Two rules of thumb for when to split:**
1. Extract code that works on the **same functionality** / is closely related
2. Extract code that requires **more interpretation** than the surrounding code

### Avoid Unexpected Side Effects

A side effect is unexpected when the function name doesn't imply it.

```typescript
// BAD: Unexpected side effect - name says "validate", but creates a session
function validateUserInput(email, password) {
  if (!isEmail(email) || passwordIsInvalid(password)) {
    throw new Error('Invalid input!');
  }
  createSession();  // unexpected!
}

// GOOD: Move side effect out, or rename to imply it
function validateAndInitSession(email, password) {
  validateUserInput(email, password);
  createSession();
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD
public void ValidateUserInput(string email, string password)
{
    if (!EmailIsValid(email) || PasswordIsInvalid(password))
        throw new ArgumentException("Invalid input.");

    CreateSession(); // unexpected side effect
}

// GOOD
public static void ValidateUserInputWithoutSideEffects(string email, string password)
{
    if (!EmailIsValid(email) || PasswordIsInvalid(password))
        throw new ArgumentException("Invalid input.");
}

public Session ValidateAndInitSession(string email, string password)
{
    ValidateUserInputWithoutSideEffects(email, password); // Validation stays side-effect free.
    return CreateSession(); // Exactly one side effect.
}
```

---

## Control Structures

Control structures (`if`, `for`, `while`, `switch`) can lead to suboptimal code. Three areas to focus on:

### Prefer Positive Checks

Use positive wording in conditions when possible.

```typescript
// GOOD: Positive check - zero thinking required
if (isEmpty(blogContent)) {
  // throw error
}

// LESS CLEAR: Negated positive - extra interpretation
if (!hasContent(blogContent)) {
  // throw error
}
```

**C#/.NET (illustrative counterpart):**
```csharp
if (string.IsNullOrWhiteSpace(blogContent))
{
    throw new ArgumentException("Blog content is required.", nameof(blogContent));
}
```

> Compatibility note (.NET): `switch` / switch expressions are often a good fit for closed sets (for example enums). Prefer polymorphism when the variation is open or repeatedly duplicated.

Exception: sometimes a negative check is cleaner when there are multiple valid states.

```typescript
// GOOD: Negative check avoids listing all invalid states
if (!isOpen(transaction)) {
  // throw error
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// GOOD: Negative check avoids listing all invalid states
if (!transaction.IsOpen)
{
    throw new InvalidOperationException("Transaction is closed.");
}
```

### Avoid Deep Nesting

Deeply nested code is hard to read and error-prone. Strategies to flatten:

#### 1. Use Guards & Fail Fast

```typescript
// BAD: Deep nesting
function messageUser(user, message) {
  if (user) {
    if (message) {
      if (user.acceptsMessages) {
        const success = user.sendMessage(message);
        if (success) {
          console.log('Message sent!');
        }
      }
    }
  }
}

// GOOD: Guard clause, fail fast
function messageUser(user, message) {
  if (!user || !message || !user.acceptsMessages) {
    return;
  }
  user.sendMessage(message);
  if (success) {
    console.log('Message sent!');
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public void MessageUser(User? user, string? message)
{
    if (user is null || string.IsNullOrWhiteSpace(message) || !user.AcceptsMessages)
        return;

    var success = user.SendMessage(message);
    if (success)
        _logger.LogInformation("Message sent.");
}
```

#### 2. Extract Control Structures into Functions

```typescript
// BAD: Nested control structure
function connectDatabase(uri) {
  if (!uri) {
    throw new Error('An URI is required!');
  }
  const db = new Database(uri);
  let success = db.connect();
  if (!success) {
    if (db.fallbackConnection) {
      return db.fallbackConnectionDetails;
    } else {
      throw new Error('Could not connect!');
    }
  }
  return db.connectionDetails;
}

// GOOD: Extract nested logic
function connectDatabase(uri) {
  validateUri(uri);

  const db = new Database(uri);
  let success = db.connect();
  if (success) {
    return db.connectionDetails;
  } else {
    return connectFallbackDatabase(db);
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public ConnectionDetails ConnectDatabase(string uri)
{
    ValidateUri(uri);

    var db = new Database(uri);
    var success = db.Connect();
    if (success)
        return db.ConnectionDetails;

    return ConnectFallbackDatabase(db);
}
```

#### 3. Use Polymorphism & Factory Functions

Replace duplicated conditional logic with polymorphic objects.

```typescript
// BAD: Repeated type checks
function processTransaction(transaction) {
  if (isPayment(transaction)) {
    if (usesCreditCard(transaction)) {
      processCreditCardPayment(transaction);
    }
    if (usesPayPal(transaction)) {
      processPayPalPayment(transaction);
    }
  } else {
    if (usesCreditCard(transaction)) {
      processCreditCardRefund(transaction);
    }
    if (usesPayPal(transaction)) {
      processPayPalRefund(transaction);
    }
  }
}

// GOOD: Factory function with polymorphic object
function getProcessors(transaction) {
  if (usesCreditCard(transaction)) {
    return { processPayment: processCreditCardPayment, processRefund: processCreditCardRefund };
  }
  if (usesPayPal(transaction)) {
    return { processPayment: processPayPalPayment, processRefund: processPayPalRefund };
  }
}

function processTransaction(transaction) {
  const processors = getProcessors(transaction);
  if (isPayment(transaction)) {
    processors.processPayment(transaction);
  } else {
    processors.processRefund(transaction);
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public interface ITransactionProcessor
{
    void ProcessPayment(Transaction transaction);
    void ProcessRefund(Transaction transaction);
}

public void ProcessTransaction(Transaction transaction, ITransactionProcessor processor)
{
    if (transaction.IsPayment)
    {
        processor.ProcessPayment(transaction);
        return;
    }

    processor.ProcessRefund(transaction);
}
```

#### 4. Embrace Errors (throw instead of error codes)

```typescript
// BAD: Synthetic errors with status codes
function createUser(email, password) {
  const inputValidity = validateInput(email, password);
  if (inputValidity.code === 1 || inputValidity.code === 2) {
    console.log(inputValidity.message);
    return;
  }
  // ... continue
}

// GOOD: Use real errors with try-catch
function handleSignupRequest(request) {
  try {
    createUser(request.email, request.password);
  } catch (error) {
    console.log(error.message);
  }
}

function createUser(email, password) {
  validateInput(email, password);
  // ... continue
}

function validateInput(email, password) {
  if (!email.includes('@') || password.length < 7) {
    throw new Error('Input is invalid!');
  }
  const existingUser = findUserByEmail(email);
  if (existingUser) {
    throw new Error('Email is already taken!');
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public void HandleSignupRequest(SignupRequest request)
{
    try
    {
        CreateUser(request.Email, request.Password);
    }
    catch (ArgumentException ex)
    {
        _logger.LogWarning(ex, "Invalid signup request.");
    }
}

public void CreateUser(string email, string password)
{
    ValidateInput(email, password);
    // ... continue
}
```

---

## Classes & Objects

### Objects vs Data Containers

Distinguish between **objects** (hide data, expose behavior) and **data containers** (expose data, no behavior).

```typescript
// Data Container: exposes data publicly
class UserData {
  public name: string;
  public age: number;
}

// Object: hides data, exposes behavior
class User {
  private name: string;
  private age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`Hi! I'm ${this.name} and I'm ${this.age} years old.`);
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// Data container (DTO)
public sealed record UserData(string Name, int Age);

// Behavior-rich object
public sealed class User
{
    private readonly string _name;
    private readonly int _age;

    public User(string name, int age)
    {
        _name = name;
        _age = age;
    }

    public string Greet() => $"Hi! I'm {_name} and I'm {_age} years old.";
}
```

> Compatibility note (.NET): records and auto-properties are fine for data containers and DTOs. Preserve behavior-rich domain objects where invariants or decisions belong.

Both are valid - use the right kind for the right job. **Don't mix types**: don't access internals of objects, and don't add behavior to data containers.

### Class Cohesion

**Cohesion** = how much methods use class properties. High cohesion means every method uses most properties.

- **High cohesion** → well-designed, focused class
- **Low cohesion** → should probably be a data container or split into smaller classes

When cohesion decreases, **split into smaller, more focused classes**.

---

## Object Calisthenics (9 Rules)

Exercises to improve OO design. Follow strictly during practice, relax slightly in production.

### 1. One Level of Indentation per Method

```typescript
// BAD: Multiple levels
function process(orders: Order[]) {
  for (const order of orders) {
    if (order.isValid()) {
      for (const item of order.items) {
        if (item.inStock) {
          // process...
        }
      }
    }
  }
}

// GOOD: Extract methods
function process(orders: Order[]) {
  orders.filter(o => o.isValid()).forEach(processOrder);
}

function processOrder(order: Order) {
  order.items.filter(i => i.inStock).forEach(processItem);
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Multiple levels
void Process(IEnumerable<Order> orders)
{
    foreach (var order in orders)
    {
        if (!order.IsValid())
            continue;

        foreach (var item in order.Items)
        {
            if (item.InStock)
            {
                ProcessItem(item);
            }
        }
    }
}

// GOOD: Extract methods
void ProcessFlat(IEnumerable<Order> orders)
{
    foreach (var order in orders.Where(o => o.IsValid()))
        ProcessOrder(order);
}

void ProcessOrder(Order order)
{
    foreach (var item in order.Items.Where(i => i.InStock))
        ProcessItem(item);
}
```

### 2. Don't Use the ELSE Keyword

Use early returns, guard clauses, or polymorphism.

```typescript
// BAD: else
function getDiscount(user: User): number {
  if (user.isPremium) {
    return 20;
  } else {
    return 0;
  }
}

// GOOD: Early return
function getDiscount(user: User): number {
  if (user.isPremium) return 20;
  return 0;
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: else
decimal GetDiscount(User user)
{
    if (user.IsPremium)
    {
        return 20m;
    }
    else
    {
        return 0m;
    }
}

// GOOD: Early return
decimal GetDiscountFast(User user)
{
    if (user.IsPremium) return 20m;
    return 0m;
}
```

### 3. Wrap All Primitives and Strings

Primitives should be wrapped in domain objects when they have meaning.

```typescript
// BAD: Primitive obsession
function createUser(email: string, age: number) { }

// GOOD: Value objects
class Email {
  constructor(private value: string) {
    if (!this.isValid(value)) throw new InvalidEmail();
  }
  private isValid(email: string): boolean { ... }
}

class Age {
  constructor(private value: number) {
    if (value < 0 || value > 150) throw new InvalidAge();
  }
}

function createUser(email: Email, age: Age) { }
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class Email
{
    public string Value { get; }

    public Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value) || !value.Contains('@'))
            throw new ArgumentException("Invalid email.", nameof(value));
        Value = value;
    }
}

public sealed class Age
{
    public int Value { get; }

    public Age(int value)
    {
        if (value < 0 || value > 150)
            throw new ArgumentOutOfRangeException(nameof(value));

        Value = value;
    }
}
```

### 4. First-Class Collections

Any class with a collection should have no other instance variables.

```typescript
// BAD: Collection mixed with other state
class Order {
  items: OrderItem[] = [];
  customerId: string;
  total: number;
}

// GOOD: Collection is its own class
class OrderItems {
  constructor(private items: OrderItem[] = []) {}

  add(item: OrderItem): void { ... }
  total(): Money { ... }
  isEmpty(): boolean { ... }
}

class Order {
  constructor(
    private items: OrderItems,
    private customerId: CustomerId
  ) {}
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Collection mixed with other state
public sealed class Order
{
    public List<OrderItem> Items { get; } = [];
    public string CustomerId { get; init; } = string.Empty;
    public decimal Total { get; private set; }
}

// GOOD: Collection is its own class
public sealed class OrderItems
{
    private readonly List<OrderItem> _items = [];

    public void Add(OrderItem item) => _items.Add(item);
    public Money Total() => _items.Aggregate(Money.Zero, (sum, item) => sum.Add(item.Total));
    public bool IsEmpty() => _items.Count == 0;
}

public sealed class OrderComposed(OrderItems items, CustomerId customerId)
{
    private readonly OrderItems _items = items;
    private readonly CustomerId _customerId = customerId;
}
```

### 5. One Dot per Line (Law of Demeter)

Don't chain through object graphs.

```typescript
// BAD: Train wreck
const city = order.customer.address.city;

// GOOD: Tell, don't ask
const city = order.getShippingCity();
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Train wreck
var city = order.Customer.Address.City;

// GOOD: Tell, don't ask
var shippingCity = order.GetShippingCity();
```

### 6. Don't Abbreviate

If a name is too long to type, the class is doing too much.

```typescript
// BAD
const custRepo = new CustRepo();
const ord = new Ord();

// GOOD
const customerRepository = new CustomerRepository();
const order = new Order();
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD
var custRepo = new CustRepo();
var ord = new Ord();

// GOOD
var customerRepository = new CustomerRepository();
var order = new Order();
```

### 7. Keep All Entities Small

- Classes: < 50 lines
- Methods: < 10 lines
- Files: < 100 lines

If larger, it's probably doing too much. Split it.

### 8. No Classes with More Than Two Instance Variables

Forces small, focused classes.

```typescript
// BAD: Too many variables
class Order {
  id: string;
  customerId: string;
  items: Item[];
  total: number;
  status: string;
}

// GOOD: Composed of smaller objects
class Order {
  constructor(
    private id: OrderId,
    private details: OrderDetails
  ) {}
}

class OrderDetails {
  constructor(
    private customer: Customer,
    private lineItems: LineItems
  ) {}
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Too many variables
public sealed class Order
{
    private readonly string _id;
    private readonly string _customerId;
    private readonly IReadOnlyCollection<Item> _items;
    private readonly decimal _total;
    private readonly string _status;

    public Order(string id, string customerId, IReadOnlyCollection<Item> items, decimal total, string status)
    {
        _id = id;
        _customerId = customerId;
        _items = items;
        _total = total;
        _status = status;
    }
}

// GOOD: Composed of smaller objects
public sealed class OrderComposed(OrderId id, OrderDetails details)
{
    private readonly OrderId _id = id;
    private readonly OrderDetails _details = details;
}

public sealed class OrderDetails(Customer customer, LineItems lineItems)
{
    private readonly Customer _customer = customer;
    private readonly LineItems _lineItems = lineItems;
}
```

### 9. No Getters/Setters/Properties

Objects should have behavior, not just data. Tell objects what to do.

```typescript
// BAD: Data bag with getters
class Account {
  getBalance(): number { return this.balance; }
  setBalance(value: number) { this.balance = value; }
}

// Caller does the work
if (account.getBalance() >= amount) {
  account.setBalance(account.getBalance() - amount);
}

// GOOD: Behavior-rich object
class Account {
  withdraw(amount: Money): WithdrawResult {
    if (!this.canWithdraw(amount)) {
      return WithdrawResult.insufficientFunds();
    }
    this.balance = this.balance.subtract(amount);
    return WithdrawResult.success();
  }
}

// Caller tells, object decides
const result = account.withdraw(amount);
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class Account
{
    private Money _balance;

    public WithdrawResult Withdraw(Money amount)
    {
        if (!CanWithdraw(amount))
            return WithdrawResult.InsufficientFunds();

        _balance = _balance.Subtract(amount);
        return WithdrawResult.Success();
    }

    private bool CanWithdraw(Money amount) => _balance >= amount;
}

var result = account.Withdraw(amount);
```

> Compatibility note (.NET): this rule is about avoiding anemic domain models, not banning C# properties. Auto-properties are idiomatic for DTOs and simple state holders.

---

## Comments

### When to Write Comments

**Only write comments to explain WHY, not WHAT or HOW.**

Code explains what and how. Comments explain business reasons, non-obvious decisions, or warnings.

```typescript
// BAD: Explains what (redundant)
// Add 1 to counter
counter++;

// GOOD: Explains why
// Compensate for 0-based indexing in legacy API
counter++;
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: explains what
counter++;

// GOOD: explains why
// Compensate for 0-based indexing required by the legacy endpoint.
counter++;
```

### Prefer Self-Documenting Code

Instead of commenting, rename to make intent clear.

```typescript
// BAD: Comment needed
// Check if user can access premium features
if (user.subscriptionLevel >= 2 && !user.isBanned) { }

// GOOD: Self-documenting
if (user.canAccessPremiumFeatures()) { }
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD: Comment needed
// Check if user can access premium features
if (user.SubscriptionLevel >= 2 && !user.IsBanned) { }

// GOOD: Self-documenting
if (user.CanAccessPremiumFeatures()) { }
```

---

## Formatting

### Vertical Spacing
- Related code together
- Blank lines between concepts
- Most important/public at top

### Horizontal Spacing
- Consistent indentation
- Space around operators
- Max line length ~80-120 characters

### Storytelling
Code should read top-to-bottom like a story. High-level at top, details below.

```typescript
class OrderProcessor {
  // Public API first
  process(order: Order): ProcessResult {
    this.validate(order);
    this.calculateTotals(order);
    return this.save(order);
  }

  // Supporting methods below, in order of appearance
  private validate(order: Order): void { ... }
  private calculateTotals(order: Order): void { ... }
  private save(order: Order): ProcessResult { ... }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class OrderProcessor
{
    // Public API first
    public ProcessResult Process(Order order)
    {
        Validate(order);
        CalculateTotals(order);
        return Save(order);
    }

    // Supporting methods below
    private void Validate(Order order) { /* ... */ }
    private void CalculateTotals(Order order) { /* ... */ }
    private ProcessResult Save(Order order) => ProcessResult.Success();
}
```

---

## Clean Code Checklist

### Naming
- [ ] Use **descriptive** and meaningful names
  - Variables & Properties: Nouns or short phrases with adjectives
  - Functions & Methods: Verbs or short phrases with adjectives
  - Classes: Nouns
- [ ] Be as **specific** as necessary and possible
- [ ] Use **yes/no "questions"** for booleans (e.g. `isValid`)
- [ ] Avoid **misleading** names
- [ ] Be **consistent** with names (e.g. stick to `get...` instead of `fetch...`)

### Comments & Formatting
- [ ] **Most comments are bad** - avoid them!
- [ ] Acceptable comments: legal info, warnings, regex explanations, todos
- [ ] Use **vertical formatting**: keep related concepts close, separate unrelated ones
- [ ] Write code **top to bottom**: called functions below calling functions
- [ ] Use **horizontal formatting**: avoid long lines, use indentation for scope

### Functions
- [ ] **Limit parameters** - fewer is better, use objects to group
- [ ] Functions should be **small and do one thing**
  - Abstraction levels in the body should be **one level below** the function name
  - **Don't mix** levels of abstraction
  - But **avoid redundant splitting**!
- [ ] Stay **DRY** (Don't Repeat Yourself)
- [ ] **Avoid unexpected side effects**

### Control Structures & Errors
- [ ] Prefer **positive checks**
- [ ] Avoid **deep nesting**
  - Use **guard** statements and fail fast
  - Use **polymorphism** and factory functions
  - **Extract** control structures into separate functions
- [ ] Use **real errors** (throw/catch) instead of synthetic error codes

### Objects & Classes
- [ ] Build either **"real objects"** or **data containers** - don't mix
- [ ] Build **small classes** focused on a **single responsibility** (not "single method"!)
- [ ] Build classes with **high cohesion**
- [ ] Follow the **Law of Demeter** for real objects (avoid `this.customer.lastPurchase.date`)
- [ ] Follow the **SOLID principles** (especially SRP and OCP for clean code)
