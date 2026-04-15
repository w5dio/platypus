# AGENTS.md — Platypus Service

## What Is This Repo

> One paragraph: what Platypus is, what the service repo's purpose is, the agent's role

## Framework Files

> List AGENTS.md and the Workflow File (`run.yml`); must not be touched; one-line description of the Workflow File with pointer to CI Workflow section

## Files to Create

> - Config (`config.yaml`): mandatory, exact name, repo root
> - Config Schema (`config.schema.json`): mandatory, exact name, repo root
> - Terraform files: no naming constraint, all at repo root

## Config

> The Config is the developer's domain — they either provide it directly or describe clearly what it should look like. It is the starting point for creating the Config Schema.

## Config Schema

> Agent generates the full Config Schema (structure + all annotations) interactively with the developer. All fields must have `description`, `examples`, and `default`. Schema errors (invalid JSON, invalid JSON Schema structure) are caught by CI on next push.
>
> **See DECISIONS.md:** Schema Authoring: Agent-Generated vs. Tool-Generated Skeleton; Schema Validation Tooling; Documentation Authoring: Manual vs. Generated from Schema

## Terraform

### Config Reading

> `yamldecode(file("config.yaml"))` pattern

### Secrets Reading

> Provider env vars or `TF_VAR_*` input variables

### State

> HCP Terraform backend

## CI Workflow

> High-level overview of what the Workflow File does: validate Config, generate README, apply Terraform, publish outputs to job summary

## Outputs

> Declare `output {}` blocks; framework publishes them to job summary; README explains to users how to access them

## Secrets

> How to record secrets in the platform (GitHub Secrets — placeholder, TBD)
>
> **See DECISIONS.md:** Secrets Management
