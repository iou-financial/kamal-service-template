# Kamal Service Template

This is a template repo for onboarding new Rails services to Kamal staging at IOU Financial.

## Placeholders

Replace these placeholders throughout all files:
- `__SERVICE__` — service name (lowercase, e.g. `delphi`, `probono`)
- `__ELASTIC_IP__` — EC2 Elastic IP from terraform output
- `__SERVICE_UPPER__` — uppercase env var prefix (e.g. `DELPHI`, `PROBONO`)

## Architecture

- **Hosting**: EC2 instances managed by Terraform (`~/git/terraform/staging/kamal/`)
- **Deploy**: Kamal 2 with kamal-proxy (SSL via Let's Encrypt)
- **Registry**: ghcr.io/iou-financial
- **Secrets**: AWS Secrets Manager (us-east-1, staging account 566883448644)
- **CI**: GitHub Actions deploys on push to `develop`
- **DNS**: Route53 — `*.staging.ioufinancial.com`

## Key conventions

- Healthcheck path: `/up` (Rails default)
- Puma port: 3000
- SSH user: `ubuntu`
- Builder: local (no remote builder)
- Docker cleanup runs in post-deploy hook
- Slack notifications on build, deploy, and failure via shared webhook
- KAMAL_REGISTRY_PASSWORD is shared across all services: `nucleus/staging/KAMAL_REGISTRY_PASSWORD`
- Service secrets go under `SERVICE_NAME/staging/*` in Secrets Manager
- Deploy branch: `develop` for staging
- Hooks must be executable (`chmod +x`)

## Onboarding steps

See README.md for the full step-by-step guide.

## Common patterns

- **Web only** (no workers): Use `allow_empty_roles: true` in staging destination
- **Workers**: Uncomment the workers role in deploy.yml, set cmd
- **MySQL instead of Postgres**: Change `postgresql-client` to `default-mysql-client` in Dockerfile, `libpq-dev` to `default-libmysqlclient-dev` in build stage
- **Dual DB** (Postgres + MySQL): Include both clients and dev packages (see delphi)
- **RDS IAM auth**: Add `curl` for CA bundle download in Dockerfile final stage
