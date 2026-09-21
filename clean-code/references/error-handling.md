# Error Handling Rule

Contents: [11. Error Handling](#11-error-handling-التعامل-مع-الأخطاء)

---

## 11. Error Handling (التعامل مع الأخطاء)

**Idea:** never assume every operation succeeds. Decide explicitly what happens when input is bad, a file is missing, the network fails, or the database refuses.

> **This rule changes behavior.** Before, a bad input might crash with a stack trace. After, it returns a clean 400. That is the point, but it means every change here must be listed separately with "before" and "after" behavior and approved by the user.

**Smells**
- `except:` or `except Exception: pass`, which hides real bugs.
- Return values ignored: a `None`, `False`, or `-1` failure result nobody checks.
- Assuming keys and values exist: `data["x"]`, `int(request.form["n"])`, `row[0]` on a possibly empty result.
- Unvalidated input from users, files, environment variables, or external APIs.
- Network calls with no timeout.
- Resources opened and never closed (files, DB connections).
- Multi-step database changes with no rollback on failure.
- Generic messages that lose the cause ("Something went wrong") and no logging.
- The opposite mistake: leaking stack traces or SQL errors to end users.
- Inconsistent failure signals: some functions raise, some return `None`, some return `False`.

**How to fix**
- **Validate at the boundary:** where data enters the system (request, file, env, external API). Inside, trust validated data.
- **Catch specific exceptions,** and only where you can do something meaningful about them (recover, retry, translate to a response, add context).
- **Let unexpected errors surface.** A crash from a real bug is better than a swallowed one. Log it at the top level.
- **Log with context** for developers (what was being done, which ids). Give the user a short clear message without internals.
- **Use context managers / `finally`** for cleanup, and rollback on failure.
- **Set timeouts** on network calls.
- **Be consistent:** choose one failure style (exceptions with domain-specific types is a good default) and one error response shape.

**Before**
```python
@app.route("/pay", methods=["POST"])
def pay():
    amount = float(request.form["amount"])
    conn = sqlite3.connect("app.db")
    conn.execute("INSERT INTO payments (amount) VALUES (?)", (amount,))
    conn.commit()
    return "ok"
```
Problems: a missing field returns Flask's generic 400 page; non-numeric input raises `ValueError` (a 500 error); negative amounts are accepted; the connection is never closed; a failure part-way has no rollback.

**After**
```python
import logging
from decimal import Decimal, InvalidOperation

logger = logging.getLogger(__name__)

def parse_amount(raw_value):
    try:
        amount = Decimal(raw_value)
    except (InvalidOperation, TypeError):
        raise ValueError("Amount must be a number")
    if amount <= 0:
        raise ValueError("Amount must be greater than zero")
    return amount

@app.route("/pay", methods=["POST"])
def pay():
    try:
        amount = parse_amount(request.form.get("amount"))
    except ValueError as error:
        return {"error": str(error)}, 400

    try:
        with get_connection() as conn:      # commits on success, rolls back on error, always closes
            insert_payment(conn, amount)
    except sqlite3.DatabaseError:
        logger.exception("Failed to save payment (amount=%s)", amount)
        return {"error": "Could not save the payment"}, 500

    return "ok"
```

**Don't**
- Wrap whole functions in one big `try/except`.
- Catch `Exception` just to make an error disappear.
- Add validation that changes accepted input (for example rejecting values that were accepted before) without listing it as a behavior change.
- Invent user-facing messages in the project's language without asking. If the project's UI is in Arabic, propose the message text and let the user confirm the wording.
- Add retries around operations that are not safe to repeat (payments, sending messages) without an idempotency plan.
