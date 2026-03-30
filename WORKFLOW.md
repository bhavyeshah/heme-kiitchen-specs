# OpenSpec Workflow — Hémé Kiitchen

Reference for the commands and steps used to propose and implement changes in this repo.

---

## Overview

This repo uses **OpenSpec** (`openspec` CLI) to manage feature proposals, design decisions, specs, and implementation tasks as versioned markdown files before any code is written.

The workflow has two phases:
1. **Propose** — generate all planning artifacts (`/opsx:propose`)
2. **Apply** — implement tasks from those artifacts (`/opsx:apply`)

---

## Phase 1: Propose a Change

### Step 1 — Create a new change

```bash
openspec new change "<name>"
```

- `<name>` is kebab-case (e.g. `heme-kiitchen-web-app`)
- Creates a scaffolded directory at `openspec/changes/<name>/`

**Example:**
```bash
openspec new change "heme-kiitchen-web-app"
```

---

### Step 2 — Check artifact status

```bash
openspec status --change "<name>" --json
```

Returns the list of artifacts, their current status (`ready`, `blocked`, `done`), and what `applyRequires` (the artifacts that must be complete before implementation can start).

**Example:**
```bash
openspec status --change "heme-kiitchen-web-app" --json
```

---

### Step 3 — Get instructions for an artifact

```bash
openspec instructions <artifact-id> --change "<name>" --json
```

Returns the `template`, `instruction`, `outputPath`, and `dependencies` for that artifact. Always run this before writing an artifact file.

**Example:**
```bash
openspec instructions proposal --change "heme-kiitchen-web-app" --json
openspec instructions design   --change "heme-kiitchen-web-app" --json
openspec instructions specs    --change "heme-kiitchen-web-app" --json
openspec instructions tasks    --change "heme-kiitchen-web-app" --json
```

---

### Step 4 — Create artifacts in dependency order

Write each artifact file at the path shown in `outputPath`. The dependency order for the `spec-driven` schema is:

```
proposal.md
    ├── design.md
    └── specs/<capability>/spec.md   (one file per capability)
            └── tasks.md
```

**Files to create:**

| Artifact | Path | What it covers |
|---|---|---|
| `proposal` | `proposal.md` | Why, what changes, capabilities list, impact |
| `design` | `design.md` | Tech decisions, architecture, risks, open questions |
| `specs` | `specs/<capability>/spec.md` | Requirements + testable scenarios per capability |
| `tasks` | `tasks.md` | Checkbox implementation tasks grouped and ordered |

After writing each file, re-run `openspec status` to confirm it moves to `done` before proceeding.

---

### Step 5 — Verify all artifacts are complete

```bash
openspec status --change "<name>"
```

When all artifacts show `[x]` and the output says **"All artifacts complete!"**, the change is ready to implement.

---

## Phase 2: Apply (Implement) a Change

```bash
# In Claude Code, run:
/opsx:apply
```

This reads `tasks.md` and walks through implementation task by task, updating checkbox state as each task completes.

---

## Phase 3: Archive a Completed Change

```bash
# In Claude Code, run:
/opsx:archive
```

Run after all tasks are done. Archives the change artifacts alongside the implemented specs.

---

## Quick Reference — All Commands

```bash
# Create a new change
openspec new change "<name>"

# Check status of a change
openspec status --change "<name>"
openspec status --change "<name>" --json

# Get instructions for a specific artifact
openspec instructions <artifact-id> --change "<name>" --json

# List all changes
openspec list
```

### Slash commands (used inside Claude Code sessions)

| Command | When to use |
|---|---|
| `/opsx:propose` | Start a new change — describe what you want to build |
| `/opsx:apply` | Begin or continue implementing tasks from a change |
| `/opsx:explore` | Think through ideas before committing to a proposal |
| `/opsx:archive` | Finalise and archive a completed change |

---

## Artifact Authoring Rules

- `context` and `rules` fields from `openspec instructions` are constraints **for the author** — never copy them into the artifact file.
- Spec files use `## ADDED Requirements` / `## MODIFIED Requirements` / `## REMOVED Requirements` headers.
- Each requirement: `### Requirement: <name>`
- Each scenario: `#### Scenario: <name>` (exactly 4 hashtags) with `WHEN` / `THEN` format.
- Every requirement must have at least one scenario.
- Task items must use `- [ ] X.Y Description` format or they won't be tracked by apply.

---

## Change Directory Structure

```
openspec/
  changes/
    heme-kiitchen-web-app/
      .openspec.yaml       # change metadata (auto-generated)
      proposal.md          # why + capabilities
      design.md            # how + decisions
      tasks.md             # implementation checklist
      specs/
        customer-storefront/spec.md
        platter-builder/spec.md
        checkout/spec.md
        order-management/spec.md
        admin-panel/spec.md
        product-catalog/spec.md
        inventory-tracking/spec.md
```
