# Structure Rules

Contents: [5. Function](#5-function-الدوال) · [6. Single Responsibility](#6-single-responsibility-المسؤولية-الواحدة) · [7. Abstraction](#7-abstraction-التجريد) · [8. Separation of Concerns](#8-separation-of-concerns-فصل-المسؤوليات)

Rules 5 to 8 are about how code is cut into pieces. They fit together but work at different sizes:

| Rule | Level | Question it answers |
|------|-------|---------------------|
| Function | one function | Does it do one job? |
| Single Responsibility | one class, module, or file | Does it have one reason to change? |
| Abstraction | repeated or noisy details | Is this detail hidden behind something stable and clear? |
| Separation of Concerns | whole layers | Are UI/API, business logic, and data kept apart? |

---

## 5. Function (الدوال)

**Idea:** each function does one specific job, at one level of detail.

**Smells**
- Longer than about 40 lines, or you must scroll to read it.
- The name needs "and" or "then": `validate_and_save`.
- Comments like `# step 1`, `# step 2` inside the body.
- More than about 4 parameters.
- Boolean flag parameters that switch behavior: `send(msg, urgent=True)`.
- Computes a value and also prints, saves, or sends it.
- Mixes high-level steps with low-level details.

**How to fix**
- Extract each named step into its own function. The original becomes a short orchestrator that reads like a table of contents.
- Replace a flag parameter with two clearly named functions.
- Group related parameters into one object (dataclass, dict, or a typed structure your project already uses).
- Separate pure calculation from side effects (I/O, database, network).

**Before**
```python
def process_order(order):
    # validate
    if not order.items:
        raise ValueError("empty order")
    # calculate
    total = sum(i.price * i.qty for i in order.items)
    if order.coupon:
        total -= total * order.coupon.rate
    # save
    db.save(order, total)
    # notify
    send_email(order.customer.email, f"Total: {total}")
```
**After**
```python
def process_order(order):
    validate_order(order)
    total = calculate_total(order)
    save_order(order, total)
    notify_customer(order.customer, total)
```

**Don't**
- Extract one-line code that has no meaningful name.
- Split so far that the reader jumps through ten tiny functions to follow one idea. Extract when the piece has a clear name and purpose.

---

## 6. Single Responsibility (المسؤولية الواحدة)

**Idea:** a class, module, or file should have one main reason to change. Function is about one function's job; this is the same principle at the module and class level.

**Smells**
- You can only describe it with "and": "this module handles payments **and** sends emails **and** builds reports".
- A `utils.py` or `helpers.js` that became a dumping ground.
- A class that changes when the business rule changes **and** when the report format changes.
- A god file with hundreds of lines covering unrelated topics.

**Test:** describe the module in one sentence without "and". If you can't, it probably needs splitting.

**How to fix**
- Split by reason to change. Name the new modules by responsibility: `pricing.py`, `invoices.py`, `notifications.py`.
- Move code in small steps. Keep old import paths working until every caller is updated, or update all callers in the same step.

**Before**
```python
# order_manager.py  (700 lines)
class OrderManager:
    def calculate_price(self, order): ...
    def save(self, order): ...
    def render_invoice_pdf(self, order): ...
    def send_confirmation_email(self, order): ...
```
**After**
```python
# pricing.py          -> calculate_price(order)
# order_repository.py -> save(order)
# invoices.py         -> render_invoice_pdf(order)
# notifications.py    -> send_confirmation_email(order)
```

**Don't**
- Make one class per method. Keep things that change together, together (cohesion).
- Split a small script that only has one job.

---

## 7. Abstraction (التجريد)

**Idea:** gather repeated or noisy details into one stable, clearly named piece so the calling code reads at the level of intent. But stop before the abstraction adds more complexity than it removes.

**Under-abstraction smells**
- Raw SQL strings, HTTP calls, date formatting, or file paths built by hand in many places.
- Magic numbers and magic strings (`if status == 3`, `"pending"` typed everywhere).
- The intent of the code is buried in low-level details.

**Over-abstraction smells**
- An interface, factory, or base class with a single implementation.
- A wrapper that only forwards to another function.
- A generic helper with many flags to cover every case.
- Everything driven by config for cases that never change.

**How to fix**
- Hide noisy details behind a small, well-named function, class, or constant.
- **Rule of three:** wait until something appears about three times (or is both noisy and clearly stable) before abstracting it.
- Good abstraction reduces what the reader must know. If the reader must open the abstraction to understand the caller, it is not helping.

**Before**
```python
if order.status == "pending" and (datetime.now() - order.created_at).days > 7:
    ...
```
**After**
```python
PENDING = "pending"
STALE_AFTER_DAYS = 7

def is_stale(order):
    age_in_days = (datetime.now() - order.created_at).days
    return order.status == PENDING and age_in_days > STALE_AFTER_DAYS

if is_stale(order):
    ...
```

**Over-engineering to remove or avoid**
```python
# One discount type, yet three layers:
class AbstractDiscountStrategyFactory: ...
# Simply:
def apply_discount(price, rate): ...
```

When in doubt, keep it concrete.

---

## 8. Separation of Concerns (فصل المسؤوليات)

**Idea:** keep UI/API handling, business logic, and data access in separate places, so each can be understood, tested, and changed on its own.

**Smells**
- A route or controller that contains SQL, business calculations, and response formatting.
- Business rules inside templates or UI event handlers.
- SQL scattered through many route functions.
- Business logic that needs `request`, `session`, or other web objects, so it can't be tested without a web server.
- An API handler that calls external services directly.

**The layers** (dependencies point downward only)

```
Presentation   routes / controllers / UI components
               parse input, call the service, shape the response
      |
Business       services / domain
               rules and calculations; no request objects, no SQL
      |
Data           repositories / DAOs / API clients
               queries and external calls; no business rules
```

Typical mappings:
- Flask/Django/Express: `routes` then `services` then `repositories`.
- Frontend: components (display) then hooks/services (logic and state) then API client.
- Desktop apps (WPF, etc.): View then ViewModel then Service/Repository.

**How to fix, safest order**
1. Extract data access into repository functions (no behavior change).
2. Extract business rules into service functions that receive plain values or objects and return results or raise domain-specific exceptions.
3. Make the route thin: parse, call, map result or exception to a response.

Keep URLs, response shapes, status codes, and user-facing messages identical.

See `examples/flask-refactor.md` for a full before/after.

**Don't**
- Force three layers on a 50-line script or a prototype. Scale the structure to the size of the project.
- Introduce a new framework or dependency-injection container for this.
