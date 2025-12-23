# KB-SEC-AUTH-005 — Password Hashing (PBKDF2)

**Knowledge Base Category:** Security / Cryptography
**System:** Django Authentication
**Severity:** High
**Status:** Active Reference

---

## 1. Purpose

This article explains **how Django hashes passwords**, why PBKDF2 is used, and how to configure it securely.

---

## 2. What Is PBKDF2

PBKDF2 (Password-Based Key Derivation Function 2):

* Uses hashing + salt
* Applies thousands of iterations
* Slows brute-force attacks

---

## 3. Django Default Configuration

```python
PASSWORD_HASHERS = [
  'django.contrib.auth.hashers.PBKDF2PasswordHasher',
]
```

* Salted
* Iteration-based
* Industry standard

---

## 4. Password Storage Format

```text
pbkdf2_sha256$iterations$salt$hash
```

Plain-text passwords are **never stored**.

---

## 5. Verification Process

```python
user.check_password(raw_password)
```

* Hashes input
* Compares hashes

---

## 6. Security Tuning

```python
PBKDF2PasswordHasher.iterations = 600_000
```

Increase over time as CPU improves.

---

## 7. Common Myths

* ❌ Encryption ≠ hashing
* ❌ Hashes cannot be reversed

---

## 8. Engineering Rules

* Never store passwords in plain text
* Never log password values
* Never transmit passwords unencrypted

---

## 9. Related KBs

* KB-SEC-AUTH-001 — Django Password Hashing & JWT Authentication Failure
* KB-SEC-AUTH-004 — Secure User Registration APIs

---

**Owner:** Security Engineering
**Last Reviewed:** 2025-12-23
