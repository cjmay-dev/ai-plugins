# Local Development Setup

Set up a local development environment for working with a compose-template application. This skill helps configure the necessary tools and environment for local development, testing, and deployment.

## What This Skill Does

This skill guides the setup of a local development environment:

1. Installs and configures required tools (Terraform, Infisical CLI)
2. Initializes Infisical environment configuration
3. Sets up environment variables for local work
4. Provides workflows for infrastructure and application development

## Usage

Invoke this skill when:
- Setting up development environment for first time
- Switching between development environments
- Need to work locally on infrastructure or compose app
- Troubleshooting environment issues

## Prerequisites

Before starting local development, ensure you have:
- Git installed
- Access to the compose-template repository
- Infisical account and project access
- (For infrastructure work) Terraform installed
- (For app deployment) Ansible installed

## Local Initialization Script

The template provides `local-init.sh` for environment setup:

```bash
./local-init.sh <mode>
```

**Modes:**
- `infra` - Set up for Terraform infrastructure development
- `app` - Set up for Ansible/Docker Compose development

## Steps for Infrastructure Development

1. **Initialize Local Environment**
   ```bash
   ./local-init.sh infra
   ```
   This will:
   - Prompt for Infisical credentials
   - Create `.infisical.json` configuration
   - Set up local environment for "dev" environment by default

2. **Load Environment Variables**
   ```bash
   source .env.local
   ```
   This loads secrets and configurations needed for Terraform

3. **Work with Terraform**
   ```bash
   # Initialize Terraform
   make tf-init
   
   # Plan infrastructure changes
   make tf-plan
   
   # Apply infrastructure changes
   make tf-apply
   ```

4. **Clean Up Secrets**
   ```bash
   make lock
   ```
   This removes `.env.local` and logs out of Infisical CLI

## Steps for App Development

1. **Initialize Local Environment**
   ```bash
   ./local-init.sh app
   ```
   Similar to infra mode but configured for app deployment

2. **Load Environment Variables**
   ```bash
   source .env.local
   ```

3. **Configure Docker Host**
   ```bash
   # Run Ansible playbook to configure the server
   make configure
   ```

4. **Deploy Compose App**
   ```bash
   # Deploy the docker-compose.yaml to the host
   make compose
   ```
   Note: This uses Infisical CLI to inject secrets just-in-time

5. **Clean Up Secrets**
   ```bash
   make lock
   ```

## Environment Configuration

### .infisical.json
Created on first run of `local-init.sh`, contains:
```json
{
  "workspaceId": "...",
  "environment": "dev",
  "gitBranchToEnvironmentMapping": null
}
```

To use a different environment (e.g., "prod"), edit this file and re-run `local-init.sh`.

### .env.local
Generated each time you run `local-init.sh`, contains:
- Infisical project secrets
- Environment-specific variables
- Credentials for services (B2, Cloudflare, etc.)

**Important:** This file should NEVER be committed. It's automatically in `.gitignore`.

## Makefile Commands

The template provides these convenience commands:

**Infrastructure:**
- `make tf-init` - Initialize Terraform
- `make tf-plan` - Plan infrastructure changes
- `make tf-apply` - Apply infrastructure changes
- `make tf-destroy` - Destroy infrastructure (use with caution!)

**Application:**
- `make configure` - Run Ansible playbook to configure host
- `make compose` - Deploy compose app with secret injection
- `make lock` - Remove local secrets and logout

## Security Best Practices

1. **Never commit secrets**
   - `.env.local` is gitignored
   - Secrets are injected just-in-time by Infisical CLI

2. **Lock after each session**
   - Always run `make lock` when done
   - Prevents secrets from sitting on development machine

3. **Use appropriate environments**
   - Use "dev" for development work
   - Use "prod" carefully and only when necessary
   - Never test destructive operations in "prod"

4. **Verify loaded environment**
   ```bash
   # Check which environment is active
   echo $ENVIRONMENT
   ```

## Working with Multiple Environments

To switch environments:

1. Edit `.infisical.json` and change "environment" field
2. Re-run `local-init.sh` with appropriate mode
3. Source the new `.env.local`
4. Verify environment is correct before operations

## Troubleshooting

**Infisical CLI Not Found**
```bash
# Install Infisical CLI
# See: https://infisical.com/docs/cli/overview
```

**Terraform Not Found**
```bash
# Install Terraform
# See: https://developer.hashicorp.com/terraform/downloads
```

**Permission Denied on local-init.sh**
```bash
chmod +x local-init.sh
```

**Secrets Not Loading**
- Verify Infisical project access
- Check `.infisical.json` configuration
- Re-run `local-init.sh`
- Ensure you've sourced `.env.local`

**Wrong Environment Active**
- Check contents of `.infisical.json`
- Edit environment field
- Re-run `local-init.sh`

## Next Steps

After local setup:
- Use `/compose-infra` for infrastructure work
- Use `/compose-deploy` for application deployment
- Use `/compose-backup` for backup configuration
