## Pattern 5 — Builder

### What it is

Builder constructs complex objects step by step. Instead of a constructor with 10 parameters or a factory method that cannot express every combination, you chain method calls to set only what you need.

### Why it applies to test automation

Test data is complex. A user might have an email, password, role, department, manager, address, phone, account status, and subscription plan. You never need all of them in every test. Builder lets each test set only what matters and leave everything else as a sensible default. This makes tests expressive and easy to read.

### When to use

Use it for any test data object with more than 4–5 fields. Use it when different tests need different combinations of the same object. Use it for API request payloads, database seed objects, and form fill data.

### When NOT to use

Do not use it for simple objects with 1–3 fields. Do not use it when all fields are always required — a constructor is cleaner for required fields.

---

### TypeScript — Playwright

```typescript
// builders/user.builder.ts
// Fluent builder — chain methods, call build() at the end

import { faker } from '@faker-js/faker';

interface Address {
  street: string;
  city: string;
  state: string;
  zip: string;
  country: string;
}

interface UserPayload {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  role: 'admin' | 'user' | 'viewer' | 'editor';
  phone?: string;
  department?: string;
  active: boolean;
  address?: Address;
  permissions: string[];
  subscriptionPlan?: 'free' | 'pro' | 'enterprise';
}

export class UserBuilder {
  // Private mutable object that gets configured by the chain
  private readonly user: UserPayload;

  constructor() {
    // Sensible defaults — tests only override what they care about
    this.user = {
      email:            `test+${Date.now()}@example.com`,
      password:         'Password123!',
      firstName:        faker.person.firstName(),
      lastName:         faker.person.lastName(),
      role:             'user',
      active:           true,
      permissions:      [],
      subscriptionPlan: 'free',
    };
  }

  // Each method returns `this` — enables method chaining
  withEmail(email: string): this {
    this.user.email = email;
    return this;
  }

  withPassword(password: string): this {
    this.user.password = password;
    return this;
  }

  withRole(role: UserPayload['role']): this {
    this.user.role = role;
    return this;
  }

  withDepartment(department: string): this {
    this.user.department = department;
    return this;
  }

  withPhone(phone: string): this {
    this.user.phone = phone;
    return this;
  }

  withAddress(address: Address): this {
    this.user.address = address;
    return this;
  }

  withPermissions(permissions: string[]): this {
    this.user.permissions = permissions;
    return this;
  }

  addPermission(permission: string): this {
    this.user.permissions.push(permission);
    return this;
  }

  withSubscription(plan: UserPayload['subscriptionPlan']): this {
    this.user.subscriptionPlan = plan;
    return this;
  }

  inactive(): this {
    this.user.active = false;
    return this;
  }

  // Pre-built combinations — convenience builders
  asAdmin(): this {
    return this.withRole('admin').withPermissions(['read', 'write', 'delete', 'admin']);
  }

  asViewer(): this {
    return this.withRole('viewer').withPermissions(['read']);
  }

  asEnterpriseAdmin(): this {
    return this.asAdmin().withSubscription('enterprise');
  }

  // Returns a plain object — detached from the builder
  // Call this once at the end of the chain
  build(): UserPayload {
    // Return a copy — builder can be reused after calling build()
    return { ...this.user, permissions: [...this.user.permissions] };
  }
}
```

```typescript
// builders/payment.builder.ts

interface PaymentPayload {
  amount: number;
  currency: string;
  cardNumber: string;
  cardHolderName: string;
  expiryMonth: string;
  expiryYear: string;
  cvv: string;
  description?: string;
  metadata?: Record<string, string>;
}

export class PaymentBuilder {
  private readonly payment: PaymentPayload;

  constructor() {
    this.payment = {
      amount:         1000,
      currency:       'USD',
      cardNumber:     '4111111111111111',
      cardHolderName: 'Test User',
      expiryMonth:    '12',
      expiryYear:     '2028',
      cvv:            '123',
    };
  }

  withAmount(amount: number): this        { this.payment.amount = amount; return this; }
  withCurrency(currency: string): this    { this.payment.currency = currency; return this; }
  withCard(cardNumber: string): this      { this.payment.cardNumber = cardNumber; return this; }
  withDescription(desc: string): this     { this.payment.description = desc; return this; }
  withMetadata(meta: Record<string, string>): this { this.payment.metadata = meta; return this; }

  // Pre-built card scenarios
  asDeclinedCard(): this   { return this.withCard('4000000000000002'); }
  asExpiredCard(): this    { return this.withCard('4000000000000069'); }
  asInsufficientFunds(): this { return this.withCard('4000000000009995'); }
  asMicropayment(): this   { return this.withAmount(1); }
  asLargePayment(): this   { return this.withAmount(999999); }

  build(): PaymentPayload { return { ...this.payment }; }
}
```

