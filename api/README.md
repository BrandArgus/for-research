# BrandArgus Research API

The BrandArgus Research API provides free access to domain intelligence data for qualified security researchers and brand protection professionals.

Two API services are available:

## NRDS Search API

Search and analyze newly registered gTLD domains within the past 30 days. Supports phishing detection, brand impersonation monitoring, malicious campaign tracking, and registration pattern analysis.

| Attribute | Details |
|-----------|---------|
| Base URL | `https://api.brandargus.com/v1/nrds-search` |
| Domain Types | Major gTLDs (com, net, org, info, xyz, etc.) |
| Time Window | Rolling 30 days |
| Update Frequency | Daily |
| Data Source | Daily updated gTLD data |

## NS Reverse Lookup API

Identify all gTLD domains hosted under a specific nameserver infrastructure. Supports threat intelligence, infrastructure correlation, and malicious domain tracking.

| Attribute | Details |
|-----------|---------|
| Base URL | `https://api.brandargus.com/v1/ns-reverse` |
| Data Scope | Active domains (Daily Snapshot) |
| Update Frequency | Daily |
| Data Source | Daily updated gTLD data |

Both APIs are currently in Beta and provided free of charge for research purposes.

---

## Quick Start

```bash
# NRDS: Search domains containing "paypal"
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.brandargus.com/v1/nrds-search/domains?keyword=paypal&tld=com"

# NS Reverse: Find domains on a nameserver
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.brandargus.com/v1/ns-reverse/domains?ns=ns1.example.com"
```

---

## Authentication

All API requests require a Bearer token in the Authorization header. Your client IP must also be within the pre-authorized range.

### IP Whitelist

| Attribute | Details |
|-----------|---------|
| IP Format | Single IP (e.g., `203.0.113.50`) or CIDR range (e.g., `203.0.113.0/24`) |
| IP Changes | Contact support. Changes may take up to 24 hours. |
| Dynamic IP | Not supported. Use a VPN or cloud server with static IP. |

API requests from non-whitelisted IPs will be rejected with a `403` error.

### Authentication Header

```
Authorization: Bearer <your-api-key>
```

---

## Rate Limiting

| Attribute | Details |
|-----------|---------|
| Requests per minute | 120 |
| Recommended interval | 1 request per second |

Exceeding the limit returns a `429 Too Many Requests` error.

### Response Headers

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests per minute |
| `X-RateLimit-Remaining` | Remaining requests in current window |
| `X-RateLimit-Reset` | Unix timestamp when the rate limit resets |

### Rate Limit Response

```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 120
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1706284800

{
  "error": "rate limit exceeded"
}
```

---

## Error Codes

| Code | Description |
|------|-------------|
| 400 | Invalid parameter, malformed cursor, or bad request |
| 401 | Missing or invalid API key |
| 403 | IP address not in allowed range |
| 429 | Rate limit exceeded |
| 500 | Internal server error |

### Error Response

```json
{
  "error": "ip not in allowed range"
}
```

---

## NRDS Search API

### Search Domains

`GET /v1/nrds-search/domains`

Search newly registered domains with flexible filtering and cursor-based pagination. Results are returned in database order.

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `keyword` | string | — | Search keyword in domain prefix (case-insensitive) |
| `match` | string | `contains` | Match type: `contains`, `prefix`, `suffix` |
| `tld` | string | all | Filter by TLD (e.g., `com`, `net`, `org`) |
| `reg_after` | string | — | Registration date from, inclusive (`YYYY-MM-DD`) |
| `reg_before` | string | — | Registration date to, inclusive (`YYYY-MM-DD`) |
| `pure_number` | bool | — | If `true`, only pure numeric domains. If `false`, exclude them. |
| `has_number` | bool | — | If `true`, must contain numbers. If `false`, must not. |
| `has_hyphen` | bool | — | If `true`, must contain hyphens. If `false`, must not. |
| `pure_alpha` | bool | `false` | Only alphabetical domains `[a-z]` |
| `min_len` | int | — | Minimum domain prefix length |
| `max_len` | int | — | Maximum domain prefix length |
| `limit` | int | `100` | Results per page (max: 1000) |
| `cursor` | string | — | Pagination cursor from previous response |

The `ns` field in the response contains multiple nameservers separated by semicolons (`;`). When `next_cursor` is `null` or absent, there are no more results.

#### Request

```bash
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/nrds-search/domains?\
keyword=paypal&tld=com&limit=100"

# Pagination
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/nrds-search/domains?\
keyword=paypal&cursor=eyJvZmZzZXQiOjEwMH0"

# Phishing detection: domains with hyphens
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/nrds-search/domains?\
keyword=paypal&has_hyphen=true"

# Short alphabetical domains
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/nrds-search/domains?\
pure_alpha=true&min_len=3&max_len=6"
```

#### Response

```json
{
  "matched": 1542,
  "returned": 100,
  "next_cursor": "eyJvZmZzZXQiOjEwMH0",
  "domains": [
    {
      "domain": "paypal-secure-login.com",
      "reg_date": "2026-01-25",
      "exp_date": "2027-01-25",
      "ns": "ns1.example.com;ns2.example.com"
    }
  ]
}
```

---

### Count Domains

`GET /v1/nrds-search/domains/count`

Count matching domains without returning the full result set. Accepts the same filtering parameters as Search (except `limit` and `cursor`).

#### Request

```bash
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/nrds-search/domains/count?\
keyword=paypal"
```

#### Response

```json
{
  "matched": 15432
}
```

---

### Export Domains

