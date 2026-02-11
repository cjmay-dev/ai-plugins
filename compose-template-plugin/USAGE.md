# Using the compose-template Plugin

This guide shows how to use the compose-template Claude Code plugin.

## Quick Start

### Installation

**Method 1: Using the plugin directory directly**
```bash
claude --plugin-dir ./compose-template-plugin
```

**Method 2: Via marketplace (if configured)**
```bash
claude marketplace add cjmay-dev https://raw.githubusercontent.com/cjmay-dev/ai-plugins/main/marketplace.json
claude plugin install compose-template
```

## Example Workflows

### Creating a New App

1. **Start Claude Code with the plugin**
   ```bash
   cd your-workspace
   claude --plugin-dir /path/to/compose-template-plugin
   ```

2. **Initialize a new app**
   ```
   I need to create a new application using compose-template.
   The app will be a blog platform.
   ```
   
   Claude will:
   - Invoke the compose-template-expert agent
   - Guide you through creating the repository
   - Help you set up the initial structure
   - Can use `/compose-init` skill for detailed steps

3. **Set up local development**
   ```
   /compose-local
   ```
   
   This skill will guide you through:
   - Installing required tools
   - Running `local-init.sh`
   - Setting up Infisical authentication

### Deploying Infrastructure

```
I need to deploy the infrastructure for my compose-blog app.
```

The agent will:
- Explain the infrastructure deployment process
- Can invoke `/compose-infra` for detailed Terraform steps
- Guide you through reviewing and applying changes

### Customizing the Application

```
I need to add a PostgreSQL database and a Redis cache to my compose app.
```

The agent will:
- Show you how to modify `docker-compose.yaml`
- Configure Traefik labels appropriately
- Guide you to add necessary secrets to Infisical
- Help deploy with `/compose-deploy`

### Configuring Backups

```
I want to back up my database every 6 hours and keep backups for 30 days.
```

The agent will:
- Explain stack-back configuration
- Can invoke `/compose-backup` for detailed settings
- Show you how to customize retention policies

## Skills Reference

### /compose-init
Initialize a new compose-template application. Use when:
- Starting a new project
- Need to understand initial setup
- Want guidance on repository structure

### /compose-local
Set up local development environment. Use when:
- First time working with a compose-template project
- Switching between environments (dev/prod)
- Need to configure Infisical or Terraform locally

### /compose-infra
Deploy and manage infrastructure with Terraform. Use when:
- Deploying infrastructure for first time
- Modifying existing infrastructure
- Troubleshooting infrastructure issues
- Need to understand deployed resources

### /compose-deploy
Deploy the Docker Compose application. Use when:
- Deploying application for first time
- Updating existing application
- Configuring Traefik routing
- Managing secrets and environment variables

### /compose-backup
Configure backups with stack-back. Use when:
- Setting up backup schedules
- Customizing what gets backed up
- Configuring retention policies
- Need to restore from backup

## Agent Usage

The **compose-template-expert** agent is automatically invoked when:
- You mention compose-template in your prompt
- Working in a compose-template repository
- Discussing Docker Compose, Terraform, or Infisical in the context of deployment

You can also explicitly invoke it:
```
@compose-template-expert how do I configure multi-stage backups?
```

## Common Scenarios

### Scenario 1: New Application from Scratch

**You say:**
```
I need to create a new web application using compose-template.
It will be a simple blog with a database.
```

**Claude (with plugin) will:**
1. Invoke compose-template-expert agent
2. Guide you through repository creation
3. Help customize docker-compose.yaml
4. Show you how to add secrets
5. Deploy the infrastructure and application
6. Set up automated backups

### Scenario 2: Debugging Deployment Issues

**You say:**
```
My compose app won't start. The web service keeps restarting.
```

**Claude (with plugin) will:**
1. Ask for logs: `docker-compose logs web`
2. Check for common issues:
   - Missing environment variables
   - Incorrect Traefik labels
   - Port conflicts
3. Guide you through verification steps
4. Help fix the configuration

### Scenario 3: Adding a New Service

**You say:**
```
I need to add Redis to my existing compose-template app for caching.
```

