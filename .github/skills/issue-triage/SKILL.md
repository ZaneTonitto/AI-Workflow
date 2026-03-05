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
### Step 2a — Inspect Source Code (MANDATORY for ISS/FIX types)

When the issue references specific files, classes, methods, error messages, or stack traces, you **MUST** read the actual source code before proceeding to Step 3 or Step 4. Do NOT produce a resolution plan based solely on stack traces, class names, or inferences from related work items.

**How to inspect code:** Use the `github-code-navigation` skill. This skill uses `gh` CLI commands to directly access source files from GitHub. The default repository is `WiseTechGlobal/CargoWise` (branch `master`) unless the issue references a different repo.

**Procedure:**

1. **Identify target files from the stack trace.** Map assembly + namespace + class to a source file path. Use `mcp_github_search_code` if the exact path is unknown:
   ```
   mcp_github_search_code
     query: "class WorkflowExceptionReader repo:WiseTechGlobal/CargoWise"
   ```
   Note: Load the tool first with `tool_search_tool_regex` pattern `mcp_github_search_code` if not already loaded.

2. **Read the crash-site method.** Fetch the file directly:
   ```
   mcp_github_get_file_contents
     owner: "WiseTechGlobal"
     repo: "CargoWise"
     path: "{path from search result}"
     branch: "master"
   ```
   Read enough context (the full method + 10 lines above/below) to understand parameters, null checks, and control flow.

3. **Read the immediate caller** (one frame below in the stack). Understand how the crash-site method is invoked — what values are passed, whether null guards exist before the call.

4. **Search for related code** if needed — callers, tests, similar patterns:
   ```
   mcp_github_search_code
     query: "SetCauseAndResolution repo:WiseTechGlobal/CargoWise"
   ```

5. **Build permalinks** using a commit SHA for the resolution plan. Get the latest commit for the file:
   ```
   mcp_github_list_commits
     owner: "WiseTechGlobal"
     repo: "CargoWise"
     path: "{path}"
     per_page: 1
   ```
   Then construct: `https://github.com/{owner}/{repo}/blob/{sha}/{path}#L{line}`

**What to extract from code inspection:**
- Exact line number of the failure (not inferred — read from actual source)
- Parameters and their types at the crash site
- Whether null guards or validation already exist
- The lookup/resolution logic that produces the null value
- Existing test class names and locations for the affected code
- Commit SHA for stable permalink generation

**If source code cannot be accessed** (auth failure, repo not found, etc.), explicitly flag this in the Risks & Unknowns section of the resolution plan and note that all line numbers and code behavior are inferred rather than verified. Do NOT silently skip this step.

### Step 2b — Knowledge & Duplicate Search
Use the `documentation` MCP server to cross-reference the issue against existing knowledge. Run these searches based on what you learned in Steps 1–2:

1. **Check for duplicate/related work items** — Search ediprod workitems for the same error message, affected component, or symptoms.
   ```
   search-knowledge-digested
     sources: ["ediprod-workitems"]
     query: "{error message or core symptom from the issue}"
     isActive: true   (check open items first, then false for closed/resolved)
   ```
   If duplicates or closely related items are found, note them in the triage summary and consider whether the existing fix applies.

2. **Check for related incidents** — Search ediprod incidents for the same symptoms reported by clients.
   ```
   search-knowledge-digested
     sources: ["ediprod-incidents"]
     query: "{client-facing symptom or error}"
     isActive: true
   ```
   Look for patterns — multiple clients hitting the same issue suggests higher urgency.

3. **Search domain documentation** — Search developer and user docs for the affected module/component to understand intended behavior and design context.
   ```
   search-knowledge-digested
     sources: ["developer-docs", "user-docs"]
     query: "{component name, feature area, or module}"
   ```
   This helps distinguish "bug" from "working as designed" and informs the fix approach.

