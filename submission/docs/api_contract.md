API contract

Success

```bash
{
   "success": true, (true/false)
   "data": {},
   "message": {}
}
```

Error

```bash
{
   "success": true, (true/false)
   "error": {
      "code": "VALIDATION_ERROR",
      "message": "Invalid request payload"
   }
}
```

Pagination approach

```bash
GET vehicle?page=1&limit=20&sort=createdAt&order=desc&vehicle_name=john
```

Authentication & multi-tenancy design
Use Prisma middleware to detect first and query tenent_id in service

```bash
tenant_id
```

Example Tenent approach

Tenent

```bash
id
name
plan (free | pro | etc)
created_at
```

Users

```bash
id
tenant_id
name
role
```

Vehicles

```bash
id
tenant_id
vehicle_name
```

Event

```bash
id
tenant_id
event_type
event_name
```

Rate limiting & usage tracking

- I would implement rate limiting using Redis + middleware and key by tenant/API key after authentication. Each request is tracked using a key based on tenant or API key
  Example. 1 min limit 100

```bash
rate:{tenant_id}:{api_key}
```

API usage for billing

- I use Redis to maintain real-time counters per tenant_id for fast access and dashboards
  and I log every API request asynchronously into a database as a usage ledger, which serves as the source of truth for billing.
  At the end of the billing cycle, I aggregate usage per tenant and apply pricing rules.
  To ensure scalability, I would use a queue to decouple request handling from database writes
