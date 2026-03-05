# GitHub Code Navigation

## Purpose
Use the GitHub MCP server tools to browse, search, and read source code in GitHub repositories during investigation and development. This skill is a building block — other skills and agents should reference it whenever they need to inspect code.

## Prerequisites
- GitHub MCP server enabled in VS Code (provides `mcp_github_*` tools).
- Access to the target repository on GitHub.

## Tool Loading
GitHub MCP tools are deferred — you must load them before first use:
```
tool_search_tool_regex with pattern "mcp_github_{tool_name}"
```
Load only the tools you need for the current operation.

## Core Operations

### Read a file
```
mcp_github_get_file_contents
  owner: "{owner}"
  repo: "{repo}"
  path: "{path}"
  branch: "{branch_or_sha}"   (optional — defaults to repo default branch)
```
Returns the decoded file content directly — no base64 decoding needed. Use when you know the file path and want to read its contents. Specify a commit SHA for a pinned version, or a branch name for latest.

### Search code in a repository
```
mcp_github_search_code
  query: "{search_terms} repo:{owner}/{repo}"
```
Use to find files or code patterns across the repo. Useful for locating classes, method usages, error messages, or configuration. The query uses GitHub code search syntax — add qualifiers like `language:csharp`, `path:some/dir`, or `filename:SomeFile.cs` to narrow results.

**Examples:**
- Find a class definition: `"class WorkflowExceptionReader" repo:WiseTechGlobal/CargoWise`
- Find method usages: `"SetCauseAndResolution" repo:WiseTechGlobal/CargoWise`
- Narrow by language: `"SetCauseAndResolution" repo:WiseTechGlobal/CargoWise language:csharp`

### List directory contents
```
mcp_github_get_file_contents
  owner: "{owner}"
  repo: "{repo}"
  path: "{directory_path}"
```
When pointed at a directory, this returns the listing of files and subdirectories. Use to explore directory structure when you're unsure of exact file names.

### Get a specific commit
```
mcp_github_get_commit
  owner: "{owner}"
  repo: "{repo}"
  sha: "{commit_sha}"
```
Use to inspect what a specific commit changed — returns files modified, status, and patch diffs.

### View recent commits to a file
```
mcp_github_list_commits
  owner: "{owner}"
  repo: "{repo}"
  path: "{file_path}"
  per_page: 10
```
Use to understand recent changes to a file — helpful when investigating regressions or finding when a bug was introduced.

### Build a permalink to a specific line
```
https://github.com/{owner}/{repo}/blob/{sha}/{path}#L{line}
```
Use to generate stable source links for resolution plans. To get the commit SHA, use `mcp_github_list_commits` with the file path and take the first result's SHA. Always prefer commit SHA over branch name for permalinks.

## When to Use

| Situation | Tool |
|---|---|
| Error references a specific file and line | `get_file_contents` with the file path |
| Need to find where a class/method is defined | `search_code` with class/method name |
| Need to trace callers of a method | `search_code` for method name |
| Need to understand recent changes to a file | `list_commits` with file path |
| Building a resolution plan — need source links | `list_commits` for SHA + build permalink |
| Exploring unfamiliar area of codebase | `get_file_contents` on directory path |
| Investigating when a bug was introduced | `list_commits` + `get_commit` for diffs |

## Guidelines

- **Prefer commit SHAs over branch names** for stable references — branches move, SHAs don't.
- **Read the actual code** — don't guess at behavior based on file names alone.
- **Limit scope** — read the specific files/methods relevant to the issue, not entire directories.
- **Cache results mentally** — if you've read a file, don't re-fetch it unless you need a different section.
- **Use search before browsing** — `search_code` is faster than manually navigating directories.
- **Load tools first** — always use `tool_search_tool_regex` to load `mcp_github_*` tools before calling them. Once loaded, they stay available for the session.
