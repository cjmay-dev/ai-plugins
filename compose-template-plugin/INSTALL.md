# Marketplace Configuration (Example)

This directory contains an example marketplace configuration for installing the compose-template plugin.

## Usage

To use this plugin with Claude Code:

### Option 1: Local Plugin Directory
```bash
# Use the plugin directly from this directory
claude --plugin-dir /path/to/compose-template-plugin
```

### Option 2: Create a Marketplace File

Create a `marketplace.json` file with the following structure:

```json
{
  "marketplaces": [
    {
      "name": "cjmay-dev-plugins",
      "source": "https://github.com/cjmay-dev/ai-plugins",
      "plugins": [
        {
          "name": "compose-template",
          "source": "./compose-template-plugin",
          "version": "1.0.0",
          "description": "Plugin for developing with compose-template",
          "author": "cjmay-dev",
          "keywords": ["docker", "compose", "terraform", "infisical"]
        }
      ]
    }
  ]
}
```

Then install with:
```bash
claude plugin install compose-template@cjmay-dev-plugins
```

### Option 3: Direct GitHub Installation

If you add the marketplace to your Claude Code settings:

```bash
# Add marketplace (one time)
claude marketplace add cjmay-dev-plugins https://raw.githubusercontent.com/cjmay-dev/ai-plugins/main/marketplace.json

# Install plugin
claude plugin install compose-template
```

## Testing Locally

To test the plugin locally during development:

1. Navigate to this repository
2. Run Claude Code with the plugin:
   ```bash
   claude --plugin-dir ./compose-template-plugin
   ```
3. Try invoking a skill:
   ```
   /compose-init
   ```
4. Or let Claude automatically invoke the compose-template-expert agent

## Plugin Components

The plugin provides:

- **5 Skills**: `/compose-init`, `/compose-local`, `/compose-infra`, `/compose-deploy`, `/compose-backup`
- **1 Agent**: `compose-template-expert` - automatically invoked for compose-template projects
- **Documentation**: Comprehensive guides for each workflow

## Updating the Plugin

When you update the plugin:

1. Increment version in `.claude-plugin/plugin.json`
2. Update CHANGELOG (if you create one)
3. Commit and push changes
4. Users can update with: `claude plugin update compose-template`
