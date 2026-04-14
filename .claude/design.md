# Platypus: Design Document

This document captures the design of the Platypus platform. It is a map, not a spec — it provides enough orientation to guide implementation, but intentionally leaves details open. The implementation of each part is the source of truth; decisions are made and recorded there as work progresses. Do not treat anything here as a complete specification.

## Contents

- [Overview](#overview)
  - [Terminology](#terminology)
- [Framework Repository](#framework-repository)
- [Service Repository](#service-repository)
  - [File Roles](#file-roles)
- [Platform Concepts](#platform-concepts)
  - [Config](#config)
  - [Terraform Implementation](#terraform-implementation)
  - [Documentation](#documentation)
  - [Service Implementation by Coding Agent](#service-implementation-by-coding-agent)
  - [CI Workflow](#ci-workflow)
  - [Resilience](#resilience)
  - [Secrets](#secrets)
- [Future Work](#future-work)

---

## Overview

Platypus is a self-service infrastructure provisioning platform intended for use across all domains (weibeld, W5D Labs, Rye & Cheese, w5d.io). Users define desired infrastructure as a declarative config; the platform takes care of provisioning and operating the infrastructure.

The `platypus` repo is the **framework** for building the platform. It provides the framework files installed into service repos and defines what a service is and how it operates. All actual platform functionality is implemented in the individual service repos. The platform is the entirety of all live service repos built on this framework.

Service implementation is developed with a coding agent — see [Service Implementation by Coding Agent](#service-implementation-by-coding-agent) in Platform Concepts.

### Terminology

| Term | Meaning |
|---|---|
| **Framework** | The `platypus` repo itself — provides the structure, conventions, and managed components that all services conform to |
| **Platform** | The entirety of all live service repos built on the framework |
| **Service** | A single provisioning unit, maintained in its own `plat-<name>` repo |
| **Framework files** | Files installed by the framework (`framework/`) — not created or edited by the service developer |
| **Service implementation** | Files provided by the service developer — Terraform files, config, schema, and generated README |

---

## Framework Repository

The framework repo contains the framework files and the script that installs them into a service repo. It contains no runtime logic of its own.

```
install
README.md
DECISIONS.md
framework/
  AGENTS.md
  .github/workflows/run.yml
```

- **`install`:** copies all files from `framework/` into the service repo, mirroring the directory structure. Safe to re-run — managed files are overwritten in full. Checks for required tools before copying and aborts with a clear error if any are missing.
- **`README.md`:** describes the framework repo and how to use it
- **`DECISIONS.md`:** permanent record of architectural and technology decisions
- **`framework/AGENTS.md`:** framework instructions for the coding agent — see [Service Implementation by Coding Agent](#service-implementation-by-coding-agent) in Platform Concepts
- **`framework/.github/workflows/run.yml`:** GitHub Actions workflow — see [CI Workflow](#ci-workflow) in Platform Concepts

---

## Service Repository

A complete service repo contains both the framework files (installed by `install`) and the service implementation (provided by the developer). Together they form a fully operational service.

```
config.yaml
config.schema.json
README.md
*.tf                 ← one or more Terraform files; structure and naming not dictated by the framework
AGENTS.md
.github/workflows/run.yml
```

- **`config.yaml`:** declarative config edited by the user to define desired infrastructure — see [Config](#config)
- **`config.schema.json`:** JSON Schema for the config, driving both validation and documentation generation — see [Config](#config), [Documentation](#documentation)
- **`README.md`:** generated automatically by the workflow from `config.schema.json` — see [Documentation](#documentation)
- **Terraform files:** implement the provisioning logic and expose outputs — see [Implementation](#implementation), [Output](#output)
- **`AGENTS.md`:** framework instructions for the coding agent — see [Service Implementation by Coding Agent](#service-implementation-by-coding-agent)
- **`.github/workflows/run.yml`:** GitHub Actions workflow — see [CI Workflow](#ci-workflow)

The developer-provided files are not present after running `install` — they are created by the coding agent guided by `AGENTS.md`.

### File Roles

Each file in a service repo can be characterised along three axes:

**Axis 1: Provenance** — where does the file come from?

| Framework-provided | Developer-provided |
|---|---|
| `AGENTS.md`, `run.yml`, `README.md` | `config.yaml`, `config.schema.json`, Terraform files |

- **Framework-provided:** either statically copied into the service repo by `install` (`AGENTS.md`, `run.yml`) or generated automatically by the workflow (`README.md`); never created or edited by the developer
- **Developer-provided:** the developer is responsible for producing it — by authoring (`config.yaml`, `config.schema.json`, Terraform files)

**Axis 2: Lifecycle** — when is the file used?

| Dev-time | Runtime |
|---|---|
| `AGENTS.md` | `config.yaml`, `config.schema.json`, `README.md`, Terraform files, `run.yml` |

- **Dev-time:** used only while building or maintaining the service
- **Runtime:** executes or is actively used once the service is live

**Axis 3: Visibility** — who is the file for?

| User-facing | Implementation |
|---|---|
| `config.yaml`, `README.md` | `config.schema.json`, Terraform files, `AGENTS.md`, `run.yml` |

- **User-facing:** seen and interacted with by the user of the service
- **Implementation:** not the user's concern — internal to how the service works

---

## Platform Concepts

How the platform works — the ideas behind the service repo structure and the framework's design.

### Config

- Each service has exactly one config file, named `config.yaml`, at the repo root
- The config is declarative — it describes desired state; the service ensures actual state matches it
- Each service must have a JSON Schema for its `config.yaml`, named `config.schema.json`
- Every resource in the config must be something the service directly creates and manages — resources managed outside the platform have no place in service config
- The fields and their meaning are determined by the service developer

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

### Service Implementation by Coding Agent

#### Development Approach

Service implementation is developed interactively with a coding agent — the same way any new project would be built with a coding agent. The developer and agent work back and forth: the developer describes what the service should do and makes decisions; the agent implements. The content and purpose of the service come entirely from the developer. `AGENTS.md` provides the agent with background context about the Platypus framework so the developer does not have to explain it in every session.

#### Agent Agnosticism

The platform is not tied to any specific agent. `AGENTS.md` is used as the instructions file because it is an emerging cross-agent standard; its content is plain Markdown with no agent-specific syntax. For compatibility with a specific agent, it is the developer's responsibility to configure their agent to read `AGENTS.md` (e.g. by pointing `CLAUDE.md` or `GEMINI.md` at it, or by renaming the file).

#### AGENTS.md

`AGENTS.md` contains all the background context about the Platypus framework that the coding agent needs — how services are structured, what files are framework-managed, how config and Terraform relate, and so on. This frees the developer from having to explain the framework in every session. It does not prescribe what a specific service should do; that comes from the developer through the session itself.

`AGENTS.md` is fully framework-managed and must not be edited by the service developer.

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

- **Framework versioning:** `run.yml` is a static copy installed into each service repo. Updates to the framework require developers to re-run `install` to receive the new version. Versioned releases (e.g. GitHub Releases) are a future option for making framework updates more explicit and auditable — following the same model as npm packages or GitHub Actions themselves (`actions/checkout@v4`).
