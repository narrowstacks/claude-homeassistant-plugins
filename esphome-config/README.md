# ESPHome YAML Configuration Skill

A skill for creating and managing ESPHome YAML configurations with proper syntax, component organization, modularization through packages, and correct usage of configuration types.

## Contents

- **SKILL.md** - Core instructions and quick reference
- **references/** - Detailed guides loaded as needed
  - yaml-syntax.md - Comments, scalars, sequences, mappings, anchors, multi-line strings, secrets, substitutions, !include, hidden items, and lambdas
  - packages.md - Local packages, remote git packages, templating with variables, extending, and removing configurations
  - configuration-types.md - ID naming rules, pin specifications, and time period formats
- **examples/** - Complete working configuration files

## Usage

Claude will automatically load this skill when you ask about ESPHome YAML configuration, device configs, reusable packages, organizing multi-device deployments, or troubleshooting YAML structure and configuration types. The skill uses progressive disclosure: core instructions are always available, detailed references are loaded only when needed.
