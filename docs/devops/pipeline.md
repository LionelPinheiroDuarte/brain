# Pipeline CI/CD

## CI — Continuous Integration

Continuous integration (CI) is the practice of automatically running a series of checks on every push. The goal is to catch regressions as early as possible, before they reach production.

A typical CI pipeline follows this order:

```
push → lint → unit tests → build → E2E tests → deploy
```

Each step blocks the next. If any step fails, the pipeline stops and the deployment never runs.

---

### E2E Tests

Unit and integration tests verify that the code works. E2E tests verify that the user can actually do what they're supposed to do — they test the delivered output, not the source code.

Problems only E2E tests catch:

- A link points to the wrong URL after a permalink change
- A page returns a directory listing instead of HTML
- The language switcher breaks after a refactor
- An element is hidden by CSS

**Place in the pipeline:** E2E tests run after the build, against the real deployed artifact. They are few in number and focused on critical user flows — navigation, key content, and core interactions.

```
        /\
       /E2E\        ← few, slow, critical paths only
      /------\
     /  Integr.\
    /------------\
   /  Unit tests  \
  /________________\
```
