# simplified-django

> Django concepts explained simply — with analogies, clean code snippets, and practical examples. No fluff, just clarity.

---

## Why This Exists

Django's documentation is thorough — but not always simple.

This repo is my personal learning notes, written the way I wish the docs were written: **start with an analogy, show a real example, explain the gotcha.**

Every note follows the same structure:
- One analogy to build the mental model
- Clean code showing the pattern
- Common mistakes and why they happen
- A quick decision rule at the end

---

## Contents

### ORM
| Topic | File |
|---|---|
| QuerySet vs Manager — when to use each, chaining, custom methods | [`orm/queryset-vs-manager.md`](orm/queryset-vs-manager.md) |

*More topics coming as I learn — views, signals, middleware, authentication, and more.*

---

## Structure

```
simplified-django/
├── orm/
│   └── queryset-vs-manager.md
├── views/
├── signals/
├── middleware/
├── authentication/
└── README.md
```

---

## How to Use

Just browse the topic you're stuck on. Each file is self-contained — no need to read in order.

If you're new to a concept, read the analogy first. If you're debugging something, jump straight to the **common mistakes** section.

---

## Contributing

Found an error or have a simpler way to explain something? PRs are welcome.

---

*Built while learning Django in depth — March 2026*