4. **Broad exploration (when needed)** — If the digested search is too narrow, use `search-knowledge-many-results` to scan more broadly:
   ```
   search-knowledge-many-results
     sources: ["ediprod-workitems", "ediprod-incidents"]
     query: "{broader terms — class name, module, area}"
     years: 2
   ```
   Skim the results to spot recurring patterns or prior fixes in the same area.

**When to search what:**

| Situation | Tool | Sources |
|---|---|---|
| Exact error message / exception type | `search-knowledge-digested` | `ediprod-workitems`, `ediprod-incidents` |
| Understanding a module's intended behavior | `search-knowledge-digested` | `developer-docs`, `user-docs` |
| Checking if this is a known/recurring issue | `search-knowledge-many-results` | `ediprod-workitems`, `ediprod-incidents` |
| Finding prior fixes in the same code area | `search-knowledge-many-results` | `ediprod-workitems` |
| Unclear domain terminology or business rules | `search-knowledge-digested` | `user-docs`, `developer-docs` |

**Guidelines:**
- **Rephrase queries** using domain vocabulary — don't just paste the raw error message. Add component names, module codes, and synonyms.
- **Use `requiredWords`** to force critical terms (e.g., a specific class name or error code) when results are noisy.
- **Search active then closed** — check `isActive: true` first for open duplicates, then `isActive: false` to find prior resolutions.
- **Incorporate findings** into the triage summary and resolution plan — note duplicate WIs, reference prior fixes, and cite relevant documentation.

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

### Step 4 — Resolution Plan (Issue-Fixer Handoff)
Produce a resolution plan structured as a **handoff document for an issue-fixer agent**. The fixer agent will receive ONLY this document — it will not re-triage, so every piece of context it needs must be included.

**Output the plan in two places:**

1. **Markdown file** — Write to `{jobNumber}-resolution-plan.md` in the repo root.
2. **Work item description** — **Append** to the work item using `update-job-details` with the plan converted to HTML (see HTML conversion rules below).

**CRITICAL — Preserving existing description content:**

The work item description may already contain notes from human developers. These notes typically start with a staff code and timestamp, e.g.:

```
AQN 03-Mar-26 09:23 GMT+11:00:
```

You MUST preserve all existing description content. To do this:
1. Read the current description from the `get-job-details` response.
2. Prepend the existing content, then add a blank line separator.
3. Add your own **bolded timestamped note** using this format:
   ```
   **triage-agent {DD-MMM-YY HH:MM} GMT+{offset}:**
   ```
   Use the current UTC time converted to the local timezone (use `+11:00` as default if unknown). Example:
   ```
   **triage-agent 06-Mar-26 14:30 GMT+11:00:**
   ```
4. Follow your timestamp with the resolution plan HTML.

The final description sent to `update-job-details` should look like:
```
{existing description content}

**triage-agent {timestamp}:**
{resolution plan HTML}
```

The plan MUST follow this structure:

