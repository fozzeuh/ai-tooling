# Object-Oriented Design

## Responsibility-Driven Design (RDD)

The key insight: **Objects are defined by their responsibilities, not their data.**

> Compatibility note (.NET): records, init-only properties, and immutable collections are idiomatic options for value-like types and DTO boundaries; keep behavioral decisions where invariants live.

### Finding Objects

Start with:
1. **Nouns** in requirements → candidate objects
2. **Verbs** → candidate methods/behaviors
3. **Domain concepts** → value objects

### Finding Responsibilities

Each object should answer:
- What does this object **know**?
- What does this object **do**?
- What does this object **decide**?

### Object Stereotypes

Every class fits one (or maybe two) stereotypes:

| Stereotype | Purpose | Example |
|------------|---------|---------|
| **Information Holder** | Knows things, holds data | `User`, `Product`, `Address` |
| **Structurer** | Maintains relationships | `OrderItems`, `UserGroup` |
| **Service Provider** | Performs work | `PaymentProcessor`, `EmailSender` |
| **Coordinator** | Orchestrates workflow | `OrderFulfillmentService` |
| **Controller** | Makes decisions, delegates | `CheckoutController` |
| **Interfacer** | Transforms between systems | `UserAPIAdapter`, `DatabaseMapper` |

### The Two Questions

For every class, ask:
1. **"What pattern is this?"** - Which stereotype? Which design pattern?
2. **"Is it doing too much?"** - Check object calisthenics rules

If you can't answer clearly, the class needs refactoring.

---

## Tell, Don't Ask

**Command objects to do work. Don't interrogate them and do the work yourself.**

```typescript
// BAD: Asking, then doing
if (account.getBalance() >= amount) {
  account.setBalance(account.getBalance() - amount);
  // more logic here...
}

// GOOD: Telling
const result = account.withdraw(amount);
if (result.isSuccess()) {
  // ...
}
```

**C#/.NET (illustrative counterpart):**
```csharp
var result = account.Withdraw(amount);
if (result.IsSuccess)
{
    // ...
}
```

The object that has the data should have the behavior.

---

## Design by Contract (DbC)

Every method has:
- **Preconditions** - What must be true BEFORE calling
- **Postconditions** - What will be true AFTER calling
- **Invariants** - What is ALWAYS true about the object

```typescript
class BankAccount {
  private balance: Money;

  // INVARIANT: balance is never negative

  // PRECONDITION: amount > 0
  // POSTCONDITION: balance decreased by amount OR error returned
  withdraw(amount: Money): WithdrawResult {
    if (amount.isNegativeOrZero()) {
      return WithdrawResult.invalidAmount();
    }

    if (this.balance.isLessThan(amount)) {
      return WithdrawResult.insufficientFunds();
    }

    this.balance = this.balance.minus(amount);
    return WithdrawResult.success(this.balance);
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class BankAccount
{
    private Money _balance;

    // INVARIANT: balance is never negative
    public WithdrawResult Withdraw(Money amount)
    {
        if (amount.IsNegativeOrZero())
            return WithdrawResult.InvalidAmount();

        if (_balance.IsLessThan(amount))
            return WithdrawResult.InsufficientFunds();

        _balance = _balance.Minus(amount);
        return WithdrawResult.Success(_balance);
    }
}
```

---

## Composition Over Inheritance

**Prefer composing objects over extending classes.**

### Why Inheritance is Problematic:
- Tight coupling between parent and child
- Fragile base class problem
- Difficult to change parent without breaking children
- Forces "is-a" relationship that may not fit

### When to Use Inheritance:
- True "is-a" relationship (rare)
- Framework requirements
- Template Method pattern (intentional)

### Prefer Composition:
```typescript
// BAD: Inheritance
class PremiumUser extends User {
  getDiscount(): number { return 20; }
}

// GOOD: Composition
class User {
  constructor(private discountPolicy: DiscountPolicy) {}

  getDiscount(): number {
    return this.discountPolicy.calculate();
  }
}

// Now discount behavior is pluggable
new User(new PremiumDiscount());
new User(new StandardDiscount());
new User(new NoDiscount());
```

