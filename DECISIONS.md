# DECISIONS.md — Platypus

Permanent record of architectural decisions and technology choices made during the design of the Platypus platform, including the rationales behind them. Serves as a reference so that already-answered questions are not revisited.

---

## Architectural Decisions

Deliberate choices about the scope, shape, and direction of the platform. Some decisions build on top of each other; others build on top of technology choices.

### Platform Purpose: Provisioning vs. Provisioning + Inventory

What's the fundamental purpose of the platform?

**Provisioning:**

- The platform creates and manages resources according to a declarative config
- The config is prescriptive — it defines what should exist, and the platform makes it so
- Every item in the config is owned by the platform; the platform is the single source of truth for its state
- The scope is strictly bounded: if it's not in the config, the platform doesn't know about it

**Provisioning + Inventory:**

- In addition to provisioning, the platform maintains a record of what resources and services exist, including those created and managed outside the platform
- The config is descriptive — it documents reality rather than prescribing it
- Items in the config may be managed by other tools, teams, or processes; the platform has no control over them
- The scope is unbounded: anything that exists can in principle be listed

**Decision:** Provisioning

**Rationale:** Mixing inventory into the platform conflates concerns. It bloats the config and requires actively recording and keeping in sync resources managed outside the platform — if not done diligently, the inventory becomes inaccurate and loses its value.