```markdown
# {jobNumber} — Resolution Plan

## Summary

**Job:** {jobNumber}
**Title:** {title}
**Status:** {status}
**Product/Module:** {product} / {module}
**Area:** {area}
**Component:** {componentName} ({componentCode})
**Type:** {type}
**Complexity:** {Low / Medium / High}

## Problem Statement

{Clear, precise description of the defect or issue. Include:
- What is happening (symptoms, error messages, exception types)
- Where it is happening (call chain or flow leading to the failure)
- When/how it is triggered (the conditions that reproduce the issue)
- Production incident IDs if applicable}

## Root Cause Hypothesis

{Best assessment of why the issue occurs. Be specific:
- Name the exact code path and failure point
- Explain the mechanism (e.g., null deref, race condition, missing validation)
- Reference the exact source file and line number with a link if available}

## Fix Specification

Describe the EXACT changes the fixer agent should make. Be as specific as possible — the fixer should not need to re-investigate the issue.

### Files to Modify

- **{path/to/file.cs}** — {Concise description of what to change and why}
- ...

### Implementation Steps

{Numbered, ordered steps describing the code changes to make. Each step should be concrete and actionable:}

1. {e.g., "Add a null guard for `exceptionType` before accessing `.PK` at line ~141 of WorkflowExceptionReader.cs"}
2. {e.g., "When null, skip setting cause/resolution and log a warning including the unrecognized type code"}
3. {Be specific about behavior: what to do, what NOT to do, what to return/skip}

### Code Changes

{Include this section when source code was inspected in Step 2a AND the fix is Low or Medium complexity.
For each change, show a before/after code snippet so the fixer can see exactly what to modify.
Omit this section for High complexity fixes where multiple interrelated changes make snippets misleading.}

**{path/to/file.cs}** — Line {N}

Before:
```csharp
{Exact code as it currently exists in the source, with enough surrounding context (3-5 lines) to locate it unambiguously}
```

After:
```csharp
{The modified code showing the fix applied, with the same surrounding context}
```

{Repeat for each file/location that needs changes.}

### Testing Requirements

1. {Specific test to write — e.g., "Unit test: pass null exceptionType to SetCauseAndResolution, verify no exception thrown and fields are left unset"}
2. {Regression test — e.g., "Unit test: pass valid exceptionType, verify cause and resolution are set correctly"}
3. {Name existing test classes/projects to add tests to, if identifiable}

## Risks & Unknowns

- **{Risk description}** — {How to mitigate or what to watch for}

## Key Files

- **{path/to/file.cs}** — {Why this file matters — crash site, caller, related logic, test location}
```

**HTML Conversion Rules for Work Item Description:**

The work item description only supports HTML formatting — not Markdown. When writing the plan to the work item via `update-job-details`, convert the Markdown plan to HTML using these rules:

