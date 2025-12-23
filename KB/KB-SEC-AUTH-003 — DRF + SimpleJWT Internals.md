# KB-SEC-AUTH-003 — DRF + SimpleJWT Internals

**Knowledge Base Category:** Security / Authentication
**System:** Django REST Framework, SimpleJWT
**Severity:** Medium
**Status:** Active Reference

---

## 1. Purpose

This article explains **how DRF and SimpleJWT authenticate users internally**, to aid debugging and secure design.

---

## 2. TokenObtainPair Authentication Flow

```
Client
  │ POST /token/
  │ { email, password }
  ▼
TokenObtainPairSerializer
  │
  │ authenticate()
  ▼
Django Authentication Backend
  │
  │ check_password()
  ▼
User Authenticated?
  │
  ├── YES → Issue Refresh + Access Tokens
  └── NO  → Authentication Error
```

---

## 3. Critical Internals

### 3.1 `authenticate()`

```python
user = authenticate(
    request=request,
    email=email,
    password=password
)
```

* Delegates to Django auth backends
* Uses `USERNAME_FIELD`

---

### 3.2 Password Validation

```python
user.check_password(raw_password)
```

* Compares PBKDF2 hashes
* Plain-text passwords always fail

---

## 4. Common Failure Modes

| Symptom                 | Root Cause          |
| ----------------------- | ------------------- |
| Admin works, user fails | Password not hashed |
| Always fails            | Wrong login field   |
| Random failures         | Inactive users      |

---

## 5. Token Payload Customization

```python
token['email'] = user.email
token['full_name'] = user.full_name
```

Safe to include **non-sensitive** fields only.

---

## 6. Security Notes

* Never include secrets in JWT payload
* Access tokens must be short-lived
* Refresh tokens must be rotated

---

## 7. Debugging Checklist

* [ ] `check_password()` returns `True`
* [ ] Correct login field used
* [ ] User is active
* [ ] Correct auth backend configured

---

## 8. Related KBs

* KB-SEC-AUTH-001 — Django Password Hashing & JWT Authentication Failure
* KB-SEC-AUTH-002 — Django Custom User Model Design

---

**Owner:** Platform Security
**Last Reviewed:** 2025-12-23
