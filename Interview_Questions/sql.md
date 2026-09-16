## SQL

## Q1. SQL query to find average salary

**Your queries — mostly correct, with notes**

```sql
-- Option 1: Using AVG() - preferred, simplest
SELECT AVG(salary) AS avg_salary
FROM Employees;

-- Option 2: Using SUM/COUNT - works, but AVG() is the better practice
SELECT SUM(salary)/COUNT(salary) AS avg_salary
FROM Employees;

-- Option 3: Average grouped by department - correct
SELECT department_id, AVG(salary) AS average_salary
FROM Employees
GROUP BY department_id;
```

**Note:** `AVG()` is preferred over manually dividing `SUM/COUNT` — it's cleaner and handles NULL values correctly by default (ignores NULLs automatically, same as COUNT does).

**Real-time example — banking**
- `SELECT branch_id, AVG(loan_amount) AS avg_loan FROM Loans GROUP BY branch_id;` — find the average loan amount issued per branch, useful for identifying which branches process larger loans on average

## Q2. SQL query to find products purchased by each employee (join query)

**Your queries are correct — here's the cleaned-up complete version**

```sql
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    p.product_id,
    p.product_name,
    o.order_date
FROM Employees e
INNER JOIN Orders o ON e.employee_id = o.employee_id
INNER JOIN Order_Details od ON o.order_id = od.order_id
INNER JOIN Products p ON od.product_id = p.product_id
ORDER BY e.employee_id, o.order_date DESC;
```

**Correct note on default join** — you're right: writing just `JOIN` without specifying a type defaults to `INNER JOIN` in every major database engine

**Real-time example — e-commerce**
- Find which products each sales rep processed: joining `Employees` → `Orders` → `Order_Details` → `Products` shows a report like "Employee John processed orders containing Product X, Y, Z" — useful for commission calculation or sales performance tracking

**Real-time example — banking**
- Find which loan officer processed which loan products: joining `Employees` → `Loan_Applications` → `Loan_Products` to see which officer handled home loans vs personal loans

## Q3. What are Aggregate Functions in SQL?

**Correct — here's the plain-English purpose of each**

- `COUNT()` — counts number of rows
- `SUM()` — adds up numeric values
- `AVG()` — calculates average
- `MAX()` — finds highest value
- `MIN()` — finds lowest value

**Real-time example — banking**
- `SELECT COUNT(*) FROM Transactions WHERE status = 'FAILED';` — count how many transactions failed today
- `SELECT MAX(transaction_amount) FROM Transactions WHERE account_id = 123;` — find the largest single transaction for an account (useful for fraud detection checks)

## Q4 & Q5. What is the purpose of GROUP BY?

**Your definition and examples are correct**
- GROUP BY groups rows that share the same value in a specified column, so aggregate functions (COUNT, SUM, AVG) can be applied **per group** instead of across the whole table

**Real-time example — e-commerce**
```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM Orders
GROUP BY customer_id;
```
- This gives total spending **per customer**, not one single total across all customers — useful for identifying top spenders for a loyalty program

**Real-time example — banking**
```sql
SELECT status, COUNT(status) AS count
FROM Transactions
GROUP BY status;
```
- Shows how many transactions fall into each status (`SUCCESS`, `FAILED`, `PENDING`) — useful for a daily operations health check

## Q6. How do you use the LIKE operator?

**Correction on your query — use single quotes, not double quotes, for string values in SQL (double quotes are used for identifiers/column names in some databases like PostgreSQL, and can cause errors elsewhere)**

**Your pattern explanations are correct — here's the cleaned-up reference**

| Pattern | Meaning |
|---|---|
| `'a%'` | Starts with "a" |
| `'%a'` | Ends with "a" |
| `'%a%'` | Contains "a" anywhere |
| `'_a'` | Exactly 2 characters, second one is "a" |
| `'a_'` | Exactly 2 characters, first one is "a" |

**Corrected query**
```sql
SELECT *
FROM Customers
WHERE first_name LIKE '_a%';   -- second letter is 'a', any length
```

**Real-time example — banking**
- `SELECT * FROM Customers WHERE email LIKE '%@gmail.com';` — find all customers using Gmail addresses, useful for testing email-domain-specific notification logic

**Real-time example — e-commerce**
- `SELECT * FROM Products WHERE product_name LIKE '%wireless%';` — search functionality showing all products with "wireless" anywhere in the name

## Q7. What is the UNIQUE constraint in SQL?

**Your definition is mostly right — one correction needed**

