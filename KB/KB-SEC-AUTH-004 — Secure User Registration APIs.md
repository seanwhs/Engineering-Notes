# KB-SEC-AUTH-004 — Secure User Registration APIs

**Knowledge Base Category:** Security / Authentication
**System:** Django, DRF
**Severity:** High
**Status:** Active Reference

---

## 1. Purpose

This article defines **secure patterns for implementing user registration APIs** in Django + DRF.

---

## 2. Threat Model

Registration endpoints are vulnerable to:

* Plain-text password storage
* Account enumeration
* Weak password policies
* Automated abuse

---

## 3. Mandatory Security Requirements

### Requirement 1 — Password Hashing

```python
user.set_password(password)
```

Never assign passwords directly.

---

### Requirement 2 — Serializer-Level Enforcement

```python
class RegisterSerializer(serializers.ModelSerializer):
    def create(self, validated_data):
        user = User(**validated_data)
        user.set_password(validated_data['password'])
        user.save()
        return user
```

---

### Requirement 3 — Input Validation

* Enforce password strength
* Validate email format
* Prevent duplicate emails

---

## 4. Recommended Registration Flow

```
Client
  │ POST /register/
  │ { email, password }
  ▼
DRF Serializer
  │
  │ set_password()
  ▼
User Created (Inactive)
  │
  ▼
Email Verification
```

---

## 5. Optional Hardening

* Email verification
* Rate limiting
* CAPTCHA
* Audit logging

---

## 6. Forbidden Practices

* Returning password fields
* Auto-activating privileged accounts
* Logging credentials

---

## 7. Engineering Rules

* Registration logic lives in serializers or services
* Passwords are never stored or logged in plain text

---

## 8. Related KBs

* KB-SEC-AUTH-001 — Django Password Hashing & JWT Authentication Failure
* KB-SEC-AUTH-005 — Password Hashing (PBKDF2)

---

**Owner:** Application Security
**Last Reviewed:** 2025-12-23