> Note: the inventory concern will be addressed in a separate future project (ideas in [w5dio/inventory](https://github.com/w5dio/inventory)) that discovers resources by querying live systems — purely read-only, no config required, and entirely unrelated to Platypus.

### Provisioning Model: Per-User vs. Shared

For whom is infrastructure provisioned?

**Per-User:**

- Infrastructure is provisioned on request for a specific user of the platform
- The lifecycle of the provisioned infrastructure is determined by the user that requested it
- To provision or modify infrastructure, the user edits the config of the corresponding platform service
- Users actively interact with the platform — it is visible and central to their workflow

**Shared:**

- Infrastructure is provisioned independently of any concrete request and is meant to be shared by any user that wants to use it
- The lifecycle of the infrastructure is determined by the platform operator, not by users
- Users access the infrastructure directly (e.g. via a URL) without going through the platform
- The platform is invisible to users of the shared infrastructure — it is only touched for operational changes that affect the shared infrastructure as a whole

**Decision:** Per-User

**Rationale:** Shared infrastructure doesn't fit the Platypus model: it is accessed directly by users without ever going through the platform, making a self-service provisioning platform the wrong tool to manage it.

> Note: shared infrastructure will be maintained in standalone repositories in the [w5d.io](https://github.com/w5dio/) GitHub organisation outside of Platypus or any other platform or framework.

### Interaction Model: Manual vs. Programmatic

How is the Platypus platform interacted with?

**Manual:**

- Human users interact with the platform by editing the service config in service repositories and reading provisioning outputs from the workflow
- Simple, no tooling dependencies on the user's side

**Programmatic:**

- Programs edit service configs in service repositories and consume provisioning outputs at runtime, enabling automated or event-driven provisioning
- Requires a reliable mechanism for programmatic output distribution — all viable options introduce significant complexity or external dependencies

**Decision:** Manual

**Rationale:** Programmatic output consumption introduces significant complexity and external dependencies. Manual usage requires no tooling and is fully sufficient for the current scope of the platform.

### Provisioning Outputs: Workflow vs. External System

Where can the user access the outputs of a provisioning run (e.g. the IP address of a provisioned compute instance)?

**Workflow:**

- Outputs are published to the GitHub Actions job summary after each provisioning run
- No new dependency and no additional service to build or maintain
- Exposes an internal implementation detail (GitHub Actions) to the user; the user must navigate the GitHub Actions UI to find outputs

**External System:**

- Outputs are stored in a dedicated external system (e.g. a key-value store, database, or separate repository) after each run
- Fully abstracts away internal implementation details
- Enables building on top of the platform (e.g. a UI or dashboard)
- Requires building a new service or adopting an external dependency

**Decision:** Workflow

**Rationale:** An external system requires building or adopting additional infrastructure. The job summary requires no additional infrastructure and is sufficient for the current manual interaction model.

### Framework Release Model: Discrete Versions vs. Always Latest

How are framework updates released to service repositories?

**Discrete Versions:**

- The framework repository publishes tagged releases (e.g. `v1.0`, `v1.1`); the install script fetches a specified version
- Different service repositories can pin to different versions; updates are explicit and opt-in
- Adds complexity to the installation process and requires ongoing version management across service repositories

**Always Latest:**

- The framework has no versions; the install script always fetches the current state of `main`
- Installation is simple — nothing to specify or track
- No record of which framework state is installed in each service repository; services installed at different times may have diverged

**Decision:** Always Latest

**Rationale:** Discrete versions add installation complexity with no practical benefit — there is no use case for deliberately running different framework versions across service repositories.

> Note: as a future improvement to the chosen approach, the install script could record the installed framework commit SHA in a `.version` file in the service repository. The GitHub Actions workflow could then compare this against the latest upstream SHA and automatically reinstall the framework when they diverge. This would automatically keep all service repositories current without manual intervention.

### Service Config Isolation: Shared Repo vs. Dedicated Repo vs. API

Where does the service config live relative to the service implementation?

**Shared Repo:**

- The config and service implementation coexist in the same repository
- Simple — no cross-repository coordination required
- Developer and user commits mix in the same Git history; no strict separation of concerns

**Dedicated Repo:**

- The config lives in a separate repository from the service implementation (GitOps pattern, e.g. ArgoCD, Flux)
- Clean separation: distinct Git histories and independent access control per repository
- Requires managing two repositories per service and cross-repository workflow coordination

**API:**

- Users submit config changes to a dedicated API server, which manages the connection to the implementation internally (Kubernetes pattern)
- Maximum abstraction for the user — no knowledge of the underlying implementation required
- Maximum complexity to build and operate — effectively building a platform on top of a platform

**Decision:** Shared Repo

**Rationale:** Split repositories and an API layer add significant complexity with no practical benefit at current scale — in practice, the service developer and service user are often the same person, so strict separation or access control is not required.

### Service Config Format: YAML vs. Terraform Variables

What format is the service config specified in?

**YAML:**

- The config is a human-readable YAML file
- Tool-agnostic — not tied to any specific IaC tool or syntax
- Low barrier to entry — no knowledge of Terraform or HCL required
- Requires decoding YAML in the Terraform code and mapping values to Terraform variables, adding a translation layer and complexity

**Terraform Variables:**

- The config is a `.tfvars` file; variables are defined in native HCL syntax and read directly by Terraform
- No translation layer — Terraform handles types and validation natively
- Ties the config format to Terraform; switching IaC tools would require rewriting the config
- Requires users to know HCL syntax

**Decision:** YAML

**Rationale:** Terraform Variables expose Terraform — an implementation detail — directly to the user. A core goal of the platform is to hide IaC tooling from users; the translation overhead of YAML is a manageable implementation detail in service of that goal.

### Service Docs Source: Developer vs. Config Schema

What is the source of the service documentation?

**Developer:**

- The developer writes the README by hand, following platform guidelines
- Flexible — documentation can cover anything beyond what the config schema describes
- Risk of docs drifting from the actual config over time; requires developer discipline to keep in sync

**Config Schema:**

- The README is generated automatically from the config schema
- The config schema is the single source of truth for both the config contract and the documentation — docs cannot drift from reality
- No separate documentation effort required; the developer only needs to maintain the schema

**Decision:** Config Schema

**Rationale:** The config schema is already required for config validation. Making it drive the documentation too eliminates any risk of docs drifting from the actual config, with no additional effort from the developer.

### Service Docs Generation: Developer vs. Workflow

Who runs the service documentation generation?

**Developer:**

- The developer runs a generation script locally and commits the result
- Requires generation tooling to be installed and run locally
- Documentation may fall out of sync if the developer forgets to regenerate after schema changes

**Workflow:**

- The GitHub Actions workflow generates the README from the config schema on every push and commits the result back if changed
- No tool dependencies on the developer — all generation tooling runs on the CI runner
- Guarantees eventual consistency — docs are always in sync with the schema without relying on developer discipline

**Decision:** Workflow

**Rationale:** Local generation requires tool dependencies and relies on developer discipline to stay in sync. The workflow imposes neither constraint.

### Service Config Schema Source: Agent vs. JSON Schema Generator

How is the service config schema created?

**Agent:**

- The developer describes the intended config to the agent — informally and incrementally — and the agent produces a complete, valid schema
- Straightforward to iterate: the developer can refine requirements conversationally
- Agent instructions can restrict how the schema must look (required annotations, forbidden keywords, etc.), making output more predictable
- Non-deterministic by nature, though constrained by the agent instructions

**JSON Schema Generator:**

- The developer runs a tool (e.g. `genson`, `quicktype`) that infers a schema from a fully specified example config
- Deterministic — the same input always produces the same schema
- Requires a fully specified example config upfront; cannot infer intent from informal descriptions
- Cannot produce annotations (field descriptions, examples, etc.) — the developer must add these to the schema manually
- Requires bundling the tool with the framework, adding a runtime dependency

**Decision:** Agent

**Rationale:** Coding agents produce reliable JSON schemas — the format is well-documented and well-represented in training data, and the agent already has the service context to write accurate field descriptions. The process can be further improved and made more predictable by restricting the schema format in the agent instructions.

### Service Config Schema Validation: Meta-Schema vs. None

How is the config schema itself validated?

**Meta-Schema:**

- A meta-schema is a schema for the schema — it validates whether the config schema itself conforms with requirements that we define
- Can only enforce negative constraints (e.g. reject disallowed keywords via `additionalProperties: false`) but cannot enforce positive requirements (e.g. a `description` annotation present on every field) — validation is therefore partial regardless
- Must be hosted at a stable public URL because validators fetch the `$schema` URI over HTTP — adding an external dependency and operational concern

**None:**

- No formal validation of the config schema; the agent is fully trusted to produce a schema that conforms to the agent instructions
- No hosting requirement, no additional infrastructure

**Decision:** None

**Rationale:** A meta-schema provides only partial validation — it cannot enforce positive requirements, meaning complete formal validation would require additional checks regardless. Since full enforcement is not achievable, fully relying on the agent as the single point of enforcement is simpler and more consistent.

> Note: it should be empirically tested in practice whether the agent can indeed reliably produce valid schemas that conform to the agent instructions.

### Service Config Schema Scope: Structure vs. Value Constraints

How detailed should the config schema be?

**Structure:**

- The schema defines the structure of the config — field types, required fields, and allowed values (`enum`) — but does not constrain values further (e.g. number ranges, string formats, array lengths)
- Smaller and more clearly bounded scope for the schema, making schema creation by the agent more predictable
- Risk that incorrectly formatted or otherwise invalid values are passed into the system

**Value Constraints:**

- In addition to structure, the schema encodes constraints on field values: ranges (`minimum`, `maximum`), string patterns (`pattern`), array lengths (`minItems`, `maxItems`), etc.
- Invalid values are detected before they enter the system
- Constraints expressed in the schema can be surfaced in the generated documentation
- May bloat the schema and make schema creation by the agent less predictable
- Cannot cover all possible constraint types — it is not guaranteed that no invalid values will ever enter the system regardless

**Decision:** Structure

**Rationale:** Schema value constraints cannot cover all possible constraints, so invalid values entering the system cannot be fully prevented regardless. Keeping the schema small and focused might weigh more than validation that is as complete as possible. Constraints on values can be informally expressed in the `description` annotations of the fields which are surfaced in the documentation.

---

## Technology Choices

Concrete technology choices for implementing the platform, recorded where multiple viable options exist and the decision is not self-evident.

### Infrastructure as Code (IaC) Tool: Terraform

**Considered options:**

- **Terraform:** industry standard, declarative HCL, excellent provider ecosystem (including Cloudflare), plan/apply is a natural validation/remediation mechanism
- **OpenTofu:** functionally identical to Terraform but no managed state service comparable to HCP Terraform's free tier
- **Pulumi:** uses real programming languages but is overkill for config-level IaC
- **Ansible:** procedural, not truly declarative, no state model
- **Direct API scripts:** no declarative or plan/diff benefits

**Decision:** Terraform

**Rationale:** Terraform is the industry standard with the best provider ecosystem and a plan/apply model that maps naturally onto the platform's validate-then-apply workflow. No other tool offers a meaningful advantage for our purpose — alternatives are either functionally identical (OpenTofu), overkill (Pulumi), or lack a declarative state model.

### Terraform State Storage: HCP Terraform

**Considered options:**

- **Local file:** not accessible from GitHub Actions — ruled out
- **Git:** state can contain secrets; causes messy diffs
- **S3 + DynamoDB:** viable but requires managing AWS credentials in GitHub Actions solely for state access
- **HCP Terraform:** free tier covers unlimited state storage and up to 500 managed resources, clean integration via `hashicorp/setup-terraform`, handles state locking out of the box

**Decision:** HCP Terraform

**Rationale:** HCP Terraform is the only option with zero additional credential management, native GitHub Actions integration, and free-tier state storage including locking — no other option matches this combination without added operational overhead.

### Automation Platform: GitHub Actions

**Considered options:**

- **GitHub Actions:** native to GitHub, free hosted runners, no additional integration needed
- **VPS + cron:** no native Git trigger, no audit trail, requires maintaining a server
- **Other CI platforms (CircleCI, Jenkins, Buildkite, etc.):** require external integration and credentials since code lives on GitHub
- **Serverless schedulers (AWS EventBridge + Lambda, GCP Cloud Scheduler + Cloud Run, etc.):** require additional wiring to integrate with GitHub

**Decision:** GitHub Actions

**Rationale:** GitHub Actions is native to GitHub — no external integration, credentials, or additional services required. All other options need some form of integration with GitHub to achieve the same result.

### Service Docs Generation: TBD

Tooling used by the workflow to generate the service documentation in the README from the config schema.

**Considered options:**

> WIP: work in progress - this list is not final.

- **[`json-schema-for-humans`](https://github.com/coveooss/json-schema-for-humans)** (Python): generates HTML or Markdown from JSON Schema; supports multiple output templates
- **[`@adobe/jsonschema2md`](https://github.com/adobe/jsonschema2md)** (Node.js): converts JSON Schema into Markdown; targets JSON Schema 2019-09
- **Custom script:** a bespoke script that generates the README directly from the config schema without relying on a third-party tool

**Decision:** TBD

**Rationale:** TBD

### Service Config Validation: TBD

Tooling used by the workflow to validate the config against the config schema.

**Considered options:**

> WIP: work in progress - this list is not final.

- **[`ajv-cli`](https://github.com/ajv-validator/ajv-cli)** (Node.js): CLI for AJV, one of the fastest and most widely used JSON Schema validators; supports YAML and JSON Schema draft-07 through 2020-12
- **[`check-jsonschema`](https://github.com/python-jsonschema/check-jsonschema)** (Python): CLI and pre-commit hooks for JSON Schema validation, maintained by the python-jsonschema team
- **[`sourcemeta/jsonschema`](https://github.com/sourcemeta/jsonschema)** (C++): comprehensive JSON Schema CLI covering validation, linting, formatting, and bundling; supports YAML

**Decision:** TBD

**Rationale:** TBD

### Secrets Management: TBD

Storage location for secrets required by the provisioning code.

**Considered options:**

> WIP: work in progress - this list is not final.

- **GitHub Actions Repository Secrets:** per-service secrets stored in each service repository; must be duplicated for shared credentials across services — impractical at scale
- **GitHub Actions Organisation Secrets:** stored once at the GitHub organisation level; accessible by all service repositories; native to GitHub, zero additional infrastructure
- **HCP Vault Secrets:** managed secrets store on HCP; natural complement to HCP Terraform which the platform already uses; accessible from GitHub Actions via the HCP Vault Secrets Action
- **Managed secret store (AWS Secrets Manager, Azure Key Vault, etc.):** dedicated cloud-hosted secrets stores; require cloud credentials in GitHub Actions to access them — adds an external dependency
- **Self-hosted secret manager (HashiCorp Vault, Infisical, OpenBao, etc.):** powerful and flexible; requires building and operating additional infrastructure
- **OIDC:** eliminates stored long-lived credentials by requesting short-lived tokens from the cloud provider at runtime; only applicable to cloud providers that support OIDC with GitHub Actions — not universal

**Decision:** TBD

**Rationale:** TBD
