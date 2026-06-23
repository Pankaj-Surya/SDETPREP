## Pattern 2 — Service Object Model

### What it is

Service Object Model mirrors POM but for the API layer. Instead of a class per UI page, you create a class per backend service or API resource. Each service class owns the API calls for one domain — users, payments, orders, etc.

### Why it applies to test automation

Most SDET work involves both UI and API testing. If you call `request.post('/api/users', ...)` raw in every test, you have the same problem as having raw CSS selectors in tests — change the endpoint, update 40 files. A Service Object centralises all calls to one resource. Tests call `userService.createUser(...)` and never see the URL or headers.

### When to use

Use it whenever you have repeated API calls in your tests. Use it for test data setup and teardown via API. Use it when building hybrid tests that set up state via API and verify via UI.

### When NOT to use

Do not use it for one-time exploratory API tests that will not be maintained. Do not create a service object for an API you call once across the entire test suite.

---

### TypeScript — Playwright

```typescript
// services/interfaces.ts
// Define interfaces first — tests depend on these, not concrete classes

import { type APIRequestContext } from '@playwright/test';

export interface User {
  id: string;
  email: string;
  role: string;
  token: string;
  createdAt: string;
}

export interface CreateUserPayload {
  email: string;
  role?: string;
  firstName?: string;
  lastName?: string;
}

export interface IUserService {
  createUser(payload: CreateUserPayload): Promise<User>;
  getUserById(id: string): Promise<User>;
  updateUser(id: string, payload: Partial<CreateUserPayload>): Promise<User>;
  deleteUser(id: string): Promise<void>;
  getUsersByRole(role: string): Promise<User[]>;
}
```

```typescript
// services/user.service.ts
// Concrete implementation — all user API calls in one place

import { type APIRequestContext } from '@playwright/test';
import type { User, CreateUserPayload, IUserService } from './interfaces';

export class UserService implements IUserService {
  private readonly baseUrl = '/api/v1/users';

  // Association: receives request context — does not create it
  // The fixture creates it and passes it here
  constructor(private readonly request: APIRequestContext) {}

  async createUser(payload: CreateUserPayload): Promise<User> {
    const response = await this.request.post(this.baseUrl, {
      data: { role: 'user', ...payload },
    });

    if (!response.ok()) {
      const body = await response.text();
      throw new Error(`createUser failed [${response.status()}]: ${body}`);
    }

    return response.json() as Promise<User>;
  }

  async getUserById(id: string): Promise<User> {
    const response = await this.request.get(`${this.baseUrl}/${id}`);

    if (!response.ok()) {
      throw new Error(`getUserById failed [${response.status()}] for id: ${id}`);
    }

    return response.json() as Promise<User>;
  }

  async updateUser(id: string, payload: Partial<CreateUserPayload>): Promise<User> {
    const response = await this.request.patch(`${this.baseUrl}/${id}`, {
      data: payload,
    });

    if (!response.ok()) {
      throw new Error(`updateUser failed [${response.status()}]`);
    }

    return response.json() as Promise<User>;
  }

  async deleteUser(id: string): Promise<void> {
    const response = await this.request.delete(`${this.baseUrl}/${id}`);

    if (!response.ok() && response.status() !== 404) {
      throw new Error(`deleteUser failed [${response.status()}] for id: ${id}`);
    }
  }

  async getUsersByRole(role: string): Promise<User[]> {
    const response = await this.request.get(this.baseUrl, {
      params: { role },
    });

    if (!response.ok()) {
      throw new Error(`getUsersByRole failed [${response.status()}]`);
    }

    const body = await response.json() as { users: User[] };
    return body.users;
  }
}
```

```typescript
// services/payment.service.ts
// Separate service for payments — single responsibility

import { type APIRequestContext } from '@playwright/test';

export interface Payment {
  id: string;
  amount: number;
  currency: string;
  status: 'pending' | 'completed' | 'failed' | 'refunded';
  userId: string;
}

export interface IPaymentService {
  createPayment(userId: string, amount: number, currency: string): Promise<Payment>;
  getPayment(id: string): Promise<Payment>;
  refundPayment(id: string): Promise<Payment>;
  getPaymentsByUser(userId: string): Promise<Payment[]>;
}

export class PaymentService implements IPaymentService {
  constructor(private readonly request: APIRequestContext) {}

  async createPayment(userId: string, amount: number, currency: string): Promise<Payment> {
    const response = await this.request.post('/api/v1/payments', {
      data: { userId, amount, currency },
    });

    if (!response.ok()) {
      throw new Error(`createPayment failed [${response.status()}]`);
    }

    return response.json() as Promise<Payment>;
  }

  async getPayment(id: string): Promise<Payment> {
    const response = await this.request.get(`/api/v1/payments/${id}`);
    return response.json() as Promise<Payment>;
  }

  async refundPayment(id: string): Promise<Payment> {
    const response = await this.request.post(`/api/v1/payments/${id}/refund`);

    if (!response.ok()) {
      throw new Error(`refundPayment failed [${response.status()}]`);
    }

    return response.json() as Promise<Payment>;
  }

  async getPaymentsByUser(userId: string): Promise<Payment[]> {
    const response = await this.request.get('/api/v1/payments', {
      params: { userId },
    });

    const body = await response.json() as { payments: Payment[] };
    return body.payments;
  }
}
```

