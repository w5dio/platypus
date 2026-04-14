# Platypus: Design Document

This document captures the design of the Platypus platform. It is input for implementation, not a final reference document.

---

## Overview

Platypus is a self-service infrastructure provisioning platform intended for use across all domains (weibeld, W5D Labs, Rye & Cheese, w5d.io). Users define desired infrastructure as a declarative config; the platform takes care of provisioning and operating the infrastructure.

The platform is named Platypus. Individual units are called services. Each service is maintained in a separate repository, prefixed with `plat-` followed by the service name (e.g. `plat-domain`). The platform's own repository is named `platypus`.

The `platypus` repo is the **framework** for building the platform. It contains no runtime logic of its own — it provides the framework files that are installed into service repos and define what a service is and how it operates. All actual platform functionality is implemented in the individual service repos. The platform is the entirety of all live service repos built on this framework.

Service development is driven by a coding agent. The platform is not tied to any specific agent — `AGENTS.md` is used as the instructions file because it is an emerging cross-agent standard, and its content is plain Markdown with no agent-specific syntax. For compatibility with a specific agent, it is the developer's responsibility to configure their agent to read `AGENTS.md` (e.g. by pointing `CLAUDE.md` or `GEMINI.md` at it, or by renaming the file).

---

## Concepts

### Terminology

| Term | Meaning |
|---|---|
| **Framework** | The `platypus` repo itself — provides the structure, conventions, and managed components that all services conform to |
| **Platform** | The entirety of all live service repos built on the framework |
| **Service** | A single provisioning unit, maintained in its own `plat-<name>` repo |
| **Framework files** | Files installed by the framework (`install/`) — not created or edited by the service developer |
| **Service implementation** | Files provided by the service developer — the Terraform files, config, schema, and generated README |

### File Roles

Each file in a service repo can be characterised along three axes:

**Axis 1: Provenance** — where does the file come from?

| Framework-installed | Developer-provided |
|---|---|
| `AGENTS.md`, `Makefile`, `jsonschema2readme.sh`, `run.yml` | `config.yaml`, `config.schema.json`, `README.md`, Terraform files |

- **Framework-installed:** statically copied into the service repo by `install`; never created or edited by the developer
- **Developer-provided:** the developer is responsible for producing it — either by authoring (`config.yaml`, `config.schema.json`, Terraform files) or by generating via framework tooling (`README.md`)

**Axis 2: Lifecycle** — when is the file used?

| Dev-time | Runtime |
|---|---|
| `AGENTS.md`, `Makefile`, `jsonschema2readme.sh` | `config.yaml`, `config.schema.json`, `README.md`, Terraform files, `run.yml` |

- **Dev-time:** used only while building or maintaining the service
- **Runtime:** executes or is actively used once the service is live

**Axis 3: Visibility** — who is the file for?

| User-facing | Implementation |
|---|---|
| `config.yaml`, `README.md` | `config.schema.json`, Terraform files, `AGENTS.md`, `Makefile`, `jsonschema2readme.sh`, `run.yml` |

- **User-facing:** seen and interacted with by the user of the service
- **Implementation:** not the user's concern — internal to how the service works

---

## platypus Repo Contents

The repo contains the framework files and a script that installs them into a service repo.

### File Structure

```
install              ← installs framework files into the service repo (safe to re-run)
README.md            ← describes the repo and how to use it
DECISIONS.md         ← permanent record of architectural and technology decisions
install/
  AGENTS.md          → root of service repo
  Makefile           → root of service repo
  jsonschema2readme.sh    → root of service repo
  .github/
    workflows/
      run.yml        → .github/workflows/ of service repo
```

Once the repo is implemented, this design document is no longer needed. `README.md` covers what is not already self-evident from the repo contents; `DECISIONS.md` is extracted from the Architectural Decisions and Technology Choices sections and kept as a permanent record.

### install

Copies all files from `install/` into the service repo, mirroring the directory structure. Safe to re-run — on subsequent runs all managed files are overwritten in full.

Before copying, checks that all tools required by the Makefile are installed and aborts with a clear error message if any are missing.

### AGENTS.md

Contains the framework instructions read by the coding agent at session start. Fully framework-managed — must not be edited by the service developer. The framework instructions are comprehensive enough to guide the agent without any service-specific additions.

### Makefile

Provides the standard developer commands, consistent across all services:

| Command | Description |
|---|---|
| `make readme` | Regenerate `README.md` from `config.schema.json` |
| `make validate` | Validate `config.yaml` against `config.schema.json` |

