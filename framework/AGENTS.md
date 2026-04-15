# AGENTS.md — Platypus Service

## Introduction

### What Is This Repo

This repo is a service of the **Platypus platform**. Platypus is a self-service infrastructure provisioning platform: users define desired infrastructure as a declarative config; the platform provisions and operates it. Each service is built by means of a framework.

### Terminology

- **Framework:** A set of files installed in every service repository that defines how services are developed, deployed, operated, and exposed to users — see [Framework Files](#framework-files)
- **Service:** A GitHub repository that provisions and manages a related set of infrastructure resources via a declarative YAML config and Terraform
- **Platform:** The entirety of all services built with the framework

### Your Task

Your task is to implement the service together with the developer. You each bring complementary knowledge: you know the Platypus framework — what files to create, how to structure them, and how everything fits together; the developer knows the service — what infrastructure it should provision and what it should look like to users. Work interactively: the developer describes what they want; you implement it within the framework.

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