```typescript
// fixtures/services.fixture.ts
// Wire up all service objects via fixtures
// Tests get services injected — they never instantiate services directly

import { test as base } from '@playwright/test';
import { UserService } from '../services/user.service';
import { PaymentService } from '../services/payment.service';
import type { IUserService } from '../services/interfaces';
import type { IPaymentService } from '../services/payment.service';

type ServiceFixtures = {
  userService: IUserService;
  paymentService: IPaymentService;
};

export const test = base.extend<ServiceFixtures>({

  userService: async ({ request }, use) => {
    await use(new UserService(request));
  },

  paymentService: async ({ request }, use) => {
    await use(new PaymentService(request));
  },
});

export { expect } from '@playwright/test';
```

```typescript
// tests/payment-flow.spec.ts
// Hybrid test: API setup + UI verification

import { test, expect } from '../fixtures/services.fixture';
import { DashboardPage } from '../pages/dashboard.page';

test.describe('Payment flow — hybrid UI/API test', () => {

  test('completed payment appears in user dashboard', async ({ page, userService, paymentService }) => {
    // Setup via API — fast, reliable, no UI clicking for data prep
    const user    = await userService.createUser({ email: `pay+${Date.now()}@test.com` });
    const payment = await paymentService.createPayment(user.id, 5000, 'USD');

    // Verify via UI
    await page.goto(`/users/${user.id}/payments`);
    await expect(page.getByText(payment.id)).toBeVisible();
    await expect(page.getByText('$50.00')).toBeVisible();

    // Cleanup via API — no UI clicking needed
    await paymentService.refundPayment(payment.id);
    await userService.deleteUser(user.id);
  });

  test('user has no payments by default', async ({ userService, paymentService }) => {
    const user     = await userService.createUser({ email: `nopay+${Date.now()}@test.com` });
    const payments = await paymentService.getPaymentsByUser(user.id);

    expect(payments).toHaveLength(0);

    await userService.deleteUser(user.id);
  });

});
```

---

### Java — Selenium with REST Assured

```java
// src/main/java/services/IUserService.java
package services;

import models.User;
import models.CreateUserPayload;
import java.util.List;

public interface IUserService {
  User createUser(CreateUserPayload payload);
  User getUserById(String id);
  User updateUser(String id, CreateUserPayload payload);
  void deleteUser(String id);
  List<User> getUsersByRole(String role);
}
```

```java
// src/main/java/services/UserService.java
package services;

import io.restassured.RestAssured;
import io.restassured.response.Response;
import io.restassured.specification.RequestSpecification;
import models.User;
import models.CreateUserPayload;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.Arrays;
import java.util.List;

public class UserService implements IUserService {

  private final RequestSpecification spec;
  private final ObjectMapper mapper = new ObjectMapper();
  private static final String BASE_PATH = "/api/v1/users";

  public UserService(String baseUrl, String authToken) {
    this.spec = RestAssured.given()
      .baseUri(baseUrl)
      .header("Authorization", "Bearer " + authToken)
      .header("Content-Type", "application/json")
      .header("Accept", "application/json");
  }

  @Override
  public User createUser(CreateUserPayload payload) {
    Response response = spec
      .body(payload)
      .when()
        .post(BASE_PATH)
      .then()
        .statusCode(201)
        .extract().response();

    return response.as(User.class);
  }

  @Override
  public User getUserById(String id) {
    return spec
      .when()
        .get(BASE_PATH + "/" + id)
      .then()
        .statusCode(200)
        .extract().as(User.class);
  }

  @Override
  public void deleteUser(String id) {
    spec
      .when()
        .delete(BASE_PATH + "/" + id)
      .then()
        .statusCode(org.hamcrest.Matchers.oneOf(200, 204, 404));
  }

  @Override
  public List<User> getUsersByRole(String role) {
    User[] users = spec
      .queryParam("role", role)
      .when()
        .get(BASE_PATH)
      .then()
        .statusCode(200)
        .extract().as(User[].class);

    return Arrays.asList(users);
  }

  @Override
  public User updateUser(String id, CreateUserPayload payload) {
    return spec
      .body(payload)
      .when()
        .patch(BASE_PATH + "/" + id)
      .then()
        .statusCode(200)
        .extract().as(User.class);
  }
}
```

---