- UNIQUE constraint ensures all values in a column are different from each other — no duplicates allowed
- **Correction:** In standard SQL, a UNIQUE column actually allows **multiple NULL values**, since NULL is considered "unknown," and two unknowns are never treated as duplicates of each other. (Note: SQL Server is a notable exception — it only allows a single NULL in a UNIQUE column, unlike MySQL/PostgreSQL/Oracle which allow multiple NULLs.) So don't state "allows single null value" as a universal rule — it depends on the database.

**Constraint types — your list is correct, cleaned up:**
- **Primary Key** — uniquely identifies each row; combines UNIQUE + NOT NULL
- **Foreign Key** — links a column to a primary key in another table, enforcing referential integrity
- **NOT NULL** — column cannot have empty/null values
- **UNIQUE** — no duplicate values allowed in the column
- **CHECK** — enforces a custom condition on the column's values (e.g., `age > 18`)
- **DEFAULT** — automatically assigns a default value if none is provided during insert

**Real-time example — banking**
- `account_number` column has a UNIQUE constraint — no two accounts can share the same account number
- `CHECK (balance >= 0)` constraint could prevent an account balance from ever going negative directly at the DB level, as an extra safety net beyond application logic

**Real-time example — e-commerce**
- `email` column on the Customers table has a UNIQUE constraint — prevents two customer accounts from registering with the same email

## Q8. SQL query to find duplicate records

**Your logic is correct — here's the cleaned-up version**

```sql
SELECT age, COUNT(age) AS duplicate_count
FROM Customers
GROUP BY age
HAVING COUNT(age) > 1;
```

**Note:** naming the aliased column `duplicate_records` in your version was a bit confusing since it's actually the count, not the records themselves — `duplicate_count` is clearer

**Real-time example — e-commerce**
```sql
SELECT email, COUNT(*) AS count
FROM Customers
GROUP BY email
HAVING COUNT(*) > 1;
```
- Finds customers who accidentally registered with the same email more than once — useful for a data-cleanup or duplicate-account bug investigation

**Real-time example — banking**
```sql
SELECT customer_id, COUNT(*) AS count
FROM Accounts
GROUP BY customer_id
HAVING COUNT(*) > 5;
```
- Finds customers with more than 5 accounts — potentially useful for flagging unusual account activity

## Q9. Query to find employees with salary greater than Adam's salary

**Correction — your query used the wrong table/columns (Orders/item="Keyboard" instead of Employees/salary). Here's the corrected version matching the actual question:**

```sql
SELECT *
FROM Employees
WHERE salary > (
    SELECT salary
    FROM Employees
    WHERE first_name = 'Adam'
);
```

**Plain English**
- The inner query (subquery) first finds Adam's exact salary
- The outer query then finds every employee whose salary is greater than that value

**Real-time example — banking**
```sql
SELECT * FROM Employees
WHERE salary > (SELECT salary FROM Employees WHERE first_name = 'Priya');
```
- Useful in a real scenario like auditing pay equity, or generating a report of "employees earning more than a specific benchmark employee"

**Real-time example — e-commerce**
```sql
SELECT * FROM Products
WHERE price > (SELECT price FROM Products WHERE product_name = 'Wireless Mouse');
```
- Find all products priced higher than a specific reference product, useful for a "similar or premium alternatives" feature

## Q10. Find the 3rd and 5th highest salary

**Explaining `LIMIT n-1, 1` — what "n" means**

- In `LIMIT offset, count`, the **offset** tells SQL how many rows to *skip* before starting to return results, and **count** tells it how many rows to return after that
- If you sort salaries in descending order (highest first) and want the **Nth highest** value, you need to skip the first `(N-1)` rows, then take the next 1 row
- Example: for the **3rd highest**, you skip the top 2 rows (offset = 2) and take 1 row → `LIMIT 2, 1`
- Example: for the **5th highest**, you skip the top 4 rows (offset = 4) and take 1 row → `LIMIT 4, 1`
- So "n" in "n-1" just refers to whichever rank you're looking for — 3rd highest means n=3, so offset = n-1 = 2

**Your queries are correct**
```sql
-- 3rd highest salary
SELECT DISTINCT salary
FROM Employees
ORDER BY salary DESC
LIMIT 2, 1;

-- 5th highest salary
SELECT DISTINCT salary
FROM Employees
ORDER BY salary DESC
LIMIT 4, 1;
```

