# Claude Home Assistant Plugin Marketplace

A collection of Claude plugins providing specialized skills for home automation and embedded systems configuration.

## Overview

This marketplace contains plugins that extend Claude Code functionality with expert skills for:

- **Home Assistant Configuration** - Create, modify, and troubleshoot Home Assistant YAML configuration files
- **ESPHome Configuration** - Configure ESP32/ESP8266 IoT devices with ESPHome firmware
- **ESPHome Display** - Write lambda code for ESPHome display components
- **ESPHome LVGL** - Configure LVGL (Light and Versatile Graphics Library) interfaces for ESPHome devices

## Available Plugins

| Plugin                 | Description                                                         | Version |
| ---------------------- | ------------------------------------------------------------------- | ------- |
| `homeassistant-config` | Create, modify, and troubleshoot Home Assistant configuration files | 1.0.0   |
| `esphome-config`       | Skill for ESPHome device configuration files                        | Latest  |
| `esphome-display`      | Skill for ESPHome display lambda code                               | Latest  |
| `esphome-lvgl`         | Skill for ESPHome LVGL config                                       | Latest  |

## Installation

### Prerequisites

- Claude Code installed on your machine
- Basic command-line familiarity

### Step 1: Add the Marketplace

Add this repository as a plugin marketplace source:

```bash
/plugin marketplace add claude-homeassistant-plugins https://github.com/narrowstacks/claude-homeassistant-plugins
```

### Step 2: Browse and Install Plugins

View available plugins and install interactively:

```bash
/plugin
```

Select "Browse Plugins" and choose the plugins you want to install.

Alternatively, install a specific plugin directly:

```bash
/plugin install homeassistant-config@claude-homeassistant-plugins
/plugin install esphome-config@claude-homeassistant-plugins
/plugin install esphome-display@claude-homeassistant-plugins
/plugin install esphome-lvgl@claude-homeassistant-plugins
```

### Step 3: Restart Claude Code

Restart Claude Code to activate the newly installed plugins. They will then be available in your workflow.

## Usage

Once installed, each plugin provides specialized skills that can be invoked during conversations:

- Ask Claude about Home Assistant configuration challenges
- Request help with ESPHome device configuration
- Get assistance writing display lambda code for ESPHome
- Configure LVGL interfaces for embedded displays

Each plugin includes detailed documentation and skill definitions to guide you.

## Plugin Development

Each plugin in this marketplace follows the Claude plugin structure:

- `plugin.json` - Plugin metadata (name, description, version, author)
- `SKILL.md` - Skill documentation and usage guidelines
- `README.md` - Plugin-specific installation and usage information
- `skills/` - Directory containing skill definitions

For more information on Claude plugin development, see the [Claude Code Plugins Documentation](https://docs.claude.com/en/docs/claude-code/plugins.md).

## Contributing

Contributions are welcome! To add a new plugin to this marketplace:

1. Create a new plugin directory following the plugin structure above
2. Add the plugin entry to `.claude-plugin/marketplace.json`
3. Include comprehensive documentation in your plugin's README.md and SKILL.md
4. Submit a pull request with your changes

## License

This marketplace is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Repository

- **Owner**: narrowstacks
- **Repository**: https://github.com/narrowstacks/claude-homeassistant-plugins

## Support

For issues, questions, or suggestions about specific plugins, please refer to the individual plugin documentation or open an issue in the repository.

## Additional Resources

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code/)
- [Claude Code Plugins Guide](https://docs.claude.com/en/docs/claude-code/plugins.md)
- [Home Assistant Documentation](https://www.home-assistant.io/docs/)
- [ESPHome Documentation](https://esphome.io/)
