# Repository Pattern in Test Automation

The **Repository pattern** in test automation acts as an abstraction or data access layer. It creates a centralized collection to manage, query, and serve test data (such as user credentials, environment configurations, API payloads, or database records) to your test scripts. 

Instead of hardcoding paths to JSON files, databases, or environment variables directly inside your Page Objects or Test Cases, the test interacts exclusively with a **Repository** interface.

---

### How the Repository Pattern Works
In standard software development, repositories abstract database operations. In test automation, repositories abstract **test data sourcing**. 

* **The Test Case:** Requests specific data (e.g., `userRepository.getAdminUser()`).
* **The Repository:** Contains the logic to find, fetch, and structure that data.
* **The Data Sources:** The actual storage mechanism, which could be local JSON files, Excel sheets, environment variables, or an external database.

---

### Benefits for Test Automation
* **Centralized Data Management:** If your test data structure changes (e.g., adding a new required field to user objects), you modify it in one repository instead of updating dozens of individual tests or data sheets.
* **Storage Agnostic:** You can switch your test data source from local JSON files to an external API or database without changing a single line of code in your test scripts.
* **Strong Typing and Autocomplete:** In TypeScript or Java, mapping your data sources to a repository allows IDE autocomplete to guide your data selection, minimizing typo risks.
* **Dynamic Data Fetching:** Repositories can contain logic to dynamically generate data on the fly (e.g., generating unique emails using libraries like Faker) or pulling unused accounts from an available pool.

---

### Implementation Code

#### 1. Playwright / TypeScript Implementation

**Define the Data Model & Repository (`UserRepository.ts`):**
```typescript
import * as fs from 'fs';
import * as path from 'path';

export interface User {
    id: string;
    username: string;
    role: 'admin' | 'standard' | 'guest';
}

export class UserRepository {
    private users: User[];

    constructor() {
        // Load data from a JSON file (could easily be swapped for an API or DB call later)
        const filePath = path.join(__dirname, '../data/users.json');
        const rawData = fs.readFileSync(filePath, 'utf-8');
        this.users = JSON.parse(rawData);
    }

    /**
     * Get a user based on their system role
     */
    getUserByRole(role: 'admin' | 'standard' | 'guest'): User {
        const user = this.users.find(u => u.role === role);
        if (!user) throw new Error(`No user found with role: ${role}`);
        return user;
    }

    /**
     * Dynamic helper to fetch data
     */
    getExpiredUser(): User {
        return this.users.find(u => u.username.includes('expired'))!;
    }
}
```

**Execute within the Test File (`login.spec.ts`):**
```typescript
import { test, expect } from '@playwright/test';
import { UserRepository } from '../repositories/UserRepository';
import { LoginPage } from '../pages/LoginPage';

test.describe('Login Security Tests', () => {
    let userRepository: UserRepository;

    test.beforeAll(() => {
        userRepository = new UserRepository();
    });

    test('Admin user should see security dashboard', async ({ page }) => {
        const loginPage = new LoginPage(page);
        
        // Fetch cleanly via repository abstracting the JSON/Database layer
        const adminUser = userRepository.getUserByRole('admin');

        await loginPage.navigate();
        await loginPage.login(adminUser.username, 'SecurePassword123');
        
        await expect(page.locator('#security-panel')).toBeVisible();
    });
});
```

---

#### 2. Selenium / Java Implementation

**Define the Model & Repository (`UserRepository.java`):**
```java
package repositories;

import models.User;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.util.Arrays;
import java.util.List;

public class UserRepository {
    private List<User> users;

    public UserRepository() {
        try {
            ObjectMapper mapper = new ObjectMapper();
            User[] userArray = mapper.readValue(new File("src/test/resources/users.json"), User[].class);
            this.users = Arrays.asList(userArray);
        } catch (Exception e) {
            throw new RuntimeException("Failed to load test user repository", e);
        }
    }

    public User getUserByRole(String role) {
        return users.stream()
                .filter(user -> user.getRole().equalsIgnoreCase(role))
                .findFirst()
                .orElseThrow(() -> new RuntimeException("User role not found: " + role));
    }
}
```

**Execute within the Test Class (`LoginTest.java`):**
```java
package tests;

import org.junit.jupiter.api.Test;
import repositories.UserRepository;
import models.User;

public class LoginTest extends BaseTest {
    private UserRepository userRepository = new UserRepository();

    @Test
    public void testAdminLogin() {
        // Retrieve strong-typed data from the repository
        User admin = userRepository.getUserByRole("admin");
        
        loginPage.open();
        loginPage.loginAs(admin.getUsername(), admin.getPassword());
        
        // Assertion logic here
    }
}
```
