# [Technology / Stack Name] Coding Rules

## Folder Structure

// [Describe the top-level layout and what each folder is responsible for]

```
src/
├── config/
│   └── [config loader]              # [description]
├── constants/
│   ├── [env file]                   # typed env variable accessors
│   └── [resources file]             # resource name constants
├── lib/
│   ├── [infrastructure layer]/
│   │   └── <feature>.[layer].ts     # [infra construct description]
│   └── [domain layer]/
│       ├── common/
│       │   └── [shared dtos]        # shared response types
│       └── <feature>/
│           ├── <feature>.model.ts   # data model / schema
│           ├── <feature>.dto.ts     # validation schemas and DTO classes
│           ├── <feature>.service.ts # handler / business logic functions
│           ├── <feature>.lambda.ts  # [platform construct — e.g. CDK NodejsFunction]
│           └── <feature>.spec.ts    # integration tests
├── utils/
│   └── [utility file]               # [describe shared helpers]
└── [stack or entry].ts              # [main wiring / entry point]
```

// Never place model, dto, service, or spec files outside their feature folder

## Model (`<feature>.model.ts`)

// [Describe the schema/ORM library used and any invariants]
// [List defaults that must always be set — e.g. id, timestamps]
// [Describe how table/collection names are sourced — env var, constants, etc.]

```typescript
// [Example model class]
```

## DTOs (`<feature>.dto.ts`)

// [Describe the validation library (e.g. zod) and DTO factory pattern]
// [Describe field reuse strategy — e.g. shared schema object, .partial() for updates]
// [Note static factory methods exposed on each DTO class]

```typescript
// [Example DTO file]
```

## Service / Handlers (`<feature>.service.ts`)

// [Describe handler shape — plain exported const, class method, etc.]
// [Describe error handling strategy — try-catch, result types, etc.]
// [Describe validation contract — safeParse before use, return 400 on failure]
// [Describe 404 handling — check existence, return 404 with ErrorDto]
// [Describe update strategy — filter undefined fields before partial update]

```typescript
// [Example service handlers for CRUD operations]
```

## [Platform Constructs] (`<feature>.[construct].ts`)

// [Describe the construct class pattern — one class per operation, shared entry file, etc.]
// [List what must be bundled externally vs. inlined]
// [Describe how resource names are read — from constants, never hardcoded]
// [Describe optional/required constructor parameters — scope, environment, logGroup, etc.]

```typescript
// [Example construct class]
```

## [Infrastructure Layer] (`lib/[infra]/<feature>.[type].ts`)

// [Describe the infrastructure wrapper pattern]
// [List required billing/scaling settings]
// [List required removal/lifecycle policies]
// [Describe how partition/primary keys are defined]

```typescript
// [Example infra construct]
```

## [Gateway / Router] (`lib/[gateway]/<feature>.[gateway].ts`)

// [Describe the gateway/router wrapper pattern]
// [Describe how names are sourced]

```typescript
// [Example gateway class]
```

## Stack / Entry Point (`[stack].ts`)

// Wire resources in this order: [list resource wiring order]
// [Describe permission grants — which lambdas need which table/resource access]
// [Describe CORS setup — when and how to apply preflight]
// [Describe environment variable passing to constructs]

```typescript
// [Example route wiring + permission grants + CORS]
```

## Constants

// `constants/[env file].ts` — typed accessor object for all `process.env` variables, no logic
// `constants/[resources file].ts` — all resource names derived from a shared prefix (e.g. `STACK_NAME`)

```typescript
// [Example env constants and resource name constants]
```

## Shared Utilities (`utils/[utility].ts`)

// [List each exported helper with its contract]
// e.g. `createResponse(code, data, headers?)` — always includes CORS headers
// e.g. `badRequest(details)` — returns 400 with ErrorDto wrapping validation errors
// e.g. `internalError(err)` — logs error and returns 500 with ErrorDto

## Shared DTOs (`lib/[domain]/common/[api.dto].ts`)

// [Describe shared error and success DTO shapes]
// [Note static factory methods — .create(), .schema]

## Tests (`<feature>.spec.ts`)

// Must be integration tests — no mocking
// Must throw early if required env variables are not set
// Must use randomized identifiers to prevent test pollution across runs
// Must follow full CRUD lifecycle in order: create → list → get → update → delete
// Must persist the created resource id in a `let` variable shared across tests
// Must include validation tests for invalid input (400) and missing resources (404)
// Must always end with a cleanup test that deletes created data even if earlier tests fail
// Must include all constants and setup inside the `describe()` block

```typescript
// [Example describe block with lifecycle tests]
```

---

<!--
HOW TO CREATE A NEW INSTRUCTIONS FILE
=======================================

1. Copy this file to instructions/<technology>.md
   - Use the same name as the paired bootstrap file (e.g. aws.node.lambda.md → aws.node.lambda.md)

2. Write rules as AI-facing constraints, not prose:
   - Lead each section with `//` comment rules that an LLM can follow directly
   - Code blocks show the canonical pattern — they are the law, not suggestions
   - Rules must be specific enough that an AI generates compliant code without extra prompting

3. Cover every layer of the stack:
   - One section per file type the AI will be asked to generate
   - Each section lists invariants the AI must never violate
   - Each section includes a minimal but complete TypeScript code example

4. Describe "never" rules explicitly:
   - e.g. "Never hardcode resource names", "Never mock in tests", "Never duplicate field definitions"
   - These prevent the most common LLM drift patterns

5. Pair it with a bootstrap file:
   - Create a matching bootstrap/<technology>.md (see bootstrap/template.md)
   - The bootstrap is the human setup guide; this file is the AI code-generation rulebook

6. Reference the existing example:
   - instructions/aws.node.lambda.md is the canonical reference implementation
-->
