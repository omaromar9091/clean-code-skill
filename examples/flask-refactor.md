# Example: Refactoring a Flask route

A checkout route that mixes everything in one function, and the same route after refactoring.

Rules applied: Naming, Early Return, Function, Single Responsibility, Separation of Concerns, Error Handling (flagged as a behavior change).

## Before: `app.py`

```python
from flask import Flask, request, render_template
import sqlite3

app = Flask(__name__)

@app.route("/orders/<int:order_id>/checkout", methods=["POST"])
def checkout(order_id):
    conn = sqlite3.connect("shop.db")
    cur = conn.cursor()
    cur.execute("SELECT id, total, status, customer_id FROM orders WHERE id = ?", (order_id,))
    row = cur.fetchone()
    if row:
        if row[2] == "pending":
            cur.execute("SELECT balance FROM customers WHERE id = ?", (row[3],))
            balance = cur.fetchone()[0]
            if balance >= row[1]:
                cur.execute("UPDATE customers SET balance = balance - ? WHERE id = ?", (row[1], row[3]))
                cur.execute("UPDATE orders SET status = 'paid' WHERE id = ?", (order_id,))
                conn.commit()
                return render_template("done.html", total=row[1])
            else:
                return "Not enough balance", 400
        else:
            return "Order already processed", 400
    else:
        return "Order not found", 404
```

### What is wrong

| Rule | Problem |
|------|---------|
| Naming | `row[2]`, `row[1]`, `row[3]`: nobody remembers what the indexes mean |
| Early Return | 4 levels of nesting, errors at the bottom, far from their conditions |
| Function / SRP | One function reads the DB, applies business rules, updates the DB, and renders HTML |
| Separation of Concerns | Route, business rules, and SQL are all in one place; you can't test the rules without a web server |
| Error Handling | Connection is never closed; `fetchone()[0]` crashes if the customer doesn't exist; no rollback |

## After

### `repository.py`: data access only

```python
import sqlite3
from contextlib import contextmanager

DB_PATH = "shop.db"

@contextmanager
def get_connection():
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

def find_order(conn, order_id):
    return conn.execute(
        "SELECT id, total, status, customer_id FROM orders WHERE id = ?", (order_id,)
    ).fetchone()

def find_customer_balance(conn, customer_id):
    row = conn.execute("SELECT balance FROM customers WHERE id = ?", (customer_id,)).fetchone()
    return row["balance"] if row else None

def deduct_balance(conn, customer_id, amount):
    conn.execute("UPDATE customers SET balance = balance - ? WHERE id = ?", (amount, customer_id))

def mark_order_paid(conn, order_id):
    conn.execute("UPDATE orders SET status = 'paid' WHERE id = ?", (order_id,))
```

### `services.py`: business rules only (no Flask, no SQL)

```python
import repository

class OrderNotFound(Exception): pass
class OrderAlreadyProcessed(Exception): pass
class InsufficientBalance(Exception): pass

def checkout_order(conn, order_id):
    order = repository.find_order(conn, order_id)
    if order is None:
        raise OrderNotFound(order_id)
    if order["status"] != "pending":
        raise OrderAlreadyProcessed(order_id)

    balance = repository.find_customer_balance(conn, order["customer_id"])
    if balance is None or balance < order["total"]:
        raise InsufficientBalance(order_id)

    repository.deduct_balance(conn, order["customer_id"], order["total"])
    repository.mark_order_paid(conn, order_id)
    return order["total"]
```

### `routes.py`: thin, only translates between HTTP and the service

```python
from flask import render_template
from repository import get_connection
from services import checkout_order, OrderNotFound, OrderAlreadyProcessed, InsufficientBalance

# Same messages and status codes as before, kept in one place.
ERROR_RESPONSES = {
    OrderNotFound: ("Order not found", 404),
    OrderAlreadyProcessed: ("Order already processed", 400),
    InsufficientBalance: ("Not enough balance", 400),
}

# `app` is your Flask app (or a Blueprint, if you use those).
@app.route("/orders/<int:order_id>/checkout", methods=["POST"])
def checkout(order_id):
    try:
        with get_connection() as conn:
            total = checkout_order(conn, order_id)
    except tuple(ERROR_RESPONSES) as error:
        message, status = ERROR_RESPONSES[type(error)]
        return message, status
    return render_template("done.html", total=total)
```

## What stayed the same (on purpose)

- The URL, HTTP method, template name, and template variable (`total`).
- Every user-facing message and status code.
- The database schema and SQL semantics.

## Behavior change to report, not hide

If the order's customer does not exist, the old code crashed with a 500 error (`fetchone()` returned `None`). The new code responds with "Not enough balance" (400). This is reasonable, but it is a change, so the agent must list it under **Behavior-changing suggestions** and get approval.

## Applied in this order

1. Rename `row[n]` accesses to named columns (`sqlite3.Row`): Naming.
2. Extract SQL to `repository.py`: Separation of Concerns, no behavior change.
3. Move rules into `services.py` with guard clauses: Early Return, Function, Single Responsibility.
4. Thin the route: Separation of Concerns.
5. Connection handling with rollback and closing, and the missing-customer case: Error Handling (approved separately).
