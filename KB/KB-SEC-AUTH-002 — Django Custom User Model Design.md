# KB-SEC-AUTH-002 — Django Custom User Model Design

**Knowledge Base Category:** Security / Authentication
**System:** Django
**Applies To:** Projects using custom `AUTH_USER_MODEL`
**Severity:** Medium–High (Design-time risk)
**Status:** Active Reference

---

## 1. Purpose

This article defines **approved design patterns, anti-patterns, and engineering rules** for implementing a **custom Django User model**.

A poorly designed user model creates **irreversible technical debt** and can silently break:

* Authentication
* Admin tooling
* Third-party libraries (DRF, SimpleJWT, allauth)
* Migrations

---

## 2. When a Custom User Model Is Required

A custom user model is justified when **any** of the following are required:

* Email-based login (`USERNAME_FIELD = 'email'`)
* Additional identity fields (e.g. `full_name`, `phone`)
* Removal or modification of `username`
* Multi-tenant or external IdP integration

If none apply, **do not customize** — use Django’s default `User`.

---

## 3. Mandatory Design Rules

### Rule 1 — Define Early

```python
AUTH_USER_MODEL = 'userauths.User'
```

* Must be set **before first migration**
* Changing later requires database rebuild

---

### Rule 2 — Use Email Login Explicitly

```python
USERNAME_FIELD = 'email'
REQUIRED_FIELDS = ['username']
```

* All auth systems will authenticate using `email`
* Frontend and APIs must align

---

### Rule 3 — Always Provide a Custom UserManager

```python
class UserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        ...
```

This enforces:

* Password hashing
* Required fields
* Consistent user creation

---

## 4. Recommended Reference Implementation

```python
class User(AbstractUser):
    username = models.CharField(max_length=255, unique=True)
    email = models.EmailField(unique=True)
    full_name = models.CharField(max_length=255)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    objects = UserManager()
```

---

## 5. Common Anti-Patterns

### ❌ Anti-Pattern 1 — Removing `username` blindly

Breaks:

* Django admin
* Third-party auth packages

---

### ❌ Anti-Pattern 2 — Creating users via `.create()`

```python
User.objects.create(email=email, password=password)
```

Results in **plain-text passwords**.

---

### ❌ Anti-Pattern 3 — Overriding `save()` for auth logic

Authentication logic **must live in managers or services**, not `save()`.

---

## 6. Migration & Compatibility Notes

* Always regenerate migrations after model changes
* Verify compatibility with:

  * Django Admin
  * DRF serializers
  * JWT / OAuth libraries

---

## 7. Engineering Checklist

* [ ] `AUTH_USER_MODEL` set before migrations
* [ ] Custom `UserManager` implemented
* [ ] `USERNAME_FIELD` aligned with API
* [ ] Passwords only set via `set_password()`

---

## 8. Related KBs

* KB-SEC-AUTH-001 — Django Password Hashing & JWT Authentication Failure
* KB-SEC-AUTH-003 — DRF + SimpleJWT Internals

---

**Owner:** Engineering Architecture
**Last Reviewed:** 2025-12-23
