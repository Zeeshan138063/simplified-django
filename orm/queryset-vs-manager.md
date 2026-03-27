# Django ORM — QuerySet vs Manager

> **Analogy:** Manager = vending machine (entry point). QuerySet = items tray (what comes out — lazy, chainable, adjustable).

---

## Core Difference

| Aspect | Manager (`objects`) | QuerySet (`.filter(…)`) |
|---|---|---|
| Role | Entry point to the DB for a model | A lazy SQL query — chainable & evaluatable |
| Lives on | The model class — `Post.objects` | Returned by manager / other QuerySets |
| Lazy? | No — always available as class attribute | Yes — SQL runs only on iteration or slice |
| Chainable? | Not really — gives you a QuerySet | Yes — `.filter().exclude().order_by()` |
| Hits DB? | Only when QuerySet is evaluated | On evaluation — `list()`, iteration, `[0]` |

---

## Where Does `Post.active` Come From?

The name on the left of `=` is the accessor — Django has no opinion on it:

```python
class Post(models.Model):
    objects = models.Manager()   # → Post.objects  (default)
    active  = ActiveManager()    # → Post.active   (custom)
```

> ⚠️ Always define `objects = models.Manager()` **first** — Django uses the first manager internally (e.g. admin).

---

## Scenario 1 — Custom Manager Only (filtered default)

Use when you want a pre-applied `WHERE` clause on every query from that manager.

```python
class ActiveManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)

class Post(models.Model):
    objects = models.Manager()
    active  = ActiveManager()

Post.active.all()             # WHERE is_active = true
Post.active.filter(author=u)  # WHERE is_active = true AND author_id = <id>
```

`get_queryset()` is the *base* — every method (`.all()`, `.filter()`, `.order_by()`) builds on top of it.

---

## Scenario 2 — Custom QuerySet + `as_manager()` (chainable methods)

Covers **~80% of real-world cases**. Use when you want semantic, chainable query methods.

```python
class PostQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')

    def recent(self):
        return self.order_by('-created_at')

    def by_author(self, user):
        return self.filter(author=user)

class Post(models.Model):
    objects = PostQuerySet.as_manager()  # no Manager subclass needed

# All of these work:
Post.objects.published()
Post.objects.published().recent()           # ✅ chained
Post.objects.by_author(user).published()    # ✅ chained with arg
Post.objects.published().count()            # ✅ built-in on top of custom
```

`as_manager()` wraps your QuerySet in a Manager automatically.

---

## Scenario 3 — Both Together

When you need a filtered default AND chainable custom methods.

```python
class PostQuerySet(models.QuerySet):
    def published(self):
        return self.filter(status='published')

class PostManager(models.Manager):
    def get_queryset(self):
        return PostQuerySet(self.model, using=self._db).filter(is_active=True)

class Post(models.Model):
    objects = PostManager()

Post.objects.published()  # already filtered to active + chainable ✅
```

---

## The Chaining Trap — Methods on Manager

A common pattern that **breaks chaining**:

```python
class PostManager(models.Manager):
    def published(self):
        return self.filter(status='published')

    def recent(self):
        return self.order_by('-created_at')

Post.objects.published()           # ✅ works
Post.objects.published().recent()  # ❌ AttributeError
```

**Why it breaks:**

```
PostManager  → .published() → plain QuerySet  → .recent() ❌
PostQuerySet → .published() → PostQuerySet    → .recent() ✅
```

Only acceptable for terminal methods that return a value (not a QuerySet):

```python
class PostManager(models.Manager):
    def published_count(self):
        return self.filter(status='published').count()  # returns int, not QS

Post.objects.published_count()  # ✅ fine — nothing to chain
```

---

## When to Create What

| Goal | What to create |
|---|---|
| Just filter the default queryset | Manager only — override `get_queryset()` |
| Chainable custom methods (`.published()`) | QuerySet only — use `as_manager()` |
| Filtered default + chainable methods | Both — Manager overrides `get_queryset()` using custom QuerySet |

---

## Quick Decision Rule

```
Adding query methods      →  QuerySet + as_manager()
Changing default behavior →  Manager (override get_queryset)
Need both                 →  Combine them
```

---

*Notes from Django ORM learning session — March 2026*
