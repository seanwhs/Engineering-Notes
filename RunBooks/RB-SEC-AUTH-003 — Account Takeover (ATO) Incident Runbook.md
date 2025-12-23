# RB-SEC-AUTH-003 — Account Takeover (ATO) Incident Runbook

**Runbook Category:** Security Incident Response / Authentication
**Severity:** SEV-1 (Critical)
**Audience:** Security, SRE, On-call Engineers

---

## 1. Purpose

This runbook defines the response to **confirmed or suspected account takeover (ATO)**, where an attacker gains unauthorized control of a legitimate user account.

---

## 2. Detection Signals

* Password changed without user consent
* MFA disabled unexpectedly
* Login from anomalous IP / geo
* Abnormal privileged actions
* Multiple failed login attempts followed by success

---

## 3. Immediate Triage (0–10 Minutes)

* [ ] Confirm affected account(s)
* [ ] Identify authentication vector (password, OAuth, JWT)
* [ ] Preserve authentication and audit logs
* [ ] Notify security incident commander

---

## 4. Immediate Containment (Mandatory)

1. **Disable affected account(s)**
2. Force logout across all sessions
3. Revoke all active tokens
4. Lock password changes temporarily

---

## 5. User Protection Actions

* Force password reset
* Re-enroll MFA
* Invalidate trusted devices
* Notify user with security guidance

---

## 6. Evidence Collection

* Login history (IP, device, time)
* Password / MFA changes
* API actions performed post-compromise

---

## 7. Root Cause Analysis (RCA)

Common causes:

* Password reuse
* Phishing
* Credential stuffing
* Weak MFA enforcement

---

## 8. Remediation & Prevention

* Enforce MFA for all users
* Enable anomaly-based login detection
* Educate users on phishing

---

## 9. Runbook Rule

> **User safety takes priority over forensic completeness. Disable first, investigate second.**

---

**Owner:** Security Engineering
