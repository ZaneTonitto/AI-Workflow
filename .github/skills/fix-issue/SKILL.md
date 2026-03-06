# Fix Issue

## Purpose
Given a work item number that has been triaged by the issue-triager agent, read the resolution plan from the work item description and implement the fix — production code changes, unit tests, and a pull request — in the locally cloned CargoWise repository.

## Trigger
User provides a job number (e.g., `WI00878427`) that already has a resolution plan in its work item description.

## Prerequisites
- The work item has been triaged and has a resolution plan written by the issue-triager agent in its description.
- The CargoWise repository is cloned locally at `C:\git\GitHub\WiseTechGlobal\CargoWise`.
- The `$env:STAFF_CODE` environment variable is set.

## Workflow

### Step 1 — Read the Resolution Plan
Retrieve the resolution plan that serves as the handoff document from the issue-triager agent.

1. **Get job details** using `get-job-details` with the provided job number.
   - Extract the resolution plan from the work item description. It will be preceded by a `triage-agent` timestamp header.
   - Parse out: Problem Statement, Root Cause Hypothesis, Fix Specification (files to modify, implementation steps, code changes), Testing Requirements, Risks & Unknowns, Key Files.
2. **Check for a local markdown plan** — look for `{jobNumber}-resolution-plan.md` in the current workspace. If it exists, prefer it over the WI description version as it contains the full plan with metadata.
3. **Validate completeness** — the plan MUST contain at minimum:
   - Specific files to modify with paths
   - Implementation steps
   - Testing requirements
   If any of these are missing, stop and ask the user to re-triage or provide the missing information.

### Step 2 — Prepare the Local Repository

1. **Navigate to the CargoWise repo:**
   ```
   cd C:\git\GitHub\WiseTechGlobal\CargoWise
   ```
2. **Ensure the repo is on `master` and up to date:**
   ```
   git checkout master
   git pull origin master
   ```
3. **Read the staff code** from the environment:
   ```
   $env:STAFF_CODE
   ```
4. **Create a feature branch** using the naming convention:
   ```
   git checkout -b {STAFF_CODE}/{jobNumber}/{wi-summary-kebab-case}
   ```
   Where `{wi-summary-kebab-case}` is the work item title converted to lowercase kebab-case (e.g., `null-reference-in-workflow-exception-reader`). Truncate to 60 characters max if needed.

### Step 3 — Verify and Understand Target Files

Before making ANY code changes, read and understand every file listed in the resolution plan.

1. **Read each target file** from the local repo using the file path from the plan. Read the full file or at minimum the class/method being modified plus 30 lines of surrounding context.
2. **Verify the plan's accuracy** — confirm that:
   - The file exists at the specified path
   - The line numbers match (they may have shifted since the plan was written)
   - The "before" code snippets in the plan match what's actually in the file
   - The class/method signatures match
3. **If discrepancies are found:**
   - Minor line number shifts: adjust and proceed.
   - Structural changes (method renamed, file moved, code refactored): stop and report to the user that the plan may be outdated.
4. **Study the surrounding code patterns** — before writing any code, understand:
   - Naming conventions used in this file and namespace (PascalCase, prefixes, suffixes)
   - Error handling patterns (how are exceptions thrown, caught, logged?)
   - Null handling patterns (null guards, null coalescing, early returns)
   - Dependency injection patterns (constructor injection, service locator)
   - Access modifiers used on similar methods
   - Documentation/comment style
   - How similar problems are solved elsewhere in the same file or namespace

### Step 4 — Implement the Fix

Apply the code changes specified in the resolution plan. Follow these principles strictly:

**Design Principles:**
- **SOLID principles** — Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion. Every change should respect these.
- **Match existing patterns** — your code must look like it was written by the same developer who wrote the surrounding code. If the file uses explicit null checks, don't introduce null-conditional operators. If it uses guard clauses, don't use nested if-else.
- **Minimal change surface** — change only what the plan specifies. Do not refactor adjacent code, rename variables for "clarity", or add unrelated improvements.
- **No side effects** — ensure your fix doesn't alter behavior for any case other than the one being fixed.

**Implementation procedure:**

1. **Apply changes file by file**, following the plan's Implementation Steps in order.
2. For each file:
   a. Read the file to get the current state.
   b. Apply the change precisely as specified, adjusting for any line number shifts.
   c. Verify the edit was applied correctly by re-reading the modified section.
