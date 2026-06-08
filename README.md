# Vulnerability Report: Unauthenticated Customer Name Disclosure via /api/reviews/all

## Overview

| Field | Value |
|-------|-------|
| **Target** | https://www.cypherock.com |
| **Endpoint** | `GET /api/reviews/all` |
| **Severity** | High |
| **Category** | Sensitive data leaks leading to funds loss (Bounty Scope #6) |
| **Authentication Required** | None |
| **Rate Limiting** | None |
| **Status** | Active |

---

## Description

The Cypherock website exposes an API endpoint at `/api/reviews/all` that returns **fully unmasked customer names** along with their review text. Unlike the `/api/purchasers.json` endpoint which applies partial character masking, this endpoint returns **complete, unredacted real names** of Cypherock hardware wallet purchasers.

This is particularly dangerous because:
1. The names are **completely unmasked** — no character replacement
2. The data confirms these individuals are **cryptocurrency hardware wallet owners** (high-value targets)
3. No authentication, session, or CSRF token is required
4. No rate limiting is applied

---

## Reproduction Steps

### Step 1: Send GET Request

```http
GET /api/reviews/all HTTP/1.1
Host: www.cypherock.com
Accept: application/json
```

**cURL:**
```bash
curl -s "https://www.cypherock.com/api/reviews/all" \
  -H "Accept: application/json"
```

### Step 2: Receive Unmasked Customer Data

The response contains an array of review objects with **fully unmasked** customer names:

```json
[
  {
    "name": "[REDACTED — Full real name returned]",
    "review": "...",
    "rating": 5
  },
  ...
]
```

**Key difference from `/api/purchasers.json`:**

| Endpoint | Masking | Example |
|----------|---------|---------|
| `/api/purchasers.json` | Partial masking | `Da*** Bri****` |
| **`/api/reviews/all`** | **No masking** | **`David Britten`** (full name) |

---

## Impact

### Direct Impact
- **Full real names** of confirmed cryptocurrency hardware wallet purchasers are exposed
- **Review content** may reveal additional personal context (use case, location, wallet type)
- Data is accessible to any unauthenticated user or automated scraper

### Attack Chain → Funds Loss
1. **Enumerate** all customer names via `/api/reviews/all` (full names, no guessing needed)
2. **Cross-reference** with social media, LinkedIn, public records to build target profiles
3. **Launch targeted spear-phishing**: "Dear [Full Name], your Cypherock X1 requires an urgent firmware update. Please connect your device and enter your recovery phrase at [malicious-url]"
4. **Harvest seed phrases** → drain cryptocurrency wallets → **funds loss**

Hardware wallet customers are **uniquely high-value targets** because:
- They are confirmed cryptocurrency holders
- They likely hold significant amounts (invested in premium hardware security)
- They trust the Cypherock brand — a phishing email referencing their exact product is highly convincing

### Amplification with Other Findings
- Combined with `/api/purchasers.json` (which includes country data), attackers can geo-locate victims
- Combined with `/api/affiliates/check-email`, attackers can verify victim email addresses
- Combined with the exposed Stripe publishable key, attackers can build convincing fake checkout/refund pages

---

## Affected Data

| Data Field | Exposed | Masked |
|------------|---------|--------|
| Customer first name | Yes | **No** |
| Customer last name | Yes | **No** |
| Review text | Yes | N/A |
| Star rating | Yes | N/A |

---

## Recommended Remediation

1. **Immediate**: Remove the `/api/reviews/all` endpoint or require authentication
2. **If reviews must be public**: Apply consistent masking (e.g., `D**** B******`) — matching `/api/purchasers.json` behavior at minimum
3. **Better**: Use pseudonyms or initials only (e.g., "D.B." or "Verified Buyer")
4. **Add rate limiting** to all `/api/*` endpoints
5. **Add Cloudflare WAF rules** to block automated scraping of API endpoints

---

## Bounty Eligibility

This finding falls under Cypherock Bug Bounty scope item **#6: "Sensitive data leaks leading to funds loss"**:

- **Sensitive data**: Full real names of confirmed cryptocurrency holders — YES
- **Leading to funds loss**: Enables targeted phishing campaigns against identified wallet owners — YES
- **Ease of exploit**: Single unauthenticated GET request — TRIVIAL
- **Risk to users**: Confirmed crypto holders become spear-phishing targets — CRITICAL

---

## Timeline

| Date | Event |
|------|-------|
| [DATE] | Vulnerability discovered during authorized security assessment |
| [DATE] | Report submitted to security@cypherock.com |
| | Awaiting acknowledgment |
