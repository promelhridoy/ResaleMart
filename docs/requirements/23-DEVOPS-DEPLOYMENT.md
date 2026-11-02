# 23 - DevOps and Deployment

## Environments
local (docker-compose), dev, staging (production-like, sandbox gateways), production.

## Local stack (docker-compose)
mongo (replica set rs0, single node), redis, minio (S3), mailhog, api, worker, web. Script `pnpm dev` starts all; `pnpm seed`.

## Containers
Multi-stage Dockerfiles, non-root user, healthcheck endpoints `/health/live`, `/health/ready` (DB, Redis).

## Production topology (v1)
- Web: Vercel or container behind CDN.
- API: >= 2 containers (e.g. AWS ECS/Fargate, or VPS cluster with Docker + Nginx/Traefik) behind load balancer, autoscale on CPU/latency.
- Worker: >= 2 containers, separate queues.
- MongoDB Atlas M30+ (3-node replica set, backups, PITR) in Singapore/Mumbai.
- Redis managed (ElastiCache/Upstash/Redis Cloud) with persistence; separate instance for queues vs cache if scale demands.
- Object storage: S3 + CloudFront (public listing images) and private bucket for NID/evidence.
- Secrets: AWS Secrets Manager / Doppler / Vault.
- WAF + rate limiting at edge (Cloudflare or AWS WAF).
- Data residency: confirm legal requirement for storing citizen data in-country; if required, host private data in a BD data centre.

## CI/CD (GitHub Actions)
PR: install, lint, typecheck, unit, integration (services), build, security scan, preview deploy.
main: build images, push, deploy staging, run E2E smoke, manual approval, deploy production (rolling/blue-green), run DB migrations (idempotent scripts via migrate-mongo), post-deploy smoke.
Rollback: previous image tag one-click; migrations backward compatible (expand/contract).

## Observability
- Logs: JSON to CloudWatch/Loki; correlation by requestId.
- Metrics: request rate/latency/error, queue depth/latency, socket connections, DB ops, ledger imbalance gauge (must be 0), payment success rate, webhook failures.
- Tracing: OpenTelemetry.
- Errors: Sentry (web + api + worker).
- Uptime: external probes for web, API, checkout path.
- Alerts (PagerDuty/Slack): API 5xx > 2%, payment webhook failures, reconciliation mismatch, DLQ non-empty, queue lag > 2 min, DB replication lag, disk > 80%, certificate expiry, SMS spend spike.

## Backups and DR
Daily snapshot + continuous oplog (PITR), 35-day retention, encrypted, cross-region copy; monthly restore drill; object storage versioning on private bucket; runbooks for DB failover, Redis loss (queues replay from outbox), gateway outage (disable checkout banner).

## Release and ops
Feature flags in `system_config`; maintenance mode switch; migration playbooks; on-call rota; incident severity levels and postmortem template; monthly cost review; capacity review before campaigns.

## Capacity planning (starting point for 100k users)
Assume 10% DAU (10k), peak 1k RPS reads with cache; 2-4 API instances (2 vCPU/4 GB), 2 workers, Mongo M30-M40, Redis 2-5 GB. Load-test and adjust before launch.
