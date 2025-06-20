# TypeScript Development Rules

## Overview & Philosophy

- **No comments**: code must be self-documenting through descriptive naming
- Only use comments for extremely obscure operations that cannot be clarified through naming
- Prefer a functional programming approach with strong typing
- Clean separation of concerns between modules
- Favor functional composition over class inheritance
- Structured error handling, clear propagation to boundaries, and comprehensive logging

### Abstraction Principles

- **Avoid Leaky Abstractions** – abstractions fully encapsulate implementation details

  ```typescript
  // ✅ Good: Factory remains generic, specifics handled by consumer
  const factory = (args, getMetadata) => ({
    metadata: getMetadata(args),
  })

  // ❌ Bad: Factory knows about specific parameter handling
  const factory = (args) => ({
    metadata: args.pluginId ? { pluginId: args.pluginId } : {},
  })
  ```

- **Single Responsibility** – each abstraction should serve a single clear purpose
- **Dependency Injection** – dependencies explicitly passed, never hardcoded
- **Interface Segregation** – minimal, focused interfaces
- **Open/Closed Principle** – open to extension, closed to modification

## Structure & Organization

### Project Architecture

- `src/service/` – Contains service-specific implementations
- `src/lib/` – Shared utilities and core functionality
- `src/types/` – TypeScript type definitions
- Services are organized into single-responsibility modules
- Clear separation between business logic and infrastructure concerns
- Files handle a single responsibility or domain concern

### Service Import Patterns

- Use barrel files (`index.ts`) for cleaner imports
- Import services through barrel files to reduce clutter

  ```typescript
  // ✅ Good
  import { serviceA, serviceB } from "./service"

  // ❌ Bad
  import { createServiceA } from "./service/service-a"
  import { createServiceB } from "./service/service-b"
  ```

- Barrel files contain only re-exports, no implementation:
  ```typescript
  // service/index.ts
  export { createServiceA } from "./service-a.js"
  export { createServiceB } from "./service-b.js"
  ```

### Global Service Philosophy

- Limited use of globally available services, strictly for core infrastructure
- Proper dependency injection, explicitly passed via constructors/factories
- Service initialization via consistent factory pattern
- Clear separation of initialization logic at application startup

## Naming Conventions

- Descriptive, self-explanatory function and variable names
- Always use named parameters (object destructuring) for multiple arguments:

  ```typescript
  // ✅ Good
  new SQSClient({ region: env.AWS_REGION })
  createService({ pool, s3Client, bucketName })

  // ❌ Bad
  new SQSClient(env.AWS_REGION)
  createService(pool, s3Client, bucketName)
  ```

- Prefer arrow functions over traditional `function` keyword
- Consistent declaration and usage of TypeScript interfaces and types

## Core Patterns & Best Practices

### Code Style Fundamentals

- Strong typing with TypeScript
- Minimal use of classes; favor plain functions and objects
- Use type guards for runtime type narrowing
- Prioritize code clarity over explanatory comments

### Async Operations

- Always use `async/await` syntax for asynchronous code
- Avoid `.then()`, `.catch()`, `.finally()` unless explicitly required by external libraries or patterns

### Dependency Injection

- Pass dependencies explicitly into services and handlers, never import them directly
- Enhances testability, service boundaries, and composition flexibility

### Control Flow Patterns

#### Map-Based Dispatch Over Switch Statements

- Use `Map` for runtime dispatch when dealing with dynamic keys or complex handler functions:

  ```typescript
  // ✅ Good: Map-based dispatch for dynamic handlers
  const getPreliminarySamplesFns = new Map([
    ["location", locationFn],
    ["deviceLocation", locationFn],
    ["samples", samplesFn],
    ["zones", zonesFn],
    ["temperatures", temperaturesFn],
  ])

  const getPreliminarySamples =
    getPreliminarySamplesFns.get(propertyName) || defaultFn
  ```

- Use for cleaner, more maintainable dispatch than large switch statements
- Particularly effective when handlers are functions or when keys are dynamic

#### Switch Expressions Over Nested If/Else

- Use `switch` expressions in immediately invoked function expressions (IIFE):
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

#### Runtime Dispatch

- Use declarative dispatch maps instead of nested conditions:
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

#### Early Exit Over Nested Branching

- Prefer early returns or guards to minimize nesting:

  ```typescript
  // ✅ Good
  if (!user || !user.isAdmin) return

  // ❌ Avoid
  if (user) {
    if (user.isAdmin) {
      // deeply nested logic
    }
  }
  ```

#### Functional Iteration

- Use functional array methods (`map`, `reduce`, `filter`, `forEach`) instead of traditional loops unless early exit or async iteration is required
- Exceptions: async iteration (`for await...of`) or when early exit is needed

#### Array Construction

- Prefer `Array.from()` for sequence creation:

  ```typescript
  // ✅ Good
  const results = Array.from({ length: count }, (_, i) =>
    processItem(data.slice(i * 2, i * 2 + 2))
  )

  // ❌ Bad
  const results = []
  for (let i = 0; i < count; i++) {
    results.push(processItem(data.slice(i * 2, i * 2 + 2)))
  }
  ```

