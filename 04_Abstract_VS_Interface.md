

# Abstract Classes vs. Interfaces in Test Automation
Use an **abstract class** when your automation classes share tight, hierarchical relationships with common code or state (e.g., base setup, logging, or shared utilities). Use an **interface** when you want to enforce a strict contract across separate, unrelated classes, or to swap out underlying execution engines entirely without altering peripheral test logic.
---## Comparison Overview
| Feature | Abstract Class | Interface |
| :--- | :--- | :--- |
| **Core Intent** | **Code reuse** with custom extensions. | Strict behavioral **contract** implementation. |
| **State/Fields** | Can hold instance fields (e.g., `driver`, `page`). | Cannot hold instance state (constants only). |
| **Inheritance** | Single inheritance only. | Multiple interfaces per class allowed. |
| **Methods** | Mix of concrete and abstract methods. | Unimplemented method signatures only*. |

*\*Note: Java 8+ allows default/static methods in interfaces, and TypeScript allows optional properties, but their primary purpose remains structure enforcement rather than core base state management.*
---## Playwright (TypeScript) Real-World Use Cases### 1. When to Use Abstract Class in PlaywrightUse an abstract class to create a foundational Page Object Model (POM). It allows you to store the Playwright `Page` instance and expose helper wrappers (e.g., waiting for elements or capturing screenshots) that every page object must reuse.
```typescript
import { Page } from '@playwright/test';

// Shared base with state and implementation
export abstract class BasePage {
  protected page: Page;

  constructor(page: Page) {
    this.page = page;
  }

  // Shared concrete utility
  async takeScreenshot(name: string) {
    await this.page.screenshot({ path: `screenshots/${name}.png` });
  }

  // Abstract method: Forces subclasses to define their unique path
  abstract open(): Promise<void>;
}

// Subclass extending the shared foundation
export class LoginPage extends BasePage {
  async open() {
    await this.page.goto('/login');
  }
}
```
### 2. When to Use Interface in PlaywrightTypeScript interfaces exist **only at compile time** and add no weight to your compiled JavaScript. Use interfaces to structure test configuration shapes, API payload responses, or component contracts where state inheritance isn't needed.
```typescript
// Enforcing structural contract for test user data
interface UserCredentials {
  username: string;
  token: string;
  role: 'admin' | 'user';
}

export class AuthSession {
  // Uses the interface to strictly validate inputs
  async loginViaApi(user: UserCredentials) {
    console.log(`Logging in ${user.username} with role ${user.role}`);
  }
}
```
