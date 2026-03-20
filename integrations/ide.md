# IDE Plugins

Proscan provides editor plugins that surface security findings directly where you write code. Findings appear as inline warnings and diagnostics — no need to switch to a browser to see scan results.

## Supported Editors

| Editor | Integration Type |
|--------|-----------------|
| VS Code | Extension |
| IntelliJ IDEA | Plugin |
| Visual Studio | Extension |
| Eclipse | Plugin |
| Neovim | LSP client |
| Sublime Text | LSP client |
| JetBrains Fleet | Plugin |
| Cursor | Extension |

## Features

- Inline findings displayed as editor diagnostics (warnings, errors)
- Hover for finding details and remediation guidance
- Quick-fix actions for supported vulnerability types
- Scan on save (configurable)
- Project-level scan from the editor command palette

## Setup

1. Install the Proscan extension or plugin from your editor's marketplace
2. Open the extension settings
3. Enter your Proscan server URL and API token
4. The plugin connects to your running Proscan instance and pulls findings for the current project

## LSP

For editors that support the Language Server Protocol, Proscan provides an LSP server. Any LSP-compatible editor can connect to it. Neovim and Sublime Text use this approach.

Configuration details depend on your editor. See the [integrations repository](https://github.com/Proscan-hub/integrations) for editor-specific setup instructions.