### Type Safety & Runtime Patterns

#### Type Guards for Runtime Safety

- Use type guards to safely narrow types at runtime boundaries:

  ```typescript
  // ✅ Good: Explicit type guard
  const isMPacket = (packet: KnownPacket): packet is MBPacket | MCPacket =>
    Object.prototype.hasOwnProperty.call(packet, "tagNumber")

  const isKnownPacket = (packet: Packet): packet is KnownPacket =>
    !Object.prototype.hasOwnProperty.call(packet, "error")
  ```

- Essential when working with external data or union types
- Provides compile-time safety with runtime checks

#### Template Method Pattern with Function Parameters

- Pass behavior as functions rather than using inheritance:
  ```typescript
  // ✅ Good: Function parameter for customization
  const decodeBinaryFields = (buffer: Buffer, fields: PacketFields): any =>
    Object.keys(fields).reduce((prev, curr) => {
      const { indices, decodeFn } = fields[curr]
      const [startIdx, endIdx] = indices
      prev[curr] = decodeFn(buffer.slice(startIdx, endIdx))
      return prev
    }, {})
  ```

### Data Processing Patterns

#### Buffer Manipulation with Functional Composition

- Chain buffer operations for clear data transformation:
  ```typescript
  // ✅ Good: Clear buffer processing pipeline
  const decodeSamples = (b: Buffer) =>
    Array(Math.floor(b.length / 2))
      .fill(0)
      .reduce((prev, _, i) => {
        const sample = b.slice(i * 2, i * 2 + 2)
        prev.push(decodeSample(sample))
        return prev
      }, [])
  ```

#### Declarative Configuration Objects

- Use configuration objects to drive behavior:
  ```typescript
  // ✅ Good: Declarative field definitions
  const getMBPacketFields = (samplesLength: number): PacketFields => ({
    WTSN: { indices: [0, 3], decodeFn: decodeWTSN },
    interval: { indices: [3, 4], decodeFn: (b: Buffer) => b.readUInt8() },
    first: { indices: [4, 10], decodeFn: decodeDate },
    samples: { indices: [10, samplesLength], decodeFn: decodeSamples },
  })
  ```

### Object Construction Patterns

#### Fluent Interface for Configuration

- Allow method chaining for builder-like patterns:

  ```typescript
  // ✅ Good: Fluent interface
  healthChecker.monitorService(service, opts).serve(port)
  ```

- Use sparingly, only when the API benefits from sequential configuration

#### Factory Pattern with Consistent Initialization

- Use factory functions for complex object creation:
  ```typescript
  // ✅ Good: Factory with dependency injection
  const createService = (opts: ServiceOpts): Service => {
    const service = new Service(opts)
    service.connect()
    return service
  }
  ```

### Event-Driven Patterns

#### Resource Cleanup with Event Handlers

- Proper resource management through event-driven cleanup:
  ```typescript
  // ✅ Good: Explicit cleanup handlers
  server
    .on("error", (error: any) => logger.warn("⚠️ Socket error", error))
    .on("close", (error: any) =>
      logger.log(error ? "error" : "info", "🚪 Socket closed", error)
    )
  ```

## Error Handling & Logging

### Error Handling Strategy

- Errors propagate to logical boundaries; catch errors only when:
  - Additional context can be provided
  - Error requires transformation at a boundary
  - Async context demands explicit error handling
- Structured errors with clear categorization
- Validate strongly at input boundaries to prevent downstream errors

### Logging Guidelines

- Visual organization via emoji prefixes
- Structured logging: clear, human-readable messages plus context objects
- Consistent log level usage: `info` for operational logs, `error` for failures
- Log at system/module boundaries only, not inside pure/internal functions:

  ```typescript
  // ✅ Good: logs at boundaries
  const fileProcessor = initFileProcessor(s3Client)
  logger.info("📄 File processor initialized")

  // ❌ Bad: logs inside internals
  function initFileProcessor(s3Client) {
    logger.info("📄 File processor initialized") // Avoid internal logging
    return { ... }
  }
  ```

## Data Processing

- Pipeline architecture: clear separation of data acquisition, transformation, and storage
- Pure functions for transformation logic
- Side effects isolated to boundary functions
- Event-driven processing with explicit state transitions

## Testing

- Unit tests emphasize core business logic
- Separate test setup clearly from assertions
- Extensive test helpers and fixtures
- Mock external dependencies at service boundaries

## Maintenance Notes

- Keep this file updated as project structure evolves
- Document any significant changes to architecture, dependencies, or infrastructure
- Ensure updates are accurately reflected here after any changes
- Clearly list manual follow-up tasks (e.g., updating `.env` values) as TODO items

## IMPORTANT

- For general code responses, explicitly state:
  "🐒 Abiding by coding laws"
