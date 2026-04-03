# Piano API Reference for Data Engineering

> Complete reference for all Piano API endpoints relevant to data ingestion pipelines.
> Source: Piano SDK v16.734.2 + VX Export API.

## Table of Contents

- [Authentication](#authentication)
- [Base URLs](#base-urls)
- [API 1: VX Export API (Async CSV)](#api-1-vx-export-api-async-csv)
- [API 2: Publisher API v3 (Real-time JSON)](#api-2-publisher-api-v3-real-time-json)
  - [Conversion Endpoints](#conversion-endpoints)
  - [Subscription Endpoints](#subscription-endpoints)
  - [User Endpoints](#user-endpoints)
  - [User Access Endpoints](#user-access-endpoints)
  - [Term Endpoints](#term-endpoints)
  - [Resource Endpoints](#resource-endpoints)
  - [Promotion Endpoints](#promotion-endpoints)
  - [Payment Endpoints](#payment-endpoints)
  - [Export Management Endpoints](#export-management-endpoints)
  - [Export Create Endpoints](#export-create-endpoints)
- [Rate Limits and Best Practices](#rate-limits-and-best-practices)

---

## Authentication

All Piano APIs authenticate via an `api_token` header:

```
api_token: YOUR_API_TOKEN
```

The token is obtained from the Piano Dashboard under **Settings > API Tokens**.

> **Important:** The token is region-specific. A token from `dashboard-eu.piano.io` only works with EU API endpoints.

---

## Base URLs

| Region | Dashboard | Publisher API v3 | VX Export API |
|--------|-----------|-----------------|---------------|
| **EU** | `https://dashboard-eu.piano.io` | `https://dashboard-eu.piano.io/api/v3/` | `https://reports-api.piano.io/rest/export/` |
| **US** | `https://dashboard.piano.io` | `https://api.piano.io/api/v3/` | `https://reports-api.piano.io/rest/export/` |
| **AP** | `https://dashboard-ap.piano.io` | `https://api-ap.piano.io/api/v3/` | `https://reports-api.piano.io/rest/export/` |
| **Sandbox** | `https://sandbox.piano.io` | `https://sandbox.piano.io/api/v3/` | `https://reports-api.piano.io/rest/export/` |

---

## API 1: VX Export API (Async CSV)

The VX Export API uses an asynchronous pattern: **create job -> poll status -> download ZIP/CSV**.

Base URL: `https://reports-api.piano.io/rest/export/`

### 1.1 Create Subscription Log Export

Generates a CSV with 75 columns per subscription event. Includes user PII, device, URL, location, template, offer, campaign codes.

```
POST /schedule/vx/subscriptionLog
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `start_date_from` | string | No | Start date filter (YYYY-MM-DD) |
| `start_date_to` | string | No | End date filter (YYYY-MM-DD) |

**Example:**

```bash
curl -X POST "https://reports-api.piano.io/rest/export/schedule/vx/subscriptionLog" \
  -H "api_token: $PIANO_API_TOKEN" \
  -d "aid=B3HsDWtPpe" \
  -d "start_date_from=2026-03-29" \
  -d "start_date_to=2026-03-29"
```

**Response:**

```json
{
  "export_id": "EX123456",
  "file_name": "subscription-log.zip",
  "job_status": "QUEUED"
}
```

### 1.2 Create Conversion Export

Generates a ZIP containing 9 aggregate CSV files (device, page, location, tag, template, term, termType, promotion, campaignCode).

```
POST /schedule/vx/conversion
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `from` | string | No | Start date (YYYY-MM-DD) |
| `to` | string | No | End date (YYYY-MM-DD) |

**Example:**

```bash
curl -X POST "https://reports-api.piano.io/rest/export/schedule/vx/conversion" \
  -H "api_token: $PIANO_API_TOKEN" \
  -d "aid=JC6giZ1Ape" \
  -d "from=2026-03-29" \
  -d "to=2026-03-29"
```

**Response:** Same structure as subscription log.

### 1.3 Check Export Status

```
GET /status
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `export_id` | string | Yes | Export ID from create step |

**Example:**

```bash
curl "https://reports-api.piano.io/rest/export/status?aid=B3HsDWtPpe&export_id=EX123456" \
  -H "api_token: $PIANO_API_TOKEN"
```

**Response:**

```json
{
  "job_status": "FINISHED"
}
```

Possible statuses: `QUEUED`, `RUNNING`, `FINISHED`, `failed`, `error`

### 1.4 Get Download URL

```
GET /download/url
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `export_id` | string | Yes | Export ID |

**Response:**

```json
{
  "url": "https://storage.googleapis.com/..."
}
```

Download the file from the returned URL, then unzip to get CSVs.

---

## API 2: Publisher API v3 (Real-time JSON)

All endpoints return JSON synchronously. Paginated endpoints use `offset` + `limit`.

### Conversion Endpoints

#### 2.1 List Conversions

Returns individual conversion events with user, term, subscription, payment, and promo details.

```
GET /api/v3/publisher/conversion/list
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `aid` | string | Yes | | Application ID |
| `uid` | string | No | | Filter by user ID |
| `date_from` | datetime | No | | Start date filter |
| `date_to` | datetime | No | | End date filter |
| `offset` | int | No | 0 | Pagination offset |
| `limit` | int | No | 100 | Page size (max 100) |

**Example:**

```bash
curl "https://dashboard-eu.piano.io/api/v3/publisher/conversion/list?aid=JC6giZ1Ape&date_from=2026-03-29&date_to=2026-03-30&offset=0&limit=100" \
  -H "api_token: $PIANO_API_TOKEN"
```

**Response:**

```json
{
  "code": 0,
  "total": 47,
  "conversions": [
    {
      "term_conversion_id": "TCPEUBHDL6W9",
      "type": "Registration",
      "aid": "JC6giZ1Ape",
      "create_date": 1774824823,
      "browser_id": "mnccre1r7ms5vep8",
      "term": {
        "term_id": "TMS2OE33RI3N",
        "name": "Maak een account aan",
        "type": "registration",
        "type_name": "Registration",
        "resource": { "rid": "RPSSCY4", "name": "Digital reading" }
      },
      "user_access": {
        "access_id": "iVpozDohyJ2O",
        "granted": true,
        "user": { "uid": "15eb0605-...", "email": "user@example.com" },
        "start_date": 1774824823
      },
      "subscription": null,
      "user_payment": null,
      "promo_code": null,
      "schedule": null,
      "period": null
    }
  ]
}
```

**Pagination:** Loop while `len(conversions) < total`, incrementing `offset` by `limit`.

#### 2.2 Get Conversion Detail

Get a single conversion by ID or access ID.

```
GET /api/v3/publisher/conversion/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `term_conversion_id` | string | No | Conversion ID |
| `access_id` | string | No | Access ID |
| `is_last_term_conversion` | bool | No | Only return latest |

#### 2.3 Get Conversion Data (Behavioral Context)

**Critical endpoint** -- returns device, page, location, tags, template, offer, and campaign data for a single conversion.

```
GET /api/v3/publisher/conversion/data/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `term_conversion_id` | string | Yes | Conversion ID from `/conversion/list` |

**Example:**

```bash
curl "https://dashboard-eu.piano.io/api/v3/publisher/conversion/data/get?aid=JC6giZ1Ape&term_conversion_id=TCPEUBHDL6W9" \
  -H "api_token: $PIANO_API_TOKEN"
```

**Response (TermConversionData):**

```json
{
  "code": 0,
  "aid": "JC6giZ1Ape",
  "offer_id": "OFG06JSSAJVS",
  "term_id": "TMS2OE33RI3N",
  "offer_template_id": "OTZ940Z1LGCH",
  "template_id": "TMPL123",
  "uid": "15eb0605-...",
  "user_country": "The Netherlands",
  "user_region": "North Holland",
  "user_city": "Amsterdam",
  "zip": "1012",
  "latitude": "52.3676",
  "longitude": "4.9041",
  "user_agent": "Mozilla/5.0 ...",
  "locale": "nl_NL",
  "hour": "14",
  "url": "https://www.cosmopolitan.com/nl/...",
  "browser": "chrome",
  "platform": "mobile",
  "operating_system": "android",
  "tags": "Content,affiliate",
  "content_created": "2026-03-15",
  "content_author": "Author Name",
  "content_section": "lifestyle",
  "campaigns": ["campaign_abc"]
}
```

> **This is the key endpoint that bridges the gap between `/conversion/list` (user/payment data) and the export CSVs (behavioral data).**

#### 2.4 Count Conversions

```
GET /api/v3/publisher/conversion/count
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `term_id` | string | No | Filter by term |

#### 2.5 Get Last Conversion Access

```
GET /api/v3/publisher/conversion/lastAccess
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `rid` | string | No | Resource ID |
| `uid` | string | No | User ID |
| `subscription_id` | string | No | Subscription ID |

#### 2.6 Log External Conversion

Write-only endpoint to log a conversion event manually. Not used for data ingestion.

```
POST /api/v3/publisher/conversion/log
```

#### 2.7 Create Registration Conversion

Write-only endpoint to create a registration conversion manually.

```
POST /api/v3/publisher/conversion/registration/create
```

---

### Subscription Endpoints

#### 2.8 List Subscriptions

```
GET /api/v3/publisher/subscription/list
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `aid` | string | Yes | | Application ID |
| `uid` | string | No | | Filter by user |
| `type` | string | No | | Filter by term type |
| `start_date` | datetime | No | | Filter start date |
| `end_date` | datetime | No | | Filter end date |
| `q` | string | No | | Search query |
| `select_by` | string | No | | Selection criteria |
| `status` | string | No | | Filter by status |
| `offset` | int | No | 0 | Pagination offset |
| `limit` | int | No | 100 | Page size |

**Returns:** `List[UserSubscriptionListItem]` with subscription_id, status, dates, term info, user info.

#### 2.9 Get Subscription

```
GET /api/v3/publisher/subscription/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `subscription_id` | string | Yes | Subscription ID |

**Returns:** Full `UserSubscription` object.

#### 2.10 Search Subscriptions (Advanced)

```
GET /api/v3/publisher/subscription/search
```

Supports ~25 filter parameters for advanced subscription searching:

| Key Parameters | Description |
|----------------|-------------|
| `search_new_subscriptions` | Filter by creation date range |
| `search_active_now_subscriptions` | Filter active subscriptions by status |
| `search_inactive_subscriptions` | Filter inactive by status and date |
| `search_updated_subscriptions` | Filter recently updated |
| `search_auto_renewing_subscriptions` | Filter by auto-renew |
| `search_subscriptions_by_next_billing_date` | Filter by next billing |
| `search_subscriptions_by_terms` | Filter by specific terms/term types |

**Returns:** `List[SubscriptionLogItem]`

#### 2.11 Count Subscriptions

```
POST /api/v3/publisher/subscription/count
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |

#### 2.12 Subscription Stats (per user)

```
POST /api/v3/publisher/subscription/stats
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `uid` | string | Yes | User ID |

**Returns:** `List[UserSubscriptionDto]` for that user.

---

### User Endpoints

#### 2.13 List Users

```
POST /api/v3/publisher/user/list
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `aid` | string | Yes | | Application ID |
| `disabled` | bool | No | false | Include disabled users |
| `q` | string | No | | Search query |
| `offset` | int | No | 0 | Pagination offset |
| `limit` | int | No | 100 | Page size |

**Returns:** `List[User]` with uid, email, first_name, last_name, create_date, last_visit.

#### 2.14 Get User

```
POST /api/v3/publisher/user/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `uid` | string | Yes | User ID |

#### 2.15 Search Users (Advanced)

```
POST /api/v3/publisher/user/search
```

Supports 60+ filter parameters for advanced user searching: registration dates, access status, conversion terms, spending amounts, payment methods, billing failures, subscription status, inquiries, licensing, custom fields, consents, and more.

---

### User Access Endpoints

#### 2.16 List User Access

```
GET /api/v3/publisher/user/access/list
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `uid` | string | Yes | User ID |
| `expand_bundled` | bool | No | Expand bundled access |
| `cross_app` | bool | No | Include cross-app access |

**Returns:** `List[AccessDTO]` with access_id, resource, granted/revoked status, dates.

#### 2.17 Check User Access

```
GET /api/v3/publisher/user/access/check
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `uid` | string | Yes | User ID |
| `rid` | string | Yes | Resource ID |

#### 2.18 Count Active Access

```
POST /api/v3/publisher/user/access/active/count
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |

---

### Term Endpoints

#### 2.19 List Terms

```
GET /api/v3/publisher/term/list
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `rid` | string | No | Filter by resource |
| `include_type` | List[str] | No | Include term types |
| `exclude_type` | List[str] | No | Exclude term types |
| `type` | string | No | Specific type filter |
| `source` | List[str] | No | Source filter |

**Returns:** `List[Term]` with term_id, name, type, resource, billing details, pricing.

**Use case:** Dimension table -- sync weekly to resolve term_id to display names.

#### 2.20 Get Term

```
GET /api/v3/publisher/term/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `term_id` | string | Yes | Term ID |

#### 2.21 Count Terms

```
GET /api/v3/publisher/term/count
```

---

### Resource Endpoints

#### 2.22 List Resources

```
GET /api/v3/publisher/resource/list
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `type` | string | No | Resource type (default "NA") |
| `q` | string | No | Search query |

**Returns:** `List[Resource]` with rid, name, description, type, publish_date, external_id.

**Use case:** Dimension table -- sync weekly.

#### 2.23 Get Resource

```
GET /api/v3/publisher/resource/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `rid` | string | Yes | Resource ID |

---

### Promotion Endpoints

#### 2.24 List Promotions

```
GET /api/v3/publisher/promotion/list
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `aid` | string | Yes | | Application ID |
| `expired` | string | No | "active" | Filter: "active", "expired", "all" |

**Returns:** `List[Promotion]` with promotion_id, name, discount_type, percentage_discount, start/end dates, uses.

#### 2.25 Get Promotion

```
GET /api/v3/publisher/promotion/get
```

#### 2.26 Get Promotion Total Sales

```
GET /api/v3/publisher/promotion/total
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `promotion_id` | string | Yes | | Promotion ID |
| `aid` | string | Yes | | Application ID |
| `currency_code` | string | No | "USD" | Currency |

---

### Payment Endpoints

#### 2.27 Get Payment

```
GET /api/v3/publisher/payment/get
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `user_payment_id` | string | Yes | Payment ID |

**Returns:** `UserPayment` with amount, currency, payment_method, state, dates.

---

### Export Management Endpoints

#### 2.28 List Exports

```
GET /api/v3/publisher/export/list
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `order_by` | string | No | Sort field |
| `order_direction` | string | No | "asc" or "desc" |
| `q` | string | No | Search query |

**Returns:** `List[Export]` with export_id, name, status, dates.

#### 2.29 Get Export

```
GET /api/v3/publisher/export/get
```

#### 2.30 Download Export

```
GET /api/v3/publisher/export/download
```

**Returns:** Download URL string.

#### 2.31 Re-run Export

```
GET /api/v3/publisher/export/run
```

#### 2.32 Delete Export

```
GET /api/v3/publisher/export/delete
```

---

### Export Create Endpoints

These create async export jobs. After calling, use `/export/get` to poll status and `/export/download` to retrieve the file.

#### 2.33 AAM Daily Report (Proof of Access)

```
POST /api/v3/publisher/export/create/aam/daily
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `export_name` | string | Yes | Name for the export |
| `date_from` | datetime | Yes | Start date |
| `date_to` | datetime | Yes | End date |
| `snowflake` | bool | No | Snowflake format |

**Use case:** Daily proof-of-access report for audience measurement.

#### 2.34 AAM Monthly Report v2

```
POST /api/v3/publisher/export/create/aam/monthly/v2
```

Same parameters as daily. Monthly aggregation.

#### 2.35 Access Report Export

```
GET /api/v3/publisher/export/create/accessReportExport
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `aid` | string | Yes | | Application ID |
| `export_name` | string | Yes | | Name for the export |
| `date_from` | datetime | Yes | | Start date |
| `date_to` | datetime | Yes | | End date |
| `access_status` | string | No | "ALL" | Filter: ALL, ACTIVE, EXPIRED, REVOKED |
| `term_type` | List[str] | No | | Filter by term types |
| `term_id` | List[str] | No | | Filter by term IDs |
| `last_payment_status` | string | No | | Filter by payment status |

**Use case:** Snapshot of all access grants.

#### 2.36 Access Report Export v2 (with timezone)

```
GET /api/v3/publisher/export/create/accessReportExport/v2
```

Same as above with timezone-aware dates.

#### 2.37 Transactions Report

```
POST /api/v3/publisher/export/create/transactionsReport
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `aid` | string | Yes | | Application ID |
| `export_name` | string | Yes | | Name for the export |
| `transactions_type` | string | No | "purchases" | Type filter |
| `order_by` | string | No | "payment_date" | Sort field |
| `date_from` | datetime | No | | Start date |
| `date_to` | datetime | No | | End date |

**Use case:** Payment transactions for revenue analytics and reconciliation.

#### 2.38 Transactions Report v2 (with timezone)

```
POST /api/v3/publisher/export/create/transactionsReport/v2
```

#### 2.39 Subscription Details Report

```
POST /api/v3/publisher/export/create/subscriptionDetailsReport
```

Supports ~25 filter parameters (same as `/subscription/search`). Creates a CSV export of matching subscriptions.

#### 2.40 Subscription Details Report v2 (with timezone)

```
POST /api/v3/publisher/export/create/subscriptionDetailsReport/v2
```

#### 2.41 Subscription Summary Report

```
POST /api/v3/publisher/export/create/subscriptionSummaryReport
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `export_name` | string | Yes | Name for the export |
| `date_from` | datetime | Yes | Start date |
| `date_to` | datetime | Yes | End date |

#### 2.42 Term Change Report

```
GET /api/v3/publisher/export/create/termChangeReportExport
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `export_name` | string | Yes | Name for the export |
| `date_from` | datetime | No | Start date |
| `date_to` | datetime | No | End date |

**Use case:** Track term upgrades/downgrades over time.

#### 2.43 Daily Activity Report

```
GET /api/v3/publisher/export/create/dailyActivityReportExport
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `export_name` | string | Yes | Name for the export |
| `date` | datetime | Yes | Report date |
| `term_type` | List[str] | No | Filter by term types |
| `currency` | string | No | Currency filter |

#### 2.44 Monthly Activity Report

```
GET /api/v3/publisher/export/create/monthlyActivityReportExport
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `aid` | string | Yes | Application ID |
| `export_name` | string | Yes | Name for the export |
| `month` | int | Yes | Month (1-12) |
| `year` | int | Yes | Year |

#### 2.45 User Export

```
POST /api/v3/publisher/export/create/userExport
```

Supports 60+ filter parameters for exporting filtered user lists with custom fields, consent data, subscription status, spending, and more.

---

## Rate Limits and Best Practices

### General Guidelines

- **No official rate limit documented**, but Piano recommends staying under ~10 requests/second.
- The `/conversion/data/get` endpoint is per-conversion (1 call per `term_conversion_id`). For 47 conversions, that is 47 calls. Add a small delay (~100-200ms) between calls.
- VX Export jobs take 30-120 seconds to complete. Poll every 10 seconds.
- Paginated endpoints max out at `limit=100`. Always paginate until `offset + len(results) >= total`.
- Use `date_from` / `date_to` filters to limit result sets. Avoid fetching all-time data daily.

### Error Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `401` | Invalid or wrong-region API token |
| `400` | Bad request (check parameter format) |
| `404` | Resource not found |
| `429` | Rate limited (back off and retry) |

### Date Formats

- VX Export API: `YYYY-MM-DD` strings
- Publisher API v3: ISO datetime or epoch seconds (varies by endpoint)
- Response timestamps: Unix epoch seconds (e.g., `1774824823`)
