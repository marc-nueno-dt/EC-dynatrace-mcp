# EC Dynatrace MCP Workshop

Hands-on workshop to set up and use the Dynatrace MCP environment.

## Workshop Goals

- Prepare your environment in AWS Workspaces.
- Install the required Node.js toolchain.
- Clone the workshop repository.
- Import and configure the project.
- Install and run the Dynatrace MCP server.

## Prerequisites

- Access to AWS Workspaces
- AWS Kiro available in your workspace
- Internet access to download dependencies

## Task 1: Introduction and Local Setup

### 1. Connect to AWS Workspaces

Sign in to your assigned AWS Workspace.

### 2. Open AWS Kiro

Launch AWS Kiro and open a terminal.

### 3. Check Node.js and npm

Run:

```bash
node -v
npm -v
```

If both commands return versions, continue to the next step.

### 4. Install Node.js (if missing)

Install `nvm`, then Node.js 24:

```bash
# Download and install nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.5/install.sh | bash

# Load nvm in the current shell
\. "$HOME/.nvm/nvm.sh"

# Install Node.js
nvm install 24

# Verify installation
node -v
npm -v
```

Expected output (or newer patch versions):

- `node -v` -> `v24.x.x`
- `npm -v` -> `11.x.x`

### 5. Clone the Repository

```bash
git clone https://github.com/marc-nueno-dt/EC-dynatrace-mcp.git
cd EC-dynatrace-mcp
```

## Task 2: Import

Import the cloned project into AWS Kiro.

Suggested checklist:

- Open the project folder.
- Verify files were indexed correctly.
- Open a terminal at the project root.

## Task 3: Install Dynatrace MCP Server Locally

For Kiro workspace-level MCP settings, use this format with `DT_ENVIRONMENT_CONFIGS`.

```json
{
  "mcpServers": {
    "dynatrace-managed-mcp": {
      "command": "npx",
      "args": [
        "-y",
        "@dynatrace-oss/dynatrace-managed-mcp-server@latest"
      ],
      "env": {
        "DT_ENVIRONMENT_CONFIGS": "[{\"dynatraceUrl\":\"https://dynatrace.development.tech.ec.europa.eu\",\"apiEndpointUrl\":\"https://intragate.development.ec.europa.eu/apmgw\",\"environmentId\":\"87889eb4-bb35-41ec-a521-adf5abcc6256\",\"alias\":\"digit-dev\",\"apiToken\":\"dt-token-here\"}]",
        "LOG_LEVEL": "DEBUG"
      },
      "autoApprove": [
        "dynatrace_managed_get_environments_info"
      ]
    }
  }
}
```


Notes:

- Keep `DT_TOKEN` out of source control.
- If you configure multiple environments, add more JSON objects inside `DT_ENVIRONMENT_CONFIGS`.
- `DT_ENVIRONMENT_CONFIGS` takes precedence for this setup, so `DT_CONFIG_FILE` is not required in this Kiro configuration style.

## Task 4: Connect to the Remote HTTP Server

For the remote HTTP server, use the standard HTTP transport shape in Kiro.

Use this workspace-level MCP configuration:

```json
{
  "mcpServers": {
    "dynatrace-managed-mcp-remote": {
      "url": "https://intragate.development.ec.europa.eu/dynatrace-mcp",
      "headers": {
        "X-Dynatrace-Tokens": "digit-dev=dt0c01.AAA"
      }
    }
  }
}
```

Use `dt-config-remote.yaml` on the remote side to define the Dynatrace environment details the server should expose:

- `alias: digit-dev`
- `dynatraceUrl: https://dynatrace.development.tech.ec.europa.eu`
- `apiEndpointUrl: https://intragate.development.ec.europa.eu/apmgw`
- `environmentId: 87889eb4-bb35-41ec-a521-adf5abcc6256`

If you need additional remote environments, add them to [dt-config-remote.yaml](dt-config-remote.yaml).


# Additional Topics

Useful Prompt Examples: https://github.com/dynatrace-oss/dynatrace-managed-mcp/tree/main/examples 

Expand DT MCP for EC Usecases