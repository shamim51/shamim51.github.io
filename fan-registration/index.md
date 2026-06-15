# Fan Registration/Signup API Guide

This guide documents the **Fan Registration** flow — how web and mobile clients register a new club fan (USER role) via OTP verification and Keycloak account creation.
All endpoints live under:

```
/api/v1/sports360/public/users
```

---

## Overview

Fan registration is a **3-step, OTP-based** flow:

```
Client                          Sports 360 Backend              Keycloak / Email
  |                                    |                              |
  |-- [1] POST /request-otp ---------->|-- send OTP email ------------>|
  |<-- 200 OK -------------------------|                              |
  |                                    |                              |
  |-- [2] POST /verify-otp ----------->|                              |
  |<-- onboardingToken + state --------|                              |
  |                                    |                              |
  |-- [3] POST /register -------------->|-- create user + assign role ->|
  |   (Bearer onboardingToken)         |-- create local user record   |
  |<-- 200 OK -------------------------|                              |
  |                                    |                              |
  |-- [4] Login via PKCE (see auth guide) ---------------------------->|
```

## After step 3, the user can log in using the standard OAuth 2.0 + PKCE flow documented in `[auth-pkce-flow-guide]`

## Tenant Resolution

Every endpoint requires the backend to know **which club (tenant)** the fan is registering for. Resolution order:

1. `**X-Tenant-ID` header** (preferred for mobile)
2. **Subdomain** from the request host (preferred for web)


| Platform   | How to identify tenant                | Example                                                                             |
| ---------- | ------------------------------------- | ----------------------------------------------------------------------------------- |
| **Web**    | Subdomain of the portal URL           | `https://club_duhok_sc.getsport360.com/...` → tenant slug club_duhok_sc             |
| **Mobile** | `X-Tenant-ID` header on every request | `X-Tenant-ID:` club_duhok_sc or `X-Tenant-ID: 550e8400-e29b-41d4-a716-446655440000` |


### Header format

```
X-Tenant-ID: {tenant-slug}
```

Accepted values:

- **Tenant UUID** — e.g. `550e8400-e29b-41d4-a716-446655440000` 
- **Domain slug** — e.g. `club_duhok_sc` 
If neither the header nor subdomain resolves a tenant, the API returns **400 Bad Request**.

---

## Base URL

```
https://api.getsport360.com/api/v1/sports360
```

---

## Step 1 — Request OTP

Sends a 6-digit OTP to the user's email. Creates or updates a pending registration record for the tenant.

### Request

```
POST /public/users/request-otp
Content-Type: application/json
X-Tenant-ID: club_duhok_sc          # required on mobile; optional on web if subdomain is present
```

**Body:**

```json
{
  "email": "fan@example.com"
}
```


| Field   | Type   | Required | Validation         |
| ------- | ------ | -------- | ------------------ |
| `email` | string | yes      | Valid email format |


### Success Response — 200 OK

```json
{
  "code": 200,
  "message": "OTP sent to fan@example.com",
  "data": "OTP sent to fan@example.com"
}
```

### Error Responses


| HTTP | Message key                                    | When                                                  |
| ---- | ---------------------------------------------- | ----------------------------------------------------- |
| 400  | `error.bad_request`                            | Tenant could not be resolved from header or subdomain |
| 400  | `error.email_required` / `error.email_invalid` | Missing or invalid email                              |
| 403  | `error.forbidden`                              | Tenant is inactive                                    |
| 404  | `tenant.not_found`                             | Tenant slug/UUID not found                            |
| 409  | `error.user_already_exists_in_tenant`          | Email already registered in this tenant               |


### Notes

- OTP expires after **10 minutes**.
- Pending registration expires after **7 days**.
- In development, otp is 123456

---

## Step 2 — Verify OTP

Validates the OTP and returns a short-lived **onboarding token** used to complete registration.

### Request

```
POST /public/users/verify-otp
Content-Type: application/json
X-Tenant-ID: club_duhok_sc
```

**Body:**

```json
{
  "email": "fan@example.com",
  "otp": "654321"
}
```


| Field   | Type   | Required | Validation                       |
| ------- | ------ | -------- | -------------------------------- |
| `email` | string | yes      | Must match the email from step 1 |
| `otp`   | string | yes      | Exactly 6 characters             |


### Success Response — 200 OK

```json
{
  "code": 200,
  "message": "OTP verified successfully",
  "data": {
    "onboardingToken": "eyJhbGciOiJIUzI1NiJ9...",
    "registration": {
      "email": "fan@example.com",
      "status": "PENDING_OTP",
      "otpVerified": true,
      "adminName": null,
      "clubName": null,
      "bio": null,
      "primaryColor": null,
      "secondaryColor": null,
      "websiteUrl": null,
      "facebookUrl": null,
      "instagramUrl": null,
      "youtubeUrl": null,
      "subscriptionPlanId": null,
      "paymentStatus": "PAID",
      "paymentRef": null,
      "expiresAt": "2026-06-21T10:00:00Z"
    }
  }
}
```

**Store `onboardingToken`** — it is required for step 3.

### Error Responses