**C#/.NET (illustrative counterpart):**
```csharp
public interface IDiscountPolicy
{
    decimal Calculate();
}

public sealed class User(IDiscountPolicy discountPolicy)
{
    public decimal GetDiscount() => discountPolicy.Calculate();
}
```

---

## The Law of Demeter (Principle of Least Knowledge)

**Only talk to your immediate friends.**

A method should only call:
1. Methods on `this`
2. Methods on parameters
3. Methods on objects it creates
4. Methods on its direct components

```typescript
// BAD: Reaching through objects
order.getCustomer().getAddress().getCity();

// GOOD: Ask the immediate friend
order.getShippingCity();
```

**C#/.NET (illustrative counterpart):**
```csharp
// BAD
var city = order.Customer.Address.City;

// GOOD
var city = order.GetShippingCity();
```

This reduces coupling - changes to `Address` don't ripple through all callers.

---

## Encapsulation

**Hide internal details, expose behavior.**

### Levels of Encapsulation:
1. **Data** - private fields, no direct access
2. **Implementation** - how things work internally
3. **Type** - concrete class hidden behind interface
4. **Design** - architectural decisions hidden from clients

```typescript
// BAD: Exposed internals
class Order {
  public items: Item[] = [];
  public total: number = 0;
}

// Client can corrupt state
order.items.push(item);
order.total = -999; // Oops!

// GOOD: Encapsulated
class Order {
  private items: OrderItems;
  private total: Money;

  addItem(item: Item): void {
    this.items.add(item);
    this.recalculateTotal();
  }

  getTotal(): Money {
    return this.total; // Returns copy or immutable
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class Order
{
    private readonly List<Item> _items;
    private readonly ReadOnlyCollection<Item> _itemsView;
    private Money _total;

    public Order(IEnumerable<Item> items, Money? providedTotal = null)
    {
        ArgumentNullException.ThrowIfNull(items);

        _items = items.ToList(); // Defensive copy: this aggregate owns its collection state.
        _itemsView = _items.AsReadOnly();

        var computedTotal = CalculateTotal(_items);
        if (providedTotal is not null && providedTotal != computedTotal)
            throw new InvalidOperationException("Initial total is inconsistent with items.");

        _total = providedTotal ?? computedTotal;
        EnsureInvariant(_total);
    }

    public IReadOnlyList<Item> Items => _itemsView;
    public Money Total => _total;

    public void AddItem(Item item)
    {
        ArgumentNullException.ThrowIfNull(item);

        var candidateTotal = _total.Add(item.LineTotal);
        EnsureInvariant(candidateTotal); // Validate next state before mutating aggregate state.

        _items.Add(item);
        _total = candidateTotal;
    }

    private static Money CalculateTotal(IEnumerable<Item> items)
        => items.Aggregate(Money.Zero, static (acc, item) => acc.Add(item.LineTotal));

    private static void EnsureInvariant(Money total)
    {
        if (total.IsNegative)
            throw new InvalidOperationException("Order total cannot be negative.");
    }
}
```

---

## Polymorphism

**Replace conditionals with types.**

```typescript
// BAD: Type checking
function calculateShipping(method: string, value: number): number {
  if (method === 'standard') return value < 50 ? 5 : 0;
  if (method === 'express') return 15;
  if (method === 'overnight') return 25;
  throw new Error('Unknown method');
}

// GOOD: Polymorphism
interface ShippingMethod {
  calculateCost(orderValue: number): number;
}

class StandardShipping implements ShippingMethod {
  calculateCost(orderValue: number): number {
    return orderValue < 50 ? 5 : 0;
  }
}

class ExpressShipping implements ShippingMethod {
  calculateCost(orderValue: number): number {
    return 15;
  }
}

// Usage - no conditionals
function calculateShipping(method: ShippingMethod, value: number): number {
  return method.calculateCost(value);
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public interface IShippingMethod
{
    decimal CalculateCost(decimal orderValue);
}

public sealed class StandardShipping : IShippingMethod
{
    public decimal CalculateCost(decimal orderValue) => orderValue < 50m ? 5m : 0m;
}

public static decimal CalculateShipping(IShippingMethod method, decimal value)
    => method.CalculateCost(value);
```

