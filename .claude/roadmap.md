# Roadmap — Platypus Framework

## Introduction

Tasks are marked `[ ]` (not started), `[/]` (in progress), or `[x]` (complete). Work on the first incomplete task.

Sub-items under a task are working notes and prior research to be consumed and incorporated into the final artefacts as the task is executed — delete them as they are used. The expected outcome is that completed tasks have no remaining sub-items.

## Tasks

- [x] DECISIONS.md: redo Terraform State Storage and Secrets Storage sections (with all decisions), add HCP Terraform rationale
- [ ] Carry out HCP Terraform research in research.md
  - **Add HCP Terraform research section to research.md**
    - **Pricing:** free tier of 500 managed resources; after that $0.10/resource/month — to be verified against https://www.hashicorp.com/en/pricing?tab=terraform; get an intuition of how quickly the 500 resource limit is reached in practice
    - **Company:** IBM acquired HashiCorp in February 2025 (https://www.hashicorp.com/en/blog/hashicorp-officially-joins-the-ibm-family)
    - **Name:** research needed — why is it called "IBM HCP Terraform" on some pages and "HCP Terraform" on others? Are they the same thing? What is the official name going forward?
    - **Drift detection:** research and verify not included in free tier; check https://www.hashicorp.com/en/pricing?product_intent=terraform&tab=terraform (Features → Visibility & optimisation tab) — appears included in Standard/Premium/Enterprise but not Essentials; clarify which tier corresponds to the free tier; note: not needed as we implement periodic drift detection via GitHub Actions workflow
    - **Local execution mode** (https://developer.hashicorp.com/terraform/cloud-docs/overview#workspace-support-for-local-execution): workspace execution mode can be set to Local — not needed; we use HCP Terraform specifically for its remote execution environment
    - **Structural organisation:**
      - Hierarchy: account → organisation → project → workspace
      - Stacks: briefly describe key idea and how it differs from workspaces (https://developer.hashicorp.com/terraform/cloud-docs/stacks, https://developer.hashicorp.com/terraform/cloud-docs/stack-workspace) — not needed for Platypus
      - Organisations: clarify whether HCP Terraform organisations are separate from HCP organisations (https://portal.cloud.hashicorp.com/)
      - Workspace: maps to a single state file and set of variables
      - **Mapping to our use case:**
        - Account: danielmweibel@gmail.com (upstream-provided email to avoid circular dependencies)
        - Organisation: w5dio (Platypus platform lives in the w5d.io domain)
        - Project: Platypus (enables assigning shared variable sets across services)
        - Workspaces: one per Platypus service (one state file, one `terraform apply`)
    - **Version control integration** (https://developer.hashicorp.com/terraform/cloud-docs/overview#version-control-integration): HCP Terraform watches a GitHub repo and runs automatically on Terraform config changes — not needed; we trigger explicitly from the GitHub Actions workflow
    - **Command line integration** (https://developer.hashicorp.com/terraform/cloud-docs/overview#command-line-integration): what we use — standard `terraform` CLI with `terraform login` to authenticate; all `terraform` commands (`plan`, `apply`, etc.) transparently execute on HCP Terraform instead of locally
    - **Dynamic provider credentials** (https://developer.hashicorp.com/terraform/cloud-docs/dynamic-provider-credentials): briefly explain basic idea; verify whether transparent to Terraform code and execution — i.e. can a service developer switch from static secrets to dynamic provider credentials without changing anything else?
    - **Variable sets** (https://developer.hashicorp.com/terraform/cloud-docs/variables/managing-variables#variable-sets): can be defined at organisation and project level; variables in a variable set are assigned automatically to every workspace in the corresponding organisation or project
    - **GitHub Actions integration:** `hashicorp/setup-terraform` is the official GitHub Action for authenticating and setting up the Terraform CLI in a workflow; research whether there are other official HCP Terraform GitHub Actions beyond this
  - **HCP Terraform workspace creation — test results**
    - `workspaces.name`: if workspace does not exist, `terraform init` silently creates it — no prompts, no output messages; `terraform plan` and `terraform apply` work seamlessly on the new workspace
    - `workspaces.tags` (no matching workspace): `terraform init` prompts for a new workspace name, creates it, then proceeds; `terraform plan` and `terraform apply` work seamlessly
    - `workspaces.tags` (one matching workspace exists): workspace implicitly used, no output messages
    - `workspaces.tags` (multiple matching workspaces exist): one matching workspace implicitly used — selection rule unclear
  - **HCP Terraform authentication — research and document all methods**
    - **Local — user token:** interactive browser-based OAuth flow via `terraform login`; issues a personal user token stored in `~/.terraform.d/credentials.tfrc.json`; tied to the individual developer's account; one-time setup per machine
    - **GitHub Actions — team/organisation token:** long-lived API token generated in the HCP Terraform UI; stored as a GitHub Actions secret (`TF_API_TOKEN`); scoped to a team or organisation, not tied to a specific user; must be added to every new service repository
    - **GitHub Actions — OpenID Connect:** *(not yet verified — to be researched)* GitHub Actions authenticates to HCP Terraform via OpenID Connect Workload Identity Federation with no stored token; motivation: eliminate the need to add a static API token to every service repository; possible lead: https://github.com/hashicorp/hcp-auth-action

- [ ] Scaffold/draft `install` script
  - **Service repository initialisation and HCP Terraform workspace creation flow**

    **Path A — dev runs Terraform locally first, then pushes:**
    1. Install script: framework files + Terraform boilerplate code generated (including `cloud` block with HCP Terraform workspace name, config file reading, etc.)
    2. Dev writes Terraform code
    3. Dev runs `terraform login` (one-time per machine)
    4. Dev runs `terraform init` → workspace silently created in HCP Terraform
    5. Dev runs `terraform plan` / `terraform apply` → executes remotely on HCP Terraform
    6. Dev pushes
    7. Workflow runs `hashicorp/setup-terraform` action with `TF_API_TOKEN` secret → Terraform CLI authenticated to HCP Terraform
    8. Workflow runs `terraform init` → connects to existing workspace
    9. Workflow runs `terraform plan` / `terraform apply` → executes remotely on HCP Terraform

    **Path B — dev pushes without running any Terraform locally:**
    1. Install script: framework files + Terraform boilerplate code generated (including `cloud` block with HCP Terraform workspace name, config file reading, etc.)
    2. Dev writes Terraform code
    3. Dev pushes
    4. Workflow runs `hashicorp/setup-terraform` action with `TF_API_TOKEN` secret → Terraform CLI authenticated to HCP Terraform
    5. Workflow runs `terraform init` → workspace silently created in HCP Terraform
    6. Workflow runs `terraform plan` / `terraform apply` → executes remotely on HCP Terraform

- [ ] Implement `scripts/generate-readme` (based on JSON Schema to Markdown Tools research and decision in DECISIONS.md)
- [ ] Decide on config validation tooling in DECISIONS.md
- [ ] Implement `scripts/validate-config` (based on decision in DECISIONS.md)
- [ ] Create Config Schema section in AGENTS.md
  - [ ] Pop the stash containing the in-progress AGENTS.md changes: `git stash pop stash@{0}` (stash@{0}: WIP on main: 0d9453d Finalise research about JSON Schema to Markdown conversion tools)
  - When working on AGENTS.md, add the following to CLAUDE.md (remove when AGENTS.md is complete):
    **Goal of AGENTS.md:** AGENTS.md is installed into every service repository. Its purpose is to allow a coding agent that spins up in a fresh service repository (containing only the framework files) to fulfil its role without any additional context. The agent's role is to build a complete service together with the developer. The developer brings domain knowledge — what infrastructure to provision, what the config should look like — and the agent brings framework knowledge: what files to create, how to structure them, what constraints apply, and how everything fits together. They work interactively. AGENTS.md must give the agent everything it needs to know: the framework's structure, constraints, and conventions. It is not a tutorial — it is a complete reference for the agent's specific task.
- [ ] TBD: further tasks (workflow file, final implementation of `install` script, etc.)
