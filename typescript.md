# TypeScript Development Rules

## Principles

- Optimize for clear domain modeling, small modules, and explicit boundaries.
- Prefer simple functions and plain data over classes. Use classes only when identity, encapsulated mutable state, or framework integration makes them the clearest tool.
- Make invalid states hard to represent with discriminated unions, branded IDs, literal unions, and narrow domain types.
- Keep generic abstractions behavior-agnostic. Inject policy and domain-specific behavior from the consumer.
- Use dependency injection at service boundaries. Do not import concrete infrastructure into business logic.
- Choose readability and correctness over style dogma. Functional composition is good when it clarifies intent; loops are fine when control flow, early exit, or performance is clearer.

```typescript
// Good: generic factory stays policy-free
const createEnvelope = <Payload, Metadata>({
  payload,
  getMetadata,
}: {
  payload: Payload
  getMetadata: (payload: Payload) => Metadata
}) => ({
  payload,
  metadata: getMetadata(payload),
})

// Bad: generic factory knows one consumer's policy
const createEnvelope = (payload: { pluginId?: string }) => ({
  payload,
  metadata: payload.pluginId ? { pluginId: payload.pluginId } : {},
})
```

## Compiler Baseline

- Keep TypeScript strict. Do not weaken compiler settings to make a change pass.
- Prefer these settings unless a project has a documented reason not to:
  - `strict`
  - `noUncheckedIndexedAccess`
  - `exactOptionalPropertyTypes`
  - `noImplicitOverride`
  - `noFallthroughCasesInSwitch`
  - `useUnknownInCatchVariables`
- Treat `skipLibCheck` as a build-performance compromise only. Do not use it to hide application type errors.
- Use the repository's module system consistently. In ESM projects, include explicit file extensions where the runtime requires them.
- Keep generated types generated. Do not hand-edit generated files unless the generator is broken and the fix is documented.

## Imports And Modules

- Prefer named exports for application code. Default exports are acceptable for framework conventions that require them.
- Use `import type` and `export type` for type-only dependencies.
- Avoid namespace imports unless the imported module is genuinely used as a namespace.
- Barrel files are optional, not automatic:
  - Use them for stable public module boundaries.
  - Avoid them when they obscure ownership, create circular imports, slow tooling, or hide large dependency trees.
  - Barrel files must contain re-exports only.
- Keep module boundaries intentional:
  - `src/lib/` for shared domain-agnostic utilities.
  - `src/types/` for shared type contracts that are not owned by one feature.
  - Feature/service directories own their internal implementation details.

```typescript
// Good
import type { Logger } from "../logging/types.js"
import { createUserService } from "./user-service.js"

// Good when exposing a stable public surface
export { createUserService } from "./user-service.js"
export type { UserService } from "./types.js"
```

## Naming And API Shape

- Use descriptive names that encode domain meaning, not implementation mechanics.
- Use object parameters for functions with multiple arguments, optional values, or same-typed values.
- Use positional parameters only for obvious, stable, tiny APIs.
- Prefer arrow functions for local functions and callbacks. Use function declarations when hoisting or stack traces make them clearer.
- Name booleans as predicates: `isReady`, `hasAccess`, `canRetry`.
- Avoid vague names such as `data`, `payload`, `result`, and `handler` when a domain name is available.

```typescript
// Good
const createInvoice = ({
  customerId,
  lineItems,
  issuedAt,
}: {
  customerId: CustomerId
  lineItems: InvoiceLineItem[]
  issuedAt: Date
}) => {
  // ...
}

// Avoid
const createInvoice = (customerId: string, items: unknown[], date: Date) => {
  // ...
}
```

## Type Modeling

- Prefer `type` for unions, mapped types, primitives, and function shapes.
- Prefer `interface` for object contracts intended to be extended or implemented by external consumers.
- Use discriminated unions for state machines and variants.
- Use `satisfies` to validate object literals while preserving narrow literal types.
- Prefer `as const` for fixed lookup tables and literal tuples.
- Prefer literal unions or `as const` objects over `enum`. Use `enum` only when interoperating with existing APIs that require it.
- Avoid broad primitives for domain identifiers. Use branded types where mixing IDs would be dangerous.
- Do not expose implementation-specific types from module boundaries.
- Avoid type assertions. If an assertion is necessary, keep it close to the boundary and explain the invariant through a guard, schema, or wrapper.
- Never use double assertions such as `value as unknown as Target` unless isolating a broken third-party type behind a narrow adapter.

