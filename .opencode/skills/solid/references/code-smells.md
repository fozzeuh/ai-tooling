# Code Smells & Anti-Patterns

## What Are Code Smells?

Indicators that something MAY be wrong. Not bugs, but design problems that make code hard to understand, change, or test.

> Compatibility note (.NET): thresholds in this guide (10 lines, 50 lines, 3+ parameters, etc.) are heuristics for discussion. Treat them as signals, not absolute limits.

## The Five Categories

### 1. Bloaters
Code that has grown too large.

| Smell | Symptom | Refactoring |
|-------|---------|-------------|
| **Long Method** | > 10 lines | Extract Method |
| **Large Class** | > 50 lines, multiple responsibilities | Extract Class |
| **Long Parameter List** | > 3 parameters | Introduce Parameter Object |
| **Data Clumps** | Same group of variables appear together | Extract Class |
| **Primitive Obsession** | Primitives instead of small objects | Wrap in Value Object |

### 2. Object-Orientation Abusers
Misuse of OO principles.

| Smell | Symptom | Refactoring |
|-------|---------|-------------|
| **Switch Statements** | Type checking, large switch/if-else | Replace with Polymorphism |
| **Parallel Inheritance** | Adding subclass requires adding another | Merge Hierarchies |
| **Refused Bequest** | Subclass doesn't use parent methods | Replace Inheritance with Delegation |
| **Alternative Classes** | Different interfaces, same concept | Rename, Extract Superclass |

### 3. Change Preventers
Code that makes changes difficult.

| Smell | Symptom | Refactoring |
|-------|---------|-------------|
| **Divergent Change** | One class changed for many reasons | Extract Class (SRP) |
| **Shotgun Surgery** | One change touches many classes | Move Method/Field together |
| **Parallel Inheritance** | (see above) | Merge Hierarchies |

### 4. Dispensables
Code that can be removed.

| Smell | Symptom | Refactoring |
|-------|---------|-------------|
| **Comments** | Explaining bad code | Rename, Extract Method |
| **Duplicate Code** | Copy-paste | Extract Method, Pull Up Method |
| **Dead Code** | Unreachable code | Delete |
| **Speculative Generality** | "Just in case" code | Delete (YAGNI) |
| **Lazy Class** | Class that does almost nothing | Inline Class |

### 5. Couplers
Excessive coupling between classes.

| Smell | Symptom | Refactoring |
|-------|---------|-------------|
| **Feature Envy** | Method uses another class's data extensively | Move Method |
| **Inappropriate Intimacy** | Classes know too much about each other | Move Method, Extract Class |
| **Message Chains** | `a.getB().getC().getD()` | Hide Delegate |
| **Middle Man** | Class only delegates | Inline Class |

---

## The Seven Most Common Code Smells

### 1. Long Method

**Symptom:** Method > 10 lines, doing multiple things.

```typescript
// SMELL
function processOrder(order: Order) {
  // Validate
  if (!order.items.length) throw new Error('Empty');
  if (!order.customer) throw new Error('No customer');

  // Calculate
  let total = 0;
  for (const item of order.items) {
    total += item.price * item.quantity;
    if (item.discount) {
      total -= item.discount;
    }
  }

  // Apply tax
  const taxRate = getTaxRate(order.customer.state);
  total = total * (1 + taxRate);

  // Save
  db.orders.insert({ ...order, total });

  // Notify
  emailService.send(order.customer.email, 'Order confirmed');
}

// REFACTORED
function processOrder(order: Order) {
  validateOrder(order);
  const total = calculateTotal(order);
  saveOrder(order, total);
  notifyCustomer(order);
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public void ProcessOrder(Order order)
{
    ValidateOrder(order);
    var total = CalculateTotal(order);
    SaveOrder(order, total);
    NotifyCustomer(order);
}
```

### 2. Large Class

**Symptom:** Class with many responsibilities, > 50 lines.

```typescript
// SMELL: God class
class User {
  // User data
  name: string;
  email: string;

  // Authentication
  login() { }
  logout() { }
  resetPassword() { }

  // Preferences
  setTheme() { }
  setLanguage() { }

  // Notifications
  sendEmail() { }
  sendSMS() { }

  // Billing
  charge() { }
  refund() { }
}

// REFACTORED: Separate classes
class User { name: string; email: string; }
class AuthService { login(); logout(); resetPassword(); }
class UserPreferences { setTheme(); setLanguage(); }
class NotificationService { sendEmail(); sendSMS(); }
class BillingService { charge(); refund(); }
```

