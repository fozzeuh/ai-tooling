# Testing Strategy

## The Testing Pyramid

```
       /\
      /  \        E2E Tests (Few)
     /----\       - Full system
    /      \      - Slow, brittle
   /--------\
  /          \    Integration Tests (Some)
 /------------\   - Multiple components
/              \  - Medium speed
----------------
      Unit Tests (Many)
      - Single unit
      - Fast, isolated
```

> Compatibility note (.NET): keep the same pyramid intent across xUnit/NUnit/MSTest. Use the framework already present in the repository.

## Test Types

### Unit Tests

Test ONE class or function in isolation.

**Characteristics:**
- Fast (milliseconds)
- No external dependencies (mocked)
- Most of your tests should be unit tests

```typescript
describe('Order', () => {
  it('calculates total correctly', () => {
    const order = new Order();
    order.addItem({ price: 100 });
    order.addItem({ price: 50 });

    expect(order.calculateTotal()).toBe(150);
  });
});
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class OrderTests
{
    [Fact]
    public void CalculatesTotalCorrectly()
    {
        var order = new Order();
        order.AddItem(new OrderItem(price: 100m));
        order.AddItem(new OrderItem(price: 50m));

        Assert.Equal(150m, order.CalculateTotal());
    }
}
```

### Integration Tests

Test multiple components together.

**Characteristics:**
- Slower (may use real DB)
- Test boundaries between components
- Fewer than unit tests

```typescript
describe('OrderService Integration', () => {
  let db: Database;
  let service: OrderService;

  beforeAll(async () => {
    db = await Database.connect();
    service = new OrderService(new PostgresOrderRepo(db));
  });

  it('saves and retrieves an order', async () => {
    const order = Order.create({ customerId: '123' });
    await service.save(order);

    const retrieved = await service.findById(order.id);
    expect(retrieved).toEqual(order);
  });
});
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class OrderServiceIntegrationTests
{
    [Fact]
    public async Task SavesAndRetrievesOrder()
    {
        await using var db = await TestDatabase.ConnectAsync();
        var service = new OrderService(new PostgresOrderRepo(db));

        var order = Order.Create(customerId: "123");
        await service.SaveAsync(order, CancellationToken.None);

        var retrieved = await service.FindByIdAsync(order.Id, CancellationToken.None);
        Assert.Equal(order, retrieved);
    }
}
```

### E2E / Acceptance Tests

Test the entire system from user perspective.

**Characteristics:**
- Slowest
- Most brittle (many moving parts)
- Test critical paths only

```typescript
describe('Checkout Flow', () => {
  it('user can complete purchase', async () => {
    await page.goto('/products');
    await page.click('[data-testid="add-to-cart"]');
    await page.click('[data-testid="checkout"]');
    await page.fill('[name="card"]', '4242424242424242');
    await page.click('[data-testid="pay"]');

    expect(await page.textContent('h1')).toBe('Order Confirmed');
  });
});
```

**C#/.NET (illustrative counterpart):**
```csharp
[Fact]
public async Task UserCanCompletePurchase()
{
    // Example only: Playwright for .NET / Selenium equivalent.
    await Page.GotoAsync("/products");
    await Page.ClickAsync("[data-testid=\"add-to-cart\"]");
    await Page.ClickAsync("[data-testid=\"checkout\"]");
    await Page.FillAsync("[name=\"card\"]", "4242424242424242");
    await Page.ClickAsync("[data-testid=\"pay\"]");

    Assert.Equal("Order Confirmed", await Page.TextContentAsync("h1"));
}
```

---

## Arrange-Act-Assert (AAA)

Structure EVERY test this way:

```typescript
it('applies discount to premium users', () => {
  // ARRANGE - Set up the test world
  const user = new User({ isPremium: true });
  const cart = new Cart(user);
  cart.addItem({ price: 100 });

  // ACT - Execute the behavior under test
  const total = cart.calculateTotal();

  // ASSERT - Verify the expected outcome
  expect(total).toBe(80); // 20% discount
});
```

**C#/.NET (illustrative counterpart):**
```csharp
[Fact]
public void AppliesDiscountToPremiumUsers()
{
    // ARRANGE
    var user = new User(isPremium: true);
    var cart = new Cart(user);
    cart.AddItem(new Item(price: 100m));

    // ACT
    var total = cart.CalculateTotal();

    // ASSERT
    Assert.Equal(80m, total);
}
```

