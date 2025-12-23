# RB-SEC-AUTH-005 — OAuth / SSO Misconfiguration Incident Runbook

**Runbook Category:** Security Incident Response / Identity Federation
**Severity:** SEV-1 (Privilege Impact), SEV-2 (Exposure)

---

## 1. Purpose

This runbook handles incidents caused by **OAuth / SSO misconfiguration**, including improper scopes, trust boundaries, or redirect handling.

---

## 2. Detection Signals

* Users gaining unintended roles
* Tokens accepted from wrong issuer
* Redirect URI abuse
* Missing audience (`aud`) validation

---

## 3. Immediate Triage

* [ ] Identify IdP involved
* [ ] Identify affected applications
* [ ] Confirm misconfigured parameter

---

## 4. Immediate Containment

1. Disable affected OAuth client
2. Reject tokens from affected issuer
3. Restrict privileged scopes

---

## 5. Configuration Validation Checklist

* Validate `iss`, `aud`, `exp`
* Enforce exact redirect URIs
* Least-privilege scopes

---

## 6. Evidence Collection

* OAuth configuration snapshots
* Token samples (redacted)
* Role mappings

---

## 7. Remediation

* Correct OAuth configuration
* Rotate client secrets
* Force re-authentication

---

## 8. Prevention Controls

* Configuration-as-code
* Peer review for IdP changes
* Automated OAuth validation tests

---

## 9. Runbook Rule

> **Federated identity failures propagate fast — isolate trust boundaries immediately.**

---

**Owner:** Security Engineering
