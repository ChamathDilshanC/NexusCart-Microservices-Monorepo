# Migrating from Azure Cosmos DB (Mongo API) to OCI Autonomous JSON Database

Goal: replace Cosmos DB (paid, RU-based) with Oracle Always Free Autonomous
JSON Database, using the Oracle Database API for MongoDB, without touching
the 8 Azure Container Apps beyond a single secret update.

## Status

- [x] Autonomous JSON Database `nexuscartdb` provisioned (Always Free, 26ai engine, region ap-singapore-1)
- [x] Network ACL set to `0.0.0.0/0` (see "Network access" below for why)
- [x] Mongo API confirmed enabled (on by default for AJD workload)
- [x] Dedicated app schema `NEXUSCART` created (SODA_APP + DB_DEVELOPER_ROLE, not ADMIN)
- [ ] `ORDS.ENABLE_SCHEMA` run for the `NEXUSCART` schema — **manual step, see below**
- [ ] Data migrated (`scripts/migrate-cosmos-to-oracle.sh`)
- [ ] Migration validated (`scripts/validate-migration.js`)
- [ ] Azure Container Apps cut over (`scripts/cutover-azure-secrets.sh`)
- [ ] Old Cosmos DB account deleted (only after the above are confirmed healthy)

## One remaining manual step: enable ORDS on the app schema

The Mongo API requires the target schema to be REST/SODA-enabled. This must
be run once, by a human, in Oracle's browser SQL Worksheet — it's an
admin-privileged operation against the database and isn't something to
automate blind:

1. Open the SQL Worksheet for `nexuscartdb` (URL in OCI Console →
   Autonomous Database → Database Actions → SQL, or the `sql-dev-web-url`
   from `oci db autonomous-database get`).
2. Log in as `ADMIN`.
3. Run:

   ```sql
   BEGIN
     ORDS.ENABLE_SCHEMA(
       p_enabled => TRUE,
       p_schema => 'NEXUSCART',
       p_url_mapping_type => 'BASE_PATH',
       p_url_mapping_pattern => 'nexuscart',
       p_auto_rest_auth => FALSE
     );
     COMMIT;
   END;
   /
   ```

## Network access

Oracle's Mongo API requires either "Secure access from allowed IPs and VCNs
only" or a private endpoint. Azure Container Apps (consumption plan) has no
static outbound IP by default, so IP-restricting the ACL to "just Azure"
isn't possible without adding a NAT Gateway + VNet integration on the Azure
side (extra cost, defeats the point of cutting spend).

Chosen approach: ACL open to `0.0.0.0/0`, mitigated by:
- TLS mandatory (Oracle enforces `ssl=true`)
- A dedicated, least-privilege app user (`NEXUSCART`) instead of `ADMIN`
- A strong random 20-character password (stored only in the container app secret and a local scratch file — not committed anywhere)

This is materially safer than the originally-considered alternative
(self-hosted MongoDB on a bare OCI VM with an open port and no TLS by
default), but isn't zero-risk. Rotate the `NEXUSCART` password periodically.

## Running the migration

```bash
# 1. Migrate data (reads live Cosmos DB — run during low traffic)
SOURCE_MONGODB_URI="<current backend/.env MONGODB_URI>" \
TARGET_MONGODB_URI="mongodb://nexuscart:<APP_PASSWORD>@<ADB_HOST>:27017/nexuscart?authMechanism=PLAIN&authSource=\$external&ssl=true&retryWrites=false&loadBalanced=true" \
  backend/scripts/migrate-cosmos-to-oracle.sh

# 2. Validate before touching production traffic
SOURCE_MONGODB_URI="..." TARGET_MONGODB_URI="..." \
  node backend/scripts/validate-migration.js

# 3. Cut over all 8 Container Apps (prompts for confirmation)
TARGET_MONGODB_URI="..." backend/scripts/cutover-azure-secrets.sh
```

All 8 services (`auth`, `product`, `order`, `payment`, `business`,
`review-rating`, `notification`, `admin`) share a single MongoDB database —
confirmed by `order-service`, `business-service`, and `admin-service` all
defining an `Order` model against the same default collection, same for
`User`. So the target is one Oracle schema (`nexuscart`), not eight.

## Notes

- Free tier limit: 1 OCPU / 20GB storage per Autonomous DB, max 2 free
  instances per tenancy. Fine for current scale; watch storage as the
  catalog/order history grows.
- `mongorestore` into Oracle's ORDS-backed Mongo API only supports one
  source database per restore invocation (can't combine multiple
  `--nsInclude` in one command) — the migration script already loops
  per-database to handle this.