| Markdown | HTML |
|---|---|
| `# Heading` | `<h1>Heading</h1>` |
| `## Heading` | `<h2>Heading</h2>` |
| `### Heading` | `<h3>Heading</h3>` |
| `**bold**` | `<b>bold</b>` |
| `*italic*` | `<i>italic</i>` |
| `` `code` `` | `<code>code</code>` |
| `- list item` | `<ul><li>list item</li></ul>` |
| `1. list item` | `<ol><li>list item</li></ol>` |
| `[text](url)` | `<a href="url">text</a>` |
| Code block (```) | `<pre><code>…</code></pre>` |
| Paragraph break | `<br/>` (preferred) |
| `- [ ] item` | `<ul><li>☐ item</li></ul>` |

- Do NOT use Markdown syntax in the HTML version — it will render as raw text.
- Keep the same structure and content as the Markdown file; only the formatting changes.
- **Keep the HTML compact** — do NOT wrap every sentence in `<p>` tags or add excessive `<br/>` between elements. Use `<br/>` only for intentional line breaks within a section. Headings and lists already create their own spacing. The goal is a dense, readable format — not a spaced-out document.
- **Do NOT use HTML tables** — they render poorly in work item descriptions (no borders, just floating text). Use bold labels with `<b>` and `<br/>` line breaks instead. For example, instead of a table use: `<b>File:</b> path/to/file.cs — description<br/>`
- **Skip redundant metadata** — do NOT include the `# {jobNumber} — Resolution Plan` heading, the `## Summary` section, or fields like Job, Title, Status, Product/Module, Area, Component, Type in the WI description — these are already visible in the work item's own panels. Start the WI description content directly from the `## Problem Statement` section. The full Summary section is only needed in the markdown file (which serves as a standalone handoff document for the fixer agent).

**Guidelines for writing the plan:**
- **Be exhaustive on context** — the fixer agent has no prior knowledge of the issue.
- **Be prescriptive on implementation** — tell the fixer exactly what to change, not just where to look.
- **Include before/after code snippets** — when source code was read in Step 2a and the fix is Low or Medium complexity, include concrete before/after code blocks in the "Code Changes" section. This removes ambiguity and lets the fixer apply changes directly. Use the same snippets in both the markdown file and the HTML work item description (using `<pre><code>` for HTML).
- **Separate investigation from action** — all investigation should be done during triage. The plan should contain conclusions, not "investigate X" steps.
- **Include source links** — GitHub permalink URLs to the exact lines where changes are needed.
- **Name specific files and line numbers** — vague references like "the reader class" are not acceptable.
- **Keep workflow/process steps out** — PR submission, WI updates, patchbacks are NOT part of the fixer's scope. Only include code changes and testing.

### Step 5 — Next Actions
Ask the user:
1. Does the resolution plan look correct, or should any hypothesis be adjusted?
2. Should any related items be investigated further?
3. Ready to hand off to the issue-fixer agent?

### Step 6 — Retrospective
After the task is complete (plan delivered and user is satisfied), reflect on your own performance. Present a brief retrospective to the user:

```
## Retrospective

### What went well
- {Things that worked effectively during this triage — e.g., "Knowledge search found a prior fix in WI00912345 that directly informed the root cause hypothesis"}

### What was difficult or slow
- {Bottlenecks, dead ends, or areas where you struggled — e.g., "Took multiple search queries to locate the relevant source file because the stack trace referenced an intermediate assembly"}

### What information was missing
- {Data you needed but couldn't find — e.g., "No developer docs existed for the WorkflowException pipeline, had to infer behavior from code alone"}

### Confidence assessment
- **Root cause confidence:** {High / Medium / Low} — {why}
- **Fix specification confidence:** {High / Medium / Low} — {why}

### Suggested improvements to agent design
{Only suggest changes when you genuinely identified a gap or friction point. Do NOT suggest changes for the sake of it. Each suggestion should reference a specific moment during this triage that motivated it.}

- **Skill/workflow change:** {e.g., "Step 2b should search closed workitems by default — I had to manually re-run with isActive: false to find the prior fix"}
- **New tool or data source:** {e.g., "A tool to resolve assembly names to source repo paths would have saved 3 search iterations"}
- **Prompt/template improvement:** {e.g., "The resolution plan template should include a 'Prior Art' section for linking related resolved WIs"}
- **Decision point gap:** {e.g., "No decision point exists for when the stack trace references third-party code vs internal code"}
```

**Retrospective guidelines:**
- Be honest and specific — reference actual moments from this triage session, not hypotheticals.
- Only suggest design changes that would have materially helped *this* triage. Do not suggest speculative improvements.
- Keep it concise — this is a reflection, not a redesign document.
- The user will decide whether to act on suggestions. Do not self-modify.

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
| Stack trace or file references present (ISS/FIX) | **MUST** complete Step 2a (source code inspection) before writing resolution plan |
| Source code inspection fails or is unavailable | Flag in Risks & Unknowns; mark all line numbers as "inferred" |

## Quality Checks
- [ ] All attached spec/HLD documents have been read
- [ ] Task notes with prior analysis have been reviewed
- [ ] Related items have been checked for duplicate/overlapping work
- [ ] Knowledge search performed — checked for duplicate WIs, related incidents, and domain docs
- [ ] Prior fixes in the same code area have been reviewed (if any exist)
- [ ] Resolution plan saved to `{jobNumber}-resolution-plan.md`
- [ ] Resolution plan written to work item description via `update-job-details` as HTML
- [ ] Plan includes specific file paths and line numbers (not vague references)
- [ ] Implementation steps are prescriptive (tell the fixer what to change, not what to investigate)
- [ ] Testing requirements name specific tests to write
- [ ] No workflow/process steps leaked into the plan (PR submission, WI updates, patchbacks)
- [ ] Risks and unknowns are explicitly called out
- [ ] A fixer agent reading ONLY this plan could implement the fix without re-triaging
