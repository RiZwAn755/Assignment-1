# BUG REPORT - Order Management System

## 1. SQL Injection in Customer Search

- **Location**: `backend/src/routes/customers.js` (line 20)
- **Issue**: The search query uses string concatenation to build the SQL query: `const query = "SELECT * FROM customers WHERE name ILIKE '%" + name + "%'";`
- **Impact**: **Security**. An attacker can perform SQL injection to bypass queries, leak data, or even drop tables.
- **Fix**: Use parameterized queries with the `pg` pool.


## 2. Race Condition / Lack of Transaction in Order Creation
- **Location**: `backend/src/routes/orders.js` (lines 57-89)
- **Issue**: Inventory is checked in one query, then an order is created, and finally inventory is decremented in a third query. These are not wrapped in a database transaction.
- **Impact**: **Data Integrity**. If two users buy the last item simultaneously, both checks might pass before either decrement happens, leading to overselling (negative inventory).
- **Fix**: Wrap the entire operation in a `BEGIN...COMMIT` transaction and use a single query to decrement inventory with a `WHERE inventory_count >= quantity` clause, or use `SELECT FOR UPDATE`.

## 3. Broken Global Error Handler
- **Location**: `backend/src/index.js` (lines 24-27)
- **Issue**: The global error handler catches all errors but returns a `200 OK` status with `{ success: true }`.
- **Impact**: **Reliability / Debuggability**. The client thinks the request succeeded even when it failed server-side. Errors are "swallowed," making them hard to track.
- **Fix**: Return an appropriate error status (e.g., 500) and a meaningful error message.

## 4. N+1 Query Problem in Order List
- **Location**: `backend/src/routes/orders.js` (lines 7-31)
- **Issue**: The endpoint fetches all orders and then loops through them to fetch customer and product details for EACH order individually.
- **Impact**: **Performance**. If there are 100 orders, it makes 201 database queries (1 to get orders + 100 for customers + 100 for products). This scales poorly.
- **Fix**: Use a single `JOIN` query to fetch orders with their customer and product information in one go.

## 5. Invalid Status Transitions
- **Location**: `backend/src/routes/orders.js` (lines 95-110)
- **Issue**: There is no validation on the `status` update. An order can be moved from 'delivered' back to 'pending'.
- **Impact**: **Correctness**. Business logic is bypassed, allowing impossible state transitions.
- **Fix**: Implement a state machine or validation logic to ensure statuses only move forward (e.g., pending -> confirmed -> shipped -> delivered).

## 6. Frontend: Missing Dependencies in `useEffect` and Workaround Refetch
- **Location**: `frontend/src/components/OrderList.js` (lines 14-20)
- **Issue**: After updating a status, the code manually calls `fetchOrders()` to refresh. However, the `useEffect` that originally fetched data is missing dependencies, and the manual refetch resets the local state unexpectedly or ignores current sorting/filtering.
- **Impact**: **UX / Reliability**. The UI might flicker or reset state (like sorting) after an action.
- **Fix**: Use a proper state management approach or ensure `useEffect` is triggered correctly by state changes.

## 7. Lack of Input Validation
- **Location**: Multiple (`backend/src/routes/customers.js`, `backend/src/routes/orders.js`)
- **Issue**: Data from `req.body` is used directly in queries without validation (checks for empty strings, valid email formats, positive quantities, etc.).
- **Impact**: **Security / Reliability**. Malicious or malformed data can crash the server or corrupt the database.
- **Fix**: Use a validation library or simple manual checks before processing requests.

## 8. Missing Debounce on Search
- **Location**: `frontend/src/components/CustomerSearch.js` (lines 15-23)
- **Issue**: The search fires an API request on every single keystroke.
- **Impact**: **Performance / UX**. Excessive API calls can lag the frontend and overwhelm the backend during fast typing.
- **Fix**: Implement a debounce (e.g., 300ms) on the search input.
