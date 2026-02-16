# Advanced Topics

## Logging Configuration

Open WebUI uses Python's standard logging with configurable levels:

```bash
# Set log level (DEBUG, INFO, WARNING, ERROR)
docker run -e LOG_LEVEL=DEBUG ghcr.io/open-webui/open-webui:main

# Enable SQL query logging (use with caution - verbose)
docker run -e SQLALCHEMY_ECHO=True ghcr.io/open-webui/open-webui:main
```

### Structured JSON Logging

For production environments using log aggregation:

```bash
docker run -e LOG_FORMAT=json ghcr.io/open-webui/open-webui:main
```

## Database Migration Procedures

### SQLite to PostgreSQL Migration

1. **Export existing data:**
```bash
docker exec open-webui sqlite3 /app/backend/data/webui.db ".dump" > backup.sql
```

2. **Start PostgreSQL instance** and set `DATABASE_URL`

3. **Import with custom script** (available in scripts/migrate.py)

### Backup Strategies

**Automated SQLite backup:**
```bash
# Daily backup cron job
0 2 * * * docker exec open-webui sqlite3 /app/backend/data/webui.db ".backup /data/backups/webui-$(date +\%Y\%m\%d).db"
```

**PostgreSQL with pg_dump:**
```bash
pg_dump -h postgres-host -U openwebui -Fc openwebui > backup.dump
```

## Custom Authentication Implementation

For organizations requiring custom auth beyond OAuth/LDAP:

1. **Create an authenticating reverse proxy** that:
   - Handles your custom authentication
   - Sets trusted headers for Open WebUI

2. **Configure Open WebUI:**
```bash
docker run \
  -e WEBUI_AUTH_TRUSTED_EMAIL_HEADER=X-Authenticated-Email \
  -e WEBUI_AUTH_TRUSTED_NAME_HEADER=X-Authenticated-Name \
  ghcr.io/open-webui/open-webui:main
```

3. **Security considerations:**
   - Ensure proxy strips incoming headers to prevent spoofing
   - Use mTLS between proxy and Open WebUI in untrusted networks

## Custom Model Configuration

### Adding Custom Models via API

For automated model deployment pipelines:

```bash
curl -X POST http://localhost:3000/api/models/create \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "custom-assistant",
    "model": "llama3.1:latest",
    "system": "You are a specialized coding assistant...",
    "temperature": 0.7
  }'
```

## Performance Monitoring

### Prometheus Metrics Endpoint

Enable metrics for monitoring:

```bash
docker run \
  -e ENABLE_METRICS=true \
  -e METRICS_PORT=9090 \
  ghcr.io/open-webui/open-webui:main
```

Access metrics at `http://localhost:9090/metrics`