| HTTP | Message key                               | When                                            |
| ---- | ----------------------------------------- | ----------------------------------------------- |
| 400  | `error.otp_expired`                       | OTP past 10-minute window                       |
| 400  | `error.otp_invalid`                       | Wrong OTP                                       |
| 400  | `error.otp_required` / `error.otp_length` | Missing or wrong-length OTP                     |
| 404  | `error.registration_not_found`            | No pending registration for this email + tenant |


### Onboarding token details

- Algorithm: **HS256** (custom JWT, not a Keycloak token)
- Validity: **24 hours**
- Purpose claim: `user_registration`
- Subject: the verified email address

---

## Step 3 — Register User

Creates the fan account in Keycloak and the local database. Assigns the **USER** role and a **FREE club membership** (if configured for the tenant).

### Request

```
POST /public/users/register
Authorization: Bearer {onboardingToken}
Content-Type: application/json
X-Tenant-ID: club_duhok_sc
```

**Body:**

```json
{
  "name": "Ahmed Fan",
  "password": "F@nP@ss123"
}
```


| Field      | Type   | Required | Validation                                   |
| ---------- | ------ | -------- | -------------------------------------------- |
| `name`     | string | yes      | 2–100 characters                             |
| `password` | string | yes      | Non-blank (Keycloak password policy applies) |


### Authentication

This endpoint requires the **onboarding token** from step 2:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

The email for registration is taken from the token's `sub` claim — **do not send email in the body**.

### Success Response — 200 OK

```json
{
  "code": 200,
  "message": "User registered successfully",
  "data": "User registered successfully"
}
```

### What happens server-side

1. Validates onboarding token and OTP-verified pending registration
2. Creates user in the tenant's Keycloak realm (`club_{slug}`)
3. Assigns `USER` realm role
4. Creates local `UserEntity` record
5. Assigns FREE club membership subscription (30-day renewal) if available
6. Deletes the pending registration record

### Error Responses


| HTTP | Message key                    | When                                          |
| ---- | ------------------------------ | --------------------------------------------- |
| 401  | —                              | Missing, invalid, or expired onboarding token |
| 403  | `error.otp_not_verified`       | OTP step was skipped                          |
| 404  | `error.registration_not_found` | No pending registration                       |
| 404  | `tenant.not_found`             | Tenant no longer exists                       |
| 500  | `error.provisioning_failed`    | Tenant Keycloak realm not configured          |


---

## Step 4 — Login (Post-Registration)

Registration does **not** return Keycloak tokens. After step 3, authenticate using the standard PKCE flow:

## See `[auth-pkce-flow-guide]` for full details.

---

## Platform Integration Examples

### Web (subdomain-based)

The browser is already on the club portal subdomain, so no header is needed:

```javascript
// Step 1 — tenant resolved from hostname automatically
await fetch('https://duhok-sc.getsport360.com/api/v1/sports360/public/users/request-otp', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email: 'fan@example.com' }),
});
```

#### In devlopment/localhost fortend may use header as mobile platform.

### Mobile (header-based)

Mobile apps call the shared API host and must send `X-Tenant-ID` on **every step**:

```javascript
const TENANT = 'club_duhok_sc'; // from app config / deep link / club selection screen
const API = 'https://api.getsport360.com/api/v1/sports360';
const headers = {
  'Content-Type': 'application/json',
  'X-Tenant-ID': TENANT,
};
// Step 1
await fetch(`${API}/public/users/request-otp`, {
  method: 'POST',
  headers,
  body: JSON.stringify({ email: 'fan@example.com' }),
});
// Step 2
const verifyRes = await fetch(`${API}/public/users/verify-otp`, {
  method: 'POST',
  headers,
  body: JSON.stringify({ email: 'fan@example.com', otp: '654321' }),
});
const { data: { onboardingToken } } = await verifyRes.json();

// Step 3
await fetch(`${API}/public/users/register`, {
  method: 'POST',
  headers: {
    ...headers,
    Authorization: `Bearer ${onboardingToken}`,
  },
  body: JSON.stringify({ name: 'Ahmed Fan', password: 'F@nP@ss123' }),
});
```

---

## Response Envelope

All endpoints return the standard `ApiResponse<T>` wrapper:

```json
{
  "code": 200,
  "message": "Human-readable message",
  "data": { }
}
```

## Errors follow the same shape with an appropriate HTTP status code and `data: null`.

## Security Notes

- Steps 1 and 2 are **public** (no auth required), but tenant must be resolved.
- Step 3 requires the **onboarding JWT** — it cannot be replaced with a Keycloak access token.
- The onboarding token is scoped to the verified email; the register endpoint uses that email automatically.
- Fan registration does **not** require payment (unlike club admin onboarding).
- `TenantFilter` skips `/public/users/`* paths — tenant resolution is handled inside the controller via `TenantResolverService`.

---

## Quick Reference


| Step | Method     | Path                        | Auth                    | Tenant                 |
| ---- | ---------- | --------------------------- | ----------------------- | ---------------------- |
| 1    | `POST`     | `/public/users/request-otp` | None                    | Header or subdomain    |
| 2    | `POST`     | `/public/users/verify-otp`  | None                    | Header or subdomain    |
| 3    | `POST`     | `/public/users/register`    | Bearer onboarding token | Header or subdomain    |
| 4    | PKCE login | `/auth/init`, `/auth/token` | —                       | Realm from club config |


