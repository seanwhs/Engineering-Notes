# RB-SEC-AUTH-006 — Privilege Escalation via JWT Claims Runbook

**Runbook Category:** Security Incident Response / Authorization
**Severity:** SEV-1 (Critical)

---

## 1. Purpose

This runbook addresses **privilege escalation** caused by improper trust in **JWT claims**.

---

## 2. Detection Signals

* Users accessing admin endpoints
* JWTs containing unexpected roles
* Claims modified client-side
* Missing server-side authorization checks

---

## 3. Immediate Triage

* [ ] Identify abused claim (`role`, `is_admin`, `scope`)
* [ ] Identify affected endpoints
* [ ] Assess blast radius

---

## 4. Immediate Containment

1. Disable affected endpoints
2. Rotate JWT signing key
3. Invalidate all tokens

---

## 5. Code Review Focus Areas

* Trust boundaries between JWT and DB
* Authorization decorators
* Role validation logic

---

## 6. Evidence Collection

* Malicious JWT samples
* API access logs
* Authorization decisions

---

## 7. Remediation

* Move authorization to server-side checks
* Minimize JWT claims
* Enforce allowlists

---

## 8. Prevention Controls

* Centralized authorization layer
* Claim validation tests
* Security reviews for auth changes

---

## 9. Runbook Rule

> **JWTs assert identity, not authority. Authority lives on the server.**

---

**Owner:** Security Engineering
