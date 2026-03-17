# Fix Issue

## Purpose
Given a work item number with an existing resolution plan, implement the fix — production code changes and unit tests — in the locally cloned CargoWise repository. Changes are left uncommitted for the developer to review.

## Trigger
User provides a job number (e.g., `WI00878427`) and a resolution plan as a local markdown file.

## Prerequisites
- A resolution plan exists for the work item (usually as a local markdown file).
- The CargoWise repository is cloned locally at `C:\git\GitHub\WiseTechGlobal\CargoWise`.
- The `$env:STAFF_CODE` environment variable is set.

## Workflow

### Step 1 — Read the Resolution Plan
Retrieve the resolution plan for the work item.

1. **Locate the resolution plan** — look for `{jobNumber}-resolution-plan.md` in the current workspace.
   - Parse out: Problem Statement, Root Cause Hypothesis, Fix Specification (files to modify, implementation steps, code changes), Testing Requirements, Risks & Unknowns, Key Files.
   - If the local file is not found, use `get-job-details` with the job number and check if a plan exists in the work item description as a fallback.
2. **Validate completeness** — the plan MUST contain at minimum:
   - Specific files to modify with paths
   - Implementation steps
   - Testing requirements
   If any of these are missing, stop and ask the user to re-investigate or provide the missing information.

### Step 2 — Prepare the Local Repository

1. **Navigate to the CargoWise repo:**
   ```
   cd C:\git\GitHub\WiseTechGlobal\CargoWise
   ```
2. **Check for uncommitted changes:**
   ```
   git status --porcelain
   ```
   If there are uncommitted changes, stop and ask the user to stash or commit them first. Do NOT create a branch with someone else's uncommitted work mixed in.
3. **Check the current branch:**
   ```
   git branch --show-current
   ```
4. **If already on a branch whose name contains the job number** (e.g., `ZTT/WI01038212/null-ref-workflow-exception` from a prior run), stay on it and continue — skip to Step 3.
5. **Otherwise (on `master`, `main`, or any unrelated branch)**, you MUST create a new feature branch before making any changes. Read the staff code from `$env:STAFF_CODE` and create the branch:
   ```
   git checkout -b {STAFF_CODE}/{jobNumber}/{wi-summary-kebab-case}
   ```
   Where `{wi-summary-kebab-case}` is the work item title converted to lowercase kebab-case (e.g., `null-reference-in-workflow-exception-reader`). Truncate to 60 characters max if needed.

   **CRITICAL:** Never make code changes directly on `master` or `main`. Always branch first. If branch creation fails, stop and report to the user.

### Step 3 — Locate and Understand Target Files

Before making ANY code changes, use the `github-code-navigation` skill to locate and read every file listed in the resolution plan. GitHub code search is faster than local filesystem search — use it as the primary file discovery mechanism.

1. **Locate each target file via GitHub code search** — use the `github-code-navigation` skill to find the file:
   ```
   mcp_github_search_code
     query: "class {ClassName} repo:WiseTechGlobal/CargoWise"
   ```
   or if you have a path from the plan:
   ```
   mcp_github_get_file_contents
     owner: "WiseTechGlobal"
     repo: "CargoWise"
     path: "{path-from-plan}"
   ```
   This lets you confirm the file exists and read its contents without slow local filesystem operations.
2. **Read the file contents via GitHub** to understand the current state. Read the full file or at minimum the class/method being modified plus 30 lines of surrounding context.
3. **Verify the plan's accuracy** — confirm that:
   - The file exists at the specified path
   - The line numbers match (they may have shifted since the plan was written)
   - The "before" code snippets in the plan match what's actually in the file
   - The class/method signatures match
4. **Check for recent changes** to each target file since the plan was written. If the plan contains a timestamp, use it; otherwise assume it was written today:
   ```
   mcp_github_list_commits
     owner: "WiseTechGlobal"
     repo: "CargoWise"
     path: "{file-path}"
     per_page: 5
   ```
   If there are recent commits touching the target files, use `mcp_github_get_commit` to read those diffs and understand whether the plan's assumptions still hold.
5. **If discrepancies are found:**
   - Minor line number shifts: adjust and proceed.
   - Structural changes (method renamed, file moved, code refactored): stop and report to the user that the plan may be outdated.
6. **Only after confirming locations and plan accuracy**, read the files from the local repo for editing. The local read is for the edit step — discovery and verification should happen via GitHub.
5. **Study the surrounding code patterns** — before writing any code, understand:
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

**When encountering unfamiliar domain concepts**, use the `documentation` MCP server to look up business rules, module behavior, or terminology before making assumptions:
```
search-knowledge-digested
  sources: ["developer-docs", "user-docs"]
  query: "{domain term, module name, or business concept}"
```
Use this whenever:
- The resolution plan references domain logic you don't fully understand
- You need to understand the intended behavior of a module before deciding how to guard against edge cases
- A method name, field name, or enum value has unclear business meaning
- You're unsure whether a behavior is "by design" or a bug

Do NOT guess at domain semantics — look them up.

