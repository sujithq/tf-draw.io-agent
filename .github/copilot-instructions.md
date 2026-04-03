# GitHub Copilot Instructions — tf-draw.io-agent

This repository contains a GitHub Copilot custom agent that generates draw.io diagrams from Terraform source folders.

## Purpose

The `tf-drawio` agent reads Terraform configuration files (`.tf`) from a given folder, understands the infrastructure topology (resources, modules, providers, data sources, variables, and outputs), and produces a draw.io diagram that visually represents the infrastructure.

## Available MCP Servers

The following MCP servers are pre-configured and available to the agent:

| Server | Key | Description |
|--------|-----|-------------|
| draw.io | `drawio` | Renders and opens draw.io diagrams from XML, CSV or Mermaid |
| Terraform Registry | `terraform` | Looks up Terraform provider schemas, resource docs, and module metadata |
| Microsoft Learn | `microsoft-learn` | Searches and fetches official Microsoft / Azure documentation |

## Agent Behaviour

When the `tf-drawio` agent is invoked it **must**:

1. Accept a path to a Terraform source folder (relative or absolute) as input.
2. Read and parse every `.tf` file in that folder (and sub-folders when `--recursive` is specified).
3. Extract all resources, data sources, modules, variables, outputs, providers, and locals.
4. Identify relationships between resources (dependencies via `depends_on`, references, module calls, etc.).
5. Group related resources visually (by provider, by module, or by logical layer).
6. Use the `drawio` MCP server to produce a draw.io diagram and return the URL/file path to the user.

## Diagram conventions

- Use AWS / Azure / GCP icons when the provider matches; fall back to generic cloud icons.
- Draw arrows to represent explicit or implicit dependencies.
- Group resources that belong to the same Terraform module inside a labelled swimlane.
- Include a legend with the icon set used.
- Keep the layout left-to-right (LR) by default; the user may request TB (top-to-bottom).

## Coding standards

- Do not modify existing Terraform files under any circumstances.
- All generated diagram artefacts should be placed in a `diagrams/` folder relative to the Terraform source folder unless the user specifies otherwise.
- Prefer draw.io XML format for output (`.drawio`); also produce `.svg` when requested.
