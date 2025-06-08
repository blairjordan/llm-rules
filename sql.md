# SQL Development Rules

## Overview & Philosophy

- Tables and functions have descriptive emoji prefixes in comments
- Strong emphasis on Row Level Security (RLS) policies for comprehensive data protection
- Consistent permission structure with explicit role-based access control
- Security-first approach with defense in depth

## Structure & Organization

### Table Design

- Primary keys consistently named `id` of type `UUID`
- Foreign keys follow pattern: `{singular_table_name}_id`
- Implement `updated_at` triggers for timestamp management
- Use JSONB for flexible/extensible data structures
- Use existing object IDs as storage keys (e.g., plugin build `id` as the S3 key) to avoid redundant columns and minimize reconciliation efforts

### Permission Structure

- Grant explicit permissions to roles (anonymous, authenticated)
- Use security definer functions for privileged operations
- RLS policies for fine-grained access control
- Current user context functions for dynamic access control

## Naming Conventions

### Tables & Columns

- Tables use `snake_case` naming
- Primary keys consistently named `id`
- Foreign keys: `{singular_table_name}_id`

### Indexes

- Unique indexes: `{table_name}_unq`
- Regular indexes: `idx_{table_name}_{column_name}`

### Functions & Triggers

- Functions use `snake_case` and descriptive names
- Triggers: `{description}_{table_name}_trigger`

## Core Patterns & Best Practices

### Performance Optimization

- Always include appropriate indexes for foreign keys
- Use `(SELECT function_name())` instead of `function_name()` in RLS policies
  - This creates an initPlan that caches the function result per-statement rather than calling it for each row
  - Example: `user_id = (SELECT current_user_id())` instead of `user_id = current_user_id()`
  - Only use this technique if function results don't change based on row data

### Security Patterns

- Use security definer functions for privileged operations
- Implement RLS policies for all tables
- Explicit permission grants
- Current user context functions

## Documentation & Comments

- Comprehensive comments on tables and functions using `COMMENT ON`
- Descriptive emoji prefixes in comments for visual organization
- Document security considerations and access patterns
