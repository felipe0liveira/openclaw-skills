# Instructions

## What are Skills?

Skills in this repository are specialized documentation resources designed to teach AI agents how to use specific tools, services, or technologies effectively.

## Purpose

Each skill serves as a focused guide that:

- Provides clear, actionable instructions for AI agents
- Explains how to interact with a specific resource or service
- Includes practical examples and best practices
- Offers cheat sheets and quick reference materials

## Structure

Skills are organized by technology or service:

- Each folder represents a different skill domain (e.g., `firestore/`, `gcloud/`)
- Documentation files (`.md`) contain the instructional content
- Cheat sheets provide quick reference for common operations

## How to Use

AI agents can reference these skills when they need to:

1. Learn how to interact with a specific service or tool
2. Find syntax and command references
3. Understand best practices and common patterns
4. Get step-by-step guidance for specific operations

## Creating New Skills

When adding a new skill:

1. Create a dedicated folder for the technology/service
2. Include a `SKILL.md` with comprehensive instructions
3. Optionally add a `cheatsheet.md` for quick reference
4. Focus on practical, actionable content that an AI agent can follow

## Guidelines for New Skills (Tools)

Use the following standards when creating any new skill/tool documentation.

### 1. Instruction Scope

- Keep instructions strictly limited to the declared purpose of the skill.
- Do not add unrelated commands, endpoints, or file access.
- Clearly define what the skill can and cannot do.
- Require context checks before execution (for example, active account/project).

### 2. Credentials and Identity

- Never ask for hardcoded secrets in the documentation.
- Prefer short-lived tokens over static credentials when possible.
- Always document that the skill inherits the permissions of the active identity.
- Require dedicated least-privilege service accounts for automation.
- Explicitly warn against using personal or admin identities.

### 3. Approval and Execution Safety

- Require explicit user approval before executing commands.
- For high-sensitivity domains, require approval for both read and write operations.
- Always show the full command before execution.
- Document destructive/mutating actions clearly.
- Include rollback or recovery guidance when applicable.

### 4. Persistence and Privilege

- Avoid persistent behavior unless strictly necessary.
- Do not request permanent background presence by default.
- Do not modify system-wide settings unless explicitly part of the skill scope.
- Use the minimum required privileges and binaries only.

### 5. Installation and Supply Chain

- Prefer official vendor documentation links for installation.
- Avoid custom install scripts unless absolutely required.
- If installation steps are needed, keep them minimal and auditable.
- Document required binaries explicitly in metadata.

### 6. Documentation Pattern

For consistency, structure new skills with:

1. `SKILL.md` (core policy, scope, workflow, safety rules)
2. `installation.md` (setup and prerequisites)
3. `examples.md` (few-shot prompts and command examples)
4. `troubleshooting.md` (errors and diagnostics)

### 7. Validation Checklist (Before Publishing)

- Name and description match actual behavior.
- Scope is proportional and does not include unrelated capabilities.
- Credentials model is explicit and least-privilege oriented.
- Approval policy is clear and consistent across all docs.
- Examples follow the same safety rules defined in `SKILL.md`.
- No unsupported frontmatter fields are used.
