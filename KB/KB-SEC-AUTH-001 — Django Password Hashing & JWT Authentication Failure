# KB-SEC-AUTH-001 — Django Password Hashing & JWT Authentication Failure

**Knowledge Base Category:** Security / Authentication
**System:** Django · DRF · SimpleJWT
**Applies To:** Custom User Models (`USERNAME_FIELD = 'email'`)
**Severity:** High (Authentication Blocking)
**Status:** Resolved

---

## 1. Overview

This knowledge base article documents a **critical authentication failure pattern** encountered when using:

* Custom Django `User` models
* Email-based authentication (`USERNAME_FIELD = 'email'`)
* Django REST Framework (DRF)
* SimpleJWT (`TokenObtainPairView`)

The issue results in valid users being **unable to obtain JWT tokens**, despite appearing correctly configured.

---

## 2. Problem Statement

### Observed Error

```json
{
  "detail": "No active account found with the given credentials"
}
```

### Impact

* Normal users cannot authenticate
* JWT token issuance fails
* Application login functionality is blocked

### Notable Symptom

* ✅ Superuser authentication succeeds
* ❌ Non-admin users consistently fail

---

## 3. System Context

### Custom User Model (Relevant Configuration)

```python
class User(AbstractUser):
    email = models.EmailField(unique=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']
```

### Authentication Mechanism

* JWT tokens issued via `TokenObtainPairView`
* Credentials used: `email + password`

---

## 4. Root Cause Analysis

### Root Cause

Passwords for affected users were **stored in plain text** rather than hashed.

This occurred because users were created using direct model instantiation or `.create()` without invoking Django’s password hashing mechanism.

### Incorrect User Creation Patterns

```python
User.objects.create(email='user@email.com', password='user')
```

```python
u = User(email='user@email.com', password='user')
u.save()
```

⚠️ Django does **not** hash passwords automatically.

---

## 5. Why Superusers Were Unaffected

Superusers were created using:

```bash
python manage.py createsuperuser
```

This internally calls:

```python
user.set_password(password)
```

Result:

* Password hashed using PBKDF2
* `check_password()` succeeds
* JWT authentication succeeds

---

## 6. Diagnosis Procedure

### Step 1 — Verify User Status

```python
u = User.objects.get(email='user@email.com')
u.is_active
```

Expected result:

```text
True
```

---

### Step 2 — Verify Password Hashing

```python
u.check_password('user')
```

| Result  | Interpretation              |
| ------- | --------------------------- |
| `True`  | Password correctly hashed   |
| `False` | Password stored incorrectly |

---

## 7. Immediate Remediation

### Applied Fix

```python
u = User.objects.get(email='user@email.com')
u.set_password('user')
u.save()
```

### Validation

```python
u.check_password('user')
# True
```

JWT token issuance succeeds immediately after remediation.

---

## 8. Permanent Preventive Measures

### 8.1 Approved User Creation Methods

#### ✅ Recommended

```python
User.objects.create_user(
    email='user@email.com',
    password='user'
)
```

#### ✅ Acceptable (Manual)

```python
u = User(email='user@email.com')
u.set_password('user')
u.save()
```

---

### 8.2 Forbidden User Creation Pattern

```python
User.objects.create(email=email, password=password)
```

❌ **Never allowed in production code**

---

## 9. Enforced Best Practice — Custom UserManager

To prevent recurrence, enforce password hashing at the model level.

```python
from django.contrib.auth.models import BaseUserManager

class UserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('Email is required')

        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        return self.create_user(email, password, **extra_fields)
```

Attach to the User model:

```python
class User(AbstractUser):
    objects = UserManager()
```

---

## 10. Authentication Flow (Reference)

```
Client
  │
  │ POST /api/v1/user/token/
  │  { email, password }
  ▼
SimpleJWT Serializer
  │
  │ authenticate(email, password)
  ▼
Django Auth Backend
  │
  │ check_password()
  ▼
PBKDF2 Hash Match?
  │
  ├── YES → Issue Access + Refresh Token
  └── NO  → Authentication Failure
```

---

## 11. Engineering Rules

* All Django users **MUST** be created via `create_user()` or `set_password()`
* Direct assignment to the `password` field is **strictly forbidden**
* Authentication failures must always validate `check_password()`

---

## 12. Related Knowledge Base Articles

* KB-SEC-AUTH-002 — Django Custom User Model Design
* KB-SEC-AUTH-003 — DRF + SimpleJWT Internals
* KB-SEC-AUTH-004 — Secure User Registration APIs
* KB-SEC-AUTH-005 — Password Hashing (PBKDF2)

---

**Owner:** Engineering
**Last Reviewed:** 2025-12-23
**Next Review:** On Auth Stack Change
