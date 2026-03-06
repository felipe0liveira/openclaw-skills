---
name: gcloud
description: Built on top of the official Google Cloud SDK. Manage all Google Cloud Platform resources through the gcloud CLI by dynamically consulting `gcloud <group> --help` before executing any command, ensuring accurate and up-to-date syntax regardless of CLI version.
homepage: https://cloud.google.com/sdk/gcloud
metadata: {"clawdbot":{"emoji":"☁️","requires":{"bins":["gcloud"]},"install":[{"id":"gcloud-sdk","kind":"manual","url":"https://docs.cloud.google.com/sdk/docs/install-sdk","bins":["gcloud"],"label":"Install Google Cloud CLI (official)"}]}}
---

# Google Cloud CLI
gcloud - manage Google Cloud resources and developer workflow

This skill is built on top of the [`gcloud` CLI](https://cloud.google.com/sdk/gcloud). It covers the **entire surface area** of the `gcloud` CLI — every group, subcommand, and flag — without hardcoding any specific command syntax. Instead, it dynamically discovers the correct usage by running `gcloud <GROUP> --help` at execution time, making it resilient to version changes and always in sync with the locally installed CLI.

Available command groups are documented in [GROUPS.md](GROUPS.md). Refer to that file to map user requests to the correct `gcloud` command group. Once the group is identified, the agent must always consult the help output to understand the available subcommands and flags before executing anything.

## Installation
Follow the official Google Cloud SDK installation guide for your operating system:
https://docs.cloud.google.com/sdk/docs/install-sdk

After installation, initialize the CLI:
```bash
gcloud init
```

## Credentials & Environment
This skill uses whatever Google Cloud credentials and configuration are already present on the machine (`~/.config/gcloud`, Application Default Credentials, etc.). It operates with the full permissions of the authenticated account.

**Recommendations:**
- Use a dedicated, least-privilege service account for automation tasks.
- Test in a sandbox or non-production project before running commands against production.
- Review your active configuration with `gcloud config list` before running commands.
- Be aware that impersonation settings (`--impersonate-service-account`) and `CLOUDSDK_ACTIVE_CONFIG_NAME` can change the effective identity.

## What You Can Do
You can perform **any operation** available through the `gcloud` CLI by leveraging the full set of GROUPS (subcommands) listed in [GROUPS.md](GROUPS.md). This includes creating, configuring, managing, and deleting resources across all Google Cloud services.

## Workflow
Before executing any `gcloud` command, you **MUST** follow this workflow:

1. **Check active context** — Run `gcloud config list --format='text(core.account,core.project)'` to display the active account and project. Present this to the user so they are aware of which credentials and project will be used.
2. **Identify the correct GROUP** from [GROUPS.md](GROUPS.md) that matches the user's request.
3. **Run `gcloud <GROUP> --help`** in the terminal to understand the available subcommands, flags, and usage patterns for that group. Never guess or assume command syntax — always consult the help output first.
4. **Drill down as needed** — if the group has nested subgroups, run `gcloud <GROUP> <SUBGROUP> --help` to get the exact command syntax.
5. **For mutating/destructive operations** — Present the full command to the user and **wait for explicit approval** before executing. See the Destructive Operations section below.
6. **Execute the command** using the correct syntax obtained from the help output.

This discover-then-execute approach ensures commands are always accurate and up-to-date with the installed `gcloud` CLI version.

## Important Rules
- **Always use `--format=json`** when you need to parse or process the output programmatically.
- **Never hardcode command syntax from memory** — always verify with `--help` first, as commands and flags may vary across `gcloud` versions.
- **Never execute mutating or destructive commands autonomously** — always present the full command to the user and wait for explicit approval before running it.
- **Only use `--quiet` or `-q`** after the user has explicitly approved the command. Never add `--quiet` preemptively to suppress confirmation prompts — those prompts exist as a safety net.

## Destructive Operations
The following types of operations are considered **destructive or mutating** and require explicit user confirmation before execution:
- **Create** — provisioning new resources (instances, databases, buckets, clusters, etc.)
- **Delete / Destroy** — removing resources permanently
- **Update / Patch** — modifying existing resource configurations
- **Deploy** — deploying services, functions, or applications
- **IAM bindings** — granting or revoking roles and permissions
- **Set / Reset** — changing project settings, passwords, configurations

For these operations, the agent must:
1. Show the full command that will be executed.
2. Display the active account and project context.
3. Wait for the user to explicitly approve before running the command.

## Few-Shot Prompting Examples

### Example 1: Creating a Cloud Storage bucket

**User prompt:** "Create a new storage bucket called my-app-data in us-central1"

**Expected agent behavior:**
1. Run `gcloud config list --format='text(core.account,core.project)'` and show the active context to the user
2. Identify the relevant group: `storage`
3. Run `gcloud storage --help` to discover subcommands
4. Run `gcloud storage buckets --help` to find the `create` subcommand
5. Run `gcloud storage buckets create --help` to get the exact flags and syntax
6. Present the command to the user: `gcloud storage buckets create gs://my-app-data --location=us-central1`
7. **Wait for user approval** before executing (this is a create operation)

### Example 2: Deploying a Cloud Run service

**User prompt:** "Deploy my Docker image gcr.io/my-project/api:latest to Cloud Run in us-east1"

**Expected agent behavior:**
1. Run `gcloud config list --format='text(core.account,core.project)'` and show the active context to the user
2. Identify the relevant group: `run`
3. Run `gcloud run --help` to discover subcommands
4. Run `gcloud run deploy --help` to understand required flags (service name, image, region, etc.)
5. Present the command to the user: `gcloud run deploy api --image=gcr.io/my-project/api:latest --region=us-east1`
6. **Wait for user approval** before executing (this is a deploy operation)

### Example 3: Listing Kubernetes clusters (read-only)

**User prompt:** "Show me all GKE clusters in my project"

**Expected agent behavior:**
1. Run `gcloud config list --format='text(core.account,core.project)'` and show the active context to the user
2. Identify the relevant group: `container`
3. Run `gcloud container --help` to discover subcommands
4. Run `gcloud container clusters --help` to find the `list` subcommand
5. Run `gcloud container clusters list --help` to confirm syntax
6. Execute directly: `gcloud container clusters list --format=json` (read-only — no approval needed)

### Example 4: Managing IAM permissions

**User prompt:** "Grant the Storage Admin role to user alice@example.com on project my-project"

**Expected agent behavior:**
1. Run `gcloud config list --format='text(core.account,core.project)'` and show the active context to the user
2. Identify the relevant group: `projects`
3. Run `gcloud projects --help` to discover subcommands
4. Run `gcloud projects add-iam-policy-binding --help` to get exact syntax and required flags
5. Present the command to the user: `gcloud projects add-iam-policy-binding my-project --member=user:alice@example.com --role=roles/storage.admin`
6. **Wait for user approval** before executing (this is an IAM binding operation)

### Example 5: Creating a SQL database instance

**User prompt:** "Create a PostgreSQL 15 instance named prod-db with 4 vCPUs and 16GB RAM"

**Expected agent behavior:**
1. Run `gcloud config list --format='text(core.account,core.project)'` and show the active context to the user
2. Identify the relevant group: `sql`
3. Run `gcloud sql --help` to discover subcommands
4. Run `gcloud sql instances --help` to find the `create` subcommand
5. Run `gcloud sql instances create --help` to understand all available flags (database version, tier, region, etc.)
6. Present the full command to the user with all flags discovered from the help output
7. **Wait for user approval** before executing (this is a create operation)
