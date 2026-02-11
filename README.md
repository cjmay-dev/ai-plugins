# AI Plugins

A collection of Claude Code plugins for various development workflows and tools.

## Available Plugins

### compose-template

A plugin for developing applications using the [cjmay-dev/compose-template](https://github.com/cjmay-dev/compose-template).

**Features:**
- Skills for common compose-template workflows (init, deploy, backup, etc.)
- Specialized agent with deep compose-template knowledge
- Guides for Terraform infrastructure, Docker Compose deployment, and Infisical secrets

**Installation:**
```bash
claude plugin install compose-template

# Or use locally
claude --plugin-dir ./compose-template-plugin
```

See [compose-template-plugin/README.md](compose-template-plugin/README.md) for detailed documentation.

## About Claude Code Plugins

These plugins extend Claude Code with specialized skills and agents. Learn more about Claude Code plugins in the [official documentation](https://code.claude.com/docs/en/plugins-reference).

## License

MIT
