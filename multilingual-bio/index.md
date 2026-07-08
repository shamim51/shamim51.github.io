# Frontend Guide — Multilingual Club/Tenant Bio API Changes

## Change summary

| Before | After |
|--------|-------|
| `bio` | `bioEn` (English — stored in existing DB column `bio`) |
| — | `bioAr` (Arabic — new) |
| — | `bioKu` (Kurdish — new) |

All three fields are optional, max **1000** characters each.

**Not affected:** user profile APIs (`/api/v1/sports360/user/*`) still use `bio`.

All responses use the standard envelope:

```json
{
  "code": 200,
  "message": "...",
  "data": { }
}
```

---

## Shared types

### `RegistrationStateDto` (onboarding responses)

```json
{
  "email": "admin@club.com",
  "status": "PENDING_INFO",
  "otpVerified": true,
  "adminName": "John Doe",
  "phone": "+9647700000000",
  "clubName": "Example FC",
  "teamId": 123,
  "logo": "https://...",
  "banner": "https://...",
  "bioEn": "Professional football club founded in 1990.",
  "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
  "bioKu": "یانەی تۆپی پێ...",
  "primaryColor": "#279846",
  "secondaryColor": "#FFFFFF",
  "websiteUrl": "https://example.com",
  "adminDomain": null,
  "userDomain": null,
  "domainProvisioned": false,
  "facebookUrl": "https://facebook.com/example",
  "instagramUrl": null,
  "youtubeUrl": null,
  "subscriptionPlanId": null,
  "paymentStatus": "UNPAID",
  "paymentRef": null,
  "expiresAt": "2026-08-07T08:00:00Z"
}
```

### `ClubBrandingResponse` (tenant admin GET/PUT)

```json
{
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Example FC",
  "domain": "example.getsport360.com",
  "adminDomain": "admin-example.getsport360.com",
  "logo": "https://...",
  "banner": "https://...",
  "primaryColor": "#279846",
  "secondaryColor": "#FFFFFF",
  "bioEn": "Professional football club founded in 1990.",
  "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
  "bioKu": "یانەی تۆپی پێ...",
  "websiteUrl": "https://example.com",
  "facebookUrl": null,
  "instagramUrl": null,
  "youtubeUrl": null,
  "subscriptionPlanName": "FREE",
  "subscriptionPlanId": "660e8400-e29b-41d4-a716-446655440001",
  "subscriptionEndDate": "2027-01-01T23:59:59Z",
  "isActive": true
}
```

### `TenantPublicResponseDto` (public branding)

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "teamId": 123,
  "name": "Example FC",
  "domain": "example.getsport360.com",
  "logo": "https://...",
  "banner": "https://...",
  "primaryColor": "#279846",
  "secondaryColor": "#FFFFFF",
  "bioEn": "Professional football club founded in 1990.",
  "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
  "bioKu": "یانەی تۆپی پێ...",
  "websiteUrl": "https://example.com",
  "facebookUrl": null,
  "instagramUrl": null,
  "youtubeUrl": null
}
```

### `TenantResponse` (super admin)

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Example FC",
  "domain": "example.getsport360.com",
  "realmName": "example-fc",
  "active": true,
  "logo": "https://...",
  "banner": "https://...",
  "primaryColor": "#279846",
  "secondaryColor": "#FFFFFF",
  "bioEn": "Professional football club founded in 1990.",
  "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
  "bioKu": "یانەی تۆپی پێ...",
  "websiteUrl": "https://example.com",
  "facebookUrl": null,
  "instagramUrl": null,
  "youtubeUrl": null,
  "createdAt": "2026-01-01T00:00:00Z",
  "updatedAt": "2026-07-08T00:00:00Z",
  "subscriptionPlan": { },
  "subscriptionStartDate": "2026-01-01T00:00:00Z",
  "subscriptionEndDate": "2027-01-01T23:59:59Z",
  "teamId": 123
}
```

---

## Affected APIs

### 1. Verify OTP (club onboarding)

**Response only** — bio fields are inside `data.registration`.

```http
POST /api/v1/sports360/public/onboarding/verify-otp
Content-Type: application/json
```

**Request**

```json
{
  "email": "admin@club.com",
  "otp": "123456"
}
```