**Claude (with plugin) will:**
1. Show you how to add Redis to docker-compose.yaml
2. Configure networking correctly
3. Update secrets in Infisical if needed
4. Deploy with `/compose-deploy`
5. Verify the service is running

### Scenario 4: Infrastructure Modifications

**You say:**
```
I need to change the backup bucket configuration for my app.
```

**Claude (with plugin) will:**
1. Guide you to the Terraform configuration
2. Show what needs to be modified
3. Help you plan and apply changes with `/compose-infra`
4. Update stack-back configuration if needed

## Tips for Best Results

1. **Be specific about the environment**
   ```
   I'm working on the dev environment for my compose-blog app.
   ```

2. **Mention the stage of your project**
   ```
   I just created the repository from the template and need to deploy infrastructure.
   ```

3. **Ask for explanations when needed**
   ```
   Why do I need to use Infisical CLI for deployment?
   ```

4. **Request best practices**
   ```
   What are the best practices for managing secrets in compose-template?
   ```

5. **Get troubleshooting help**
   ```
   My Terraform deployment failed with this error: [paste error]
   ```

## Getting Help

The plugin provides comprehensive documentation:
- Each skill has detailed guides
- The agent has deep knowledge of compose-template
- All documentation is contextually available

Just ask naturally:
```
How do I set up local development?
What's the best way to configure backups?
Can you explain how Traefik routing works in compose-template?
```

## Integration with Claude Code Features

The plugin integrates seamlessly with Claude Code:
- **Automatic skill invocation**: Claude uses skills when appropriate
- **Agent expertise**: The compose-template-expert provides specialized knowledge
- **Context awareness**: Works with your local files and git repository
- **Tool integration**: Uses bash, git, and other tools as needed

## Examples of Plugin in Action

### Example 1: Complete Setup Flow

```
User: I want to create a new Minecraft server using compose-template.

Claude: I'll help you create a Minecraft server using compose-template! 
        Let me guide you through the process.

        [Invokes compose-template-expert agent]
        
        First, let's create the repository. Since it will be a Minecraft 
        server, I recommend naming it "compose-minecraft". This will make 
        your server accessible at minecraft.yourdomain.com.
        
        Here's what we'll do:
        1. Create repo from template
        2. Wait for GitHub Actions to deploy infrastructure
        3. Customize docker-compose.yaml for Minecraft
        4. Deploy the server
        
        Would you like to proceed? I can help with each step.

User: Yes, let's start. I already created the repo.

Claude: Great! Let's set up your local development environment.
        
        [Invokes /compose-local skill]
        
        Run these commands:
        ```bash
        git clone <your-repo-url>
        cd compose-minecraft
        ./local-init.sh app
        source .env.local
        ```
        ...
```

### Example 2: Troubleshooting

```
User: My app is showing 502 Bad Gateway

Claude: [Invokes compose-template-expert agent]
        
        A 502 Bad Gateway usually means Traefik can't reach your backend.
        Let's check a few things:
        
        1. First, verify your containers are running:
        ```bash
        docker ps
        ```
        
        2. Check the Traefik logs:
        ```bash
        docker logs traefik | tail -50
        ```
        
        3. Verify your Traefik labels are correct...
        ...
```

## Advanced Usage

### Combining Skills

You can chain skills for complex workflows:
```
/compose-init    # Initialize project
/compose-local   # Set up local env
/compose-infra   # Deploy infrastructure
/compose-deploy  # Deploy application
/compose-backup  # Configure backups
```

### Custom Workflows

The agent can help create custom workflows:
```
I need a deployment checklist for my production environment.
```

The agent will create a tailored checklist based on compose-template best practices.

## Feedback and Issues

If you encounter issues with the plugin:
1. Check the plugin is loaded: Use `/debug` in Claude Code
2. Verify plugin structure is correct
3. Review Claude Code plugin documentation
4. Report issues to the repository

## Next Steps

- Try the plugin with a test project
- Explore all five skills
- Let the agent guide you through complex tasks
- Customize the plugin for your specific needs

Happy coding with compose-template! 🚀
