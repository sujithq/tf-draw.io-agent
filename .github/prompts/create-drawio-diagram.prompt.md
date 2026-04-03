---
name: create-drawio-diagram
description: Generate a draw.io infrastructure diagram from a Terraform source folder
mode: agent
tools:
  - codebase
  - readFile
  - createFile
  - runCommand
  - drawio
  - terraform
  - microsoft-learn
---

Analyse the Terraform configuration files in the folder `${input:terraformFolder}` and generate a draw.io infrastructure diagram.

## Steps

1. Discover all `*.tf` files inside `${input:terraformFolder}` (recursively).
2. Parse every file and extract:
   - Providers (name, alias, region/location)
   - Resources (type, name, key attributes)
   - Data sources (type, name)
   - Modules (source, version)
   - Variables and outputs (name, description)
3. Use the **Terraform MCP server** to look up provider schemas and module documentation for any registry-sourced module.
4. Use the **Microsoft Learn MCP server** to fetch Azure resource documentation for any `azurerm_*` resource type found.
5. Build a directed dependency graph from:
   - Explicit `depends_on` lists
   - Implicit attribute references (e.g. `aws_vpc.main.id`)
   - Module output references
6. Compose draw.io XML:
   - One shape per resource/data source/module, labelled with its type and name.
   - Provider-specific icons where available (AWS, Azure, GCP).
   - Directed arrows for each dependency edge.
   - Module-scoped containers (swimlanes) for resources inside a module.
   - Provider-coloured backgrounds for ungrouped resources.
   - Legend in the top-right corner.
   - Left-to-right layout (default) or top-to-bottom if `${input:layout}` is `tb`.
7. Call the **draw.io MCP server** (`open_drawio_xml`) to open the diagram.
8. Save the output to `${input:terraformFolder}/diagrams/infrastructure.drawio`.
9. Report back with:
   - A summary table of resources found (type, count).
   - The path to the saved `.drawio` file.
   - The draw.io URL returned by the MCP server.