**Response**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "onboardingToken": "eyJ...",
    "registration": {
      "email": "admin@club.com",
      "status": "PENDING_INFO",
      "otpVerified": true,
      "adminName": null,
      "phone": null,
      "clubName": null,
      "teamId": null,
      "logo": null,
      "banner": null,
      "bioEn": null,
      "bioAr": null,
      "bioKu": null,
      "primaryColor": null,
      "secondaryColor": null,
      "websiteUrl": null,
      "adminDomain": null,
      "userDomain": null,
      "domainProvisioned": false,
      "facebookUrl": null,
      "instagramUrl": null,
      "youtubeUrl": null,
      "subscriptionPlanId": null,
      "paymentStatus": "UNPAID",
      "paymentRef": null,
      "expiresAt": "2026-08-07T08:00:00Z"
    }
  }
}
```

---

### 2. Get registration state

**Response only**

```http
GET /api/v1/sports360/onboarding/me
Authorization: Bearer {onboardingToken}
```

**Response**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "email": "admin@club.com",
    "status": "PENDING_INFO",
    "otpVerified": true,
    "adminName": "John Doe",
    "phone": "+9647700000000",
    "clubName": "Example FC",
    "teamId": 123,
    "logo": "https://...",
    "banner": "https://...",
    "bioEn": "Professional football club founded in 1990.",
    "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
    "bioKu": "یانەی تۆپی پێ...",
    "primaryColor": "#279846",
    "secondaryColor": "#FFFFFF",
    "websiteUrl": "https://example.com",
    "adminDomain": null,
    "userDomain": null,
    "domainProvisioned": false,
    "facebookUrl": null,
    "instagramUrl": null,
    "youtubeUrl": null,
    "subscriptionPlanId": null,
    "paymentStatus": "UNPAID",
    "paymentRef": null,
    "expiresAt": "2026-08-07T08:00:00Z"
  }
}
```

---

### 3. Update basic info (club onboarding)

**Request + response**

```http
PATCH /api/v1/sports360/onboarding/basic-info
Authorization: Bearer {onboardingToken}
Content-Type: multipart/form-data
```

**Request (form fields)**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `adminName` | string | Yes | 2–100 chars |
| `phone` | string | Yes | |
| `teamId` | number | Yes | |
| `clubName` | string | No | |
| `bioEn` | string | No | max 1000 — **replaces old `bio`** |
| `bioAr` | string | No | max 1000 |
| `bioKu` | string | No | max 1000 |
| `primaryColor` | string | No | `#RGB` or `#RRGGBB` |
| `secondaryColor` | string | No | `#RGB` or `#RRGGBB` |
| `websiteUrl` | string | No | |
| `facebookUrl` | string | No | |
| `instagramUrl` | string | No | |
| `youtubeUrl` | string | No | |
| `subscriptionPlanId` | string | No | UUID |
| `logo` | file | No | |
| `banner` | file | No | |

**Example (FormData)**

```javascript
const formData = new FormData();
formData.append('adminName', 'John Doe');
formData.append('phone', '+9647700000000');
formData.append('teamId', '123');
formData.append('bioEn', 'Professional football club founded in 1990.');
formData.append('bioAr', 'نادي كرة قدم محترف تأسس عام 1990.');
formData.append('bioKu', 'یانەی تۆپی پێ...');
formData.append('primaryColor', '#279846');
formData.append('secondaryColor', '#FFFFFF');
```

**Response**

Same shape as **Get registration state** (`RegistrationStateDto` in `data`).

---

### 4. Get club branding (tenant admin)

**Response only**

```http
GET /api/v1/sports360/tenant-admin/branding
Authorization: Bearer {tenantAdminToken}
```