3. **If the plan includes before/after code snippets**, use them as the primary guide. Match them exactly, adjusting only for context shifts.
4. **If the plan describes changes in prose only** (no before/after snippets), use the `github-code-navigation` skill to read additional context, then write the code yourself following the surrounding patterns.
5. **When writing new code:**
   - Use the same indentation style (tabs vs spaces, width) as the file.
   - Follow the same brace placement style.
   - Use the same `using`/`import` conventions.
   - Match the existing logging pattern and log levels.
   - Use the same exception types as similar code in the same namespace.

**When in doubt about a pattern**, use the `github-code-navigation` skill to search for similar code in the repo:
```
mcp_github_search_code
  query: "{pattern or method name} repo:WiseTechGlobal/CargoWise"
```

### Step 5 — Write Unit Tests

Every fix must have corresponding unit tests. Follow CargoWise conventions strictly.

**Test discovery — find existing tests first:**

1. **Find the test project** for the module being changed. CargoWise test projects typically follow the pattern `{ProjectName}.Tests` or `{ProjectName}.UnitTests`. Search for existing tests:
   ```
   mcp_github_search_code
     query: "class {ClassName}Tests repo:WiseTechGlobal/CargoWise"
   ```
   or search the local filesystem:
   ```
   Get-ChildItem -Path "C:\git\GitHub\WiseTechGlobal\CargoWise" -Recurse -Filter "*{ClassName}*Tests*" -File
   ```
2. **Read existing test files** for the class being modified. Understand:
   - Test naming convention (e.g., `MethodName_Condition_ExpectedResult`, `Should_DoSomething_When_Condition`)
   - Test framework used (NUnit, xUnit, MSTest)
   - Mocking framework used (Moq, NSubstitute, FakeItEasy)
   - Setup/teardown patterns (constructors, `[SetUp]`, `[OneTimeSetUp]`)
   - Assertion style (Assert.That, Should, FluentAssertions)
   - How dependencies are mocked or stubbed
   - Test data patterns (builders, factories, inline)

**Write the tests:**

1. **Follow the Testing Requirements** from the resolution plan — these specify which test scenarios to cover.
2. **Add tests to the existing test class** if one exists for the target class. Create a new test class only if none exists.
3. **Name tests using the exact same convention** as existing tests in the same file/project. Do not introduce a new naming style.
4. **For each test:**
   - Arrange: Set up the scenario that triggers the fixed behavior
   - Act: Execute the method under test
   - Assert: Verify the expected outcome
5. **Include both positive and negative tests:**
   - Test that the fix resolves the original issue (the case that was broken)
   - Test that existing behavior is preserved (regression test — the case that was already working)
6. **Build the test project** to verify compilation:
   ```
   dotnet build "{path-to-test-project.csproj}"
   ```
7. **Run the new tests** to verify they pass:
   ```
   dotnet test "{path-to-test-project.csproj}" --filter "{TestClassName}"
   ```
   If tests fail, diagnose and fix. Do not proceed with a failing test.

### Step 6 — Validate the Changes

Before committing, validate the implementation:

1. **Build the production project** to ensure no compilation errors:
   ```
   dotnet build "{path-to-production-project.csproj}"
   ```
2. **Run the relevant test suite** (not just new tests — run the full test class/project):
   ```
   dotnet test "{path-to-test-project.csproj}"
   ```
3. **Review the diff** to ensure only intended changes are included:
   ```
   git diff
   ```
   Check for:
   - No unintended whitespace changes
   - No unrelated files modified
   - No debug code left in (console.log, System.Diagnostics.Debug, commented-out code)
   - No TODO comments left by you
4. **If build or tests fail**, diagnose the issue, fix it, and re-validate. Do not commit broken code.

### Step 7 — Commit and Create Pull Request

1. **Stage the changes:**
   ```
   git add -A
   ```
2. **Commit with a clear message:**
   ```
   git commit -m "{jobNumber}: {concise summary of the fix}"
   ```
   Example: `git commit -m "WI01038212: Add null guard for exceptionType in WorkflowExceptionReader"`
3. **Push the branch:**
   ```
   git push -u origin {branch-name}
   ```
4. **Create the pull request** using the GitHub MCP tool:
   ```
   mcp_github_create_pull_request
     owner: "WiseTechGlobal"
     repo: "CargoWise"
     title: "{STAFF_CODE}/{jobNumber}/{WI Summary}"
     body: "{PR description — see template below}"
     head: "{branch-name}"
     base: "master"
   ```

