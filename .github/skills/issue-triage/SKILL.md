# Issue Triage & Analysis

## Purpose
Given a work item number (WI, CS, or PRJ), perform a structured triage to understand the issue and produce an actionable resolution plan.

## Trigger
User provides a job number (e.g., `WI00878427`, `CS00034343`, `PRJ00049378`).

## Workflow

### Step 1 — Gather Context
Fetch all available information about the job in parallel:

1. **Get job details** using `get-job-details` with the provided job number.
   - Captures: title, description, status, product, module, criticality, related items, attached documents, conversation history (for incidents).
2. **Get workflows** using `get-job-workflows` with the job number.
   - Captures: workflow IDs, component names, constrained status, tags, release groups.
3. **Get tasks** using `get-job-tasks` with the job number.
   - Captures: task sequence, types, assigned staff/capabilities, statuses, durations, whether notes exist.

### Step 2 — Deep Dive
Based on Step 1 results, gather additional detail as needed:

- **Read attached documents**: For each attached document URL returned by `get-job-details`, use `read-file` to read its content (specs, screenshots, correspondence, HLDs, etc.).
- **Read task notes**: For any task where "Has Notes" is true, use `read-task-notes` with the task ID to understand prior analysis or developer notes.
- **Follow related items**: If the job references linked WIs, CSs, or PRJs, use `get-job-details` on the most relevant related items (limit to 3 unless the user asks for more) to understand the broader context.
- **Check incident conversations**: For CS (incident) jobs, review the conversation history included in job details for client-reported symptoms and reproduction steps.

### Step 3 — Triage Summary
Present a structured summary:

```
## Triage Summary

**Job:** {jobNumber} — {title}
**Status:** {status}
**Product/Module:** {product} / {module}
**Criticality:** {criticality} (if incident)
**Component:** {componentName} (from workflow)
**Tags:** {tags}

### Problem Statement
{One paragraph distilling the core issue from description, conversations, and documents.}

### Key Findings
- {Bullet list of important facts discovered during context gathering}
- {Include symptoms, reproduction steps, affected areas, client impact}

### Related Items
- {List of linked WIs/CSs/PRJs with their titles and statuses}
```

### Step 4 — Resolution Plan
Produce a concrete plan to resolve the issue:

```
## Resolution Plan

### Root Cause Hypothesis
{Best assessment of the likely root cause based on available evidence.}

### Proposed Approach
1. {Numbered steps to investigate/fix the issue}
2. {Be specific — name files, modules, areas to look at}
3. {Include validation/testing steps}

### Risks & Unknowns
- {Anything that needs clarification or could complicate the fix}

### Estimated Complexity
{Low / Medium / High — based on scope of changes and risk}
```

### Step 5 — Next Actions
Ask the user:
1. Does the resolution plan look correct, or should any hypothesis be adjusted?
2. Should any related items be investigated further?
3. Ready to proceed with implementation, or need more analysis?

## Decision Points

| Condition | Action |
|---|---|
| Job number starts with `CS` | Prioritize conversation history and client-reported symptoms |
| Job number starts with `WI` | Focus on description, HLD/spec docs, and task notes |
| Job number starts with `PRJ` | Summarize project scope and check child work items |
| Multiple workflows exist | Summarize each workflow's status and component |
| Attached HLD/SPE documents exist | Always read these — they contain design intent |
| Job is CR1 or CR2 criticality | Flag as **URGENT**; front-load the resolution plan |
| No description or documents | Flag as insufficient information; list what's needed |
| Related items reference other WIs/CSs | Follow up to 3 most relevant to check for duplicate/overlapping work |

## Quality Checks
- [ ] All attached spec/HLD documents have been read
- [ ] Task notes with prior analysis have been reviewed
- [ ] Related items have been checked for duplicate/overlapping work
- [ ] Resolution plan includes specific, actionable steps (not vague)
- [ ] Risks and unknowns are explicitly called out