**C#/.NET (illustrative counterpart):**
```csharp
// SMELL: God class
public sealed class User
{
    public string Name { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;

    public void Login() { }
    public void Logout() { }
    public void ResetPassword() { }

    public void SetTheme() { }
    public void SetLanguage() { }

    public void SendEmail() { }
    public void SendSms() { }

    public void Charge() { }
    public void Refund() { }
}

// REFACTORED: Separate classes
public sealed class UserProfile
{
    public string Name { get; init; } = string.Empty;
    public string Email { get; init; } = string.Empty;
}

public sealed class AuthService { public void Login() { } public void Logout() { } public void ResetPassword() { } }
public sealed class UserPreferences { public void SetTheme() { } public void SetLanguage() { } }
public sealed class NotificationService { public void SendEmail() { } public void SendSms() { } }
public sealed class BillingService { public void Charge() { } public void Refund() { } }
```

### 3. Feature Envy

**Symptom:** Method uses another class's data more than its own.

```typescript
// SMELL: Order envies Customer
class Order {
  calculateShipping(customer: Customer): number {
    if (customer.country === 'US') {
      if (customer.state === 'CA') return 10;
      return 15;
    }
    return 25;
  }
}

// REFACTORED: Move to Customer
class Customer {
  getShippingCost(): number {
    if (this.country === 'US') {
      if (this.state === 'CA') return 10;
      return 15;
    }
    return 25;
  }
}

class Order {
  calculateShipping(): number {
    return this.customer.getShippingCost();
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class Customer
{
    public decimal GetShippingCost() => Country == "US" && State == "CA" ? 10 : Country == "US" ? 15 : 25;

    public string Country { get; init; } = string.Empty;
    public string State { get; init; } = string.Empty;
}

public sealed class Order
{
    private readonly Customer _customer;
    public Order(Customer customer) => _customer = customer;

    public decimal CalculateShipping() => _customer.GetShippingCost();
}
```

### 4. Primitive Obsession

**Symptom:** Using primitives for domain concepts.

```typescript
// SMELL
function createUser(email: string, age: number, zipCode: string) {
  // No validation, easy to pass wrong values
  if (!email.includes('@')) throw new Error();
  if (age < 0) throw new Error();
}

// REFACTORED: Value objects
class Email {
  constructor(private value: string) {
    if (!value.includes('@')) throw new InvalidEmail();
  }
}

class Age {
  constructor(private value: number) {
    if (value < 0 || value > 150) throw new InvalidAge();
  }
}

function createUser(email: Email, age: Age, address: Address) {
  // Type system prevents invalid data
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class Email
{
    public string Value { get; }
    public Email(string value)
    {
        if (string.IsNullOrWhiteSpace(value) || !value.Contains('@'))
            throw new ArgumentException("Invalid email", nameof(value));
        Value = value;
    }
}

public sealed class Age
{
    public int Value { get; }

    public Age(int value)
    {
        if (value is < 0 or > 150)
            throw new ArgumentOutOfRangeException(nameof(value));

        Value = value;
    }
}
```

### 5. Switch Statements

**Symptom:** Switching on type, repeated across codebase.

```typescript
// SMELL
function getArea(shape: Shape): number {
  switch (shape.type) {
    case 'circle': return Math.PI * shape.radius ** 2;
    case 'rectangle': return shape.width * shape.height;
    case 'triangle': return 0.5 * shape.base * shape.height;
  }
}

function getPerimeter(shape: Shape): number {
  switch (shape.type) { // Same switch again!
    case 'circle': return 2 * Math.PI * shape.radius;
    // ...
  }
}

// REFACTORED: Polymorphism
interface Shape {
  getArea(): number;
  getPerimeter(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}
  getArea(): number { return Math.PI * this.radius ** 2; }
  getPerimeter(): number { return 2 * Math.PI * this.radius; }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public interface IShape
{
    double GetArea();
    double GetPerimeter();
}

public sealed class Circle(double radius) : IShape
{
    public double GetArea() => Math.PI * radius * radius;
    public double GetPerimeter() => 2 * Math.PI * radius;
}
```

> Compatibility note (.NET): a `switch` on a closed enum can still be the clearest choice when there is one stable decision point. Prefer polymorphism when similar switches are duplicated or variation is open.

### 6. Inappropriate Intimacy

**Symptom:** Classes know too much about each other's internals.

