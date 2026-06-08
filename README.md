# Security Disclosure: Cypherock Web Application Vulnerabilities

> **Status: Pending Resolution via Cypherock Team**
>
> These findings have been reported to Cypherock's security team per their [responsible disclosure policy](https://www.cypherock.com/bug-bounty). Specific technical details, payloads, and exploitation steps are intentionally withheld until remediation is confirmed.

---

## Summary

During an authorized security assessment of `cypherock.com`, multiple vulnerabilities were identified in the web application's API layer. These findings collectively enable the enumeration of real customer identities — individuals who have purchased Cypherock hardware wallets and are confirmed cryptocurrency holders.

**No vulnerabilities were found in the X1 hardware wallet or X1 cards themselves.**

---

## Findings Overview

### CYPH-01: Customer Identity Disclosure (Unmasked)

| Field | Detail |
|-------|--------|
| **Severity** | High |
| **Type** | Information Disclosure |
| **CWE** | CWE-200: Exposure of Sensitive Information to an Unauthorized Actor |
| **Auth Required** | None |
| **Status** | Pending Resolution |

An unauthenticated API endpoint returns **complete, unmasked real names** of Cypherock customers along with associated user-generated content. No rate limiting is applied.

---

### CYPH-02: Customer Identity Disclosure (Partially Masked)

| Field | Detail |
|-------|--------|
| **Severity** | Medium |
| **Type** | Information Disclosure |
| **CWE** | CWE-200: Exposure of Sensitive Information to an Unauthorized Actor |
| **Auth Required** | None |
| **Status** | Pending Resolution |

A second unauthenticated API endpoint returns partially masked customer names and geographic locations. The masking implementation preserves the exact character length of the original name, significantly reducing the search space for de-anonymization. Each request returns a randomized subset from a pool of approximately 500-1000+ customer records.

Additionally, a subset of this same data is embedded in the client-side JavaScript of **every page** on the site (including error pages), serving as "social proof" purchase notifications.

---

### CYPH-03: User Email Enumeration Oracle

| Field | Detail |
|-------|--------|
| **Severity** | Medium |
| **Type** | Information Disclosure / Authentication Weakness |
| **CWE** | CWE-204: Observable Response Discrepancy |
| **Auth Required** | None |
| **Status** | Pending Resolution |

An unauthenticated API endpoint accepts an email address and returns a differential response indicating whether the email is registered in the Cypherock system. This allows bulk verification of email addresses against the Cypherock user base with no rate limiting.

---

### CYPH-04: Payment Provider Key Exposure

| Field | Detail |
|-------|--------|
| **Severity** | Low |
| **Type** | Sensitive Data Exposure |
| **CWE** | CWE-312: Cleartext Storage of Sensitive Information |
| **Auth Required** | None |
| **Status** | Pending Resolution |

A client-side JavaScript bundle contains a live payment provider API key. While this specific key type is designed for client-side use, its exposure alongside other findings enables construction of convincing phishing infrastructure.

---

### CYPH-05: Third-Party Service Identifier Exposure

| Field | Detail |
|-------|--------|
| **Severity** | Informational |
| **Type** | Information Disclosure |
| **Auth Required** | None |
| **Status** | Noted |

Multiple third-party analytics, session recording, and marketing service identifiers are embedded in the site source. These enable event injection and session replay manipulation.

---

## Combined Risk Assessment

These findings are most dangerous **in combination**:

```
CYPH-01 (full names) + CYPH-02 (names + countries) + CYPH-03 (email verification)
    = Complete identity records of confirmed cryptocurrency holders
        = High-value spear-phishing target list
            = Potential seed phrase harvesting
                = Funds loss
```

This attack chain falls within Cypherock's bounty scope item #6: *"Sensitive data leaks leading to funds loss."*

---

## What Is NOT Affected

- X1 Hardware Wallet firmware
- X1 Card cryptographic operations
- Seed phrase generation or storage
- Device-to-device communication protocols
- Shamir Secret Sharing implementation

All findings are limited to the **web application and its API layer**.

---

## Disclosure Timeline

| Date | Event |
|------|-------|
| 2026-06-XX | Vulnerabilities discovered during authorized assessment |
| 2026-06-XX | Initial report submitted to security@cypherock.com |
| — | Awaiting acknowledgment from Cypherock security team |
| — | 90-day disclosure window per Cypherock policy |

This disclosure follows Cypherock's [Bug Bounty Program](https://www.cypherock.com/bug-bounty) guidelines. Public technical details will not be released until:
1. Cypherock confirms remediation, **OR**
2. The 90-day disclosure window expires

---

## Researcher

J (@suffers)

---

> **Note:** If you are a Cypherock customer concerned about your data exposure, contact Cypherock support directly. Do not attempt to access, test, or exploit any of the endpoints referenced in this disclosure.
