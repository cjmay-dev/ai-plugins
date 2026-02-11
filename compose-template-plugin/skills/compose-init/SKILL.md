---
name: compose-init
description: Initialize a new application from the compose-template with Docker Compose, Terraform infrastructure, Infisical secrets, and automated backups
---

# Initialize Compose Template App

Initialize a new application from the compose-template. This skill helps set up the basic project structure for a new Docker Compose application with built-in backups, secrets management, and public access.

## What This Skill Does

This skill guides the process of creating a new application from the compose-template:

1. Creates or uses an existing repository based on the compose-template
2. Ensures the repository follows naming conventions (repo name becomes subdomain)
3. Verifies the basic template files are present
4. Provides guidance on next steps

## Usage

Invoke this skill when:
- Starting a new compose-template application
- Converting an existing project to use compose-template
- Need help understanding the initial setup process

## Template Repository Structure

The compose-template includes:

```
compose-template/
├── .github/
│   └── workflows/          # GitHub Actions for automated deployment
├── terraform/              # Infrastructure as code
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── ansible/               # Configuration management
│   ├── playbook.yml
│   └── roles/
├── docker-compose.yaml    # Main compose file (customize for your app)
├── Makefile              # Convenience commands
├── local-init.sh         # Local development setup script
└── README.md             # Template documentation
```

## Key Concepts

### Repository Naming
The repository name determines the subdomain where your app will be accessible:
- `compose-minecraft` → `minecraft.yourdomain.com`
- `compose-blog` → `blog.yourdomain.com`
- Any "compose-" prefix is automatically stripped

### Infrastructure Components
The template automatically deploys:
- Ubuntu server with Docker
- Backblaze B2 bucket for backups
- Backblaze B2 app key
- Cloudflare Tunnel for public access
- DNS record pointing to the tunnel
- Infisical project for secrets management

## Steps to Initialize

1. **Create Repository**
   ```bash
   # Create a new repository from the template on GitHub
   # Name it according to desired subdomain (e.g., compose-myapp)
   ```

2. **Clone Repository**
   ```bash
   git clone <repository-url>
   cd <repository-name>
   ```

3. **Verify Template Files**
   Check that these essential files exist:
   - `docker-compose.yaml`
   - `Makefile`
   - `local-init.sh`
   - `terraform/` directory
   - `ansible/` directory

4. **Wait for GitHub Actions**
   - GitHub Actions will automatically deploy infrastructure
   - Check Actions tab to monitor deployment progress
   - Alternatively, deploy manually using local development setup

5. **Customize for Your App**
   Once initialized, you'll need to:
   - Modify `docker-compose.yaml` with your app containers
   - Set Traefik labels on frontend container
   - Add app-specific secrets to Infisical project
   - Update README.md with app documentation

## Next Steps

After initialization:
- Use `/compose-local` to set up local development
- Use `/compose-infra` to manually deploy or modify infrastructure
- Use `/compose-deploy` to deploy the application
- Use `/compose-backup` to configure backup settings

## Important Notes

- This template requires pre-existing infrastructure (see cjmay.dev for details)
- Infisical CLI is used for just-in-time secret injection
- Secrets never touch the disk on the server (except container volumes)
- The template uses git submodules for easy updates
- Default backup schedule backs up all volumes and databases

## Troubleshooting

**GitHub Actions Not Running**
- Verify repository secrets are configured
- Check workflow files in `.github/workflows/`

**Template Files Missing**
- Ensure you created from template, not cloned
- Verify submodules are initialized: `git submodule update --init --recursive`

**Naming Issues**
- Repository name should not contain special characters
- Subdomain will be lowercase regardless of repo name case
