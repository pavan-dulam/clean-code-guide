# The Ultimate Guide to Clean and Scalable Node.js Codebases

## Introduction
A poorly structured monolithic codebase can be difficult to maintain, debug, and scale. Common issues include:
- Business logic mixed with controllers or models
- Lack of reusable utility functions
- Large functions performing multiple tasks
- Oversized files
- Inconsistent error handling and excessive console logs
- Hardcoded values and string messages scattered throughout the code
- Lack of a structured approach to using classes

This guide provides a structured approach to refactoring and maintaining a clean Node.js codebase using TypeScript and best practices.

---

## 1. Choosing the Right Architecture

### Monolith vs. Microservices vs. Modular Monolith
- **Monolith:** Simple to start with but can become difficult to manage as the project grows.
- **Microservices:** Suitable for large-scale applications but introduces complexity.
- **Modular Monolith (Recommended):** Keeps services modular within a monolithic structure, offering scalability without unnecessary complexity.

### Layered Architecture (Separation of Concerns)
- **Routes/Controllers** – Handles HTTP requests and responses.
- **Services** – Contains business logic.
- **Repositories/DAOs** – Manages database interactions.
- **Utils/Helpers** – Reusable functions.
- **Middlewares** – Request validation, authentication, logging, etc.
- **Config** – Environment variables and application settings.
- **Models/Schemas** – Database schema definitions.
- **Enums** – Centralized fixed sets of options.

---

## 2. Codebase Structure
```
📂 src/
 ┣ 📂 config/        # Configuration files (env, database, etc.)
 ┣ 📂 controllers/   # API route handlers (should be thin)
 ┣ 📂 services/      # Business logic (should be fat)
 ┣ 📂 repositories/  # Database interactions (DB queries)
 ┣ 📂 middlewares/   # Middleware functions
 ┣ 📂 utils/         # Helper functions
 ┣ 📂 models/        # Database models/schemas
 ┣ 📂 enums/         # Fixed sets of options
 ┣ 📂 routes/        # Route definitions
 ┣ 📂 logs/          # Log files (if using a logging library)
 ┣ 📜 app.ts         # Main application entry point
 ┣ 📜 server.ts      # Server setup
```
This structure ensures that code is well-organized and maintainable.

---

## 3. Understanding Layers and Responsibilities

### Controllers (Thin Layer)
**Do:**
- Only handle request/response.
- Validate input (preferably using middleware).
- Call the service layer for business logic.
- Forward errors to a centralized error handler.

**Don't:**
- Place business logic here.
- Make direct database calls.
- Perform complex data manipulations.

### Services (Business Logic Layer)
**Do:**
- Implement all business logic.
- Call repositories for database interactions.
- Validate and process data before saving or returning it.
- Throw custom errors when necessary.

**Don't:**
- Interact with request/response directly.
- Return raw database results without processing.

### Repositories (Data Access Layer)
**Do:**
- Perform database operations.
- Abstract database logic from services.
- Use ORM/SQL queries efficiently.

**Don't:**
- Place business logic here.
- Expose raw database models directly.

### Utils (Helper Functions)
**Do:**
- Store reusable utility functions (e.g., formatting dates, hashing passwords).
- Keep functions stateless and reusable.

### Enums (Fixed Sets of Options)
Using enums ensures consistent values and avoids magic strings.
```ts
export enum UserRole {
  ADMIN = "admin",
  USER = "user",
  GUEST = "guest"
}
```
Usage:
```ts
const userRole: UserRole = UserRole.ADMIN;
if (userRole === UserRole.ADMIN) {
  console.log("User has admin privileges");
}
```

---

## 4. Best Practices for Clean Code

### Use TypeScript Instead of JavaScript
- Enforces type safety and reduces runtime errors.
- Provides better IDE support and refactoring capabilities.
- Improves code documentation with interfaces and types.

### Controllers Should Be Thin
**Bad Example (Business logic inside controllers):**
```js
app.post('/order', async (req, res) => {
  const orderData = req.body;
  const user = await User.findById(orderData.userId);
  if (!user) {
    return res.status(404).json({ message: "User not found" });
  }
  const order = new Order(orderData);
  await order.save();
  res.json(order);
});
```
**Good Example (Move logic to a service):**
```ts
// Controller
import { OrderService } from '../services/order.service';
router.post('/order', async (req, res, next) => {
  try {
    const order = await OrderService.createOrder(req.body);
    res.json(order);
  } catch (error) {
    next(error);
  }
});
```
```ts
// Service
import { OrderRepository } from '../repositories/order.repository';
export class OrderService {
  static async createOrder(data: OrderDTO) {
    const user = await UserRepository.findById(data.userId);
    if (!user) throw new Error("User not found");
    return await OrderRepository.create(data);
  }
}
```

### Use Proper Error Handling
- Implement a centralized error handler.
- Define custom error classes.
```ts
class AppError extends Error {
  constructor(public message: string, public statusCode: number) {
    super(message);
  }
}
```

### Use a Centralized Logger Instead of `console.log`
- Use logging libraries like Winston or Pino.
```ts
import winston from 'winston';
const logger = winston.createLogger({
  level: 'info',
  transports: [new winston.transports.Console()]
});
logger.info('Server started');
```

---

## 5. Common Mistakes to Avoid
- **Avoid putting business logic inside controllers.**
- **Avoid using `console.log` for debugging.**
- **Avoid hardcoded values or magic numbers.**
- **Avoid functions that do multiple things; keep them single-purpose.**
- **Avoid excessive use of global variables.**
- **Avoid large, unmanageable files; break them into modules.**
- **Avoid ignoring error handling and proper logging.**

---

## Conclusion
Refactoring a chaotic monolithic codebase takes time, but by following these best practices, you can create a well-structured, maintainable, and scalable system. Start small, refactor one module at a time, and enforce these guidelines across your team. This approach will lead to a more efficient development process and a more robust application.

This guide is designed to help the community improve their Node.js codebases by adopting clean architecture and best practices. By implementing these principles, debugging and feature development will become much more manageable.

