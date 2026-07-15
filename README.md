# Kamal New Service Template

Template files for adding a new service to Kamal staging.

## Steps

### 1. Terraform — add EC2 instance

In `~/git/terraform/staging/kamal/terraform.tfvars`, add the instance:

```hcl
"SERVICE_NAME" = {
  instance_type = "t3.small"   # adjust based on resource needs
  volume_size   = 60
}
```

Run:
```bash
cd ~/git/terraform/staging/kamal
tf init -upgrade
tf plan
tf apply
```

Note the Elastic IP from the output.

### 2. Add SSH key to instance

The new instance only has the `staging` keypair. Add your iou key so Kamal SSH doesn't fail (agent has 6+ keys):

```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/staging.pem ubuntu@<ELASTIC_IP> \
  'cat >> ~/.ssh/authorized_keys' < ~/.ssh/iou.pub
```

### 3. Copy template files into your repo

Copy these files into your service repo and replace all `__SERVICE__` placeholders:

- `config/deploy.yml` — base Kamal config
- `config/deploy.staging.yml` — staging destination
- `.docker/Dockerfile.kamal` — production Docker image
- `.kamal/secrets.staging` — secrets from AWS Secrets Manager
- `.kamal/hooks/pre-build` — Slack: building
- `.kamal/hooks/pre-deploy` — Slack: deploying
- `.kamal/hooks/post-deploy` — Slack: deployed + docker cleanup
- `.kamal/hooks/deploy-failed` — Slack: deploy failed
- `.github/workflows/deploy-staging.yml` — CI auto-deploy

### 4. Create secrets in AWS Secrets Manager

```bash
export AWS_PROFILE=staging

# At minimum you need SECRET_KEY_BASE
aws secretsmanager create-secret \
  --name "SERVICE_NAME/staging/SECRET_KEY_BASE" \
  --secret-string "$(openssl rand -hex 64)" \
  --region us-east-1

# Add other secrets as needed (DATABASE_URL, API keys, etc.)
```

### 5. Deploy

```bash
cd ~/git/SERVICE_NAME
export AWS_PROFILE=staging
kamal deploy -d staging
```

### 6. Verify

```bash
curl https://SERVICE_NAME.staging.ioufinancial.com/up
```

### 7. Add to kamal-migration-plan.md

Add deploy steps for the service in `~/gitlab/iou-admin-tasks/kamal/kamal-migration-plan.md`.