**When creating new files** (new classes, interfaces, etc.):
- **Placement**: find an existing file in the same module/namespace and place the new file alongside it. Use `Get-ChildItem` or `github-code-navigation` to confirm the correct directory.
- **Namespace**: match the namespace of sibling files in the same directory.
- **Project file**: if the project uses explicit file includes (not default glob), add the new file to the `.csproj` `<Compile Include="..." />` items. Check the `.csproj` first — modern SDK-style projects auto-include and don't need this.
- **Naming**: follow the existing naming convention in the directory (e.g., `I{Name}.cs` for interfaces, `{Name}Base.cs` for abstract classes).
- **Boilerplate**: match the `using` block, license header (if any), and file structure of adjacent files.

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
6. **Build the test project** to verify compilation. Build only the specific `.csproj`, not the entire solution:
   ```
   dotnet build "{path-to-test-project.csproj}" --no-dependencies
   ```
   If the test project has dependencies that also need building, drop `--no-dependencies`. But never build the full `.sln` unless explicitly needed.
7. **Run the new tests immediately** to verify they pass. Do NOT skip this step — tests must be executed after being written to confirm they work:
   ```
   dotnet test "{path-to-test-project.csproj}" --filter "{TestClassName}" --no-build
   ```
   If tests fail, diagnose and fix. **Maximum 3 attempts** — if the tests still fail after 3 fix-and-retry cycles, stop and report the failure to the user with the error details. Do not loop indefinitely.

   **CRITICAL:** Do not proceed to the next step until you have actually run the tests and confirmed they pass. Writing tests without running them is incomplete work.

### Step 6 — Validate the Changes

Before finishing, validate the implementation:

1. **Build the production project** to ensure no compilation errors. Build only the specific `.csproj`:
   ```
   dotnet build "{path-to-production-project.csproj}"
   ```
2. **Run only the new tests** to confirm they pass. Filter by the specific test method names you wrote:
   ```
   dotnet test "{path-to-test-project.csproj}" --no-build --filter "FullyQualifiedName~{TestMethodName1}|FullyQualifiedName~{TestMethodName2}"
   ```
   If tests fail, diagnose and fix. **Maximum 3 attempts** — if they still fail after 3 fix-and-retry cycles, stop and report the failure to the user with the error details. Do not loop indefinitely.
3. **Review the diff** to ensure only intended changes are included:
   ```
   git diff
   ```
   Check for:
   - No unintended whitespace changes
   - No unrelated files modified
   - No debug code left in (console.log, System.Diagnostics.Debug, commented-out code)
   - No TODO comments left by you
4. **If the build fails**, diagnose the issue, fix it, and re-validate. **Maximum 3 attempts** — if it still fails after 3 cycles, stop and report to the user with the full error output. Do not loop indefinitely or leave broken code.

### Step 7 — Change Summary

Present a concise summary of what was done so the developer can quickly orient before reviewing:

```
## Change Summary

**Job:** {jobNumber} — {title}
**Branch:** {branch-name}

### Files Modified
- `{path/to/file.cs}` — {what was changed}
- `{path/to/file.cs}` — {what was changed}

### Files Created
- `{path/to/new-file.cs}` — {purpose}
(omit this section if no files were created)

### Tests Added
- `{TestClassName}.{TestMethodName}` — {what it tests}
- `{TestClassName}.{TestMethodName}` — {what it tests}

### Build & Test Status
- Production build: ✅ Pass
- Test build: ✅ Pass
- Test run: ✅ {N} passed, {N} failed
```

### Step 8 — Retrospective

After the implementation is complete (builds pass, tests pass, diff reviewed), reflect on your performance. Present a brief retrospective to the user:

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
| Resolution plan is missing or empty | Stop. Ask user to provide or generate a resolution plan first. |
| Plan lacks specific file paths | Stop. Ask user to re-investigate with source code inspection. |
| Plan's before/after snippets don't match current code | Report discrepancy; attempt to adapt if minor. Stop if structural. |
| Target file has been heavily modified since the plan | Stop. Report that the plan may be stale and ask for re-investigation. |
| Build fails after applying the fix | Diagnose and attempt to fix compilation errors. If the root cause is unclear, report to user. |
| Tests fail after applying the fix | Diagnose test failure. If the test logic is wrong, fix it. If the production code is wrong, revisit the fix. |
| Plan specifies a High-complexity fix without before/after snippets | Present a mini-proposal to the user BEFORE writing code (see below). |
| Working tree has uncommitted changes | Stop. Ask the user to stash or commit existing changes before proceeding. |
| Build or test failure persists after 3 fix attempts | Stop. Report the full error output to the user and ask for guidance. |

**High-complexity mini-proposal format** — when the plan is High complexity and lacks before/after snippets, present this to the user for approval before writing any code:
```
## Proposed Approach — {jobNumber}

### What I plan to change
- {File 1}: {specific change description}
- {File 2}: {specific change description}

### Patterns I'll follow
- {Pattern observed in surrounding code that I'll match}

### Open questions
- {Anything unclear in the plan that could affect the implementation}

### Risks
- {Anything that might go wrong with this approach}
```
Wait for user approval before proceeding to Step 4. |
| No existing test class found for the target class | Create a new test class following the naming and structure conventions of the nearest test project. |
| Already on a branch matching the job number | Stay on it and continue — do not create a new branch. |
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
- [ ] Changes are left uncommitted for developer review
- [ ] Change summary presented to user
- [ ] Retrospective completed
