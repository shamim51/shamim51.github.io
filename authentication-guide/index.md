# Sports 360 — Authentication Guide (OAuth 2.0 + PKCE)

[⬅️ Back to Index](../README.md)

This guide explains how frontend and mobile clients authenticate users through the Sports 360 backend using the **Authorization Code Flow with PKCE**. This is the only supported login mechanism.

---

## Before You Start — Finding Your Realm Name

The **realm name** identifies which club/tenant you are authenticating against. It is required on every auth call.

| Platform | Where to find it |
|---|---|
| **Web (frontend)** | It is the subdomain of the app URL. For example, `https://club-duhok-sc.getsport360.com` → realm is `club_duhok_sc` |
| **Mobile** | Obtain the realm name from DevOps / your technical contact. It will be a fixed string configured per app build (e.g. stored in your `.env` or build config). |

---

## Base URL

All Sports 360 API calls go to:

```
https://api.getsport360.com/api/v1/sports360
```
---

## Step 1 — Init: Fetch Auth Configuration

Before building the login URL, fetch the Keycloak configuration for your realm. This gives you the `clientId` and the exact endpoints to use.

### Request

```
GET /api/v1/sports360/auth/init?realm={realmName}
```

**Example:**

```
GET /api/v1/sports360/auth/init?realm=club_duhok_sc
```

No body. No auth header required.

### Response

```json
{
  "code": 200,
  "message": "Operation successful.",
  "data": {
    "clientId": "club-duhok_sc-app",
    "authorizationEndpoint": "https://dev-kc.getsport360.com/realms/club_duhok_sc/protocol/openid-connect/auth",
    "tokenEndpoint": "https://dev-kc.getsport360.com/realms/club_duhok_sc/protocol/openid-connect/token",
    "issuerUri": "https://dev-kc.getsport360.com/realms/club_duhok_sc"
  }
}
```

**Store these values** — you will need `clientId` and `authorizationEndpoint` in the next step.

---

## Step 2 — Generate PKCE Values & Build the Authorization URL

### 2a. Generate PKCE Values

You must generate a `code_verifier` and derive a `code_challenge` from it. **Do this on the client side — never send `code_verifier` to any server until Step 4.**

#### Rules for `code_verifier`
- Random URL-safe string
- **Length: 43–128 characters** (RFC 7636 — Keycloak enforces this strictly)
- Allowed characters: `A–Z`, `a–z`, `0–9`, `-`, `.`, `_`, `~`

#### Computing `code_challenge`

```
code_challenge = BASE64URL(SHA256(code_verifier))
```

> `BASE64URL` means standard Base64 with `+` → `-`, `/` → `_`, and `=` padding **removed**.

#### Code examples

**JavaScript / TypeScript**
```typescript
async function generatePKCE() {
  // Generate code_verifier (43-128 URL-safe chars)
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  const codeVerifier = btoa(String.fromCharCode(...array))
    .replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');

  // Derive code_challenge
  const encoder = new TextEncoder();
  const data = encoder.encode(codeVerifier);
  const digest = await crypto.subtle.digest('SHA-256', data);
  const codeChallenge = btoa(String.fromCharCode(...new Uint8Array(digest)))
    .replace(/\+/g, '-').replace(/\//g, '_').replace(/=/g, '');

  return { codeVerifier, codeChallenge };
}
```

**Dart / Flutter**
```dart
import 'dart:math';
import 'dart:convert';
import 'package:crypto/crypto.dart';

String generateCodeVerifier() {
  final random = Random.secure();
  final values = List<int>.generate(32, (_) => random.nextInt(256));
  return base64UrlEncode(values).replaceAll('=', '');
}

String generateCodeChallenge(String codeVerifier) {
  final bytes = utf8.encode(codeVerifier);
  final digest = sha256.convert(bytes);
  return base64UrlEncode(digest.bytes).replaceAll('=', '');
}
```

> **Important:** Store the `codeVerifier` in memory (or secure session storage). You will need the exact same string in Step 4.

