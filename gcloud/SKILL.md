---
name: gcloud
description: Built on top of the official Google Cloud SDK. Manage all Google Cloud Platform resources through the gcloud CLI by dynamically consulting `gcloud <group> --help` before executing any command, ensuring accurate and up-to-date syntax regardless of CLI version.
homepage: https://cloud.google.com/sdk/gcloud
metadata: {"clawdbot":{"emoji":"☁️","requires":{"bins":["gcloud"]},"install":[{"id":"gcloud-sdk","kind":"manual","url":"https://docs.cloud.google.com/sdk/docs/install-sdk","bins":["gcloud"],"label":"Install Google Cloud CLI (official)"}]}}
---

# Google Cloud CLI
gcloud - manage Google Cloud resources and developer workflow

This skill is built on top of the [`gcloud` CLI](https://cloud.google.com/sdk/gcloud). It covers the **entire surface area** of the `gcloud` CLI — every group, subcommand, and flag — without hardcoding any specific command syntax. Instead, it dynamically discovers the correct usage by running `gcloud <GROUP> --help` at execution time, making it resilient to version changes and always in sync with the locally installed CLI.

The GROUPS section below serves as a reference to map user requests to the correct `gcloud` command group. Once the group is identified, the agent must always consult the help output to understand the available subcommands and flags before executing anything.

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
You can perform **any operation** available through the `gcloud` CLI by leveraging the full set of GROUPS (subcommands) listed below. This includes creating, configuring, managing, and deleting resources across all Google Cloud services.

## Workflow
Before executing any `gcloud` command, you **MUST** follow this workflow:

1. **Check active context** — Run `gcloud config list --format='text(core.account,core.project)'` to display the active account and project. Present this to the user so they are aware of which credentials and project will be used.
2. **Identify the correct GROUP** from the GROUPS section below that matches the user's request.
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

## Gcloud CLI Reference (gcloud --help)

SYNOPSIS
    gcloud GROUP | COMMAND [--account=ACCOUNT]
        [--billing-project=BILLING_PROJECT] [--configuration=CONFIGURATION]
        [--flags-file=YAML_FILE] [--flatten=[KEY,...]] [--format=FORMAT]
        [--help] [--project=PROJECT_ID] [--quiet, -q]
        [--verbosity=VERBOSITY; default="warning"] [--version, -v] [-h]
        [--access-token-file=ACCESS_TOKEN_FILE]
        [--impersonate-service-account=SERVICE_ACCOUNT_EMAILS] [--log-http]
        [--trace-token=TRACE_TOKEN] [--no-user-output-enabled]

DESCRIPTION
    The gcloud CLI manages authentication, local configuration, developer
    workflow, and interactions with the Google Cloud APIs.

    For a quick introduction to the gcloud CLI, a list of commonly used
    commands, and a look at how these commands are structured, run gcloud
    cheat-sheet or see the `gcloud` CLI cheat sheet
    (https://cloud.google.com/sdk/docs/cheatsheet).

GLOBAL FLAGS
     --account=ACCOUNT
        Google Cloud user account to use for invocation. Overrides the default
        core/account property value for this command invocation.

     --billing-project=BILLING_PROJECT
        The Google Cloud project that will be charged quota for operations
        performed in gcloud. If you need to operate on one project, but need
        quota against a different project, you can use this flag to specify the
        billing project. If both billing/quota_project and --billing-project
        are specified, --billing-project takes precedence. Run $ gcloud config
        set --help to see more information about billing/quota_project.

     --configuration=CONFIGURATION
        File name of the configuration to use for this command invocation. For
        more information on how to use configurations, run: gcloud topic
        configurations. You can also use the CLOUDSDK_ACTIVE_CONFIG_NAME
        environment variable to set the equivalent of this flag for a terminal
        session.

     --flags-file=YAML_FILE
        A YAML or JSON file that specifies a --flag:value dictionary. Useful
        for specifying complex flag values with special characters that work
        with any command interpreter. Additionally, each --flags-file arg is
        replaced by its constituent flags. See $ gcloud topic flags-file for
        more information.

     --flatten=[KEY,...]
        Flatten name[] output resource slices in KEY into separate records for
        each item in each slice. Multiple keys and slices may be specified.
        This also flattens keys for --format and --filter. For example,
        --flatten=abc.def flattens abc.def[].ghi references to abc.def.ghi. A
        resource record containing abc.def[] with N elements will expand to N
        records in the flattened output. This allows us to specify what
        resource-key the filter will operate on. This flag interacts with other
        flags that are applied in this order: --flatten, --sort-by, --filter,
        --limit.

     --format=FORMAT
        Sets the format for printing command output resources. The default is a
        command-specific human-friendly output format. If both core/format and
        --format are specified, --format takes precedence. --format and
        core/format both take precedence over core/default_format. The
        supported formats are limited to: config, csv, default, diff, disable,
        flattened, get, json, list, multi, none, object, table, text, value,
        yaml. For more details run $ gcloud topic formats. Run $ gcloud config
        set --help to see more information about core/format

     --help
        Display detailed help.

     --project=PROJECT_ID
        The Google Cloud project ID to use for this invocation. If omitted,
        then the current project is assumed; the current project can be listed
        using gcloud config list --format='text(core.project)' and can be set
        using gcloud config set project PROJECTID.

        --project and its fallback core/project property play two roles in the
        invocation. It specifies the project of the resource to operate on. It
        also specifies the project for API enablement check, quota, and
        billing. To specify a different project for quota and billing, use
        --billing-project or billing/quota_project property.

     --quiet, -q
        Disable all interactive prompts when running gcloud commands. If input
        is required, defaults will be used, or an error will be raised.

        Overrides the default core/disable_prompts property value for this
        command invocation. This is equivalent to setting the environment
        variable CLOUDSDK_CORE_DISABLE_PROMPTS to 1.

     --verbosity=VERBOSITY; default="warning"
        Override the default verbosity for this command. Overrides the default
        core/verbosity property value for this command invocation. VERBOSITY
        must be one of: debug, info, warning, error, critical, none.

     --version, -v
        Print version information and exit. This flag is only available at the
        global level.

     -h
        Print a summary help and exit.

OTHER FLAGS
     --access-token-file=ACCESS_TOKEN_FILE
        A file path to read the access token. Use this flag to authenticate
        gcloud with an access token. The credentials of the active account (if
        exists) will be ignored. The file should only contain an access token
        with no other information. Overrides the default auth/access_token_file
        property value for this command invocation.

     --impersonate-service-account=SERVICE_ACCOUNT_EMAILS
        For this gcloud invocation, all API requests will be made as the given
        service account or target service account in an impersonation
        delegation chain instead of the currently selected account. You can
        specify either a single service account as the impersonator, or a
        comma-separated list of service accounts to create an impersonation
        delegation chain. The impersonation is done without needing to create,
        download, and activate a key for the service account or accounts.

        In order to make API requests as a service account, your currently
        selected account must have an IAM role that includes the
        iam.serviceAccounts.getAccessToken permission for the service account
        or accounts.

        The roles/iam.serviceAccountTokenCreator role has the
        iam.serviceAccounts.getAccessToken permission. You can also create a
        custom role.

        You can specify a list of service accounts, separated with commas. This
        creates an impersonation delegation chain in which each service account
        delegates its permissions to the next service account in the chain.
        Each service account in the list must have the
        roles/iam.serviceAccountTokenCreator role on the next service account
        in the list. For example, when --impersonate-service-account=
        SERVICE_ACCOUNT_1,SERVICE_ACCOUNT_2, the active account must have the
        roles/iam.serviceAccountTokenCreator role on SERVICE_ACCOUNT_1, which
        must have the roles/iam.serviceAccountTokenCreator role on
        SERVICE_ACCOUNT_2. SERVICE_ACCOUNT_1 is the impersonated service
        account and SERVICE_ACCOUNT_2 is the delegate.

        Overrides the default auth/impersonate_service_account property value
        for this command invocation.

     --log-http
        Log all HTTP server requests and responses to stderr. Overrides the
        default core/log_http property value for this command invocation.

     --trace-token=TRACE_TOKEN
        Token used to route traces of service requests for investigation of
        issues. Overrides the default core/trace_token property value for this
        command invocation.

     --user-output-enabled
        Print user intended output to the console. Overrides the default
        core/user_output_enabled property value for this command invocation.
        Use --no-user-output-enabled to disable.

GROUPS
    GROUP is one of the following:

     access-approval
        Manage Access Approval requests and settings.

     access-context-manager
        Manage Access Context Manager resources.

     active-directory
        Manage Managed Microsoft AD resources.

     ai
        Manage entities in Vertex AI.

     ai-platform
        Manage AI Platform jobs and models.

     alloydb
        Create and manage AlloyDB databases.

     anthos
        Anthos command Group.

     api-gateway
        Manage Cloud API Gateway resources.

     apigee
        Manage Apigee resources.

     app
        Manage your App Engine deployments.

     apphub
        Manage App Hub resources.

     artifacts
        Manage Artifact Registry resources.

     asset
        Manage the Cloud Asset Inventory.

     assured
        Read and manipulate Assured Workloads data controls.

     audit-manager
        Enroll resources, audit workloads and generate reports.

     auth
        Manage oauth2 credentials for the Google Cloud CLI.

     backup-dr
        Manage Backup and DR resources.

     batch
        Manage Batch resources.

     bigtable
        Manage your Cloud Bigtable storage.

     billing
        Manage billing accounts and associate them with projects.

     bms
        Manage Bare Metal Solution resources.

     bq
        Manage Bq resources.

     builds
        Create and manage builds for Google Cloud Build.

     certificate-manager
        Manage SSL certificates for your Google Cloud projects.

     cloud-shell
        Manage Google Cloud Shell.

     cloudlocationfinder
        Manage Cloudlocationfinder resources.

     colab
        Manage Colab Enterprise resources.

     components
        List, install, update, or remove Google Cloud CLI components.

     composer
        Create and manage Cloud Composer Environments.

     compute
        Create and manipulate Compute Engine resources.

     config
        View and edit Google Cloud CLI properties.

     container
        Deploy and manage clusters of machines for running containers.

     data-catalog
        Manage Data Catalog resources.

     database-migration
        Manage Database Migration Service resources.

     dataflow
        Manage Google Cloud Dataflow resources.

     dataplex
        Manage Dataplex resources.

     dataproc
        Create and manage Google Cloud Dataproc clusters and jobs.

     datastore
        Manage your Cloud Datastore resources.

     datastream
        Manage Cloud Datastream resources.

     deploy
        Create and manage Cloud Deploy resources.

     deployment-manager
        Manage deployments of cloud resources.

     developer-connect
        Manage Developer Connect resources.

     dns
        Manage your Cloud DNS managed-zones and record-sets.

     domains
        Manage domains for your Google Cloud projects.

     edge-cache
        Manage Media CDN resources.

     edge-cloud
        Manage edge-cloud resources.

     emulators
        Set up your local development environment using emulators.

     endpoints
        Create, enable and manage API services.

     essential-contacts
        Manage Essential Contacts.

     eventarc
        Manage Eventarc resources.

     filestore
        Create and manipulate Filestore resources.

     firebase
        Work with Google Firebase.

     firestore
        Manage your Cloud Firestore resources.

     functions
        Manage Google Cloud Functions.

     gemini
        Manage resources associated with Gemini Code Assist and Gemini Cloud
        Assist.

     healthcare
        Manage Cloud Healthcare resources.

     iam
        Manage IAM service accounts and keys.

     iap
        Manage IAP policies.

     identity
        Manage Cloud Identity Groups and Memberships resources.

     ids
        Manage Cloud IDS.

     immersive-stream
        Manage Immersive Stream resources.

     infra-manager
        Manage Infra Manager resources.

     kms
        Manage cryptographic keys in the cloud.

     logging
        Manage Cloud Logging.

     looker
        Manage Looker resources.

     lustre
        Manage Lustre resources.

     managed-kafka
        Administer Managed Service for Apache Kafka clusters, topics, and
        consumer groups.

     memcache
        Manage Cloud Memorystore Memcached resources.

     memorystore
        Manage Memorystore resources.

     metastore
        Manage Dataproc Metastore resources.

     migration
        The root group for various Cloud Migration teams.

     ml
        Use Google Cloud machine learning capabilities.

     model-armor
        Model Armor is a service offering LLM-agnostic security and AI safety
        measures to mitigate risks associated with large language models
        (LLMs).

     monitoring
        Manage Cloud Monitoring dashboards.

     netapp
        Create and manipulate Cloud NetApp Files resources.

     network-connectivity
        Manage Network Connectivity resources.

     network-management
        Manage Network Management resources.

     network-security
        Manage Network Security resources.

     network-services
        Manage Network Services resources.

     notebooks
        Notebooks Command Group.

     observability
        Manage Observability resources.

     oracle-database
        Manage Oracle Database resources.

     org-policies
        Create and manage Organization Policies.

     organizations
        Create and manage Google Cloud Platform Organizations.

     pam
        Manage Privileged Access Manager (PAM) entitlements and grants.

     parametermanager
        Parameter Manager is a single source of truth to store, access and
        manage the lifecycle of your application parameters.

     policy-intelligence
        A platform to help better understand, use, and manage policies at
        scale.

     policy-troubleshoot
        Troubleshoot Google Cloud Platform policies.

     privateca
        Manage private Certificate Authorities on Google Cloud.

     projects
        Create and manage project access policies.

     publicca
        Manage accounts for Google Trust Services' Certificate Authority.

     pubsub
        Manage Cloud Pub/Sub topics, subscriptions, and snapshots.

     recaptcha
        Manage reCAPTCHA Enterprise Keys.

     recommender
        Manage Cloud recommendations and recommendation rules.

     redis
        Manage Cloud Memorystore Redis resources.

     resource-manager
        Manage Cloud Resources.

     run
        Manage your Cloud Run applications.

     scc
        Manage Cloud SCC resources.

     scheduler
        Manage Cloud Scheduler jobs and schedules.

     secrets
        Manage secrets on Google Cloud.

     service-directory
        Command groups for Service Directory.

     service-extensions
        Manage Service Extensions resources.

     services
        List, enable and disable APIs and services.

     source
        Cloud git repository commands.

     source-manager
        Manage Secure Source Manager resources.

     spanner
        Command groups for Cloud Spanner.

     sql
        Create and manage Google Cloud SQL databases.

     storage
        Create and manage Cloud Storage buckets and objects.

     tasks
        Manage Cloud Tasks queues and tasks.

     telco-automation
        Manage Telco Automation resources.

     topic
        gcloud supplementary help.

     transcoder
        Manage Transcoder jobs and job templates.

     transfer
        Manage Transfer Service jobs, operations, and agents.

     vmware
        Manage Google Cloud VMware Engine resources.

     workbench
        Workbench Command Group.

     workflows
        Manage your Cloud Workflows resources.

     workspace-add-ons
        Manage Google Workspace Add-ons resources.

     workstations
        Manage Cloud Workstations resources.

COMMANDS
    COMMAND is one of the following:

     cheat-sheet
        Display gcloud cheat sheet.

     docker
        (DEPRECATED) Enable Docker CLI access to Google Container Registry.

     feedback
        Provide feedback to the Google Cloud CLI team.

     help
        Search gcloud help text.

     info
        Display information about the current gcloud environment.

     init
        Initialize or reinitialize gcloud.

     survey
        Invoke a customer satisfaction survey for Google Cloud CLI.

     version
        Print version information for Google Cloud CLI components.