```typescript
// tests/payment.spec.ts — Builder in action

import { test, expect } from '../fixtures/services.fixture';
import { UserBuilder } from '../builders/user.builder';
import { PaymentBuilder } from '../builders/payment.builder';

test.describe('Payment scenarios', () => {

  test('admin can process large payment', async ({ userService, paymentService }) => {
    const userData    = new UserBuilder().asAdmin().withSubscription('enterprise').build();
    const paymentData = new PaymentBuilder().asLargePayment().withCurrency('GBP').build();

    const user    = await userService.createUser(userData);
    const payment = await paymentService.createPayment(user.id, paymentData.amount, paymentData.currency);

    expect(payment.status).toBe('completed');

    await paymentService.refundPayment(payment.id);
    await userService.deleteUser(user.id);
  });

  test('viewer cannot initiate payment', async ({ userService }) => {
    const userData = new UserBuilder().asViewer().build();
    const user     = await userService.createUser(userData);

    // Verify viewer has no payment permissions
    expect(userData.permissions).not.toContain('payment:create');

    await userService.deleteUser(user.id);
  });

  test('inactive user payment is rejected', async ({ userService }) => {
    const userData = new UserBuilder().inactive().withRole('user').build();
    const user     = await userService.createUser(userData);

    expect(user.active ?? userData.active).toBe(false);

    await userService.deleteUser(user.id);
  });

});
```

---

### Java — Selenium

```java
// src/main/java/builders/UserBuilder.java
package builders;

import models.UserPayload;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class UserBuilder {

  private String email       = "test+" + System.currentTimeMillis() + "@example.com";
  private String password    = "Password123!";
  private String firstName   = "Test";
  private String lastName    = "User";
  private String role        = "user";
  private boolean active     = true;
  private List<String> permissions = new ArrayList<>();

  public UserBuilder withEmail(String email)         { this.email = email; return this; }
  public UserBuilder withPassword(String password)   { this.password = password; return this; }
  public UserBuilder withRole(String role)            { this.role = role; return this; }
  public UserBuilder inactive()                       { this.active = false; return this; }
  public UserBuilder withPermissions(String... perms) {
    this.permissions = Arrays.asList(perms);
    return this;
  }

  public UserBuilder asAdmin() {
    return withRole("admin")
      .withPermissions("read", "write", "delete", "admin");
  }

  public UserBuilder asViewer() {
    return withRole("viewer").withPermissions("read");
  }

  public UserPayload build() {
    return new UserPayload(email, password, firstName, lastName, role, active, permissions);
  }
}
```
### With Static Innter Class

```Java
import com.fasterxml.jackson.annotation.JsonInclude;

@JsonInclude(JsonInclude.Include.NON_NULL) // Skips null fields in JSON
public class UserPayload {
    // 1. Private fields matching JSON keys
    private String name;
    private String job;
    private List<String> skills;

    // 2. Private constructor forces use of the Builder
    private UserPayload(Builder builder) {
        this.name = builder.name;
        this.job = builder.job;
        this.skills = builder.skills;
    }

    // 3. Getters (Required by Jackson for serialization)
    public String getName() { return name; }
    public String getJob() { return job; }
    public List<String> getSkills() { return skills; }

    // 4. Static Inner Builder Class
    public static class Builder {
        private String name;
        private String job;
        private List<String> skills;

        // Setter methods that return the Builder instance
        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder job(String job) {
            this.job = job;
            return this;
        }

        public Builder skills(List<String> skills) {
            this.skills = skills;
            return this;
        }

        // Final build method to instantiate the payload
        public UserPayload build() {
            return new UserPayload(this);
        }
    }
}

```

```java
import static io.restassured.RestAssured.given;
import io.restassured.http.ContentType;
import org.testng.annotations.Test;
import java.util.Arrays;

public class UserApiTest {

    @Test
    public void createUserTest() {
        // Step-by-step construction using the inner class builder
        UserPayload payload = new UserPayload.Builder()
                .name("Alice")
                .job("QA Automation Engineer")
                .skills(Arrays.asList("Java", "REST Assured"))
                .build();

        given()
            .baseUri("https://reqres.in")
            .contentType(ContentType.JSON)
            .body(payload) // Jackson converts the object to JSON
        .when()
            .post("/api/users")
        .then()
            .statusCode(201);
    }
}

```

---