### Writing AAA Backwards

Sometimes easier to write in reverse:

1. **Assert first** - What do you want to verify?
2. **Act** - What action produces that result?
3. **Arrange** - What setup is needed?

---

## Test Naming

### Bad: Abstract, Technical

```typescript
it('should work correctly')
it('handles the edge case')
it('sets the data property')
```

**C#/.NET (illustrative counterpart):**
```csharp
[Fact]
public void WorksCorrectly() { }

[Fact]
public void HandlesEdgeCase() { }

[Fact]
public void SetsDataProperty() { }
```

### Good: Concrete Examples, Domain Language

```typescript
it('calculates 20% discount for premium users')
it('returns error when cart is empty')
it('recognizes "racecar" as a palindrome')
```

**C#/.NET (illustrative counterpart):**
```csharp
[Fact]
public void CalculatesTwentyPercentDiscountForPremiumUsers() { }

[Fact]
public void ReturnsErrorWhenCartIsEmpty() { }

[Fact]
public void RecognizesRacecarAsPalindrome() { }
```

### Format

```typescript
// Option 1: should + behavior
it('should apply tax based on shipping state')

// Option 2: when + then
it('when adding 2 + 3, then returns 5')

// Option 3: Given-When-Then (for complex scenarios)
describe('given a premium user', () => {
  describe('when they checkout', () => {
    it('then they receive 20% discount', () => { ... });
  });
});
```

**C#/.NET (illustrative counterpart):**
```csharp
[Fact]
public void ShouldApplyTaxBasedOnShippingState() { }

[Fact]
public void WhenAdding2And3_ThenReturns5() { }
```

---

## Test Doubles

### Dummy

Object passed but never used.

```typescript
const dummyLogger = {} as Logger;
new UserService(realRepo, dummyLogger);
```

**C#/.NET (illustrative counterpart):**
```csharp
var dummyLogger = NullLogger<UserService>.Instance;
var service = new UserService(realRepo, dummyLogger);
```

### Stub

Returns predefined values.

```typescript
const stubRepo: UserRepo = {
  findById: () => Promise.resolve(new User({ name: 'Test' })),
  save: () => Promise.resolve(),
};
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class StubUserRepo : IUserRepo
{
    private readonly User _user = new(name: "Test");

    public Task<User?> FindByIdAsync(string id, CancellationToken cancellationToken)
        => Task.FromResult<User?>(_user);

    public Task SaveAsync(User user, CancellationToken cancellationToken)
        => Task.CompletedTask;
}
```

### Spy

Records how it was called.

```typescript
const emailSpy = {
  sentEmails: [] as string[],
  send(to: string, message: string) {
    this.sentEmails.push(to);
  }
};

// Later
expect(emailSpy.sentEmails).toContain('user@example.com');
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class EmailSpy : IEmailService
{
    public List<string> SentEmails { get; } = [];

    public Task SendAsync(string to, string message, CancellationToken cancellationToken)
    {
        SentEmails.Add(to);
        return Task.CompletedTask;
    }
}

// Later
Assert.Contains("user@example.com", emailSpy.SentEmails);
```

### Mock

Verifies expected interactions.

```typescript
const mockRepo = jest.fn<UserRepo>();
mockRepo.save.mockResolvedValue(undefined);

// After test
expect(mockRepo.save).toHaveBeenCalledWith(expectedUser);
```

**C#/.NET (illustrative counterpart — xUnit + Moq):**
```csharp
var mockRepo = new Mock<IUserRepo>();
mockRepo
    .Setup(repo => repo.SaveAsync(It.IsAny<User>(), It.IsAny<CancellationToken>()))
    .Returns(Task.CompletedTask);

// After test
mockRepo.Verify(
    repo => repo.SaveAsync(expectedUser, It.IsAny<CancellationToken>()),
    Times.Once);
```

### Fake

Working implementation (simplified).

