# Snipe-IT Assets API Reference (Sample)

> **Note:** This is a writing sample. It documents the public, real-world [Snipe-IT REST API](https://snipe-it.readme.io/reference/api-overview) as an independent exercise — it is not official Snipe-IT documentation, and some request/response examples are illustrative.

## 1. Introduction

The Snipe-IT REST API lets you manage hardware assets, licenses, and users programmatically — the same operations available through the web UI, exposed as a versioned JSON API. This document covers the **Assets** resource: the core object representing a single piece of tracked hardware (a laptop, monitor, or any serialized item).

### 1.1 Base URL and Versioning

```
https://<your-domain>/api/v1
```

All endpoints in this document are relative to the base URL above. The API is versioned at the path level (`v1`); breaking changes are introduced under a new version rather than modifying existing behavior in place.

### 1.2 Authentication

Every request requires a bearer token in the `Authorization` header:

```
Authorization: Bearer <your-api-token>
```

Tokens are generated per-user from **Settings → API** in the Snipe-IT web UI. Treat tokens as credentials — they inherit the permissions of the user who created them.

### 1.3 Response Format

All responses are `application/json`. Successful list responses are wrapped in an envelope containing pagination metadata:

```json
{
  "total": 214,
  "rows": [ ... ]
}
```

## 2. Resource: Asset

### 2.1 Asset Representation

| Field | Type | Description |
|---|---|---|
| `id` | integer | Unique identifier, assigned by the server. Read-only. |
| `asset_tag` | string | Human-readable inventory tag. Required, unique. |
| `serial` | string | Manufacturer serial number. |
| `model_id` | integer | Reference to the asset's model (defines category, manufacturer, depreciation). Required. |
| `status_id` | integer | Reference to the current status label (e.g., Ready to Deploy, Deployed). Required. |
| `assigned_to` | integer \| null | ID of the user, asset, or location the item is currently checked out to. |
| `assigned_type` | string \| null | Type of the assignee: `App\Models\User`, `App\Models\Asset`, or `App\Models\Location`. |
| `purchase_date` | string (date) | ISO 8601 date. Optional. |
| `purchase_cost` | string | Decimal value as a string, for precision. Optional. |
| `notes` | string | Free-text notes field. |

### 2.2 List Assets

**`GET /hardware`**

Returns a paginated list of assets.

**Query parameters**

| Parameter | Description |
|---|---|
| `limit` | Number of results per page (default 50, max 500). |
| `offset` | Number of results to skip, for pagination. |
| `status_id` | Filter to assets with a specific status. |
| `category_id` | Filter to assets within a specific category. |
| `search` | Free-text search across asset tag, serial, notes, and assigned-user name. Slower than field-based filters on large datasets since it scans multiple joined columns. |

**Example request**

```
GET /api/v1/hardware?status_id=2&limit=25
Authorization: Bearer eyJ0eXAiOiJKV1Qi...
```

**Example response — `200 OK`**

```json
{
  "total": 3,
  "rows": [
    {
      "id": 1204,
      "asset_tag": "SNP-01204",
      "serial": "C02XG2QQJGH7",
      "model": { "id": 12, "name": "MacBook Pro 14\"" },
      "status_label": { "id": 2, "name": "Deployed" },
      "assigned_to": { "id": 88, "name": "Priya Nair", "type": "user" },
      "purchase_date": "2024-03-15",
      "purchase_cost": "2399.00"
    }
  ]
}
```

### 2.3 Create Asset

**`POST /hardware`**

**Request body**

```json
{
  "asset_tag": "SNP-01300",
  "model_id": 12,
  "status_id": 1,
  "serial": "C02XG3RRJGH8",
  "purchase_date": "2026-05-01",
  "purchase_cost": "2399.00"
}
```

**Response — `200 OK`**

```json
{
  "status": "success",
  "messages": "Asset created successfully",
  "payload": {
    "id": 1301,
    "asset_tag": "SNP-01300"
  }
}
```

**Validation error — `200 OK`**

Snipe-IT returns HTTP 200 even for validation failures, distinguishing success from failure through the `status` field rather than the status code. Check `status` before assuming success:

```json
{
  "status": "error",
  "messages": {
    "model_id": ["The selected model id is invalid."],
    "status_id": ["The selected status id is invalid."]
  }
}
```

### 2.4 Checkout / Checkin

Assigning an asset to a user, another asset, or a location is a distinct operation from updating the asset record, since it also creates an audit log entry.

**`POST /hardware/{id}/checkout`**

```json
{
  "checkout_to_type": "user",
  "assigned_user": 88,
  "note": "Deployed to new hire, onboarding batch May 2026"
}
```

**`POST /hardware/{id}/checkin`**

```json
{
  "note": "Returned — employee offboarding"
}
```

### 2.5 Delete Asset

**`DELETE /hardware/{id}`**

Soft-deletes the asset (marks it archived; does not remove history). Returns `200 OK` with a status payload, following the same success/error convention as create and update operations.

## 3. Error Handling

| HTTP Status | Meaning |
|---|---|
| `200` | Request succeeded **or** failed validation — always check the `status` field in the body. |
| `401` | Missing or invalid bearer token. |
| `404` | Resource not found — note that some endpoints return `200` with an empty payload instead; this is a known API inconsistency to design defensively around. |
| `429` | Rate limit exceeded. Respect the `Retry-After` header before retrying. |

## 4. Pagination Notes

List endpoints use `offset`/`limit` rather than page numbers. To page through a full result set:

1. Request with `limit=100&offset=0`.
2. Read `total` from the response envelope.
3. Repeat with `offset` incremented by `limit` until `offset >= total`.

---
*Sample documentation written by Matheus [Last Name] as a portfolio exercise, based on the publicly available Snipe-IT API.*
