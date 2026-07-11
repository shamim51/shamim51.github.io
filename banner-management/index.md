# Banner APIs — Frontend Reference

[⬅️ Back to Index](../README.md)

**Base URL:** `/api/v1/sports360`

**Required header (all banner list/create/update/delete requests):**

```http
X-Tenant-ID: {tenant-uuid}
```

Or realm name: `X-Tenant-ID: duhok-fc`

---

## What changed (short)

- `screen` field is **removed** → use `type`: `MOBILE` or `SHOP`
- Old public endpoint `GET /banners/screen/home` → `GET /banners?type=MOBILE`
- Tenant is resolved from **`X-Tenant-ID` header** — no `tenantId` query param
- **Shop banners** are new — same APIs, `type=SHOP`
- **Club branding banner** (`/tenant-admin/branding/banner`) — **no changes**

---

## Banner types

| Type | Used for |
|------|----------|
| `MOBILE` | Mobile app home carousel |
| `SHOP` | Shop/ecommerce carousel |
| Club banner | Separate system — not `MOBILE`/`SHOP` |

---

## 1. List banners

```http
GET /api/v1/sports360/banners?type=MOBILE
X-Tenant-ID: {uuid}
```

| Param | Required | Values |
|-------|----------|--------|
| `type` | No | `MOBILE`, `SHOP` — omit to get all types (admin) |

**Auth:** None for public read; `TENANT_ADMIN` token optional for admin panel

**Examples:**

```http
GET /api/v1/sports360/banners?type=MOBILE
GET /api/v1/sports360/banners?type=SHOP
GET /api/v1/sports360/banners
X-Tenant-ID: 3fa85f64-5717-4562-b3fc-2c963f66afa6
```

**Response:**

```json
{
  "code": 200,
  "message": "Banner list retrieved successfully",
  "data": [
    {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "type": "MOBILE",
      "image": "https://s3.amazonaws.com/...?X-Amz-Signature=...",
      "redirectUrl": "https://example.com/promo",
      "createdAt": "2026-07-11T10:00:00Z",
      "updatedAt": "2026-07-11T10:00:00Z"
    }
  ]
}
```

---

## 2. Create banner

```http
POST /api/v1/sports360/banners
Content-Type: multipart/form-data
X-Tenant-ID: {uuid}
Authorization: Bearer <token>
```

| Field | Required | Type |
|-------|----------|------|
| `type` | Yes | `MOBILE` or `SHOP` |
| `redirectUrl` | Yes | string |
| `image` | Yes | file |

**Response:**

```json
{
  "code": 200,
  "message": "Banner created successfully.",
  "data": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

---

## 3. Update banner

```http
PUT /api/v1/sports360/banners/{bannerId}
Content-Type: multipart/form-data
X-Tenant-ID: {uuid}
Authorization: Bearer <token>
```

| Field | Required | Type |
|-------|----------|------|
| `type` | No | `MOBILE` or `SHOP` |
| `redirectUrl` | No | string |
| `image` | No | file |

**Response:**

```json
{
  "code": 200,
  "message": "Banner updated successfully.",
  "data": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "type": "MOBILE",
    "image": "https://s3.amazonaws.com/...?X-Amz-Signature=...",
    "redirectUrl": "https://example.com/new-promo",
    "createdAt": "2026-07-11T10:00:00Z",
    "updatedAt": "2026-07-11T11:00:00Z"
  }
}
```

---

## 4. Delete banner

```http
DELETE /api/v1/sports360/banners/{bannerId}
X-Tenant-ID: {uuid}
Authorization: Bearer <token>
```

**Response:**

```json
{
  "code": 200,
  "message": "Banner deleted successfully.",
  "data": null
}
```


## Club branding banner (unchanged)

### Upload / replace club banner

```http
POST /api/v1/sports360/tenant-admin/branding/banner
Content-Type: multipart/form-data
X-Tenant-ID: {uuid}
Authorization: Bearer <token>
```

| Field | Required | Type |
|-------|----------|------|
| `file` | Yes | file |

**Response:**

```json
{
  "code": 200,
  "message": "Banner updated successfully.",
  "data": "https://s3.amazonaws.com/...?X-Amz-Signature=..."
}
```

### Get club branding (includes banner)

```http
GET /api/v1/sports360/tenant-admin/branding
X-Tenant-ID: {uuid}
Authorization: Bearer <token>
```

**Response (relevant fields):**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "logo": "https://s3.amazonaws.com/...",
    "banner": "https://s3.amazonaws.com/...",
    "primaryColor": "#FF0000",
    "secondaryColor": "#0000FF"
  }
}
```

### Public club info (includes banner)

```http
GET /api/v1/sports360/tenants/public/me
X-Tenant-ID: {uuid}
```

**Response (relevant field):**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "banner": "https://s3.amazonaws.com/..."
  }
}
```

### Onboarding — upload club banner

```http
PATCH /api/v1/sports360/onboarding/basic-info
Content-Type: multipart/form-data
Authorization: Bearer <onboarding-token>
```

| Field | Required | Type |
|-------|----------|------|
| `banner` | No | file |

**Response (relevant field in registration state):**

```json
{
  "code": 200,
  "message": "...",
  "data": {
    "banner": "https://s3.amazonaws.com/..."
  }
}
```

---

## Quick lookup

| API | Method | `X-Tenant-ID` | Auth | Changed? |
|-----|--------|---------------|------|----------|
| `/banners?type=` | GET | **Required** | Public | **Yes** |
| `/banners` | POST | **Required** | TENANT_ADMIN | **Yes** — `type` instead of `screen` |
| `/banners/{id}` | PUT | **Required** | TENANT_ADMIN | **Yes** — `type` instead of `screen` |
| `/banners/{id}` | DELETE | **Required** | TENANT_ADMIN | No |
| `/tenant-admin/branding/banner` | POST | **Required** | TENANT_ADMIN | No |
| `/tenant-admin/branding` | GET | **Required** | TENANT_ADMIN | No |
| `/tenants/public/me` | GET | **Required** | Public | No |
| `/onboarding/basic-info` | PATCH | Excluded | Onboarding token | No |
