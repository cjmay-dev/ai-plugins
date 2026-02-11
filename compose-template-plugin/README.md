# Compose Template Plugin

A Claude Code plugin to help agents develop applications using the [cjmay-dev/compose-template](https://github.com/cjmay-dev/compose-template).

## About compose-template

The compose-template is a project template that uses Terraform and Docker Compose to deploy applications with:

- **Automated backups** using [stack-back](https://github.com/lawndoc/stack-back)
- **Public access** via [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
- **Just-in-time secret injection** using [Infisical](https://infisical.com)

The template deploys minimal containers (<50MB memory, <0.5GB disk) and handles both infrastructure (Terraform) and application deployment (Docker Compose + Ansible).

## Installation

Install this plugin to add compose-template development capabilities to Claude Code:

```bash
# Install from a marketplace
claude plugin install compose-template

# Or install locally
claude --plugin-dir /path/to/compose-template-plugin
```

## What This Plugin Provides

### Skills

The plugin provides the following skills for working with compose-template projects:

- `/compose-init` - Initialize a new compose app from the template
- `/compose-local` - Set up local development environment
- `/compose-infra` - Deploy or manage infrastructure with Terraform
- `/compose-deploy` - Deploy the compose application
- `/compose-backup` - Configure backup settings with stack-back

### Agent

A specialized **compose-template-expert** agent that understands the template's architecture, workflow patterns, and best practices. Claude will automatically invoke this agent when working on compose-template projects.

## Usage

Once installed, you can use the skills directly:

```
/compose-init my-app
```

Or let Claude automatically use them and invoke the compose-template-expert agent when needed.

## Template Workflow

The typical workflow when using compose-template:

1. **Create repo from template** - The repo name becomes the subdomain (e.g., compose-app → app.yourdomain.com)
2. **Infrastructure deployment** - GitHub Actions or manual Terraform deployment
3. **Modify docker-compose.yaml** - Add your application containers
4. **Configure Traefik labels** - Set up routing on the frontend container
5. **Add secrets** - Configure app secrets in Infisical
6. **Deploy** - Ansible deploys and configures the app with secrets injected

## Local Development

For local development:

1. Install Terraform and Infisical CLI
2. Run `./local-init.sh <infra|app>`
3. Run `source .env.local`
4. Use `make` commands for operations:
   - `make tf-init`, `make tf-plan`, `make tf-apply` - Terraform
   - `make configure` - Ansible configuration
   - `make compose` - Deploy compose app
   - `make lock` - Remove local secrets

## License

MIT
