# go-backend

Claude Code plugin for Go backend engineering.

It provides the `go-backend` skill, which guides Claude through small, repository-aware changes to Go HTTP APIs and backend services.

## What it does

- Inspects the repository before proposing changes.
- Detects greenfield vs. existing projects and classifies the task.
- Reuses existing routing, validation, persistence, response, test, and verification conventions.
- Applies a minimal Clean Architecture structure only when the project needs one.
- Follows test-driven development for new behavior, bug fixes, and refactors:
  1. Red: add a focused failing test.
  2. Green: implement the smallest fix.
  3. Refactor: clean up while tests stay green.
- Applies baseline security checks to every backend task.
- Adds deeper security review for authentication, authorization, tokens, secrets, outbound requests, and related sensitive code.
- Runs repository verification and reports failures honestly.
- Avoids speculative layers, dependencies, workers, storage, DDD, and unrelated refactors.

The skill is intended for Go server-side services. It does not apply to standalone Go CLIs, scripts, utilities, or non-Go code.

## Install

### From a marketplace

```text
/plugin install go-backend --marketplace backend-timedoor/go-backend
```

### Local development

From the plugin repository:

```bash
claude --plugin-dir /path/to/go-backend
```

Or install the plugin directory through Claude Code's plugin management UI/commands.

## Use

Start Claude Code in a Go backend repository, then describe the task normally:

```text
Add POST /api/v1/products. Inspect the existing conventions first. Include tests.
```

```text
Investigate why concurrent user updates affect the wrong record. Diagnose first; do not edit until I approve the fix.
```

```text
Review the authentication middleware for token, authorization, secret-handling, and error-response issues.
```

Claude loads the skill when the task is a Go backend task. The skill then:

1. Inspects the repository and existing conventions.
2. Identifies blocking decisions, defaults, and deferred scope.
3. Proposes before editing when the task requires approval.
4. Writes focused tests and the minimum implementation.
5. Runs applicable verification such as `gofmt`, `go test`, and repository-defined checks.
6. Reports changed files, tests, verification results, security checks, failures, and deferred decisions.

If auto-detection misses the task or you want to force-load the skills, input this in Claude Code:
```text
/go-backend:go-backend
```

## Scope rules

The plugin does not choose greenfield router, database, or linter tooling without approval. Existing repository conventions take precedence. Missing optional architecture is not added “for later.”

## Repository layout

```text
.claude-plugin/plugin.json  Plugin metadata
skills/go-backend/SKILL.md   Skill instructions
```

## License

No license is currently declared.
