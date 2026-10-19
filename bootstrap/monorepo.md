# Bootstrap: Turbo Monorepo

Step-by-step guide to create a new Turbo monorepo from zero using this project as template.

---

## Prerequisites

- Node.js 22.x
- Yarn 1.22.x (`npm install -g yarn`)

---

## 1. Project Initialization

```bash
mkdir my-monorepo && cd my-monorepo
yarn init -y
```

---

## 2. Directory Structure

```
my-monorepo/
├── apps/          # Frontend / backend applications
├── packages/      # Shared libraries and configs
├── services/      # Backend services (e.g. Lambda APIs)
├── tests/         # Standalone test suites
├── package.json
├── turbo.json
└── .gitignore
```

```bash
mkdir apps packages services tests
```

---

## 3. Root `package.json`

Replace the generated `package.json`:

```json
{
  "name": "my-monorepo",
  "private": true,
  "version": "1.0.0",
  "scripts": {
    "dev": "turbo run dev",
    "build": "turbo run build",
    "lint": "turbo run lint",
    "test": "turbo run test"
  },
  "dependencies": {
    "turbo": "^2.10.3"
  },
  "packageManager": "yarn@1.22.22",
  "workspaces": ["apps/*", "packages/*", "services/*", "tests/*"]
}
```

```bash
yarn install
```

---

## 4. `turbo.json`

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["out/**", "dist/**", "cdk.out/**"],
      "cache": false
    },
    "dev": {
      "cache": true,
      "persistent": true
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "test": {
      "dependsOn": ["^build"]
    }
  }
}
```

---

## 5. `.gitignore`

```gitignore
# Folders
node_modules/
out/
cdk.out/
dist/
coverage/
_tests/
.expo/
.turbo/

# Files
*.log
*.env
*.apk
```

---

## 6. Adding a Workspace Package

Each workspace (`apps/*`, `packages/*`, `services/*`, `tests/*`) must have its own `package.json` with a `name` scoped to its workspace folder.

**Example: `packages/ui/package.json`**

```json
{
  "name": "@packages/ui",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "lint": "eslint src"
  }
}
```

**Example: `apps/web/package.json`**

```json
{
  "name": "@apps/web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "build": "vite build",
    "dev": "vite",
    "lint": "eslint src"
  },
  "dependencies": {
    "@packages/ui": "*"
  }
}
```

Use `"*"` as the version when referencing internal workspace packages.

---

## 7. Workspace Commands

```bash
# Run a script in a specific workspace
yarn workspace @apps/web dev

# Add a dependency to a specific workspace
yarn workspace @apps/web add react

# Add a dev dependency to a specific workspace
yarn workspace @services/my-service add -D typescript

# Add an internal package as a dependency
yarn workspace @apps/web add @packages/ui
```

---

## 8. Running Turbo Tasks

```bash
# Run dev servers for all workspaces (persistent)
yarn dev

# Build all workspaces in dependency order
yarn build

# Lint all workspaces
yarn lint

# Run all tests (after build)
yarn test

# Run a task only for a specific workspace
yarn turbo run build --filter=@apps/web

# Run a task for a workspace and its dependencies
yarn turbo run build --filter=@apps/web...
```

---

## 9. Conventions

| Rule            | Detail                                                                                     |
| --------------- | ------------------------------------------------------------------------------------------ |
| Package names   | Scoped to workspace folder: `@apps/`, `@packages/`, `@services/`, `@tests/`                |
| `build` task    | Runs after all dependencies build (`^build`); outputs go to `out/`, `dist/`, or `cdk.out/` |
| `dev` task      | Persistent and cached; runs all dev servers in parallel                                    |
| `lint` task     | Runs after all dependencies lint (`^lint`)                                                 |
| `test` task     | Runs after build (`^build`); use integration tests, not unit mocks                         |
| Internal deps   | Reference with `"*"` version in `package.json` dependencies                                |
| Cache           | `build` cache is off by default; `dev` cache is on                                         |
| `.turbo/`       | Auto-generated cache dir — always gitignored                                               |
| `node_modules/` | Hoisted to root by Yarn workspaces; always gitignored                                      |

---

## 10. Adding New Workspaces

1. Create the folder under the appropriate workspace directory (`apps/`, `packages/`, `services/`, `tests/`)
2. Add a `package.json` with a scoped name and the required scripts (`build`, `dev`, `lint`, `test`)
3. Run `yarn install` from the root to hoist dependencies
4. Reference internal packages with `"*"` in `dependencies`
5. Run `yarn turbo run build --filter=@<scope>/<name>` to verify the build pipeline
