# Security & Stability Audit

Below are the critical bugs identified in the Order Management System. Per the project requirements, I have focused on implementing fixes for the top 3 high-priority issues (SQL Injection, Race Conditions, and Global Error Handling).

## Completed Fixes

### 1. SQL Injection Vulnerability
The search functionality was unsafely concatenating user input into SQL queries. This would have allowed attackers to manipulate the database. I have refactored this to use **parameterized queries** which ensures all user input is sanitized before execution.
*   **Status:** Fixed
*   **File:** `backend/src/routes/customers.js`

### 2. Transaction Integrity (Race Condition)
The order creation process was fragmented into separate database calls without a transaction. This meant two simultaneous orders could exceed available stock. I implemented a **PostgreSQL Transaction** with **`FOR UPDATE` row locking** to ensure that inventory checks and updates happen atomically.
*   **Status:** Fixed
*   **File:** `backend/src/routes/orders.js`

### 3. Global Error Handling
The server was "swallowing" errors by returning a 200 OK status even when the database crashed. I’ve updated the global middleware to properly return a **500 Internal Server Error** and log the actual error trace so that issues can be identified and fixed quickly.
*   **Status:** Fixed
*   **File:** `backend/src/index.js`

---

## Further Findings (Proposed Solutions)

While the focus was on the above fixes, I identified several other areas for improvement:

### N+1 Query Performance
The current order list fetches data in a loop, which will slow down significantly as the database grows.
**Proposed Solution**: Use a single SQL `JOIN` query to fetch order, customer, and product data in one trip to the database.

### Lack of Input Validation
The API currently accepts raw data from the frontend without checking for valid email formats or positive numbers.
**Proposed Solution**: Implement a validation layer (like Joi or Zod) to sanitize all incoming request bodies.

### Code Organization & Structure
The current structure mixes API routes and business logic in the same files. This makes the code harder to test as it grows.
**Proposed Solution**: Refactor the codebase to separate **Routes** (endpoints/middleware) from **Controllers** (business logic/DB queries). This would make the logic reusable and much easier to unit test.

### Missing Search Debounce
The search bar triggers an API call on every keystroke, which is inefficient.

**Proposed Solution**: Add a 300ms debounce to the frontend input to reduce unnecessary server load.
