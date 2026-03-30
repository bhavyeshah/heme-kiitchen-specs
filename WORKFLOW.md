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

---

## Spec Change Log

A record of every `/opsx:explore` session and the spec modifications made.

---

### [1] Initial Proposal — `heme-kiitchen-web-app`

**Prompt:** Full product brief for Hémé Kiitchen — premium Jain-friendly dip brand, online + offline orders, admin panel, JSON storage.

**Artifacts created (all new):**
- `proposal.md` — 7 capabilities identified: customer-storefront, platter-builder, checkout, order-management, admin-panel, product-catalog, inventory-tracking
- `design.md` — tech decisions: Next.js + Express, JSON repository pattern, `/admin` route group, `/public/images/` asset convention
- `specs/customer-storefront/spec.md`
- `specs/platter-builder/spec.md`
- `specs/checkout/spec.md`
- `specs/order-management/spec.md`
- `specs/admin-panel/spec.md`
- `specs/product-catalog/spec.md`
- `specs/inventory-tracking/spec.md`
- `tasks.md` — 40 tasks across 11 groups

---

### [2] Delivery Type + Delivery Charges

**Prompt:** Orders should support Pickup or Home Delivery. Orders above ₹1500 get free delivery. Below ₹1500, delivery charges apply, paid directly to the third-party delivery partner (amount unknown at order time — notice only).

**Decisions made:**
- `delivery_charges_applicable` is a boolean flag; no amount stored
- Shared minimal status set; `dispatched` skipped for pickup
- `completed` replaces `delivered` to unify pickup and delivery end states

**Files updated:**
| File | What changed |
|---|---|
| `specs/order-management/spec.md` | Added `delivery_type`, conditional `delivery_address`, `delivery_charges_applicable` flag with ₹1500 rule to schema; status set simplified to pending → confirmed → preparing → dispatched → completed → cancelled |
| `specs/checkout/spec.md` | Added delivery type selector requirement; conditional address field; live delivery charge notice; two confirmation page scenarios (pickup vs home delivery) |

---

### [3] Order Cancellation + Edit + Soft Delete

**Prompt:** Admin should be able to cancel any order. Admin should also be able to update (broad scope — all fields) or remove (soft delete) an order to fix erroneous entries.

**Decisions made:**
- `completed` and `cancelled` are terminal states — no transitions out
- Broad edit allowed on any non-terminal order; recalculates total and delivery charge flag when items or delivery type change
- Soft delete: `deleted=true` + `deleted_at` timestamp; deleted orders excluded from default list views

**Files updated:**
| File | What changed |
|---|---|
| `specs/order-management/spec.md` | Added `deleted` + `deleted_at` to schema; added cancellation scenarios (terminal state enforcement); added "Admin can edit order details" requirement (broad scope, 4 scenarios); added "Admin can soft-delete an erroneous order" requirement (4 scenarios) |

---

### [4] Inventory Deferred

**Prompt:** Keep inventory management as a placeholder only. Inventory will be managed manually for now and plugged in later when the business scales.

**Decisions made:**
- No stock-level enforcement anywhere in the system at launch
- No `/admin/inventory` page built
- Plug-in points documented for future: product schema field, admin page, platter builder enforcement, stock decrement on order

**Files updated:**
| File | What changed |
|---|---|
| `specs/inventory-tracking/spec.md` | All requirements removed; replaced with placeholder comment listing future plug-in points |
| `specs/admin-panel/spec.md` | Inventory page requirement removed; replaced with a deferred comment |
| `tasks.md` | Section 3 (5 inventory data layer tasks) removed; tasks 10.4–10.5 removed; task 11.7 (out-of-stock test) removed; task 1.6 updated (dropped inventory.json); task 6.3 updated (removed out-of-stock state) |

---

### [5] Order Lifecycle, Accept/Decline, Payment Rules

**Prompt:** Order lifecycle should start only when admin accepts. Admin can decline. COD only for pickup. Home delivery requires UPI. Customer shares payment screenshot on WhatsApp; admin manually marks payment as complete.

**Decisions made:**
- `declined` and `cancelled` are separate statuses — declined = never accepted (admin rejected at pending stage); cancelled = accepted but subsequently stopped
- `pending` is now an admin-review gate; fulfilment only begins at `confirmed`
- Full terminal state set: `completed`, `cancelled`, `declined` — none can transition out
- Pending orders use Accept/Decline, not status update selector
- COD available for pickup only; UPI required for home delivery; switching delivery type auto-updates payment selector
- Payment screenshot flow is entirely off-system (WhatsApp); system only provides the instruction and the manual "Mark as Paid" action

**Files updated:**
| File | What changed |
|---|---|
| `specs/order-management/spec.md` | Status list expanded to 7: pending, confirmed, preparing, dispatched, completed, cancelled, declined — each defined; `pending` redefined as admin-review gate; accept (pending→confirmed) and decline (pending→declined) scenarios added; cancel restricted to confirmed/preparing/dispatched; all three terminal states block edits and transitions |
| `specs/checkout/spec.md` | Payment method requirement rewritten — COD pickup only, UPI home delivery only; delivery type switch auto-updates payment selector; confirmation page split: pickup gets WhatsApp note, home delivery gets UPI screenshot instruction with business number |
| `specs/admin-panel/spec.md` | New requirement: Accept + Decline actions shown only on pending orders; "Mark as Paid" updated to reference WhatsApp screenshot trigger |
| `tasks.md` | Updated tasks 1.4, 4.1, 4.3, 4.4, 4.6, 8.2–8.7, 9.3–9.5, 11.4–11.8; added task 4.7 (soft-delete endpoint); added task 11.9 (README) |