```typescript
// SMELL
class Order {
  process() {
    const inventory = new Inventory();
    // Reaching into inventory's internals
    for (const item of this.items) {
      const stock = inventory.stockLevels[item.sku];
      if (stock.quantity < item.quantity) {
        throw new Error('Out of stock');
      }
      inventory.stockLevels[item.sku].quantity -= item.quantity;
    }
  }
}

// REFACTORED: Tell, don't ask
class Inventory {
  reserve(items: OrderItem[]): ReserveResult {
    // Inventory manages its own state
    for (const item of items) {
      if (!this.canReserve(item)) {
        return ReserveResult.outOfStock(item);
      }
    }
    this.deductStock(items);
    return ReserveResult.success();
  }
}

class Order {
  process(inventory: Inventory) {
    const result = inventory.reserve(this.items);
    if (!result.isSuccess()) {
      throw new OutOfStockError(result.failedItem);
    }
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
// SMELL: état d'inventaire public et mutable
public sealed class LegacyInventory
{
    public Dictionary<string, StockLevel> StockLevels { get; } = new(StringComparer.Ordinal);
}

public sealed class LegacyOrder
{
    public IReadOnlyCollection<OrderItem> Items { get; init; } = Array.Empty<OrderItem>();

    public void Process()
    {
        var inventory = new LegacyInventory();
        foreach (var item in Items)
        {
            var stock = inventory.StockLevels[item.Sku];
            if (stock.Quantity < item.Quantity)
                throw new InvalidOperationException("Out of stock");

            inventory.StockLevels[item.Sku].Quantity -= item.Quantity;
        }
    }
}

// REFACTORED: Tell, don't ask + encapsulation
public sealed class Inventory
{
    private readonly Dictionary<string, int> _stockLevels;

    public Inventory(IReadOnlyDictionary<string, int> initialStock)
    {
        ArgumentNullException.ThrowIfNull(initialStock);
        _stockLevels = new Dictionary<string, int>(initialStock, StringComparer.Ordinal); // Defensive copy
    }

    public ReserveResult Reserve(IEnumerable<OrderItem> items)
    {
        ArgumentNullException.ThrowIfNull(items);

        var requestedBySku = items
            .GroupBy(static item => item.Sku, StringComparer.Ordinal)
            .Select(static group => new RequestedStock(
                group.Key,
                group.Sum(static item => item.Quantity),
                group.First()))
            .ToList();

        foreach (var request in requestedBySku)
        {
            if (!_stockLevels.TryGetValue(request.Sku, out var available) || available < request.Quantity)
                return ReserveResult.OutOfStock(request.SampleItem);
        }

        foreach (var request in requestedBySku)
            _stockLevels[request.Sku] -= request.Quantity;

        return ReserveResult.Success();
    }

    private sealed record RequestedStock(string Sku, int Quantity, OrderItem SampleItem);
}

public sealed class Order
{
    public IReadOnlyCollection<OrderItem> Items { get; init; } = Array.Empty<OrderItem>();

    public void Process(Inventory inventory)
    {
        ArgumentNullException.ThrowIfNull(inventory);

        var result = inventory.Reserve(Items);
        if (!result.IsSuccess)
            throw new OutOfStockError(result.FailedItem!);
    }
}

public static class InventoryUsageExample
{
    public static void Run()
    {
        var inventory = new Inventory(new Dictionary<string, int>(StringComparer.Ordinal)
        {
            ["SKU-RED-42"] = 5
        });

        var firstReservation = inventory.Reserve(new[] { new OrderItem("SKU-RED-42", 3) });
        var secondReservation = inventory.Reserve(new[] { new OrderItem("SKU-RED-42", 3) });

        var firstSucceeded = firstReservation.IsSuccess;   // true
        var secondSucceeded = secondReservation.IsSuccess; // false
    }
}

// Placeholders de domaine (non développés ici) :
// OrderItem, StockLevel, ReserveResult, OutOfStockError.
```

### 7. Speculative Generality

**Symptom:** "Just in case" abstractions that aren't used.

```typescript
// SMELL: Over-engineered for hypothetical needs
interface PaymentProcessor {
  process(): void;
  rollback(): void;
  audit(): void;
  generateReport(): void;
  scheduleRecurring(): void;
}

class StripeProcessor implements PaymentProcessor {
  process() { /* actual code */ }
  rollback() { throw new Error('Not implemented'); }
  audit() { throw new Error('Not implemented'); }
  generateReport() { throw new Error('Not implemented'); }
  scheduleRecurring() { throw new Error('Not implemented'); }
}

// REFACTORED: YAGNI
interface PaymentProcessor {
  process(): void;
}

class StripeProcessor implements PaymentProcessor {
  process() { /* actual code */ }
}
// Add other methods when actually needed
```

**C#/.NET (illustrative counterpart):**
```csharp
public interface IPaymentProcessor
{
    Task ProcessAsync(CancellationToken cancellationToken);
}

public sealed class StripeProcessor : IPaymentProcessor
{
    public Task ProcessAsync(CancellationToken cancellationToken) => Task.CompletedTask;
}
```

---

## Prevention Strategies

1. **Follow Object Calisthenics** - Rules prevent most smells
2. **Practice TDD** - Tests reveal design problems early
3. **Review in pairs** - Fresh eyes catch smells
4. **Refactor continuously** - Don't let smells accumulate
5. **Apply SOLID** - Prevents structural smells
6. **Use static analysis** - Tools catch common issues

---

## When You Find a Smell

1. **Confirm it's a problem** - Not all smells need fixing
2. **Ensure test coverage** - Before refactoring
3. **Refactor in small steps** - Keep tests passing
4. **Commit frequently** - Easy to revert if needed
