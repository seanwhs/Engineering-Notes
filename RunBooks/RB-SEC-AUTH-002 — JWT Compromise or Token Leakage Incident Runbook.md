# RB-SEC-AUTH-002 — JWT Compromise / Token Leakage Incident Runbook

**Runbook Category:** Security Incident Response / Authentication
**Systems:** Django · DRF · SimpleJWT · API Gateway / Frontend
**Severity Levels:** SEV-1 (Critical), SEV-2 (High)
**Audience:** On-call Engineers, Security Engineers, SREs
**Last Reviewed:** 2025-12-23

---

## 1. Purpose

This runbook defines **incident response procedures** for suspected or confirmed **JWT compromise or token leakage**, including:

* Stolen access or refresh tokens
* Tokens exposed via logs, browser storage, or client-side code
* Replay attacks using valid JWTs

---

## 2. Incident Classification

### SEV-1 — Confirmed Token Compromise

* Tokens used from unexpected IPs / geographies
* Privileged actions performed without user intent
* Refresh tokens leaked
* Evidence of automated replay

### SEV-2 — Suspected Token Exposure

* JWTs logged accidentally
* Tokens embedded in URLs
* Client-side storage concerns (localStorage exposure)

---

## 3. Immediate Triage (First 10 Minutes)

* [ ] Identify token type involved (Access vs Refresh)
* [ ] Identify affected user scope (single / multiple / global)
* [ ] Confirm environment (Prod / Staging)
* [ ] Preserve logs and evidence
* [ ] Notify security lead / incident commander

---

## 4. Decision Tree

```
Is refresh token compromised?
  │
  ├── YES → SEV-1 → Section 6
  │
  └── NO (Access token only)
        │
        ├── Short TTL? → Section 5
        └── Long TTL?  → Section 6
```

---

## 5. Scenario A — Access Token Exposure (Short-Lived)

### Characteristics

* Access token TTL ≤ 15 minutes
* No evidence of refresh token leakage

### Immediate Actions

1. Allow token to expire naturally
2. Force logout on frontend (if possible)
3. Monitor suspicious activity

### Notes

Access tokens are **stateless** and cannot be revoked individually without blacklist support.

---

## 6. Scenario B — Refresh Token or Long-Lived Token Compromise

### Immediate Actions (Mandatory)

1. **Rotate signing key immediately**
2. Invalidate all existing tokens
3. Force global logout
4. Disable sensitive endpoints temporarily

#### Django / SimpleJWT

```python
SIGNING_KEY = os.environ['NEW_JWT_SIGNING_KEY']
```

Redeploy application.

---

## 7. Token Revocation Strategy

### Option 1 — Signing Key Rotation (Global)

* Fastest containment
* Forces re-authentication for all users

### Option 2 — Blacklisting (If Enabled)

```python
INSTALLED_APPS += ['rest_framework_simplejwt.token_blacklist']
```

Use only if already deployed.

---

## 8. Evidence Collection

Collect and preserve:

* Sample compromised JWTs (redacted)
* Request logs with token usage
* IP addresses / user agents
* Token TTL and claims

---

## 9. User Impact Mitigation

* Notify affected users
* Force password reset if needed
* Revoke sessions across devices

---

## 10. Root Cause Analysis (RCA)

### Common Root Causes

* Tokens logged in plaintext
* Tokens stored in `localStorage`
* Excessive token lifetime
* Missing HTTPS

---

## 11. Permanent Remediation

* Shorten access token TTL
* Enable refresh token rotation
* Move tokens to secure HTTP-only cookies
* Add anomaly detection (IP / geo)

---

## 12. Prevention Controls

* Never log JWTs
* Enforce HTTPS everywhere
* Use `HttpOnly` + `Secure` cookies
* Set minimal token claims

---

## 13. Related Knowledge Base

* KB-SEC-AUTH-003 — DRF + SimpleJWT Internals
* KB-SEC-AUTH-005 — Password Hashing (PBKDF2)

---

## 14. Runbook Rule

> **JWT compromise is a containment problem first, a root-cause problem second. Rotate keys before investigating.**

---

**Owner:** Security Engineering
