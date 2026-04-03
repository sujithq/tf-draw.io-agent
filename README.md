# tf-draw.io-agent

> A GitHub Copilot custom agent that generates [draw.io](https://www.drawio.com/) infrastructure diagrams directly from a Terraform source folder.

---

## Overview

`tf-drawio` reads your Terraform configuration files (`.tf`), maps every resource, module, and their dependencies into a directed graph, and renders the result as a draw.io diagram — all from within GitHub Copilot Chat.

```
@tf-drawio Please create a draw.io diagram for the Terraform code in ./infra/prod
```

---

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| [Visual Studio Code](https://code.visualstudio.com/) ≥ 1.90 | Copilot Chat with agent/MCP support |
| [GitHub Copilot](https://github.com/features/copilot) subscription | Copilot Chat must be enabled |
| [Node.js](https://nodejs.org/) ≥ 18 | Required to run `npx @drawio/mcp` |
| [Docker](https://www.docker.com/) (optional) | Required for the Terraform MCP server |

---

## MCP Server Configuration

Three MCP servers are pre-configured in [`.vscode/mcp.json`](.vscode/mcp.json):

| Server | Package / Endpoint | Purpose |
|--------|--------------------|---------|
| **draw.io** | `npx -y @drawio/mcp` | Opens and renders draw.io diagrams (`open_drawio_xml`, `open_drawio_csv`, `open_drawio_mermaid`) |
| **Terraform Registry** | `docker run hashicorp/terraform-mcp-server:latest` | Looks up provider schemas, resource docs, and module metadata from the Terraform Registry |
| **Microsoft Learn** | `https://learn.microsoft.com/api/mcp` | Searches and fetches official Microsoft / Azure documentation |

### First-time setup

1. **Clone the repository** (or open it in VS Code).
2. VS Code will detect `.vscode/mcp.json` and prompt you to enable the MCP servers.
3. When prompted, supply your **HCP Terraform API token** (only needed for private registry or workspace operations — leave blank if not required).
4. Ensure Docker is running if you want Terraform Registry lookups.

> **No API token is required** for the draw.io MCP server or the Microsoft Learn MCP server.

---

## Usage

### Via Copilot Chat (recommended)

Open Copilot Chat in VS Code, switch to **Agent mode**, select the `tf-drawio` agent, and type:

```
@tf-drawio Create a draw.io diagram for ./infra/prod
```

The agent will:

1. Scan all `*.tf` files in the specified folder.
2. Parse resources, modules, data sources, providers, variables, and outputs.
3. Enrich the diagram with Terraform Registry and Azure documentation.
4. Build a dependency graph.
5. Render the diagram via the draw.io MCP server.
6. Save the output to `<terraform-folder>/diagrams/infrastructure.drawio`.

### Via the reusable prompt

Open the Command Palette (`⌘/Ctrl+Shift+P`) → **Copilot: Run Prompt File** → select **create-drawio-diagram**.

You will be asked:

| Input | Description | Default |
|-------|-------------|---------|
| `terraformFolder` | Path to the Terraform source folder | (required) |
| `layout` | Diagram layout: `lr` (left-to-right) or `tb` (top-to-bottom) | `lr` |

---

## Project Structure

```
tf-draw.io-agent/
├── .github/
│   ├── agents/
│   │   └── tf-drawio.agent.md          # Custom agent definition
│   ├── prompts/
│   │   └── create-drawio-diagram.prompt.md  # Reusable prompt
│   └── copilot-instructions.md         # Repo-wide Copilot instructions
└── .vscode/
    └── mcp.json                        # MCP server configuration
```

---

## Diagram Conventions

- **Shapes**: Provider-specific icons for AWS, Azure, and GCP; generic cloud shape as fallback.
- **Arrows**: Directed edges representing explicit (`depends_on`) or implicit (attribute reference) dependencies.
- **Containers**: Resources inside a Terraform module are grouped in a labelled swimlane.
- **Colours**: Each provider gets a distinct background colour when no module grouping applies.
- **Legend**: Top-right corner — lists the icon set and arrow semantics.
- **Layout**: Left-to-right by default; `tb` flag switches to top-to-bottom.

---

## MCP Servers — Details

### draw.io MCP (`@drawio/mcp`)

- **Repository**: [jgraph/drawio-mcp](https://github.com/jgraph/drawio-mcp)
- **npm**: [`@drawio/mcp`](https://www.npmjs.com/package/@drawio/mcp)
- **Tools exposed**:
  - `open_drawio_xml` — open a diagram from draw.io XML
  - `open_drawio_csv` — open a diagram from CSV
  - `open_drawio_mermaid` — open a diagram from Mermaid syntax

### Terraform MCP (`hashicorp/terraform-mcp-server`)

- **Repository**: [hashicorp/terraform-mcp-server](https://github.com/hashicorp/terraform-mcp-server)
- **Docker image**: `hashicorp/terraform-mcp-server:latest`
- **Documentation**: [HashiCorp Developer — Terraform MCP Server](https://developer.hashicorp.com/terraform/mcp-server)
- **Environment variable**: `TFE_TOKEN` — HCP Terraform API token (optional)

### Microsoft Learn MCP

- **Endpoint**: `https://learn.microsoft.com/api/mcp`
- **Repository**: [MicrosoftDocs/mcp](https://github.com/MicrosoftDocs/mcp)
- **Documentation**: [Microsoft Learn MCP Server overview](https://learn.microsoft.com/en-us/training/support/mcp)
- **Tools exposed**:
  - `microsoft_docs_search` — search Microsoft documentation
  - `microsoft_docs_fetch` — fetch a specific documentation page

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `@drawio/mcp` not found | Run `npm install -g @drawio/mcp` or ensure Node.js ≥ 18 is installed |
| Terraform MCP server won't start | Verify Docker is running and the image is pulled: `docker pull hashicorp/terraform-mcp-server:latest` |
| Microsoft Learn MCP not connecting | Check your internet connection; the endpoint is `https://learn.microsoft.com/api/mcp` |
| No `.tf` files found | Verify the folder path passed to the agent; use an absolute path if needed |

---

## License

[MIT](LICENSE)
