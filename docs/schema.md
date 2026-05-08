# Task Manager CLI Technical Design

## 0. Project conventions

### Language and runtime
- JavaScript on Node.js 20+.
- ES module syntax only using import/export.

### Code style
- 2-space indentation.
- Single quotes for strings.
- Use const by default, use let only when reassignment is needed, and do not use var.
- Add JSDoc comments to all exported functions and classes.

### Error handling conventions
- Wrap operations that can fail in try/catch.
- Throw Error objects with descriptive messages.
- Log failures with console.error.

### Testing conventions
- Use the built-in Node assert module.
- Test file names end with .test.js.
- Each test verifies one behavior.

### Data storage convention
- Keep all task data in memory using plain JavaScript data structures.

## 1. Data models

### Task

| Property | Type | Required | Validation rules |
| --- | --- | --- | --- |
| id | string | Yes | Non-empty. Generated at create time. Must be unique across in-memory store. |
| title | string | Yes | Trimmed string, length 1-200. |
| description | string | No | Trimmed string, length 0-1000. Defaults to ''. |
| status | 'todo' \| 'in-progress' \| 'done' | Yes | Must be one of allowed enum values. Defaults to 'todo' on create. |
| priority | 'low' \| 'medium' \| 'high' | Yes | Must be one of allowed enum values. Defaults to 'medium' on create. |
| createdAt | string (ISO 8601 datetime) | Yes | Set once at creation using `new Date().toISOString()`. |
| updatedAt | string (ISO 8601 datetime) | Yes | Set at creation and updated on each successful mutation. |

### CreateTaskInput

| Property | Type | Required | Validation rules |
| --- | --- | --- | --- |
| title | string | Yes | Same as `Task.title`. |
| description | string | No | Same as `Task.description`. |
| status | 'todo' \| 'in-progress' \| 'done' | No | Defaults to 'todo' when omitted. |
| priority | 'low' \| 'medium' \| 'high' | No | Defaults to 'medium' when omitted. |

### UpdateTaskInput

| Property | Type | Required | Validation rules |
| --- | --- | --- | --- |
| id | string | Yes | Must match an existing task id. |
| title | string | No | If provided, same rules as `Task.title`. |
| description | string | No | If provided, same rules as `Task.description`. |
| status | 'todo' \| 'in-progress' \| 'done' | No | If provided, must be valid enum value. |
| priority | 'low' \| 'medium' \| 'high' | No | If provided, must be valid enum value. |

### ListTasksQuery

| Property | Type | Required | Validation rules |
| --- | --- | --- | --- |
| status | 'todo' \| 'in-progress' \| 'done' | No | Optional filter. If provided, must be valid enum value. |
| priority | 'low' \| 'medium' \| 'high' | No | Optional filter. If provided, must be valid enum value. |
| sortBy | 'priority' \| 'createdAt' | No | Defaults to 'createdAt'. |
| sortOrder | 'asc' \| 'desc' | No | Defaults to 'desc'. |

### In-memory store

| Property | Type | Required | Validation rules |
| --- | --- | --- | --- |
| tasks | Task[] | Yes | Single source of truth in runtime memory. No database or file persistence. |

## 2. File structure

```text
.
├── docs/
│   ├── project-plan.md      # Product and implementation plan used as design input
│   └── schema.md            # Technical design for data schema, modules, and errors
├── src/
│   ├── index.js             # CLI entry point, argument parsing, command dispatch
│   ├── task.js              # Task model factory/class and timestamp/id initialization
│   ├── taskManager.js       # In-memory CRUD, filtering, and sorting orchestration
│   ├── validators.js        # Input validation and normalization helpers
│   ├── sorter.js            # Priority/date sort helpers and ordering rules
│   ├── formatter.js         # Console output formatters for success, list, and errors
│   └── errors.js            # Custom error classes used across modules
└── tests/
    ├── taskManager.test.js  # CRUD/filter/sort behavior tests
    └── validators.test.js   # Validation rule tests
```