**Response**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "tenantId": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Example FC",
    "domain": "example.getsport360.com",
    "adminDomain": "admin-example.getsport360.com",
    "logo": "https://...",
    "banner": "https://...",
    "primaryColor": "#279846",
    "secondaryColor": "#FFFFFF",
    "bioEn": "Professional football club founded in 1990.",
    "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
    "bioKu": "یانەی تۆپی پێ...",
    "websiteUrl": "https://example.com",
    "facebookUrl": null,
    "instagramUrl": null,
    "youtubeUrl": null,
    "subscriptionPlanName": "FREE",
    "subscriptionPlanId": "660e8400-e29b-41d4-a716-446655440001",
    "subscriptionEndDate": "2027-01-01T23:59:59Z",
    "isActive": true
  }
}
```

---

### 5. Update club branding (tenant admin)

**Request + response**

```http
PUT /api/v1/sports360/tenant-admin/branding
Authorization: Bearer {tenantAdminToken}
Content-Type: application/json
```

**Request**

```json
{
  "name": "Example FC",
  "bioEn": "Professional football club founded in 1990.",
  "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
  "bioKu": "یانەی تۆپی پێ...",
  "primaryColor": "#279846",
  "secondaryColor": "#FFFFFF",
  "websiteUrl": "https://example.com",
  "facebookUrl": "https://facebook.com/example",
  "instagramUrl": null,
  "youtubeUrl": null
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `name` | string | No | |
| `bioEn` | string | No | max 1000 — **replaces old `bio`** |
| `bioAr` | string | No | max 1000 |
| `bioKu` | string | No | max 1000 |
| `primaryColor` | string | No | |
| `secondaryColor` | string | No | |
| `websiteUrl` | string | No | |
| `facebookUrl` | string | No | |
| `instagramUrl` | string | No | |
| `youtubeUrl` | string | No | |

Only non-null fields in the request are applied (partial update).

**Response**

Same shape as **Get club branding** (`ClubBrandingResponse` in `data`).

---

### 6. Get public tenant branding

**Response only**

```http
GET /api/v1/sports360/tenants/public/me
```

No auth — tenant is resolved from subdomain/domain.

**Response**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "teamId": 123,
    "name": "Example FC",
    "domain": "example.getsport360.com",
    "logo": "https://...",
    "banner": "https://...",
    "primaryColor": "#279846",
    "secondaryColor": "#FFFFFF",
    "bioEn": "Professional football club founded in 1990.",
    "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
    "bioKu": "یانەی تۆپی پێ...",
    "websiteUrl": "https://example.com",
    "facebookUrl": null,
    "instagramUrl": null,
    "youtubeUrl": null
  }
}
```

---

### 7. List tenants (super admin)

**Response only**

```http
GET /api/v1/sports360/super-admin/tenant?page=0&size=10
Authorization: Bearer {superAdminToken}
```

**Response**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "content": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "name": "Example FC",
        "domain": "example.getsport360.com",
        "realmName": "example-fc",
        "active": true,
        "logo": "https://...",
        "banner": "https://...",
        "primaryColor": "#279846",
        "secondaryColor": "#FFFFFF",
        "bioEn": "Professional football club founded in 1990.",
        "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
        "bioKu": "یانەی تۆپی پێ...",
        "websiteUrl": "https://example.com",
        "facebookUrl": null,
        "instagramUrl": null,
        "youtubeUrl": null,
        "createdAt": "2026-01-01T00:00:00Z",
        "updatedAt": "2026-07-08T00:00:00Z",
        "subscriptionPlan": { },
        "subscriptionStartDate": "2026-01-01T00:00:00Z",
        "subscriptionEndDate": "2027-01-01T23:59:59Z",
        "teamId": 123
      }
    ],
    "totalElements": 1,
    "totalPages": 1,
    "size": 10,
    "number": 0
  }
}
```

---

### 8. Get tenant detail (super admin)

**Response only**

```http
GET /api/v1/sports360/super-admin/tenant/{id}
Authorization: Bearer {superAdminToken}
```

**Response**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "tenant": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Example FC",
      "domain": "example.getsport360.com",
      "realmName": "example-fc",
      "active": true,
      "logo": "https://...",
      "banner": "https://...",
      "primaryColor": "#279846",
      "secondaryColor": "#FFFFFF",
      "bioEn": "Professional football club founded in 1990.",
      "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
      "bioKu": "یانەی تۆپی پێ...",
      "websiteUrl": "https://example.com",
      "facebookUrl": null,
      "instagramUrl": null,
      "youtubeUrl": null,
      "createdAt": "2026-01-01T00:00:00Z",
      "updatedAt": "2026-07-08T00:00:00Z",
      "subscriptionPlan": { },
      "subscriptionStartDate": "2026-01-01T00:00:00Z",
      "subscriptionEndDate": "2027-01-01T23:59:59Z",
      "teamId": 123
    },
    "totalUsers": 42,
    "subscriptionHistory": [ ]
  }
}
```

---

### 9. Legacy tenant onboard

**Request only** (response is tenant ID string — no bio fields)

```http
POST /api/v1/sports360/onboarding/onboard
Authorization: Bearer {token}
Content-Type: application/json
```

**Request**

```json
{
  "tenantName": "Example FC",
  "domain": "example",
  "cognitoGroup": "example-fc",
  "subscriptionPlanId": "660e8400-e29b-41d4-a716-446655440001",
  "primaryColor": "#279846",
  "secondaryColor": "#FFFFFF",
  "bioEn": "Professional football club founded in 1990.",
  "bioAr": "نادي كرة قدم محترف تأسس عام 1990.",
  "bioKu": "یانەی تۆپی پێ...",
  "websiteUrl": "https://example.com",
  "facebookUrl": null,
  "instagramUrl": null,
  "youtubeUrl": null,
  "paymentId": null
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `bioEn` | string | No | max 1000 — **replaces old `bio`** |
| `bioAr` | string | No | max 1000 |
| `bioKu` | string | No | max 1000 |

**Response**

```json
{
  "code": 200,
  "message": "...",
  "data": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

## APIs not affected

These still use `bio` (user profile, not club/tenant branding):

| Method | Path |
|--------|------|
| `GET` | `/api/v1/sports360/user/me` |
| `PATCH` | `/api/v1/sports360/user/profile` |

---

## Existing data after deploy

- Old English text remains available as **`bioEn`**
- **`bioAr`** and **`bioKu`** are `null` until the club admin fills them in