---

## Value Objects vs Entities

### Value Objects
- Defined by their attributes (no identity)
- Immutable
- Comparable by value
- Examples: `Money`, `Email`, `Address`, `DateRange`

```typescript
class Money {
  constructor(
    private readonly amount: number,
    private readonly currency: string
  ) {}

  equals(other: Money): boolean {
    return this.amount === other.amount &&
           this.currency === other.currency;
  }

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new CurrencyMismatch();
    }
    return new Money(this.amount + other.amount, this.currency);
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public readonly record struct Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (!string.Equals(Currency, other.Currency, StringComparison.Ordinal))
            throw new InvalidOperationException("Currency mismatch.");

        return new Money(Amount + other.Amount, Currency);
    }
}
```

### Entities
- Have identity (survives attribute changes)
- Usually mutable (via methods)
- Comparable by identity
- Examples: `User`, `Order`, `Product`

```typescript
class User {
  constructor(
    private readonly id: UserId,
    private email: Email,
    private name: Name
  ) {}

  equals(other: User): boolean {
    return this.id.equals(other.id); // Identity comparison
  }

  changeEmail(newEmail: Email): void {
    this.email = newEmail; // Still same user
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class User : IEquatable<User>
{
    private readonly UserId _id;
    private Email _email;
    private Name _name;

    public User(UserId id, Email email, Name name)
    {
        _id = id;
        _email = email;
        _name = name;
    }

    public UserId Id => _id;
    public Email Email => _email;
    public Name Name => _name;

    public bool Equals(User? other)
    {
        if (ReferenceEquals(null, other))
            return false;
        if (ReferenceEquals(this, other))
            return true;

        return _id.Equals(other._id);
    }

    public override bool Equals(object? obj) => obj is User other && Equals(other);

    public override int GetHashCode() => _id.GetHashCode();

    public static bool operator ==(User? left, User? right) => Equals(left, right);
    public static bool operator !=(User? left, User? right) => !Equals(left, right);

    public void ChangeEmail(Email newEmail) => _email = newEmail;
}
```

---

## Aggregates

A cluster of objects treated as a single unit for data changes.

- One object is the **aggregate root** (entry point)
- External code only references the root
- Root enforces invariants for the entire cluster

```typescript
// Order is the aggregate root
class Order {
  private items: OrderItem[] = [];

  // All access through the root
  addItem(product: Product, quantity: number): void {
    const item = new OrderItem(product, quantity);
    this.items.push(item);
    this.validateTotal();
  }

  removeItem(itemId: ItemId): void {
    this.items = this.items.filter(i => !i.id.equals(itemId));
  }

  // Root enforces invariants
  private validateTotal(): void {
    if (this.calculateTotal().exceeds(MAX_ORDER_VALUE)) {
      throw new OrderTotalExceeded();
    }
  }
}

// BAD: Accessing items directly
order.items.push(new OrderItem(...)); // Bypasses validation!

// GOOD: Through the root
order.addItem(product, 2); // Validation happens
```
**C#/.NET (illustrative counterpart):**

> Compatibility note (.NET): the TypeScript source snippet above is kept verbatim for source fidelity and mutates before validation; treat it as illustrative. The C# counterpart validates a candidate total before mutating aggregate state.

```csharp
public sealed class Order
{
    private readonly List<OrderItem> _items = [];

    public void AddItem(Product product, int quantity)
    {
        var item = new OrderItem(product, quantity);
        var candidateTotal = CalculateTotal().Add(item.Total);
        EnsureTotalWithinLimit(candidateTotal);
        _items.Add(item);
    }

    private static void EnsureTotalWithinLimit(Money total)
    {
        if (total.Exceeds(MaxOrderValue))
            throw new OrderTotalExceeded();
    }
}
```
