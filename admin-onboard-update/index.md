# Frontend Guide — Team Selection & Onboarding API Changes

This document describes the backend changes for club onboarding **team selection** and the updated **BASIC_INFO** / **COMPLETE** steps. Use it when building or updating the onboarding wizard UI.

**Base URL:** `/api/v1/sports360`

## Onboarding flow (updated)

```
PENDING_OTP
  → request-otp / verify-otp

PENDING_INFO
  → Search teams (public dropdown API)
  → PATCH basic-info (teamId + banner + other fields)

PENDING_PAYMENT / PENDING_PROVISIONING
  → select-plan (unchanged)

PENDING_PROVISIONING
  → POST complete (password only)

PROVISIONED
  → redirect to loginUrl
```

At the **PENDING_INFO** step, the UI should:

1. Show a **searchable team dropdown** (call team search API as user types).
2. Let the user pick a team (`teamId` + display `nameEn` and `logo`).
3. Collect admin info, colors, social links, and optional **banner** image.
4. Submit everything via **PATCH basic-info** as `multipart/form-data`.

---

## 1. Team search API (dropdown)

Used to find teams and obtain the `teamId` sent in BASIC_INFO.

### Request

```http
GET /api/v1/sports360/club/teams/search?name={searchText}
```


| Property           | Value                                                           |
| ------------------ | --------------------------------------------------------------- |
| Auth               | **None** (public)                                               |
| Query param `name` | Search text. If missing or blank, response is an **empty list** |
| Max results        | 20                                                              |


### Example

```http
GET /api/v1/sports360/club/teams/search?name=duhok
```

### Success response

```json
{
  "code": 200,
  "message": "Teams matching search query",
  "data": [
    {
      "id": 12345,
      "nameEn": "Duhok SC",
      "logo": "https://cdn.example.com/teams/duhok.png"
    },
    {
      "id": 67890,
      "nameEn": "Duhok FC",
      "logo": "https://cdn.example.com/teams/duhok-fc.png"
    }
  ]
}
```


| Field    | Type     | Description                                                  |
| -------- | -------- | ------------------------------------------------------------ |
| `id`     | `number` | **Use this as `teamId` in BASIC_INFO**                       |
| `nameEn` | `string` | Display name in the dropdown                                 |
| `logo`   | `string` | Team logo URL — show in dropdown and preview after selection |


### Frontend usage pattern

```javascript
async function searchTeams(query) {
  if (!query?.trim()) return [];

  const res = await fetch(
    `/api/v1/sports360/club/teams/search?name=${encodeURIComponent(query.trim())}`
  );
  const json = await res.json();
  return json.data ?? [];
}

// On user selection, store:
// selectedTeamId = team.id
// selectedTeamName = team.nameEn   (for UI only — do not send as clubName)
// selectedTeamLogo = team.logo     (for UI preview only)
```

**Recommendations:**

- Debounce search input (e.g. 300–500 ms).
- Only call the API when the user has typed at least 2 characters.
- Show `logo` + `nameEn` in each dropdown option.
- Store `team.id` in form state as `teamId`.

---

## 2. BASIC_INFO API (updated)

Saves admin details, **team selection**, colors, socials, and optional banner.

### Request

```http
PATCH /api/v1/sports360/onboarding/basic-info
Authorization: Bearer <onboarding-token>
Content-Type: multipart/form-data
```

### Form fields


| Field                | Required | Type                | Notes                                                         |
| -------------------- | -------- | ------------------- | ------------------------------------------------------------- |
| `teamId`             | **Yes**  | number (form field) | From team search `id`                                         |
| `adminName`          | Yes      | string              | 2–100 chars                                                   |
| `clubName`           | No       | string              | **Ignored by backend** — kept for backward compatibility only |
| `bio`                | No       | string              | Max 1000 chars                                                |
| `primaryColor`       | No       | string              | Hex, e.g. `#279846`                                           |
| `secondaryColor`     | No       | string              | Hex, e.g. `#FFFFFF`                                           |
| `websiteUrl`         | No       | string              | `http://` or `https://` or empty                              |
| `facebookUrl`        | No       | string              | Same                                                          |
| `instagramUrl`       | No       | string              | Same                                                          |
| `youtubeUrl`         | No       | string              | Same                                                          |
| `subscriptionPlanId` | No       | string (UUID)       | Optional pre-selection                                        |
| `banner`             | No       | file                | **Club banner image** — only branding file uploaded here      |


**Important:**

- `clubName` is **not** used. Backend sets it from the selected team's `nameEn`.
- `logo` is **not** sent by the frontend. Backend copies it from the team record.
- Send `teamId` as a plain form field (string is fine; backend parses to number).

### Example with `FormData`

