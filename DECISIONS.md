# DECISIONS.md — Platypus

Permanent record of architectural and technology decisions made during the design of the Platypus framework.

---

## Architectural Decisions

Deliberate choices about the scope, shape, and direction of the platform — what it is, what it does, and what it explicitly does not do.

### Platform Purpose: Provisioning vs. Inventory

**Decision:** provisioning only. Every resource in a service's config must be something the service directly creates and manages.

Two fundamentally different purposes are possible for a platform that manages infrastructure:

**Provisioning**

- The platform creates and manages resources according to a declarative config
- The config is prescriptive — it defines what should exist, and the platform makes it so
- Every item in the config is owned by the platform; the platform is the single source of truth for its state
- The scope is strictly bounded: if it's not in the config, the platform doesn't know about it

**Inventory**

- The platform maintains a record of what resources and services exist, including those created and managed outside the platform
- The config is descriptive — it documents reality rather than prescribing it
- Items in the config may be managed by other tools, teams, or processes; the platform has no control over them
- The scope is unbounded: anything that exists can in principle be listed

Platypus focuses exclusively on provisioning. Mixing inventory into the same config introduces irresolvable conflicts: a provisioning platform must be able to act on its config (create, update, delete resources), but acting on inventory items it doesn't own would be destructive. Keeping the two concerns separate avoids this entirely.

> Note: the inventory concern may be addressed in a separate future project (initial ideas in [w5dio/inventory](https://github.com/w5dio/inventory)) that discovers resources by querying live systems — purely read-only, no config required, and entirely unrelated to Platypus.

### Service Lifecycle: On-Demand Provisioning vs. Shared Infrastructure

**Decision:** on-demand provisioning only.

**On-demand provisioning**

- Infrastructure is provisioned on request for a specific app or user
- The lifecycle of the provisioned infrastructure is determined by the app or user that requested it
- To provision or modify infrastructure, the user edits the config of the corresponding platform service
- Users actively interact with the platform — it is visible and central to their workflow

**Shared infrastructure**

- Infrastructure is provisioned independently of any concrete request and is meant to be shared by any app or user that wants to use it
- The lifecycle of the infrastructure is determined by the platform operator, not by individual users
- Users or apps access the infrastructure directly (e.g. via a URL) without going through the platform
- The platform is invisible to users in practice — it is only touched for operational changes that affect the infrastructure as a whole

Platypus provides on-demand provisioning only. A unified platform framework only provides value when users actively interact with it.

> Note: shared infrastructure will be maintained as standalone repositories in the w5d.io GitHub organisation, outside of the Platypus project.

### Usage Mode: Manual vs. Programmatic

**Decision:** manual.

- **Config editing:** a developer edits `config.yaml` by hand; no program writes to it
- **Output reading:** a developer reads outputs from the GitHub Actions job summary and manually wires relevant values into app config; no program reads platform outputs at runtime

Rejected alternative — dynamic/programmatic usage:
- Introduces excessive complexity, coupling, and failure vectors for a modest gain
- Options considered for dynamic/programmatic output consumption:
  - **Terraform remote state:** requires the client to also use Terraform (dealbreaker: clients should need no IaC tooling)
  - **SSM Parameter Store:** clients need AWS credentials just to read a value (introduces an AWS dependency for unrelated clients)
  - **Shared state repo/output file committed to a repo:** pollutes git history with machine-generated commits; concurrency risk; conflates human intent (config) with machine state (outputs)
  - **HCP Terraform API:** workable but against the grain — HCP Terraform is not designed for cross-system output distribution
  - **Purpose-built outputs store:** cleanest interface, but requires building and hosting a new service

### Config Format: Custom YAML vs. IaC Config

**Decision:** custom YAML.

Rejected alternative — Terraform HCL directly:
- Requires the user to know HCL syntax and Terraform concepts
- Ties the config format to Terraform; switching provisioning tools would require rewriting user-facing config
- Higher barrier to entry; harder to read at a glance

### Output Surface: GitHub Actions Job Summary vs. Key-Value Store

**Decision:** GitHub Actions job summary.

**GitHub Actions job summary**

- No new dependency and no additional service to build or maintain
- Exposes an internal implementation detail (GitHub Actions) to the user
- The user must navigate the GitHub Actions UI to find the output

**Hosted key-value store**

- Fully abstracts away the internal implementation
- Extensible: could support a future GUI or dashboard
- Requires building a new service or taking on an external dependency

The job summary was chosen for its simplicity and zero additional infrastructure.

> Note: the key-value store remains an option for a future version, particularly if a more polished output UX becomes a priority.

---

## Technology Choices

### IaC Tool: Terraform

**Decision:** all IaC in the platform uses Terraform.

- **Terraform:** industry standard, declarative HCL, excellent provider ecosystem (including Cloudflare), plan/apply is a natural validation/remediation mechanism
- **OpenTofu:** functionally identical but no managed state service comparable to HCP Terraform's free tier
- **Pulumi:** uses real programming languages but is overkill for config-level IaC
- **Ansible:** procedural, not truly declarative, no state model
- **Direct API scripts:** no declarative or plan/diff benefits

### Terraform State Storage: HCP Terraform

**Decision:** all Terraform state is stored in HCP Terraform.

- **Local file:** not accessible from GitHub Actions — ruled out
- **Git:** state can contain secrets, causes messy diffs
- **S3 + DynamoDB:** viable but requires managing AWS credentials in GitHub Actions solely for state access
- **HCP Terraform:** free tier covers unlimited state storage and up to 500 managed resources, clean integration via `hashicorp/setup-terraform`, handles state locking out of the box

### Automation Platform: GitHub Actions

**Decision:** all automated workflows run on GitHub Actions.

Natively supports all three required triggers: on config change (push), periodically (schedule), and on demand (workflow_dispatch).

- **GitHub Actions:** native to GitHub, free hosted runners, no additional integration needed
- **VPS + cron:** no native git trigger, no audit trail, requires maintaining a server
- **Other CI platforms (CircleCI, Jenkins, Buildkite etc.):** require external integration since code lives on GitHub
- **Serverless schedulers (AWS EventBridge + Lambda, GCP Cloud Scheduler + Cloud Run, etc.):** none natively support the git push trigger without additional wiring

### Documentation Generation Tooling

> **TBD:** tooling for `jsonschema2readme.sh`.

Options considered:

- **`json-schema-for-humans`** (Python): generates Markdown from JSON Schema in various formats
- **`@adobe/jsonschema2md`** (npm): generates Markdown from JSON Schema, well-maintained
- **Custom script:** a wrapper around one of the above to enforce the specific README structure the platform requires

The custom script layer is likely necessary regardless of which tool is chosen.

### Secrets Management

> **TBD:** options for maintaining and exposing secrets for provisioning runs.
