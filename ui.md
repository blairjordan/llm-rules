# UI/UX Development Rules

## Overview & Philosophy

- Clarity, directness, and user-centered design principles
- Apple's Human Interface Guidelines as foundational inspiration
- Prioritize user understanding and intention over aesthetic complexity
- Consistent visual hierarchy and interaction patterns
- Accessibility-first approach ensuring inclusive design

## Structure & Organization

### Component Hierarchy

- Primary actions should have the most visual weight
- Secondary actions should have less visual prominence
- Clear separation between primary and secondary actions in forms and interfaces
- Consistent spacing and alignment patterns throughout the application

### Visual Design System

- Establish consistent color semantics across all components
- Maintain predictable interaction patterns
- Use visual cues to guide user attention and actions
- Design for multiple screen sizes and input methods

## Naming Conventions

### Component and Action Labels

- **Use Verb Phrases**: Label buttons with specific actions they perform
- **Be Concise**: Keep button labels short and clear
- **Be Specific**: The label should clearly communicate what will happen when clicked
- Examples: "Delete", "Save", "Create Account" instead of generic "Yes/No" or "OK/Cancel"

## Core Patterns & Best Practices

### Button Design Principles

#### Color Semantics

- **Primary Actions**: Use the primary color for main actions (continue, submit, save)
- **Destructive Actions**: Use red/danger color for destructive actions (delete, remove)
- **Secondary Actions**: Use neutral colors or flat/ghost variants for secondary actions (cancel, back)

#### Button Hierarchy

- Primary action buttons should have the most visual weight
- Secondary actions should have less visual prominence
- In forms, the primary action (e.g., Submit) should be visually distinct from the secondary action (e.g., Cancel)

### Modal and Confirmation Patterns

#### Destructive Action Confirmations

For destructive actions, use confirmation modals with:

- Clear heading indicating the action
- Brief explanation of consequences
- Default action should be the safe option (e.g., Cancel)
- Destructive action should be color-coded (red/danger)
- Position the safe action first, destructive action second

#### Modal Structure

- Clear, descriptive headings
- Concise but complete explanation of what will happen
- Obvious primary and secondary actions
- Easy escape routes (close button, backdrop click, ESC key)

### Form Design

#### Input Validation

- Real-time validation feedback where appropriate
- Clear error messaging that guides users toward solutions
- Visual indicators for required fields
- Progressive disclosure for complex forms

#### Form Actions

- Primary submission action clearly distinguished
- Secondary actions (cancel, back) less prominent but easily accessible
- Loading states for form submissions
- Success confirmation patterns

### Navigation Patterns

#### Information Architecture

- Clear navigation hierarchy
- Consistent navigation patterns across the application
- Breadcrumbs for deep navigation structures
- Clear indication of current location

#### Interactive Elements

- Consistent hover and active states
- Clear visual feedback for all interactive elements
- Accessible focus indicators
- Touch-friendly target sizes for mobile interfaces

## Error Handling & Feedback

### Error States

- Clear, actionable error messages
- Visual distinction between different types of errors
- Guidance toward resolution where possible
- Graceful fallbacks for system errors

### Loading and Progress Indicators

- Clear feedback during loading states
- Progress indicators for long-running operations
- Skeleton loading patterns for better perceived performance
- Timeout handling and retry mechanisms

### Success and Confirmation States

- Clear confirmation of successful actions
- Visual feedback that doesn't interrupt user flow
- Undo capabilities where appropriate
- Status indicators for ongoing processes