```typescript
type UserId = string & { readonly __brand: "UserId" }
type InvoiceId = string & { readonly __brand: "InvoiceId" }

type SyncState =
  | { status: "idle" }
  | { status: "running"; startedAt: Date }
  | { status: "failed"; error: Error }

const statusLabels = {
  idle: "Idle",
  running: "Running",
  failed: "Failed",
} as const satisfies Record<SyncState["status"], string>

const syncStatuses = ["idle", "running", "failed"] as const
type SyncStatus = (typeof syncStatuses)[number]
```

## Avoid `any`

- Never use `any` to appease the type system.
- Prefer `unknown` at untrusted boundaries, then validate or narrow immediately.
- Always check library-provided types first. If they are incomplete, prefer:
  - Narrower union types
  - Type guards or assertion functions
  - Interface augmentation
  - A local wrapper around the weakly typed API
- `any` is acceptable only at a narrow boundary when there is no stable type contract, and the value is validated or contained immediately.
- If a code change includes `any`, call it out in the response with a 😢 and explain why it remains.

```typescript
// Good
const parseMessage = (value: unknown): Message => {
  if (!isMessage(value)) {
    throw new Error("Invalid message")
  }

  return value
}

// Avoid
const parseMessage = (value: any): Message => value
```

## Runtime Boundaries

- Validate all external input: HTTP bodies, query strings, environment variables, JSON files, queues, webhooks, local storage, and third-party SDK responses.
- Prefer a schema library already used by the repository, such as Zod, Valibot, ArkType, or io-ts. Do not add a new schema library when a suitable one already exists.
- Convert untrusted input into trusted domain types at the boundary.
- Keep validation errors useful at the boundary and simple inside the domain.
- Do not trust generated API client types without checking how runtime validation is handled.

```typescript
const configSchema = z.object({
  databaseUrl: z.string().url(),
  logLevel: z.enum(["debug", "info", "warn", "error"]).default("info"),
})

const config = configSchema.parse({
  databaseUrl: process.env.DATABASE_URL,
  logLevel: process.env.LOG_LEVEL,
})
```

## Control Flow

- Prefer guard clauses and early returns to reduce nesting.
- Use exhaustive `switch` statements for discriminated unions.
- Use dispatch maps for dynamic handlers when they improve readability.
- Do not replace clear conditionals with maps just to avoid `switch`.
- Use `for...of` when early exit, async sequencing, retries, mutation of local accumulators, or performance-sensitive iteration is clearer.
- Use `map`, `filter`, `reduce`, and `flatMap` when transforming collections without side effects.
- Avoid `forEach` with async callbacks.

```typescript
const assertNever = (value: never): never => {
  throw new Error(`Unexpected value: ${String(value)}`)
}

const renderStatus = (state: SyncState): string => {
  switch (state.status) {
    case "idle":
      return "Idle"
    case "running":
      return `Running since ${state.startedAt.toISOString()}`
    case "failed":
      return state.error.message
    default:
      return assertNever(state)
  }
}
```

## Async And Concurrency

- Use `async`/`await` for asynchronous code.
- Use `.then`, `.catch`, or `.finally` only when integrating with APIs where promise chaining is clearer.
- Never use `Array.prototype.forEach` for async work.
- Use `Promise.all` for independent concurrent work.
- Use `Promise.allSettled` when partial failure is expected and handled.
- Use sequential loops when order, rate limits, retries, transactions, or cancellation matter.
- Pass `AbortSignal` through cancellable operations where supported.
- Always handle fire-and-forget tasks explicitly with logging and lifecycle ownership.

```typescript
// Concurrent independent work
const users = await Promise.all(userIds.map((userId) => loadUser({ userId })))

// Sequential work with retry/rate-limit semantics
for (const job of jobs) {
  await processJob({ job, signal })
}
```

## Error Handling

- Catch errors only when adding context, transforming to a boundary error, cleaning up, or recovering.
- Preserve the original cause when wrapping errors.
- Use typed domain errors where callers need to branch on failure reason.
- In `catch`, treat the caught value as `unknown`.
- Do not throw strings or plain objects.
- Do not log and rethrow at the same layer unless the log adds boundary-level operational context.

