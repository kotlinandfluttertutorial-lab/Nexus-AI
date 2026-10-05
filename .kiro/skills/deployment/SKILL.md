# Skill: Deployment & Production AI Applications

## Purpose

Provides reusable engineering expertise for containerizing Nexus AI backend services,
deploying to Google Cloud Run, configuring environments and secrets, implementing health checks,
structured logging, and validating release and rollback procedures.

## When to Use

Activate this skill when:
- Writing Dockerfiles for FastAPI or Flask services
- Creating Docker Compose configurations for local development
- Configuring Cloud Run services (service YAML, CPU/memory, scaling)
- Wiring secrets via Google Secret Manager or environment variables
- Implementing `/health` and `/readiness` endpoints
- Setting up structured JSON logging for Cloud Logging
- Writing or reviewing CI/CD deployment pipelines
- Documenting rollback and environment promotion procedures

## Core Rules

1. **Secrets come from Secret Manager or environment injection — never from source code or image layers.**
2. **Containers are stateless.** No local model files, uploaded documents, or session state stored inside the container. Use Cloud Storage or a managed database.
3. **Health and readiness endpoints are mandatory.** Every service exposes `GET /health` (liveness) and `GET /readiness` (checks dependencies).
4. **Structured JSON logging.** All log output is newline-delimited JSON with `severity`, `message`, `timestamp`, and `service` fields — consumed by Cloud Logging.
5. **Minimum privilege.** Cloud Run service accounts have only the permissions they need (Secret Manager accessor, Cloud Storage object user, etc.).
6. **Image tags are immutable.** Production deployments always reference an exact image SHA or a versioned tag — never `latest`.
7. **Rollback is one command.** Every release is reversible via `gcloud run deploy` with the previous image tag.

## Project Structure

```
infrastructure/
├── docker/
│   ├── fastapi.Dockerfile
│   ├── flask.Dockerfile
│   └── docker-compose.yml        — Local multi-service development
├── cloud-run/
│   ├── fastapi-service.yaml      — Cloud Run service definition
│   └── flask-service.yaml
├── monitoring/
│   ├── alerts.yaml               — Cloud Monitoring alert policies
│   └── dashboard.json            — Cloud Monitoring dashboard
├── ci-cd/
│   ├── build-push.yml            — GitHub Actions: build + push image
│   └── deploy.yml                — GitHub Actions: deploy to Cloud Run
└── deployment/
    ├── environments/
    │   ├── local.env.example
    │   ├── staging.env.example
    │   └── production.env.example
    └── runbook.md                — Deployment and rollback procedures
```

## Implementation Patterns

### Dockerfile (FastAPI)

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies first for layer caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Non-root user
RUN adduser --disabled-password --gecos "" appuser
USER appuser

EXPOSE 8080

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080", "--workers", "1"]
```

### Health and Readiness Endpoints

```python
from fastapi import APIRouter, status
from fastapi.responses import JSONResponse

router = APIRouter(tags=["health"])

@router.get("/health")
async def health():
    """Liveness probe — is the process alive?"""
    return {"status": "ok"}

@router.get("/readiness")
async def readiness(provider_service: ProviderService = Depends()):
    """Readiness probe — are dependencies available?"""
    checks = {"llm_provider": await provider_service.ping()}
    all_healthy = all(checks.values())
    code = status.HTTP_200_OK if all_healthy else status.HTTP_503_SERVICE_UNAVAILABLE
    return JSONResponse({"status": "ready" if all_healthy else "degraded", "checks": checks}, status_code=code)
```

### Structured JSON Logging

```python
import logging
import json
import sys
from datetime import datetime, timezone

class CloudLoggingFormatter(logging.Formatter):
    SERVICE_NAME = "nexus-ai-fastapi"

    def format(self, record: logging.LogRecord) -> str:
        severity_map = {
            logging.DEBUG: "DEBUG",
            logging.INFO: "INFO",
            logging.WARNING: "WARNING",
            logging.ERROR: "ERROR",
            logging.CRITICAL: "CRITICAL",
        }
        return json.dumps({
            "severity": severity_map.get(record.levelno, "DEFAULT"),
            "message": record.getMessage(),
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "service": self.SERVICE_NAME,
            "logger": record.name,
        })

def configure_logging():
    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(CloudLoggingFormatter())
    logging.basicConfig(handlers=[handler], level=logging.INFO)
```

### Cloud Run Service YAML

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: nexus-ai-fastapi
  annotations:
    run.googleapis.com/ingress: all
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "10"
    spec:
      serviceAccountName: nexus-ai-sa@PROJECT_ID.iam.gserviceaccount.com
      containers:
        - image: gcr.io/PROJECT_ID/nexus-ai-fastapi:IMAGE_TAG
          ports:
            - containerPort: 8080
          resources:
            limits:
              cpu: "2"
              memory: "1Gi"
          env:
            - name: OPENAI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: openai-api-key
                  key: latest
```

### GitHub Actions Deploy

```yaml
- name: Deploy to Cloud Run
  run: |
    gcloud run deploy nexus-ai-fastapi \
      --image gcr.io/$PROJECT_ID/nexus-ai-fastapi:$IMAGE_TAG \
      --region $REGION \
      --platform managed \
      --no-traffic   # deploy without shifting traffic
    gcloud run services update-traffic nexus-ai-fastapi \
      --to-latest \
      --region $REGION
```

### Rollback

```bash
# List previous revisions
gcloud run revisions list --service nexus-ai-fastapi --region us-central1

# Rollback to a specific revision
gcloud run services update-traffic nexus-ai-fastapi \
  --to-revisions nexus-ai-fastapi-00042-abc=100 \
  --region us-central1
```

## Environment Configuration

| Environment | Configuration Source |
|---|---|
| Local | `.env` file (git-ignored) |
| Staging | Cloud Run env vars + Secret Manager |
| Production | Cloud Run env vars + Secret Manager |

Never commit `.env` files. Provide `.env.example` with placeholder values and documentation.

## Testing Guidance

- Test health and readiness endpoints in integration tests — assert `200` and response shape
- Test structured logging by capturing stdout and asserting valid JSON output
- Test deployment scripts in CI with a staging Cloud Run project before promoting to production
- Verify rollback by deploying a known-bad image to staging and confirming rollback restores health

## Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Storing secrets in Dockerfile `ENV` | Use Secret Manager and mount at runtime |
| Using `latest` image tag in production | Pin to SHA or versioned tag |
| Running container as root | Add `adduser` and `USER` in Dockerfile |
| Missing readiness endpoint | All services need `/readiness` for load-balancer health checks |
| Not setting `--no-traffic` on deploy | Deploy to new revision first; shift traffic after validation |
| Logging sensitive data | Never log API keys, tokens, or user content |

## Relationship to Steering

This skill governs `infrastructure/`. `05-security-standards.md` takes precedence on all secrets and credential handling decisions.