```typescript
class InMemoryUserRepo implements UserRepo {
  private users: Map<string, User> = new Map();

  async save(user: User): Promise<void> {
    this.users.set(user.id, user);
  }

  async findById(id: string): Promise<User | null> {
    return this.users.get(id) || null;
  }
}
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class InMemoryUserRepo : IUserRepo
{
    private readonly Dictionary<string, User> _users = new();

    public Task SaveAsync(User user, CancellationToken cancellationToken)
    {
        _users[user.Id] = user;
        return Task.CompletedTask;
    }

    public Task<User?> FindByIdAsync(string id, CancellationToken cancellationToken)
        => Task.FromResult(_users.TryGetValue(id, out var user) ? user : null);
}
```

---

## Testing Strategies by Layer

### Domain Layer (Most Tests)

- Unit tests with no mocks
- Test business rules, value objects, entities
- Fast, comprehensive

```typescript
describe('Money', () => {
  it('adds amounts with same currency', () => {
    const a = Money.dollars(10);
    const b = Money.dollars(20);
    expect(a.add(b).equals(Money.dollars(30))).toBe(true);
  });

  it('throws when adding different currencies', () => {
    const usd = Money.dollars(10);
    const eur = Money.euros(10);
    expect(() => usd.add(eur)).toThrow(CurrencyMismatch);
  });
});
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class MoneyTests
{
    [Fact]
    public void AddsAmountsWithSameCurrency()
    {
        var a = Money.Dollars(10);
        var b = Money.Dollars(20);
        Assert.True(a.Add(b).Equals(Money.Dollars(30)));
    }

    [Fact]
    public void ThrowsWhenAddingDifferentCurrencies()
    {
        var usd = Money.Dollars(10);
        var eur = Money.Euros(10);
        Assert.Throws<CurrencyMismatch>(() => usd.Add(eur));
    }
}
```

### Application Layer

- Integration tests with mocked infrastructure
- Test use case orchestration

```typescript
describe('CreateOrderUseCase', () => {
  it('creates order and sends confirmation', async () => {
    const orderRepo = new InMemoryOrderRepo();
    const emailService = { send: jest.fn() };
    const useCase = new CreateOrderUseCase(orderRepo, emailService);

    await useCase.execute({ customerId: '123', items: [...] });

    expect(orderRepo.count()).toBe(1);
    expect(emailService.send).toHaveBeenCalled();
  });
});
```

**C#/.NET (illustrative counterpart — xUnit):**
```csharp
[Fact]
public async Task CreatesOrderAndSendsConfirmation()
{
    var orderRepo = new InMemoryOrderRepo();
    var emailService = new EmailSpy();
    var useCase = new CreateOrderUseCase(orderRepo, emailService);

    await useCase.ExecuteAsync(new CreateOrderCommand("123", [/* ... */]), CancellationToken.None);

    Assert.Equal(1, orderRepo.Count);
    Assert.NotEmpty(emailService.SentEmails);
}
```

### Infrastructure Layer

- Integration tests with real dependencies
- Test database, API integrations

```typescript
describe('PostgresOrderRepo', () => {
  let repo: PostgresOrderRepo;

  beforeAll(async () => {
    repo = new PostgresOrderRepo(testDb);
  });

  it('persists and retrieves order', async () => {
    const order = Order.create({ ... });
    await repo.save(order);

    const found = await repo.findById(order.id);
    expect(found).toEqual(order);
  });
});
```

**C#/.NET (illustrative counterpart — xUnit):**
```csharp
public sealed class PostgresOrderRepoTests
{
    [Fact]
    public async Task PersistsAndRetrievesOrder()
    {
        await using var db = await TestDatabase.ConnectAsync();
        var repo = new PostgresOrderRepo(db);

        var order = Order.Create(/* ... */);
        await repo.SaveAsync(order, CancellationToken.None);

        var found = await repo.FindByIdAsync(order.Id, CancellationToken.None);
        Assert.Equal(order, found);
    }
}
```

---

## High-Value Integration Tests

Focus integration tests on:

1. **Boundaries** - Where systems meet
2. **Critical paths** - Money, security, core features
3. **Complex queries** - Database operations

### Contract Tests

Verify implementations match interfaces.

```typescript
// Shared contract test
function testUserRepoContract(createRepo: () => UserRepo) {
  describe('UserRepo Contract', () => {
    let repo: UserRepo;

    beforeEach(() => {
      repo = createRepo();
    });

    it('saves and retrieves user', async () => {
      const user = User.create({ name: 'Test' });
      await repo.save(user);
      const found = await repo.findById(user.id);
      expect(found).toEqual(user);
    });

    it('returns null for missing user', async () => {
      const found = await repo.findById('nonexistent');
      expect(found).toBeNull();
    });
  });
}

// Apply to all implementations
testUserRepoContract(() => new InMemoryUserRepo());
testUserRepoContract(() => new PostgresUserRepo(testDb));
```