`GET /v1/nrds-search/domains/export`

Download matching domains as a CSV file. Accepts the same filtering parameters as Search (except `limit` and `cursor`).

> Large exports with millions of domains may take several seconds. Set your HTTP client timeout to 30s+.

#### Response Headers

| Header | Value |
|--------|-------|
| `Content-Type` | `text/csv; charset=utf-8` |
| `Content-Disposition` | `attachment; filename=nrds-export.csv` |

#### Request

```bash
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/nrds-search/domains/export?\
keyword=paypal&tld=com" \
  -o domains.csv
```

#### Response (CSV)

```csv
domain,reg_date,exp_date,ns
paypal-secure.com,2026-01-25,2027-01-25,ns1.host.com;ns2.host.com
mypaypal-login.com,2026-01-24,2027-01-24,ns1.reg.com;ns2.reg.com
```

---

### Health Check

`GET /v1/nrds-search/health`

Check API service status and data freshness. No authentication required.

#### Response

```json
{
  "status": "ok",
  "updated_at": "2026-01-26T02:30:00Z"
}
```

---

## NS Reverse Lookup API

### Domain Count

`GET /v1/ns-reverse/domains/count`

Retrieve the total number of domains hosted under a specific nameserver.

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ns` **(required)** | string | — | Nameserver FQDN (e.g., `ns1.example.com`) |

#### Request

```bash
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/ns-reverse/domains/count?\
ns=ns1.example.com"
```

#### Response

```json
{
  "ns": "ns1.example.com",
  "total": 125432
}
```

---

### Query Domains

`GET /v1/ns-reverse/domains`

Query domains hosted under a specific nameserver with filtering and cursor-based pagination.

#### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ns` **(required)** | string | — | Nameserver FQDN (e.g., `ns1.example.com`) |
| `limit` | int | `100` | Results per page (max: 1000) |
| `cursor` | string | — | Pagination cursor from previous response |
| `tld` | string | all | Filter by TLD (e.g., `com`, `net`) |
| `keyword` | string | — | Filter by keyword in domain prefix (case-insensitive) |
| `match` | string | `contains` | Match type: `contains`, `prefix`, `suffix` |
| `has_number` | bool | — | If `true`, must contain numbers. If `false`, must not. |
| `has_hyphen` | bool | — | If `true`, must contain hyphens. If `false`, must not. |
| `pure_alpha` | bool | `false` | Only alphabetical domains `[a-z]` |
| `min_len` | int | — | Minimum domain prefix length |
| `max_len` | int | — | Maximum domain prefix length |
| `reg_after` | string | — | Registration date from (`YYYY-MM-DD`) |
| `reg_before` | string | — | Registration date to (`YYYY-MM-DD`) |

The `domains` array contains full domain names (e.g., `"example.com"`). When `next_cursor` is `null` or absent, there are no more results.

#### Request

```bash
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/ns-reverse/domains?\
ns=ns1.example.com&tld=com&limit=100"

# Search by keyword
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/ns-reverse/domains?\
ns=ns1.example.com&keyword=bank&match=prefix"

# Filter by registration date
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/ns-reverse/domains?\
ns=ns1.example.com&reg_after=2025-01-01&reg_before=2025-12-31"
```

#### Response

```json
{
  "ns": "ns1.example.com",
  "total": 125432,
  "returned": 100,
  "next_cursor": "eyJvZmZzZXQiOjEwMH0",
  "domains": [
    "example.com",
    "domainkits.com",
    "brandargus.com"
  ]
}
```

---

### Export Domains

`GET /v1/ns-reverse/domains/export`

Download all domains as a CSV file. Supports the same filtering parameters as Query (except `limit` and `cursor`).

> Large nameservers with millions of domains may take several seconds. Set timeout to 30s+.

#### Response Headers

| Header | Value |
|--------|-------|
| `Content-Type` | `text/csv; charset=utf-8` |
| `Content-Disposition` | `attachment; filename=ns1_example_com.csv` |

#### Request

```bash
curl -H "Authorization: Bearer sk-xxx" \
  "https://api.brandargus.com/v1/ns-reverse/domains/export?\
ns=ns1.example.com&tld=com&pure_alpha=true" \
  -o filtered.csv
```

#### Response (CSV)

```csv
domain,reg_date
example.com,2024-05-15
domainkits.com,2023-11-20
brandargus.com,
```

The `reg_date` field may be empty if registration date is unavailable.

---

### Health Check

`GET /v1/ns-reverse/health`

Check API service status and data freshness. No authentication required.

#### Response

```json
{
  "status": "ok",
  "updated_at": "2026-01-23T20:30:00Z"
}
```

---

## Terms of Use

BrandArgus.com is operated by Lyalpha GmbH. API access is granted exclusively for legitimate protection and research purposes.

### Permitted Use

- Security research and threat intelligence
- Brand protection monitoring
- Academic and non-commercial research

### Restrictions

- Do NOT use for spamming, harassment, or illegal activities
- No redistribution, sublicensing, or resale of raw data
- API keys are non-transferable and bound to your registered IP range

### GDPR Compliance

| Attribute | Details |
|-----------|---------|
| Data Minimization | No PII of domain registrants is provided |
| Log Anonymization | Requesting IPs are anonymized (last two octets masked) |
| Retention Period | Access logs retained for max 90 days |
| Account Termination | All logs deleted within 30 days upon request |

---

## Free Beta Access

This API is provided free of charge for research purposes. We welcome bug reports, feature suggestions, and feedback on data quality.


Contact: info@lyalpha-gmbh.com