**PR title format:** `{STAFF_CODE}/{jobNumber}/{WI Summary}`
- Example: `AQN/WI01038212/Null reference in WorkflowExceptionReader`

**PR body template:**
```markdown
## {jobNumber} — {WI Title}

### Problem
{One-sentence summary of the issue from the resolution plan's Problem Statement.}

### Fix
{Concise description of what was changed and why.}

### Changes
- `{path/to/file.cs}` — {what was changed}
- `{path/to/test.cs}` — {tests added}

### Testing
- {List of test scenarios added}
- All existing tests in `{TestClassName}` pass.

### Resolution Plan
See work item description for the full triage and resolution plan.
```

### Step 8 — Update the Work Item

After the PR is created:

1. **Add a note to the work item description** via `update-job-details`, preserving existing content and prepending a timestamped update:
   ```
   **fix-agent {DD-MMM-YY HH:MM} GMT+{offset}:**
   Implementation complete. PR: {PR URL}
   Branch: {branch-name}
   ```

### Step 9 — Retrospective

After the task is complete (PR created and work item updated), reflect on your performance. Present a brief retrospective to the user:

```
## Retrospective

### What went well
- {Things that worked effectively — e.g., "Resolution plan had exact before/after snippets that matched the current code, enabling direct application"}

### What was difficult or slow
- {Bottlenecks or friction — e.g., "Test project wouldn't build due to a missing NuGet dependency, required manual investigation"}

### What didn't match the plan
- {Discrepancies between the resolution plan and reality — e.g., "Line numbers had shifted by 15 due to a recent commit; had to re-locate the target code"}

### Confidence assessment
- **Fix correctness:** {High / Medium / Low} — {why}
- **Test coverage:** {High / Medium / Low} — {why}
- **Risk of regression:** {High / Medium / Low} — {why}

### Suggested improvements to agent design
{Only suggest changes when you genuinely identified a gap or friction point. Do NOT suggest changes for the sake of it. Each suggestion should reference a specific moment during this fix that motivated it.}

- **Skill/workflow change:** {e.g., "Step 3 should include checking recent commits to the target files to detect plan staleness before starting work"}
- **Plan quality issue:** {e.g., "The resolution plan didn't specify which test project to add tests to — had to search for it manually"}
- **Tool gap:** {e.g., "No tool to run a single test by name — had to filter by class which ran unrelated tests too"}
- **Convention gap:** {e.g., "No reference doc for CargoWise test naming conventions — had to infer from existing tests"}
```

**Retrospective guidelines:**
- Be honest and specific — reference actual moments from this fix session, not hypotheticals.
- Highlight plan quality issues — these feed back to the triage agent's improvement.
- Only suggest design changes that would have materially helped *this* fix.
- Keep it concise.

## Decision Points

| Condition | Action |
|---|---|
| Resolution plan is missing or empty | Stop. Ask user to run issue-triager first. |
| Plan lacks specific file paths | Stop. Ask user to re-triage with source code inspection. |
| Plan's before/after snippets don't match current code | Report discrepancy; attempt to adapt if minor. Stop if structural. |
| Target file has been heavily modified since the plan | Stop. Report that the plan may be stale and ask for re-triage. |
| Build fails after applying the fix | Diagnose and attempt to fix compilation errors. If the root cause is unclear, report to user. |
| Tests fail after applying the fix | Diagnose test failure. If the test logic is wrong, fix it. If the production code is wrong, revisit the fix. |
| Plan specifies a High-complexity fix without before/after snippets | Proceed carefully; read extra context; confirm approach with user before writing code. |
| No existing test class found for the target class | Create a new test class following the naming and structure conventions of the nearest test project. |
| $env:STAFF_CODE is not set | Stop. Ask the user to set the environment variable. |

## Quality Checks
- [ ] Resolution plan was read and validated against current codebase
- [ ] All target files were read and understood before modification
- [ ] Code changes match existing design patterns and naming conventions
- [ ] SOLID principles were respected in all new/modified code
- [ ] No unrelated changes were made (minimal change surface)
- [ ] Unit tests cover both the fix scenario and regression scenarios
- [ ] Tests follow CargoWise naming conventions and framework usage
- [ ] Production project builds successfully
- [ ] Test project builds and all tests pass
- [ ] Git diff contains only intended changes
- [ ] Commit message references the job number
- [ ] PR created with correct title format: `{STAFF_CODE}/{jobNumber}/{WI Summary}`
- [ ] Work item updated with fix-agent note and PR link
- [ ] Retrospective completed