---

### 2b. Build the Authorization URL

Use `authorizationEndpoint` and `clientId` from the Step 1 response.

```
{authorizationEndpoint}
  ?response_type=code
  &client_id={clientId}
  &redirect_uri={yourRedirectUri}
  &scope=openid profile email
  &code_challenge={codeChallenge}
  &code_challenge_method=S256
  &state={randomState}
```

**Important for mobile**: you should inform about your callback url to us, so that we can whitelist that callback url on keycloak. 

| Parameter | Description |
|---|---|
| `response_type` | Always `code` |
| `client_id` | From the `/auth/init` response |
| `redirect_uri` | The URL Keycloak will redirect back to after login. **Must be registered in Keycloak.** |
| `scope` | At minimum `openid`. Add `profile email` as needed. |
| `code_challenge` | Computed in Step 2a |
| `code_challenge_method` | Always `S256` |
| `state` | A random string you generate to prevent CSRF. Verify it matches when Keycloak redirects back. |

**Example URL:**

```
https://dev-kc.getsport360.com/realms/club_duhok_sc/protocol/openid-connect/auth
  ?response_type=code
  &client_id=club-duhok_sc-app
  &redirect_uri=https://club-duhok-sc.getsport360.com/callback
  &scope=openid profile email
  &code_challenge=NrvlDtloQdEEQ7y2cNZVTwo0t2G-Z-ycSorSwMRMpCw
  &code_challenge_method=S256
  &state=abc123xyz
```

**Open this URL in the browser.** The user will see the Keycloak login page for the club.

---

## Step 3 — Handle the Redirect (Receive the Authorization Code)

After the user logs in, Keycloak redirects back to your `redirect_uri` with query parameters:

```
https://your-app.com/callback?code=78f2affd-1572-...&state=abc123xyz&session_state=...
```

**What to do:**
1. Verify `state` matches the value you generated in Step 2b. If not, abort — possible CSRF attack.
2. Extract the `code` value.
3. Proceed to Step 4 immediately — **authorization codes expire in ~60 seconds and can only be used once.**

---

## Step 4 — Token Exchange: Get Access & Refresh Tokens

Send the authorization `code` and your `codeVerifier` to the Sports 360 backend. The backend will securely exchange it with Keycloak using the client secret.

### Request

```
POST /api/v1/sports360/auth/token
Content-Type: application/json
```

**Body:**

```json
{
  "realm": "club_duhok_sc",
  "code": "<authorization code from Step 3>",
  "redirectUri": "https://club-duhok-sc.getsport360.com/callback",
  "codeVerifier": "<the exact code_verifier string from Step 2a>"
}
```

> `redirectUri` must be **byte-for-byte identical** to what was used in Step 2b.
> `codeVerifier` must be **the exact same string** that was used to compute `code_challenge`.

### Response

```json
{
  "code": 200,
  "message": "Operation successful.",
  "data": {
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsIn...",
    "id_token": "eyJhbGciOiJSUzI1NiIsInR5...",
    "token_type": "Bearer",
    "expires_in": 300,
    "refresh_expires_in": 1800,
    "scope": "openid profile email",
    "session_state": "a1b2c3d4-..."
  }
}
```

| Field | Description |
|---|---|
| `access_token` | JWT — attach as `Authorization: Bearer <token>` on all protected API requests |
| `refresh_token` | Use to get a new `access_token` when it expires (see Step 5) |
| `expires_in` | Seconds until `access_token` expires (typically 300 = 5 min) |
| `refresh_expires_in` | Seconds until `refresh_token` expires (typically 1800 = 30 min) |

**Store the tokens securely:**
- Web: `sessionStorage` or an in-memory store (avoid `localStorage` for security)
- Mobile: Keychain (iOS) / EncryptedSharedPreferences (Android)

---

## Step 5 — Refresh Tokens

When the `access_token` expires, use the `refresh_token` to get a new one without forcing the user to log in again.

### Request