**C#/.NET (illustrative counterpart):**
```csharp
// Illustrative xUnit v2 contract; UserId, Email and Name are domain placeholders.
// Email and Name expose their persisted values through a Value property.
public abstract class UserRepoContractTestsV2
{
    protected abstract IUserRepo CreateRepo();

    [Fact]
    public async Task SavesAndRetrievesUser()
    {
        var repo = CreateRepo();
        var id = UserId.New(); // Placeholder ID factory.
        var user = new User(id, new Email("test@example.com"), new Name("Test"));

        await repo.SaveAsync(user, CancellationToken.None);
        var found = Assert.IsType<User>(await repo.FindByIdAsync(id, CancellationToken.None));

        Assert.Equal(user, found); // Entity equality checks identity only.
        Assert.Equal(id, found.Id);
        Assert.Equal("test@example.com", found.Email.Value);
        Assert.Equal("Test", found.Name.Value);
    }

    [Fact]
    public async Task ReturnsNullForMissingUser()
    {
        var repo = CreateRepo();

        var missingUserId = UserId.New(); // Placeholder ID factory.
        var found = await repo.FindByIdAsync(missingUserId, CancellationToken.None);

        Assert.Null(found);
    }
}

public sealed class InMemoryUserRepoContractTests : UserRepoContractTestsV2
{
    protected override IUserRepo CreateRepo() => new InMemoryUserRepo();
}

[Collection(nameof(PostgresUserRepoCollection))]
public sealed class PostgresUserRepoContractTests : UserRepoContractTestsV2
{
    private readonly PostgresUserRepoFixture _fixture;

    public PostgresUserRepoContractTests(PostgresUserRepoFixture fixture)
    {
        _fixture = fixture;
    }

    protected override IUserRepo CreateRepo() => _fixture.CreateRepository();
}

[CollectionDefinition(nameof(PostgresUserRepoCollection))]
public sealed class PostgresUserRepoCollection : ICollectionFixture<PostgresUserRepoFixture>
{
}

public sealed class PostgresUserRepoFixture : IAsyncLifetime
{
    public Task InitializeAsync() => Task.CompletedTask; // e.g., provision/reset isolated test DB.
    public Task DisposeAsync() => Task.CompletedTask;

    public IUserRepo CreateRepository() => new PostgresUserRepo(/* test connection */);
}
```

---

## Test Builders

Create test objects easily.

```typescript
class OrderBuilder {
  private props: Partial<OrderProps> = {
    id: 'order-1',
    customerId: 'cust-1',
    items: [],
    status: 'pending',
  };

  withId(id: string): OrderBuilder {
    this.props.id = id;
    return this;
  }

  withItems(items: Item[]): OrderBuilder {
    this.props.items = items;
    return this;
  }

  paid(): OrderBuilder {
    this.props.status = 'paid';
    return this;
  }

  build(): Order {
    return Order.create(this.props as OrderProps);
  }
}

// Usage
const order = new OrderBuilder()
  .withItems([{ sku: 'ABC', price: 100 }])
  .paid()
  .build();
```

**C#/.NET (illustrative counterpart):**
```csharp
public sealed class OrderBuilder
{
    private string _id = "order-1";
    private string _customerId = "cust-1";
    private List<Item> _items = [];
    private OrderStatus _status = OrderStatus.Pending;

    public OrderBuilder WithItems(IEnumerable<Item> items)
    {
        _items = items.ToList();
        return this;
    }

    public OrderBuilder Paid()
    {
        _status = OrderStatus.Paid;
        return this;
    }

    public Order Build() => Order.Create(_id, _customerId, _items, _status);
}
```

---

## Common Testing Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Testing implementation | Brittle tests | Test behavior only |
| Too many mocks | Tests prove nothing | Use real objects when possible |
| Shared state | Flaky tests | Isolate each test |
| No assertions | False confidence | Always assert something meaningful |
| Testing trivial code | Wasted effort | Focus on logic and edge cases |
| Slow tests | Reduced feedback | Optimize, use unit tests |
