---
name: compose-infra
description: Deploy and manage Terraform infrastructure for compose-template including Ubuntu server, B2 storage, Cloudflare Tunnel, DNS, and Infisical secrets
---

# Deploy Infrastructure with Terraform

Deploy and manage the infrastructure for a compose-template application using Terraform. This skill helps with infrastructure deployment, modification, and troubleshooting.

## What This Skill Does

This skill guides infrastructure deployment and management:

1. Deploying infrastructure using Terraform
2. Modifying infrastructure resources
3. Understanding deployed resources
4. Troubleshooting infrastructure issues

## Usage

Invoke this skill when:
- Deploying infrastructure for a new application
- Modifying existing infrastructure
- Troubleshooting infrastructure problems
- Need to understand what resources are deployed

## Deployed Resources

The compose-template Terraform code deploys:

### Compute
- **Ubuntu Server** with Docker installed
  - Configured via cloud-init
  - Ready for Docker Compose deployments

### Storage
- **Backblaze B2 Bucket** for backups
  - Stores volume and database backups
  - Created with unique name based on app

- **Backblaze B2 App Key**
  - Scoped to the specific bucket
  - Stored in Infisical for secure access

### Networking
- **Cloudflare Tunnel**
  - Provides secure public access to app
  - No need to expose ports on server

- **DNS Record**
  - Points subdomain to Cloudflare Tunnel
  - Based on repository name (e.g., compose-app → app.yourdomain.com)

### Secrets Management
- **Infisical Project**
  - Central secrets storage
  - Secrets for all deployed resources

- **Infisical Secrets**
  - B2 credentials
  - Cloudflare credentials
  - Server access details
  - Other app-specific secrets

## Infrastructure Workflow

### Automated Deployment (GitHub Actions)

By default, GitHub Actions handles infrastructure deployment:

1. Push to repository triggers workflow
2. Terraform runs automatically
3. Infrastructure is created/updated
4. Ansible configures the server
5. App is deployed with secrets

**Monitoring GitHub Actions:**
- Go to repository → Actions tab
- Check workflow status
- View logs for any issues

### Manual Deployment

For manual control or local development:

1. **Set Up Local Environment**
   ```bash
   ./local-init.sh infra
   source .env.local
   ```

2. **Initialize Terraform**
   ```bash
   make tf-init
   ```
   This:
   - Downloads provider plugins
   - Initializes backend configuration
   - Prepares workspace

3. **Review Planned Changes**
   ```bash
   make tf-plan
   ```
   This shows:
   - Resources to be created
   - Resources to be modified
   - Resources to be destroyed
   - Estimated costs (if available)

4. **Apply Changes**
   ```bash
   make tf-apply
   ```
   - Applies planned changes
   - Outputs resource information
   - Updates state

5. **Clean Up**
   ```bash
   make lock
   ```

## Terraform Structure

The template's Terraform code is organized as:

```
terraform/
├── main.tf           # Main resource definitions
├── variables.tf      # Input variables
├── outputs.tf        # Output values
├── versions.tf       # Provider versions
└── terraform.tfvars  # Variable values (from Infisical)
```

## Common Infrastructure Tasks

### View Current State
```bash
terraform show
```

### List Resources
```bash
terraform state list
```

### Get Resource Details
```bash
terraform state show <resource_name>
```

### Refresh State
```bash
terraform refresh
```

### Validate Configuration
```bash
terraform validate
```

### Format Code
```bash
terraform fmt -recursive
```

## Modifying Infrastructure

When you need to change infrastructure:

1. **Edit Terraform Files**
   - Modify resources in `terraform/main.tf`
   - Update variables in `terraform/variables.tf`
   - Adjust outputs if needed

2. **Validate Changes**
   ```bash
   terraform validate
   terraform fmt
   ```

3. **Plan Changes**
   ```bash
   make tf-plan
   ```
   - Review planned changes carefully
   - Ensure no unexpected deletions

4. **Apply Changes**
   ```bash
   make tf-apply
   ```

5. **Test Application**
   - Verify app still works
   - Check all services are accessible

## Infrastructure Dependencies

The template assumes these pre-existing resources:
- Backblaze account and credentials
- Cloudflare account with domain
- Infisical workspace
- Cloud provider account (for Ubuntu server)

See cjmay.dev for details on setting up these prerequisites.

## Troubleshooting

### Terraform Init Fails
**Problem:** Provider download issues
**Solution:**
```bash
# Clear Terraform cache
rm -rf .terraform/
make tf-init
```

### Plan Shows Unexpected Changes
**Problem:** State drift or configuration mismatch
**Solution:**
```bash
# Refresh state
terraform refresh
# Review actual vs. desired state
terraform plan
```

### Apply Fails Midway
**Problem:** Resource creation error
**Solution:**
```bash
# Review error message
# Fix the issue (credentials, quotas, etc.)
# Re-run apply (Terraform will resume)
make tf-apply
```

### Resources Not Created
**Problem:** Missing credentials or permissions
**Solution:**
- Verify Infisical secrets are set
- Check cloud provider permissions
- Review Terraform logs for specific errors

### State Lock Errors
**Problem:** Previous operation didn't complete cleanly
**Solution:**
```bash
# Force unlock (use carefully!)
terraform force-unlock <lock-id>
```

### Cannot Destroy Resources
**Problem:** Dependencies or protection enabled
**Solution:**
- Remove resource dependencies first
- Disable deletion protection if enabled
- Use targeted destroy: `terraform destroy -target=<resource>`

## Environment Separation

The template supports multiple environments:
- **dev** - Development/testing infrastructure
- **prod** - Production infrastructure

To work with different environments:
1. Edit `.infisical.json` to set environment
2. Re-run `./local-init.sh infra`
3. Source `.env.local`
4. Run Terraform commands

**Important:** Always verify which environment is active before running destructive operations.

## Cost Considerations

The template is designed to be cost-effective:
- Ubuntu server: Small instance (varies by provider)
- B2 storage: Pay for what you use
- Cloudflare Tunnel: Free tier available
- Infisical: Free tier available

Monitor your usage to avoid unexpected costs.

## State Management

**Important:** Terraform state contains sensitive information.
- State is typically stored remotely (S3, Terraform Cloud, etc.)
- Never commit state files to Git
- Ensure state backend is properly configured
- Regular state backups are recommended

## Next Steps

After infrastructure is deployed:
- Use `/compose-deploy` to deploy the application
- Use `/compose-backup` to configure backups
- Monitor resource usage and costs
- Keep Terraform code updated with template improvements