Terraform commands (`terraform validate`, `terraform plan`, etc.) are run directly — no Make wrapper needed since Terraform files live at the repo root.

### jsonschema2readme.sh

Called by `make readme`. Generates `README.md` from `config.schema.json` using the schema's root `title` and `description` fields as the README heading and introductory paragraph, with a full config reference derived from field-level annotations. Wraps an existing JSON Schema documentation tool with custom logic for the overall README structure.

> **TBD:** tooling — candidates are `json-schema-for-humans` (Python) and `@adobe/jsonschema2md` (npm)

### run.yml

The GitHub Actions workflow. Fully managed — must not be edited by the service developer.

Responsible for:

- Running Terraform
- Reading all Terraform output values defined in the service's `outputs.tf`
- Formatting and publishing them to the GitHub Actions job summary in a consistent, platform-defined format

### Example Service

The files in `example/` form a minimal runnable Platypus service. They demonstrate the complete pattern end to end — declarative config read via `yamldecode()`, a resource provisioned per config entry, and outputs exposed via `outputs.tf` and published to the job summary by the managed workflow — so a developer can see the full flow immediately before replacing the content with their real implementation.

---

## Service Repo Structure

A complete service repo contains both the framework files (installed and managed by the framework) and the service implementation files (authored by the developer). Together they form a fully operational service.

```
config.yaml              ← developer-authored · runtime · user-facing
config.schema.json       ← developer-authored · runtime
README.md                ← generated from config.schema.json · runtime · user-facing
main.tf                  ← developer-authored · runtime
variables.tf             ← developer-authored · runtime
outputs.tf               ← developer-authored · runtime
AGENTS.md                ← framework-managed · dev-time
Makefile                 ← framework-managed · dev-time
jsonschema2readme.sh     ← framework-managed · dev-time
.github/
  workflows/
    run.yml              ← framework-managed · runtime
```

The developer-authored files are not present after running `install` — they are created by the coding agent guided by `AGENTS.md`.

---

## Service Development

How a developer uses the `platypus` repo to create and develop a service.

### Init Workflow

To create a new Platypus service repo:

1. `mkdir plat-<name> && cd plat-<name> && git init`
2. `curl https://raw.githubusercontent.com/w5dio/platypus/refs/heads/main/install | bash` — installs framework files
3. Start the coding agent — it reads `AGENTS.md` at session start and guides all implementation from there

To update framework files at any time: re-run `install`.

> **Note:** the raw GitHub URL above is intended to be replaced by a short URL once the `plat-shorturl` service is deployed (e.g. `w5d.io/platypus/install`).

### Config

- Each service has exactly one config file, named `config.yaml`, located in the root of the repo
- The config is declarative — it describes the desired state, and the service ensures the actual state matches it
- The format is YAML
- Each service must have a JSON Schema for its `config.yaml`
- Every resource in the config must be something the service directly creates and manages — resources managed outside the platform have no place in service config
- The fields and their meaning in the config file are determined by the service developer

### Implementation

- Provisioning is implemented with Terraform; Terraform files live at the repo root
- HCP Terraform is used for storing Terraform state
- Terraform reads `config.yaml` via `yamldecode()` — the user is completely decoupled from Terraform concepts and provider-specific naming
- All automated execution runs on GitHub Actions via the managed `run.yml` workflow

### Output

- The service developer declares what outputs to expose by defining them in `outputs.tf`
- The managed `run.yml` workflow reads these values, formats them, and publishes them to the GitHub Actions job summary automatically
- Output is for human consumption; apps do not read platform outputs at runtime

### Documentation

- `config.schema.json` is the single source of truth for both the config contract and all user-facing documentation
- `README.md` is auto-generated from `config.schema.json` by running `make readme`; it is never manually authored
- The schema's root `title` and `description` fields provide the README heading and introductory paragraph; all other content is derived from field-level annotations
- Field-level `description`, `examples`, and `default` annotations are **mandatory**

### Secrets

> **TBD:** how secrets are managed (likely GitHub Secrets, but not yet decided)

### Resilience

- The provisioning state is periodically validated and automatically corrected if it differs from the desired state in config
- Validation and correction are done with Terraform; periodic executions are scheduled with GitHub Actions

> **TBD:** determine whether to include resilience at all — it can be seen as an optional add-on. If included, the frequency of periodic runs must also be determined (platform-wide fixed value vs. configured per service).

