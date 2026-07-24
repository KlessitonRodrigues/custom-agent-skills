# Bootstrap: [Technology / Stack Name]

Step-by-step guide to create a new [describe what this bootstraps] from zero using this project as template.

---

## Prerequisites

- [Runtime/tool 1 with version]
- [Runtime/tool 2]
- [CLI tool] configured (`[config command]`)
- [Global package] (`npm install -g [package]`)
- [Cloud/service account] with permissions to [list required permissions]

---

## 1. Project Initialization

```bash
mkdir my-service && cd my-service
[package manager] init -y
```

---

## 2. package.json

Replace the generated `package.json`:

```json
{
  "name": "@services/my-service",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "[local dev command]",
    "build": "[build command]",
    "test": "[test command]",
    "format": "[format command]"
  },
  "devDependencies": {
    "[dev-dep]": "^[version]"
  },
  "dependencies": {
    "[dep]": "^[version]"
  }
}
```

```bash
[package manager] install
```

---

## 3. [Config File 1 — e.g. TypeScript, Linter, Bundler]

**`[filename]`**

```[language]
[config content]
```

---

## 4. [Config File 2 — e.g. Formatter, Test Runner]

**`[filename]`**

```[language]
[config content]
```

---

## 5. Environment Variables

**`.env`** — never commit this file

```env
STACK_NAME="my-service-dev"
[VAR_NAME]="[value or placeholder]"
```

Add `.env` to `.gitignore`.

---

## 6. Directory Structure

```
src/
├── [entry point file]              # [description]
├── config/
│   └── [config loader]             # [description]
├── constants/
│   └── [constants files]           # [description]
├── utils/
│   └── [utility files]             # [description]
└── lib/
    └── [domain]/
        └── [domain files]          # [description]
```

---

## 7. Source Files

### `src/[file].ts`

```typescript
[file content]
```

<!-- Repeat section per shared/infrastructure file -->

---

## 8. Entity Files (repeat per domain entity)

Replace `[entity]` / `[Entity]` with your entity name throughout.

### `src/lib/[entity]/[entity].[layer].ts`

```typescript
[entity file content]
```

<!-- Repeat per layer: dto, model, service, construct/handler, spec -->

---

## 9. Stack / Entry Point

### `src/[stack or main entry].ts`

```typescript
[stack wiring content]
```

---

## 10. Local Dev Server (if applicable)

### `src/[dev server file].ts`

```typescript
[dev server content]
```

---

## 11. Integration Tests

### `src/lib/[entity]/[entity].spec.ts`

```typescript
[test content]
```

> Tests run against [describe target — e.g., deployed URL, local server]. Set [required env var] in `.env` first.

---

## 12. Deploy / Run

```bash
# [First-time setup step if any]
[bootstrap command]

# Verify the project compiles / synths
[verify command]

# Deploy / start
[deploy or run command]
```

---

## 13. Development Workflow

```bash
# Local dev
[dev command]

# Run tests
[test command]

# Coverage report
[coverage command]

# Format source files
[format command]
```

---

## 14. Conventions

| Rule | Detail |
|------|--------|
| [Convention topic] | [Convention detail] |
| [Convention topic] | [Convention detail] |
| [Convention topic] | [Convention detail] |

---

## 15. Adding a New Entity

1. Create `src/lib/[entity]/` with these files: `[list files]`
2. Add resource/config entries for the entity in `[constants file]`
3. Add env vars to `[env file]` and `.env`
4. [Additional wiring steps]
5. Add routes to `[dev server file]` for local dev

---

<!--
HOW TO CREATE A NEW BOOTSTRAP FILE
===================================

1. Copy this file to bootstrap/<technology>.md
   - Name it after the primary technology + purpose (e.g. aws.node.lambda.md, gcp.python.cloudrun.md)

2. Fill in every section:
   - Replace all [placeholders] with real content
   - Remove sections that do not apply to this stack
   - Add stack-specific sections as needed (e.g. "Docker setup", "Database migrations")

3. Keep it runnable end-to-end:
   - Every code block must be complete and copy-pasteable
   - Entity file sections must be a self-contained working example (use a "task" entity as the demo)
   - The deploy section must result in a working service

4. Pair it with an instructions file:
   - Create a matching instructions/<technology>.md (see instructions/template.md)
   - The instructions file contains the coding rules an AI uses when generating code for this stack
   - The bootstrap file is for humans setting up a new service from scratch

5. Reference the existing example:
   - bootstrap/aws.node.lambda.md is the canonical reference implementation
-->
