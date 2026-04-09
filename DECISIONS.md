# DECISIONS.md — Platform Constitution

This file records the rationale behind key platform technology decisions. It is not used during operation — it exists purely as a historical record of what options were considered and why specific choices were made.

## Terraform as IaC Tool

**Decision:** All IaC systems in the platform use Terraform.

**Options considered:**

- **Terraform:** industry standard, declarative HCL, excellent provider ecosystem (including Cloudflare), plan/diff output is a natural validation mechanism, apply is a natural remediation mechanism
- **OpenTofu:** functionally identical to Terraform but has no managed state service comparable to HCP Terraform's free tier — no practical advantage for this platform
- **Pulumi:** uses real programming languages but is overkill for config-level IaC
- **Ansible:** procedural, not truly declarative, no state model
- **Direct API scripts:** no declarative or plan/diff benefits

## HCP Terraform for Terraform State Storage

**Decision:** All Terraform state is stored in HCP Terraform.

**Options considered:**

- **Local file:** not accessible from GitHub Actions — ruled out
- **Git:** not recommended — state can contain secrets, causes messy diffs
- **S3 + DynamoDB:** viable but requires managing AWS credentials in GitHub Actions solely for state access
- **HCP Terraform:** free tier covers unlimited state storage and up to 500 managed resources, clean integration with GitHub Actions via the official `hashicorp/setup-terraform` action, handles state locking out of the box

## GitHub Actions as Automation Platform

**Decision:** All automated validation and remediation workflows run on GitHub Actions.

GitHub Actions is the natural choice as all code already lives on GitHub. It natively supports all three validation triggers required by the constitution: on config change (push), periodically (schedule), and on demand (workflow_dispatch).

**Options considered:**

- **GitHub Actions:** native to GitHub, free hosted runners, supports all three triggers, works for both Terraform and custom scripts — no additional integration needed
- **VPS + cron:** simple but no native git trigger, no audit trail, requires maintaining a server
- **Other CI platforms (CircleCI, Jenkins, Buildkite etc.):** functionally similar but require external integration since code lives on GitHub — strictly worse for this use case
- **Serverless schedulers (AWS EventBridge + Lambda, GCP Cloud Scheduler + Cloud Run, Azure Logic Apps etc.):** support cron scheduling and are pay-per-execution, but none natively support the git push trigger without additional wiring, and they add cloud infrastructure overhead just for running scripts