```javascript
async function submitBasicInfo(onboardingToken, form) {
  const body = new FormData();

  body.append('teamId', String(form.teamId));       // required — from dropdown
  body.append('adminName', form.adminName);
  body.append('bio', form.bio ?? '');
  body.append('primaryColor', form.primaryColor ?? '');
  body.append('secondaryColor', form.secondaryColor ?? '');
  body.append('websiteUrl', form.websiteUrl ?? '');
  body.append('facebookUrl', form.facebookUrl ?? '');
  body.append('instagramUrl', form.instagramUrl ?? '');
  body.append('youtubeUrl', form.youtubeUrl ?? '');

  if (form.subscriptionPlanId) {
    body.append('subscriptionPlanId', form.subscriptionPlanId);
  }

  if (form.bannerFile) {
    body.append('banner', form.bannerFile);         // optional image file
  }

  const res = await fetch('/api/v1/sports360/onboarding/basic-info', {
    method: 'PATCH',
    headers: {
      Authorization: `Bearer ${onboardingToken}`,
      // Do NOT set Content-Type — browser sets multipart boundary
    },
    body,
  });

  return res.json();
}
```

### Success response

```json
{
  "code": 200,
  "message": "Basic info updated",
  "data": {
    "email": "admin@club.com",
    "status": "PENDING_INFO",
    "otpVerified": true,
    "adminName": "John Doe",
    "clubName": "Duhok SC",
    "teamId": 12345,
    "logo": "https://cdn.example.com/teams/duhok.png",
    "banner": "https://s3.amazonaws.com/bucket/tenants/...?X-Amz-Signature=...",
    "bio": "Professional football club...",
    "primaryColor": "#279846",
    "secondaryColor": "#FFFFFF",
    "websiteUrl": "https://duhokfc.com",
    "facebookUrl": "",
    "instagramUrl": "",
    "youtubeUrl": "",
    "subscriptionPlanId": null,
    "paymentStatus": "UNPAID",
    "paymentRef": null,
    "expiresAt": "2026-07-16T10:00:00Z"
  }
}
```

**New / notable response fields:**


| Field      | Description                                                                                                 |
| ---------- | ----------------------------------------------------------------------------------------------------------- |
| `teamId`   | Selected team ID (echoed back)                                                                              |
| `clubName` | Set by backend from team `nameEn` — use for display                                                         |
| `logo`     | Team logo URL from DB — use for preview                                                                     |
| `banner`   | Presigned S3 URL (valid ~1 hour) when a banner was uploaded; `null` otherwise — use directly in `<img src>` |


`GET /api/v1/sports360/onboarding/me` returns the same `RegistrationStateDto` shape, including `teamId`, `logo`, and `banner` — use it to resume the wizard.

### Error responses

Errors use this shape:

```json
{
  "timestamp": "2026-06-16T10:30:00",
  "status": 409,
  "error": "Conflict",
  "message": "This team is already registered on the platform.",
  "path": "/api/v1/sports360/onboarding/basic-info"
}
```


| HTTP status | When                                                     | User-facing message (approx.)                      |
| ----------- | -------------------------------------------------------- | -------------------------------------------------- |
| `400`       | Validation failed (e.g. missing `teamId`, invalid color) | Field-specific validation message                  |
| `404`       | `teamId` not found in team table                         | "Selected team was not found."                     |
| `409`       | Team already has a tenant                                | "This team is already registered on the platform." |
| `409`       | Another onboarding in progress for same team             | "This team already has an onboarding in progress." |


On `409`, prompt the user to pick a different team.

---

## 3. UI checklist

### Team dropdown step

- [ ] Search input with debounce calling `GET /club/teams/search?name=...`
- [ ] Render options with `logo` + `nameEn`
- [ ] On select, store `team.id` as `teamId`
- [ ] Show selected team logo preview (from search result or `registration.logo` on resume)

### Basic info form

- [ ] Remove free-text **club name** input (or make read-only preview from selected team)
- [ ] Add **banner** file input (optional)
- [ ] Remove **logo** upload from this step and from complete step
- [ ] Submit as `**multipart/form-data`** with `teamId` required
- [ ] Handle `409` — team already taken or onboarding in progress

### Complete step

- [ ] Only password + confirm password
- [ ] Remove logo/banner upload UI

---

## 4. Migration notes for existing frontend

1. **BASIC_INFO** must switch from `application/json` to `**FormData`** (`multipart/form-data`).
2. Replace manual `clubName` input with team dropdown + `teamId`.
3. `clubName` in the request body can still be sent (old clients won't crash) but has no effect.
4. Team search is public — no onboarding token needed; can be called before OTP if needed for UX, but typically used at the `PENDING_INFO` step.

---

## 5. Quick reference — endpoints


| Step         | Method  | Path                                        | Auth             |
| ------------ | ------- | ------------------------------------------- | ---------------- |
| Search teams | `GET`   | `/api/v1/sports360/club/teams/search?name=` | Public           |
| Get state    | `GET`   | `/api/v1/sports360/onboarding/me`           | Onboarding token |
| Basic info   | `PATCH` | `/api/v1/sports360/onboarding/basic-info`   | Onboarding token |
| Select plan  | `POST`  | `/api/v1/sports360/onboarding/select-plan`  | Onboarding token |


**Onboarding token:** `Authorization: Bearer <token>` from `verify-otp` response (`data.onboardingToken`).