**Important — why DISTINCT matters here (using your sample data: 12000, 400, 400, 300, 250)**
- Without `DISTINCT`, duplicate salary values (like the two 400s) would each occupy their own row position, throwing off the ranking
- Sorted descending without DISTINCT: `12000, 400, 400, 300, 250` — the "2nd highest" here would incorrectly show `400` again instead of moving to the next actual distinct value
- With `DISTINCT`, duplicates collapse into one: `12000, 400, 300, 250` — now the 2nd highest correctly shows `400` only once, and ranking reflects genuinely different salary values

**Alternative subquery approach — your version is correct too**
```sql
SELECT MAX(salary)
FROM Employees
WHERE salary < (SELECT MAX(salary) FROM Employees);
```
- This finds the 2nd highest specifically (highest salary that's still less than the overall highest) — this exact pattern only works cleanly for 2nd highest; for 3rd/5th highest, you'd need to nest this logic further, which is why the `LIMIT` + `DISTINCT` approach is much simpler and more commonly used in interviews

**Real-time example — banking**
- Finding the 3rd highest transaction amount in a day (`ORDER BY amount DESC LIMIT 2,1`) — useful for reviewing top unusual transactions for fraud monitoring without listing every single transaction

**Real-time example — e-commerce**
- Finding the 5th best-selling product by revenue (`ORDER BY total_revenue DESC LIMIT 4,1`) — useful for a "top 5 products" dashboard where you need to identify exactly the 5th-ranked item

## SQL (continued)

## Q11. SQL query: Delete duplicates (keep only one)

**Your logic direction is right, but the JOIN condition needs fixing — you had the WHERE and JOIN condition mixed up**

```sql
-- Corrected SELECT to identify duplicates first (always verify with SELECT before DELETE)
SELECT o1.*
FROM Orders o1
JOIN Orders o2 ON o1.amount = o2.amount
WHERE o1.order_id > o2.order_id;

-- Corrected DELETE
DELETE o1
FROM Orders o1
JOIN Orders o2 ON o1.amount = o2.amount
WHERE o1.order_id > o2.order_id;
```

**What was off**
- The join condition (what makes two rows a "matching pair") should go in the `ON` clause — `o1.amount = o2.amount`
- The filter condition (which one of the pair to delete) goes in `WHERE` — `o1.order_id > o2.order_id`, meaning: for every pair of rows with the same amount, delete the one with the higher order_id, keeping the earliest one

**Important notes**
- This `DELETE ... JOIN` syntax is **MySQL-specific** — it won't work the same way in SQL Server or Oracle; those need a different approach (e.g., using a CTE with `ROW_NUMBER()`)
- **Always run the SELECT version first** to see exactly which rows would be deleted, before running the actual DELETE — this is a critical safety habit in real work, especially in production databases

**Real-time example — e-commerce**
- A checkout retry bug caused duplicate orders with identical amount, customer, and timestamp to be created. Running the SELECT first shows exactly which duplicate rows will be removed (keeping the original, lowest `order_id`), then the DELETE cleans them up safely.

## Q12. SQL query: Customers who never placed an order

**Your query works — one important caveat to know for the interview**

```sql
SELECT *
FROM Customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM Orders
);
```

**The hidden trap with `NOT IN`**
- If the `Orders.customer_id` column contains even **one NULL value**, `NOT IN` will return **zero rows** for the entire query — silently broken, with no error — because comparing anything to NULL with `NOT IN` evaluates as unknown, not true/false
- This is a well-known SQL gotcha that's worth mentioning proactively in an interview — it shows depth

**Safer alternative — `NOT EXISTS`**
```sql
SELECT *
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o WHERE o.customer_id = c.customer_id
);
```

**Another safe alternative — `LEFT JOIN` with NULL check**
```sql
SELECT c.*
FROM Customers c
LEFT JOIN Orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

**Real-time example — banking**
- Finding customers who opened an account but never made a single transaction — useful for a "inactive account" report to trigger re-engagement emails or compliance review. Using `NOT EXISTS` here is safer than `NOT IN` in case any transaction record has a NULL customer reference due to a data entry issue.

## Q13. SQL query: Employees earning above company average

**Correction — wrong table and columns were used. Here's the corrected version for the actual question asked:**

```sql
SELECT *
FROM Employees
WHERE salary > (SELECT AVG(salary) FROM Employees);
```

**Real-time example — banking**
- Finding all employees earning above the company-wide average salary — useful for HR compensation analysis, or flagging outliers for a pay-equity review

**Real-time example — e-commerce**
```sql
SELECT * FROM Products
WHERE price > (SELECT AVG(price) FROM Products);
```
- Find all products priced above the average product price — useful for a "premium products" filter or category

## Q14. SQL query: Top 3 salaries in each department

**Important data-type issue to flag first**
- The `SALARY` column was created as `varchar(20)` instead of a numeric type like `INT` or `DECIMAL` — this is a real bug worth catching. String comparison doesn't work like number comparison: `'90000' < '145000'` as **strings** actually evaluates incorrectly in lexicographic (character-by-character) order in some contexts, and sorting/comparing salaries as text can silently give wrong results. **Salary should always be stored as a numeric type** (`DECIMAL(10,2)` is ideal for currency/salary to avoid floating-point rounding issues).

**Logic correction on your query — the comparison and threshold were backwards**

Your original query counts how many salaries are **less than** the current row and checks if that count is under 2 — this actually selects the **lowest** salaries, not the top 3, and only compares up to a threshold of 2 (top 2, not top 3).

**Corrected version**
```sql
SELECT DEPT, EMP_NAME, CAST(SALARY AS UNSIGNED) AS salary_numeric
FROM EMPLOYEE e1
WHERE (
    SELECT COUNT(*)
    FROM EMPLOYEE e2
    WHERE e2.DEPT = e1.DEPT
      AND CAST(e2.SALARY AS UNSIGNED) > CAST(e1.SALARY AS UNSIGNED)
) < 3
ORDER BY DEPT, salary_numeric DESC;
```

**Plain English of the fix**
- For each employee, count how many people **in the same department** earn **more** than them
- If fewer than 3 people earn more, that employee is in the top 3 by definition
- `CAST(... AS UNSIGNED)` converts the varchar salary to a number for correct comparison (this is a MySQL-specific cast; SQL Server would use `CAST(... AS INT)`, and ideally this cast wouldn't even be needed if the column were the correct numeric type from the start)

**Cleaner modern alternative — using window functions (worth mentioning to sound current)**
```sql
SELECT DEPT, EMP_NAME, salary_numeric
FROM (
    SELECT DEPT, EMP_NAME, 
           CAST(SALARY AS UNSIGNED) AS salary_numeric,
           DENSE_RANK() OVER (PARTITION BY DEPT ORDER BY CAST(SALARY AS UNSIGNED) DESC) AS rnk
    FROM EMPLOYEE
) ranked
WHERE rnk <= 3;
```
- `DENSE_RANK()` assigns a rank within each department (partitioned by DEPT), ordered by salary descending — much more readable than a correlated subquery, and this is the pattern most interviewers actually want to see today since window functions are the modern standard for "top N per group" problems

**Real-time example — e-commerce**
- Same pattern applied to find the top 3 best-selling products by revenue **within each category** — `DENSE_RANK() OVER (PARTITION BY category ORDER BY revenue DESC) <= 3`

## Q15. SQL query: Print odd/even rows

**Important distinction to catch here**
- Your query checks whether the **EMPNO value itself** is odd/even (`MOD(EMPNO, 2)`) — this only coincidentally works because your `EMPNO` values happen to be sequential (101, 102, 103...)
- "Print odd/even **rows**" in SQL usually means odd/even by **row position in the result set**, not by the value of a particular column — these are different things, and an interviewer may specifically be testing whether you know the difference

**Correct approach for actual row position — using ROW_NUMBER()**
```sql
-- Odd rows (1st, 3rd, 5th... by position)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY EMPNO) AS rn
    FROM EMPLOYEE
) t
WHERE MOD(rn, 2) = 1;

-- Even rows (2nd, 4th, 6th... by position)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER (ORDER BY EMPNO) AS rn
    FROM EMPLOYEE
) t
WHERE MOD(rn, 2) = 0;
```

**Also note:** since `EMPNO` is stored as `varchar`, `MOD(EMPNO, 2)` relies on MySQL implicitly converting the string to a number — this works for simple numeric-looking strings like `'101'`, but it's fragile and not something to rely on; if `EMPNO` ever contained a non-numeric value or a prefix like `'EMP101'`, this would break or behave unpredictably

**Real-time example — banking**
- If asked to split a large customer list into two batches for parallel processing (e.g., sending statements), using `ROW_NUMBER()` to assign odd/even rows ensures an even, position-based split regardless of what the actual customer ID values look like — whereas relying on `MOD(customer_id, 2)` would create an uneven split if IDs aren't perfectly sequential (e.g., some IDs deleted/skipped over time).