```typescript
try {
  await publishInvoice({ invoiceId })
} catch (error: unknown) {
  throw new InvoicePublishError("Failed to publish invoice", {
    cause: error,
    invoiceId,
  })
}
```

## Logging

- Log at system boundaries: request handlers, workers, jobs, CLI commands, and integrations.
- Keep pure functions and domain transformations free of logging.
- Include structured context objects rather than interpolating complex values into strings.
- Use consistent log levels:
  - `debug` for diagnostic detail.
  - `info` for lifecycle events.
  - `warn` for degraded but recoverable behavior.
  - `error` for failed operations requiring attention.
- Avoid logging secrets, tokens, full request bodies, credentials, or personal data unless explicitly approved and redacted.

```typescript
logger.info("Invoice published", {
  invoiceId,
  customerId,
})
```

## Data And Immutability

- Prefer immutable inputs and outputs for domain functions.
- Use `readonly` for object properties and arrays where mutation is not part of the contract.
- Avoid hidden mutation of arguments.
- Use mutation locally when it is simpler and contained.
- Prefer `Array.from` for sequence construction when it is clearer than manual pushing.
- Avoid deep cloning as a default. Clone only the specific data that needs isolation.

```typescript
const groupByCustomer = (invoices: readonly Invoice[]) => {
  const grouped = new Map<CustomerId, Invoice[]>()

  for (const invoice of invoices) {
    const existing = grouped.get(invoice.customerId) ?? []
    existing.push(invoice)
    grouped.set(invoice.customerId, existing)
  }

  return grouped
}
```

## Comments

- Comments are allowed only to explain:
  - Domain intent or business invariants
  - Non-obvious constraints or regulatory rules
  - Performance tradeoffs or algorithmic guarantees
- Comments must explain why, never what.
- Prefer naming, type modeling, and extraction over explanatory comments.
- Never leave residual comments after refactoring.

```typescript
// Bad: residual comment left behind
function connectToDatabase(uri: string) {
  // Removed retry logic; handled by connection pool now
  return new DbClient(uri)
}
```

## Testing

- Test behavior at public boundaries, not implementation details.
- Unit test pure domain logic and type-driven edge cases.
- Integration test infrastructure boundaries such as databases, queues, file systems, and third-party clients.
- Prefer real parsers, serializers, and validators in tests unless the dependency is slow, flaky, or external.
- Keep fixtures typed with `satisfies` so tests fail when contracts drift.
- Add regression tests for bug fixes before or alongside the fix.
- Avoid snapshots for complex data unless the snapshot is intentionally reviewed and stable.

```typescript
const validInvoice = {
  id: "invoice_123",
  status: "draft",
  lineItems: [],
} satisfies Invoice
```

## Tooling And Linting

- Use the repository formatter and do not hand-format against it.
- Prefer lint rules that catch correctness issues, especially:
  - Floating promises
  - Unsafe `any`
  - Unhandled async errors
  - Exhaustiveness gaps
  - Inconsistent type imports
- Do not disable lint rules inline unless the exception is local, justified, and difficult to express another way.
- Keep `tsc --noEmit` or the repository equivalent passing before claiming a TypeScript change is complete.
- Treat dependency type upgrades as code changes: inspect breaking type changes rather than assuming they are noise.

## Framework And Platform Practices

- Follow the repository's established framework conventions before introducing new patterns.
- Keep server-only code out of client bundles.
- Isolate environment access behind typed config modules.
- Prefer native platform APIs where they are clear and supported by the target runtime.
- Use framework-provided request, response, routing, and caching primitives instead of custom wrappers unless there is a clear cross-cutting need.
- Treat date, time zone, currency, locale, and Unicode handling as domain-sensitive. Use explicit libraries or platform APIs rather than ad hoc string manipulation.

## Maintenance

- Keep this file aligned with the repository's actual TypeScript version, runtime, lint rules, and formatter.
- Do not add rules that cannot realistically be enforced or reviewed.
- When a rule conflicts with local project conventions, update the rule or document the exception.
- For general code responses, explicitly state: "🐒 Abiding by coding laws"
- After each code change, include a Conventional Commit message:
  - Use `feat:`, `fix:`, `refactor:`, `chore:`, `test:`, or `docs:`.
  - Keep the subject at or below 72 characters.
  - Use imperative, present tense.
  - Include scope when useful: `type(scope): description`.
