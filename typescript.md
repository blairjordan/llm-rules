# TypeScript Development Rules

## Overview & Philosophy

- **No comments** - code should be self-documenting through descriptive naming
- Only use comments for extremely obscure operations that cannot be clarified through better naming
- Functional programming approach where practical with strong typing
- Clean separation of concerns between modules
- Prefer functional composition over class inheritance
- Error handling through structured logging and propagation to boundaries

## Structure & Organization

### Project Architecture

- `src/service/` - Contains service-specific implementations
- `src/lib/` - Shared utilities and core functionality
- `src/types/` - TypeScript type definitions
- Services are organized into focused, single-responsibility modules
- Clear separation between service logic and infrastructure concerns
- Files are highly focused, handling a single responsibility or domain concern

### Global Service Philosophy

- Selective use of globally available services - only when truly needed across the application
- Services are initialized at startup with proper dependency injection
- Dependencies are passed explicitly to service constructors/factories rather than imported directly
- Global instances are limited to core infrastructure services (logging, database, etc.)
- Non-global services receive their dependencies as parameters
- Service initialization follows a consistent pattern: factory functions that return initialized instances
- Top-level init keeps startup clean, enables better testing and instrumentation

## Naming Conventions

- Self-explanatory function and variable names that make code reading intuitive
- **Named Parameters**: Always use named parameters (object destructuring) for function calls with multiple arguments
  - ✅ Good: `new SQSClient({ region: env.AWS_REGION })`
  - ❌ Bad: `new SQSClient(env.AWS_REGION)`
  - ✅ Good: `createService({ pool, s3Client, bucketName })`
  - ❌ Bad: `createService(pool, s3Client, bucketName)`
- Prefer arrow functions over "function" keyword
- Consistent declaration of TypeScript interfaces and types for enhanced code clarity

## Core Patterns & Best Practices

### Code Style Fundamentals

- Strong typing with TypeScript
- Minimal use of classes, prefer plain functions and objects
- Type guards to narrow and validate types at runtime
- Factory pattern for creating service instances
- Prioritize code clarity and structure over explanatory comments

### Async Operations

- Use async/await for all async operations
- Avoid .then(), .catch(), and .finally() unless required by a library or for a specific pattern

### Dependency Injection

- Services receive their dependencies as parameters rather than importing them
- Message/event handlers are passed into services rather than imported directly
- This enables better testability, clearer service boundaries, and more flexible composition

### Control Flow Patterns

#### Switch Expressions Over Nested If/Else

- Prefer `switch` expressions wrapped in immediately invoked functions (IIFEs) over deeply nested `if/else` blocks when assigning values based on a condition

```typescript
const result = (() => {
  switch (val) {
    case "pdf":
      return "Generating PDF..."
    case "rtf":
      return "Generating RTF..."
    default:
      return "Unknown format"
  }
})()
```

#### Runtime Dispatch Style

- Avoid deeply nested `if/else` blocks with runtime type checks
- Prefer a **declarative dispatch map** or pattern-match-like structure for runtime branching

```typescript
const fieldTypeHandlers: Record<string, (field: any) => void> = {
  PDFTextField: (f) => {
    ;(f as PDFTextField).setText(String(value))
    filledCount++
    logger.debug(`✅ Filled text field: ${fieldName} = ${value}`)
  },
  PDFCheckBox: (f) => {
    if (value) {
      ;(f as PDFCheckBox).check()
      filledCount++
      logger.debug(`✅ Checked field: ${fieldName}`)
    }
  },
}
```

#### Early Exit over Nested Branching

- Use early returns/guards to eliminate irrelevant branches as soon as possible
- Avoid wrapping large blocks of logic inside nested if or else blocks

```typescript
// Prefer
if (!user || !user.isAdmin) return
// continue with relevant logic

// Avoid
if (user) {
  if (user.isAdmin) {
    // deeply nested logic
  }
}
```

#### Functional Iteration

- Prefer functional iteration methods (`map`, `reduce`, `filter`, `forEach`) over traditional loops
- Exceptions: async iteration (`for await...of`) or when early exit is needed

#### Array Construction

- Prefer `Array.from()` with mapping function over `Array().fill().map()` for creating sequences

```typescript
// Prefer
const results = Array.from({ length: count }, (_, i) =>
  processItem(data.slice(i * 2, i * 2 + 2))
)

// Avoid
const results = []
for (let i = 0; i < count; i++) {
  results.push(processItem(data.slice(i * 2, i * 2 + 2)))
}
```

## Error Handling & Quality

### Error Handling Strategy

- Prefer error propagation to the nearest logical boundary
- Only catch errors when:
  1. Additional context can be added to the error
  2. The error needs transformation at a boundary
  3. The error occurs in an async context that won't be caught by boundary error handlers
- Error handling should be done at service boundaries where the full context is available
- Structured error objects with error codes for categorization
- Strong validation at input boundaries to prevent downstream issues

### Logging Guidelines

#### Visual Organization

- Visual organization through emoji prefixes for different operations
- Structured contextual logging with clear message and separate context object
- Consistent log level usage (info for operations, error for failures)
- Log messages should be human-readable with context as machine-parseable object

#### Logging at Boundaries, Not Internals

- **Log at system or module boundaries** — not inside pure or narrowly scoped internal functions
- Internal functions should remain clean and focused on their logic without logging
- Logging should capture observable system behavior, not implementation details

```typescript
// Good: logs at the boundary where effects happen
const fileProcessor = initFileProcessor(s3Client)
logger.info("📄 File processor initialized")

// Avoid: internal logging that clutters logic
function initFileProcessor(s3Client) {
  logger.info("📄 File processor initialized"); // ❌ Don't log here
  return { ... };
}
```

### Data Processing

- Pipeline architecture for processing flows
- Clear separation between data acquisition, transformation, and storage
- Strong validation at input/output boundaries
- Pure functions for transformation logic
- Side effects isolated to specific boundary functions
- Event-driven processing with clear state transitions

### Testing

- Unit tests focused on core business logic
- Clear separation of test setup from assertions
- Extensive use of test helpers and fixtures
- Consistent test organization
- Mock external dependencies at boundaries

## Maintenance Notes

- This file should be kept up-to-date as the project structure evolves
- Any significant changes to project architecture, dependencies, or infrastructure should be documented here
- When making changes, ensure to update this file to reflect the current state of the project
- Once a set of changes are made, return any manually required tasks as a TODO list (such as adding .env values)
