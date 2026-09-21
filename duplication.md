# Duplication Rules

Contents: [9. DRY](#9-dry-عدم-تكرار-الكود) · [10. Don't Repeat Logic](#10-dont-repeat-logic-عدم-تكرار-المنطق)

The two rules sound alike but catch different problems:

| | DRY | Don't Repeat Logic |
|---|-----|--------------------|
| What repeats | The **text** of the code | A **business rule or piece of knowledge** |
| Looks like | Copy-pasted blocks | The same rule written in different shapes in different places |
| Easy to spot? | Yes, visually and with search | No, you must know the domain |
| Risk | Annoying edits | Bugs when the copies drift apart |

---

## 9. DRY (عدم تكرار الكود)

**Idea:** write it once, call it from everywhere it is needed.

**Smells**
- The same block of about 3+ lines copy-pasted in 2+ places.
- Functions identical except for one value.
- The same literal repeated: status strings, URLs, limits, column names.
- Repeated validation or query fragments.
- Repeated UI markup (the same card, table, or form).

**How to fix**
- Extract a function; make the difference a parameter.
- Move repeated literals to named constants or an enum.
- Replace repeated `if/elif` branches with a lookup table (dict).
- Repeated UI: extract a template partial, macro, or component.

**Before**
```python
def get_active_members():
    rows = db.execute("SELECT * FROM members WHERE status = 'active'").fetchall()
    return [dict(r) for r in rows]

def get_suspended_members():
    rows = db.execute("SELECT * FROM members WHERE status = 'suspended'").fetchall()
    return [dict(r) for r in rows]
```
**After**
```python
def get_members_by_status(status):
    rows = db.execute("SELECT * FROM members WHERE status = ?", (status,)).fetchall()
    return [dict(r) for r in rows]
```

**Lookup table instead of repeated branches**
```python
# Before
if kind == "a": rate = 0.1
elif kind == "b": rate = 0.15
elif kind == "c": rate = 0.2

# After
RATES = {"a": 0.1, "b": 0.15, "c": 0.2}
rate = RATES[kind]
```

**Don't**
- Merge code that only *looks* the same but changes for different reasons. A wrong abstraction costs more than a little duplication.
- Unify by adding many flags. If the shared function needs 5 flags to serve its callers, the duplication was not real.
- Apply it to tests when repeating makes them easier to read.
- Abstract on the second copy if it is unclear they belong together. Wait for the third (rule of three).

---

## 10. Don't Repeat Logic (عدم تكرار المنطق)

**Idea:** every business rule lives in one place. If the rule changes, you edit one place.

**Smells**
- A rule like "who is eligible" or "how the late fee is computed" appears in the route, the report, the template, a SQL `WHERE`, and the JavaScript, each written slightly differently.
- Fixing a rule means editing several files and hoping you found them all.
- Frontend and backend validation that have drifted apart.
- The same calculation (tax, totals, discounts, fees) written twice in different ways.

**How to find it**
- Search for the domain words and literals: the status names, thresholds (`> 30`, `0.14`), and field combinations.
- Search in every layer: backend code, SQL, templates, JavaScript, reports, exports.
- Ask "if this rule changed tomorrow, what would I have to edit?"

**How to fix**
1. List every place the rule appears.
2. **Compare the copies.** If they disagree, stop. This is a probable bug. Report it and ask which behavior is correct. Never silently pick one.
3. Choose one source of truth: a function in the business layer, a constant, or a database view.
4. Make all other places call it.
5. If the rule must exist in two runtimes (for example Python and JavaScript), keep one written spec, add tests on both sides, and leave a comment in each pointing to the source of truth.

**Before** (same rule, three shapes)
```python
# routes.py
if order.status == "delivered" and order.days_since_delivery <= 14:
    allow_refund()

# reports.py
refundable = [o for o in orders if o["days_since_delivery"] <= 14 and o["status"] == "delivered"]
```
```html
<!-- template -->
{% if order.status == 'delivered' and order.days_since_delivery < 14 %}
```
(Note the template uses `< 14`, not `<= 14`. Two copies already disagree.)

**After**
```python
# rules.py
REFUND_WINDOW_DAYS = 14

def is_refundable(order):
    return order.status == "delivered" and order.days_since_delivery <= REFUND_WINDOW_DAYS
```
Routes, reports, and templates all call `is_refundable(order)`. The `<` versus `<=` difference is raised with the user first, because fixing it changes behavior.

**Don't**
- Centralize rules that are only coincidentally similar.
- Move a rule without checking that all copies really mean the same thing.
- Move logic into SQL, or out of it, without confirming the performance and testing implications.