```
POST /api/v1/sports360/auth/refresh
Content-Type: application/json
```

**Body:**

```json
{
  "realm": "club_duhok_sc",
  "refreshToken": "<refresh_token from Step 4>"
}
```

### Response

Same structure as the Step 4 response — you receive a fresh `access_token` and `refresh_token`. **Replace both stored tokens with the new values.**

> If the `refresh_token` has also expired, the user must log in again from Step 2.

---

## Step 6 — Logout

To end the session, revoke the `refresh_token` on Keycloak.

### Request

```
POST /api/v1/sports360/auth/logout?realm={realmName}&refreshToken={refreshToken}
```

**Example:**

```
POST /api/v1/sports360/auth/logout?realm=club_duhok_sc&refreshToken=eyJhbGci...
```

No body required.

### Response

```json
{
  "code": 200,
  "message": "Operation successful.",
  "data": "Logged out successfully"
}
```

After logout, discard all stored tokens on the client side.

---

## Using the Access Token on API Calls

Attach the `access_token` as a Bearer token on every protected API request:

```
GET /api/v1/sports360/user/me
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5...
```

---

## Common Errors & How to Fix Them

| Error | Cause | Fix |
|---|---|---|
| `404 tenant.not_found` | Realm name doesn't exist in the system | Verify the realm name with DevOps |
| `403 error.forbidden` | Tenant account is inactive | Contact support |
| `invalid_grant` / PKCE verification failed | `codeVerifier` doesn't match the `code_challenge` sent to Keycloak, OR the verifier is too short (< 43 chars) | Ensure you use the same `codeVerifier` in Steps 2a and 4; ensure length ≥ 43 |
| `invalid_grant` / code expired | Authorization code was already used or took > ~60 seconds | Restart from Step 2 and complete Step 4 immediately |
| `redirect_uri_mismatch` | `redirectUri` in Step 4 doesn't exactly match what was sent in Step 2b | Use byte-for-byte identical URI including trailing slashes |
| `401 auth.failed` | General token exchange failure | Check logs; re-verify realm, code, and codeVerifier |

---

## Complete Flow Summary

```
1. GET  /auth/init?realm=club_duhok_sc
         → get clientId + authorizationEndpoint

2. Generate codeVerifier (43-128 chars) + codeChallenge = BASE64URL(SHA256(verifier))
   Open browser → {authorizationEndpoint}?response_type=code&client_id=...&code_challenge=...

3. Keycloak → redirects to your redirect_uri?code=...
   Extract the "code" value

4. POST /auth/token
   { realm, code, redirectUri, codeVerifier }
   → get access_token + refresh_token

5. Use access_token as  Authorization: Bearer <token>  on all API calls

6. When access_token expires → POST /auth/refresh { realm, refreshToken }
   → get new access_token + refresh_token

7. On logout → POST /auth/logout?realm=...&refreshToken=...
```
end....


##  Club Admin Onboarding Flow(for web team)

### State Machine

The frontend drives the multi-step onboarding wizard based on the `status` and `paymentStatus` fields returned by the API.

```
PENDING_OTP ? PENDING_INFO ? PENDING_PAYMENT ? PENDING_PROVISIONING ? PROVISIONED
```

| Status | UI Step | Next Action |
|---|---|---|
| `PENDING_OTP` | Email input + OTP entry | Call `request-otp`, then `verify-otp` |
| `PENDING_INFO` | Club info form (name, bio, colors, socials) | Call `PATCH basic-info` |
| `PENDING_PAYMENT` | Plan selection + payment redirect | Call `POST select-plan` ? redirect to FastPay |
| `PENDING_PROVISIONING` | Set password + upload logo/banner | Call `POST complete` (multipart) |
| `PROVISIONED` | Success ? show login URL | Redirect to club subdomain |

### Resume Logic

When a user returns to the onboarding page:
1. If they have a stored onboarding token, call `GET /onboarding/me`
2. The response contains the full `registration` state (drives frontend resume logic)
3. Jump directly to the UI step matching the current status
