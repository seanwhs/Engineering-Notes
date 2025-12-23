# RB-SEC-AUTH-004 — Credential Stuffing / Brute Force Attack Runbook

**Runbook Category:** Security Incident Response / Authentication
**Severity:** SEV-1 (Active Attack), SEV-2 (Attempted)

---

## 1. Purpose

This runbook covers **automated login abuse**, including credential stuffing and brute-force attacks.

---

## 2. Detection Signals

* High volume login attempts
* Repeated failures across many accounts
* Known bot user agents
* Spikes from single IP / ASN

---

## 3. Immediate Triage

* [ ] Confirm automation (not user error)
* [ ] Identify targeted endpoints
* [ ] Assess blast radius

---

## 4. Immediate Containment

1. Enable aggressive rate limiting
2. Block offending IPs / ASNs
3. Enable CAPTCHA / challenge-response
4. Temporarily lock targeted accounts

---

## 5. Defensive Configuration

* Enforce MFA
* Reduce login attempts per IP
* Add exponential backoff

---

## 6. Evidence Collection

* Failed login logs
* IP addresses and geolocation
* Targeted usernames

---

## 7. Remediation

* Force password resets for impacted users
* Monitor for delayed success attempts

---

## 8. Prevention Controls

* Bot detection
* Password breach detection
* Strong password policy

---

## 9. Runbook Rule

> **Automated abuse requires automation in defense — block fast, tune later.**

---

**Owner:** Security Engineering