## 3. Module responsibilities

### src/index.js
- Exports: `main(argv)`.
- Responsibilities: Parse CLI args, map commands (`create`, `list`, `update`, `delete`), call `TaskManager`, and print formatted output.
- Depends on: `taskManager.js`, `validators.js`, `formatter.js`, `errors.js`.

### src/task.js
- Exports: `createTask(input)`.
- Responsibilities: Build canonical `Task` object with defaults and timestamps.
- Depends on: `validators.js` for final field checks.

### src/taskManager.js
- Exports: `TaskManager` class.
- Responsibilities: Own in-memory `tasks` array and implement create/list/update/delete/filter/sort flows.
- Depends on: `task.js`, `validators.js`, `sorter.js`, `errors.js`.

### src/validators.js
- Exports: `validateCreateTaskInput`, `validateUpdateTaskInput`, `validateListQuery`, `validateTaskId`, `normalizeText`.
- Responsibilities: Enforce all input constraints and enum checks before business logic mutates data.
- Depends on: `errors.js`.

### src/sorter.js
- Exports: `sortTasks(tasks, sortBy, sortOrder)`.
- Responsibilities: Apply deterministic ordering for `priority` and `createdAt` sorts.
- Depends on: none.

### src/formatter.js
- Exports: `formatTask`, `formatTaskList`, `formatSuccess`, `formatError`.
- Responsibilities: Keep presentation concerns separate from business logic.
- Depends on: none (except standard library).

### src/errors.js
- Exports: `AppError`, `ValidationError`, `NotFoundError`, `CommandError`.
- Responsibilities: Provide typed errors with stable names/codes and user-safe messages.
- Depends on: none.

### tests/taskManager.test.js
- Exports: none (test runner entry file).
- Responsibilities: Verify one behavior per test for CRUD, filters, and sorting using Node `assert`.
- Depends on: `src/taskManager.js`.

### tests/validators.test.js
- Exports: none (test runner entry file).
- Responsibilities: Verify validation failures/success paths per rule.
- Depends on: `src/validators.js`.

## 4. Error handling strategy

### Error types
- `ValidationError`
- `NotFoundError`
- `CommandError`
- `AppError` (base type)

### Where errors are thrown
- `src/validators.js`
- Throws `ValidationError` for invalid title/description length, invalid status/priority, invalid sort fields/order, and malformed ids.

- `src/taskManager.js`
- Throws `NotFoundError` when update/delete/get target id does not exist.
- Throws `ValidationError` when mutation payload fails second-layer validation.

- `src/index.js`
- Throws `CommandError` for invalid command names or missing required CLI flags.
- Wraps command execution in `try/catch`, logs with `console.error`, and sets non-zero exit code on failure.

### Handling rules
- All thrown errors are `Error` objects (never plain strings).
- Validation runs at CLI boundary and again in manager methods before mutation.
- On any thrown error, no partial data mutation is committed.
- User-facing messages use a consistent prefix: `Error: <message>`.

## 5. Architect Agent workflow for schema

### Objective
- Convert the plan in docs/project-plan.md into a minimal, implementation-ready technical design.

### Inputs
- docs/project-plan.md for scope, required features, and validation expectations.
- .github/copilot-instructions.md for coding and testing conventions.

### Required output structure
- Data models: every entity property with type, required flag, and validation rules.
- File structure: complete tree with one-line purpose per file.
- Module responsibilities: each module exports and dependency relationships.
- Error handling strategy: error types and where each error is thrown.

### Guardrails
- Keep design minimal and aligned to required CLI features only.
- Do not introduce database or external runtime dependencies.
- Keep naming and module boundaries consistent with Node.js ES modules.

### Definition of done
- docs/schema.md includes all required sections and maps directly to the project plan.
- Validation and error behavior are explicit enough to implement without re-interpreting requirements.
