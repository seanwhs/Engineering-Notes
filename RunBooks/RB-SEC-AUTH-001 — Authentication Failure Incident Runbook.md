# RB-SEC-AUTH-001 — Authentication Failure Incident Runbook

**Runbook Category:** Security / Authentication Incident Response
**Systems:** Django · DRF · SimpleJWT
**Severity Levels:** SEV-2 / SEV-1
**Audience:** On-call Engineers, SREs, Backend Engineers
**Last Reviewed:** 2025-12-23

---

## 1. Purpose

This runbook provides **step-by-step incident response procedures** for authentication failures in Django-based systems, specifically:

* Users unable to log in
* JWT token issuance failures
* Errors such as:

```json
{
  "detail": "No active account found with the given credentials"
}
```

---

## 2. Incident Classification

### SEV-1 (Critical)

* All users unable to authenticate
* Production login completely unavailable
* Revenue or core business impact

### SEV-2 (High)

* Subset of users affected
* New registrations cannot log in
* Admin users still able to authenticate

---

## 3. Immediate Triage Checklist (5 Minutes)

* [ ] Confirm environment (Prod / Staging / Dev)
* [ ] Identify affected user scope (all vs subset)
* [ ] Verify error message returned by auth endpoint
* [ ] Confirm admin login status

---

## 4. Fast Diagnosis Decision Tree

```
Can admin log in?
  │
  ├── NO → Global auth failure → Go to Section 8
  │
  └── YES
        │
        ├── New users only failing → Section 5
        │
        └── Random users failing → Section 6
```

---

## 5. Scenario A — New / Normal Users Cannot Authenticate

### Likely Causes

* Passwords stored in plain text
* User creation bypassed `set_password()`

### Verification

```python
u = User.objects.get(email=user_email)
u.check_password(provided_password)
```

* `False` → Password hashing failure confirmed

### Immediate Remediation

```python
u.set_password(provided_password)
u.save()
```

⚠️ Temporary fix only — proceed to **Post-Incident Actions**.

---

## 6. Scenario B — Some Existing Users Failing

### Checks

```python
u.is_active
```

* `False` → Reactivate user

```python
u.check_password(password)
```

* `False` → Reset password

---

## 7. Scenario C — JWT Endpoint Broken for All Users

### Checks

* Verify auth backend configuration
* Validate `AUTH_USER_MODEL`
* Check recent deployments or migrations

```python
AUTHENTICATION_BACKENDS
```

Rollback if necessary.

---

## 8. Global Authentication Failure (SEV-1)

### Immediate Actions

1. Disable affected deployment (rollback)
2. Switch traffic to known-good version
3. Notify stakeholders

### Do NOT

* Attempt mass password resets blindly
* Modify password hashers during incident

---

## 9. Evidence Collection

Capture the following:

* Failing request payload (redacted)
* User creation code path
* Deployment diff
* Database password format sample

---

## 10. Post-Incident Root Cause Analysis (RCA)

### Mandatory Questions

* How were affected users created?
* Was `create_user()` bypassed?
* Was a custom UserManager enforced?

---

## 11. Permanent Fixes (Must Be Actioned)

* Enforce custom `UserManager`
* Add registration API tests
* Add `check_password()` assertions in CI

---

## 12. Prevention Controls

* Code review rule: no direct `password=` assignment
* Unit tests for registration & login
* Monitoring: auth failure rate alerts

---

## 13. Related Knowledge Base

* KB-SEC-AUTH-001 — Django Password Hashing & JWT Authentication Failure
* KB-SEC-AUTH-002 — Django Custom User Model Design
* KB-SEC-AUTH-003 — DRF + SimpleJWT Internals
* KB-SEC-AUTH-004 — Secure User Registration APIs
* KB-SEC-AUTH-005 — Password Hashing (PBKDF2)

---

## 14. Runbook Rule

> **Never fix authentication incidents by bypassing password hashing or weakening security controls.**

---

**Owner:** Platform / Security Engineering
