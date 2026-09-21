# Readability Rules

Contents: [1. Naming](#1-naming-التسمية) · [2. Comment](#2-comment-التعليقات) · [3. Consistency](#3-consistency-الاتساق) · [4. Early Return](#4-early-return-الخروج-المبكر)

These are the lowest-risk rules. They are mostly local edits inside one function or file.

---

## 1. Naming (التسمية)

**Idea:** a name should tell the reader what a thing is or does, using the project's domain vocabulary.

**Smells**
- Meaningless names: `calc1`, `data`, `tmp`, `x`, `do_stuff`, `flag`, `result2`.
- Names that lie: `get_user()` that also updates the database.
- Booleans without a question form: `active` vs `is_active`, `has_access`, `should_retry`.
- One concept, several names: `user`, `usr`, `member`, `account` all meaning the same thing.
- Abbreviations the team does not use.

**How to fix**
- Function = verb phrase (`apply_discount`). Variable = noun (`discounted_price`). Boolean = question (`is_paid`).
- Length grows with scope: `i` in a 3-line loop is fine, a module-level name needs to be descriptive.
- Pick one word per concept and use it everywhere.
- Rename across the whole project: search usages in code, templates, tests, config, and strings.

**Before**
```python
def calc1(d, r):
    return d - d * r
```
**After**
```python
def apply_discount(price, discount_rate):
    return price - price * discount_rate
```

**A name that lies, split by what it really does**
```python
# Before: "get" but it writes
def get_user(user_id):
    user = db.find(user_id)
    user.last_seen = now()
    db.save(user)
    return user

# After
def find_user(user_id):
    return db.find(user_id)

def touch_last_seen(user):
    user.last_seen = now()
    db.save(user)
```

**Don't**
- Rename public APIs, route URLs, DB columns, or JSON fields (see guardrails).
- Rename domain terms the team already uses, even if you dislike them.
- Make names long just to be long. `customer_who_placed_the_order_id` is worse than `customer_id`.

---

## 2. Comment (التعليقات)

**Idea:** comments explain **why** a decision was made. The code itself should explain **what** it does.

**Smells**
- Comments that repeat the code: `# increment i`.
- Commented-out code (version control remembers it).
- Stale comments that no longer match the code.
- A comment used to excuse a bad name, or to label sections inside a long function.
- Missing "why" on things a reader would question: magic numbers, workarounds, business rules, odd ordering.

**How to fix**
- Delete comments that only restate the code, or make the code say it (better name, extracted function).
- Delete dead code.
- Add a short why-comment where a reader would ask "why?".

**Before**
```python
# loop through items and add up the price
total = 0
for item in items:
    total += item.price
```
**After**
```python
total = sum(item.price for item in items)
```

**A why-comment worth keeping**
```python
# The payment gateway rejects amounts with more than 2 decimals.
amount = round(amount, 2)
```

**Keep**: public API docstrings, license headers, TODOs that have context or a ticket link, explanations of regexes and tricky algorithms, links to specs.

---

## 3. Consistency (الاتساق)

**Idea:** the same thing is done the same way everywhere in the project.

**Smells**
- Mixed naming styles inside one layer (`snake_case` and `camelCase` together).
- Several ways to do the same task: three different ways to open the database, two error response shapes, mixed date formats.
- Different file and folder organization for similar features.
- Mixed formatting where no formatter is configured.

**How to fix**
1. Look at how the project does it now. Count which style is the majority.
2. State the dominant convention in one line in your report.
3. Align the minority to it. New code follows the surrounding code, not a generic preference.
4. Use the project's own formatter or linter if there is one. Do not introduce a new one.

**Before** (two error shapes in the same API)
```python
return {"error": "not found"}, 404
...
return {"success": False, "message": "invalid input"}, 400
```
**After**
```python
return {"error": "not found"}, 404
...
return {"error": "invalid input"}, 400
```
(Only if no client depends on the old shape. Otherwise report it and ask.)

**Don't**
- Mass-reformat files. Do style-only changes in a separate step.
- Push your preferred style when the project consistently uses another.

---

## 4. Early Return (الخروج المبكر)

**Idea:** handle the invalid or edge cases first and leave immediately. The main path then reads straight down without deep indentation.

**Smells**
- Nesting deeper than about 3 levels ("arrow code").
- An `if ok:` that wraps the whole function body.
- Long `else` blocks after a `return`, `raise`, or `continue`.

**How to fix**
- Turn each precondition into a guard clause: `if bad: return / raise / continue`.
- In loops use `continue` to skip the bad case.
- Put the happy path last, unindented.

**Before**
```python
def ship_order(order):
    if order is not None:
        if order.is_paid:
            if order.items:
                do_ship(order)
                return True
            else:
                return False
        else:
            return False
    else:
        return False
```
**After**
```python
def ship_order(order):
    if order is None or not order.is_paid or not order.items:
        return False
    do_ship(order)
    return True
```

**Don't**
- Force this when resource cleanup is needed. Use `with` or `try/finally` so cleanup still runs.
- Create so many return points that the function becomes hard to follow. Group related guards.
- Change what the function returns or raises for each case.
