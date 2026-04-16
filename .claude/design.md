# Platypus: Design Document

This document captures the design of the Platypus platform. It is a map, not a spec — it provides enough orientation to guide implementation, but intentionally leaves details open. The implementation of each part is the source of truth; decisions are made and recorded there as work progresses. Do not treat anything here as a complete specification.

## Contents

- [Platform Concepts](#platform-concepts)
  - [Framework Installation](#framework-installation)
  - [Terraform Implementation](#terraform-implementation)
  - [Documentation](#documentation)
  - [Agent Agnosticism](#agent-agnosticism)
  - [CI Workflow](#ci-workflow)
  - [Resilience](#resilience)
  - [Secrets](#secrets)
- [Future Work](#future-work)

---

## Platform Concepts

How the platform works — the ideas behind the service repo structure and the framework's design.

### Framework Installation

The framework is installed into a service repo by running the `install` script via `curl | bash`:

```sh
curl https://raw.githubusercontent.com/w5dio/platypus/refs/heads/main/install | bash
```

The command is run from the service repo root. No local clone of the framework repo is required — the script fetches and copies the framework files directly. Re-running the command is safe; managed files are overwritten in full.

The URL is intended to eventually be replaced by a shortened URL (e.g. `https://w5d.io/platypus/install`), keeping the install command concise and memorable.

### Terraform Implementation

Provisioning is implemented with Terraform. Terraform files live at the repo root; state is stored in HCP Terraform. All automated execution runs via the managed `run.yml` workflow on GitHub Actions.

#### Reading Config

`config.yaml` is read via `yamldecode(file("config.yaml"))` into a local value. Terraform then iterates over config entries — typically using `for_each` — to create one resource per entry. The user is completely decoupled from Terraform concepts and provider-specific naming.

#### Typical Files

Terraform processes all `*.tf` files in the root. The framework does not dictate naming or structure, but most implementations will have:

- **`main.tf`:** provider configuration and resources
- **`outputs.tf`:** Terraform output definitions (see [Output Requirements](#output-requirements) below) — the filename is convention only; output blocks may be in any `*.tf` file
- **`variables.tf`:** optional — only needed if the implementation uses Terraform input variables (see [Secrets](#secrets) below)

#### Secrets

Secrets (API tokens, credentials) cannot go in `config.yaml`. Two options:

- **Provider environment variables** (preferred for provider credentials): most providers read credentials directly from environment variables (e.g. `CLOUDFLARE_API_TOKEN` for the Cloudflare provider). The workflow sets these from GitHub Secrets; no Terraform variable declaration is needed.
- **Input variables:** declare `variable {}` blocks and inject values via `TF_VAR_*` environment variables set by the workflow from GitHub Secrets. Requires `variables.tf` or equivalent.

> **TBD:** how GitHub Secrets are defined and managed across service repos.

#### Output Requirements

The service developer declares outputs by defining Terraform `output {}` blocks. The `run.yml` workflow reads all defined outputs after apply (via `terraform output -json`) and publishes them to the GitHub Actions job summary. Any `*.tf` file may contain output blocks — the filename is not significant.

Output is for human consumption; apps do not read platform outputs at runtime.

### Documentation

- `config.schema.json` is the single source of truth for both the config contract and all user-facing documentation — it is the only file the developer needs to maintain to keep docs accurate
- `README.md` is generated automatically by the workflow from `config.schema.json`; it is never manually authored or edited
- The workflow generates the README on every run and commits it back if it changed — ensuring docs always stay in sync without any developer action
- Field-level `description`, `examples`, and `default` annotations in the schema are **mandatory**

> **TBD:** tooling for README generation — candidates are `json-schema-for-humans` (Python) and `@adobe/jsonschema2md` (npm)

### Agent Agnosticism

The platform is not tied to any specific agent. `AGENTS.md` is used as the instructions file because it is an emerging cross-agent standard; its content is plain Markdown with no agent-specific syntax. For compatibility with a specific agent, it is the developer's responsibility to configure their agent to read `AGENTS.md` (e.g. by pointing `CLAUDE.md` or `GEMINI.md` at it, or by renaming the file).

### CI Workflow

The GitHub Actions workflow. Fully managed — must not be edited by the service developer. Runs on push, on a schedule, and on demand (`workflow_dispatch`).

Responsible for:

- Validating `config.yaml` against `config.schema.json`
- Generating `README.md` from `config.schema.json` and committing it back if changed
- Running Terraform
- Reading all Terraform output values and publishing them to the GitHub Actions job summary

### Resilience

- The provisioning state is periodically validated and automatically corrected if it differs from the desired state in config
- Periodic execution is handled by the scheduled trigger in `run.yml`

> **TBD:** determine whether to include resilience at all — it can be seen as an optional add-on. If included, the frequency of periodic runs must be determined.

### Secrets

> **TBD:** how secrets are managed (likely GitHub Secrets, but not yet decided)

---

## Future Work
