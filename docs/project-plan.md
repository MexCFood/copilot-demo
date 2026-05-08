# Task Manager CLI Application Project Plan

## Project Overview

The Task Manager CLI is a command-line application that allows users to manage their tasks efficiently. Users can create, list, update, and delete tasks with properties including title, description, status, and priority. The application supports filtering and sorting operations to help users organize and track their work. All data is stored in memory during the session, providing a simple and fast interface without database dependencies.

## User Stories

1. **Create a Task**
   - As a user, I want to create a new task with a title, description, priority, and initial status
   - Acceptance Criteria:
     - Task is added to the task list with a unique ID
     - createdAt and updatedAt timestamps are automatically set
     - Task status defaults to "todo" if not specified
     - Task priority defaults to "medium" if not specified

2. **List All Tasks**
   - As a user, I want to view all tasks with their details
   - Acceptance Criteria:
     - All tasks are displayed in a readable format
     - Each task shows ID, title, description, status, priority, and timestamps
     - Output is formatted as a table or JSON structure

3. **View Task by ID**
   - As a user, I want to view a single task with its complete details
   - Acceptance Criteria:
     - Task can be retrieved by its unique ID
     - Full task details are displayed
     - An error message is shown if the task ID does not exist

4. **Update a Task**
   - As a user, I want to update a task's title, description, status, or priority
   - Acceptance Criteria:
     - Selected task properties can be modified
     - updatedAt timestamp is refreshed when changes are made
     - An error message is shown if the task ID does not exist

5. **Delete a Task**
   - As a user, I want to remove a task from the list
   - Acceptance Criteria:
     - Task is removed permanently
     - An error message is shown if the task ID does not exist

6. **Filter Tasks by Status**
   - As a user, I want to filter and display tasks by their status (todo, in-progress, or done)
   - Acceptance Criteria:
     - Only tasks matching the selected status are displayed
     - Works in combination with listing operations

7. **Filter Tasks by Priority**
   - As a user, I want to filter and display tasks by their priority (low, medium, or high)
   - Acceptance Criteria:
     - Only tasks matching the selected priority are displayed
     - Works in combination with listing operations

8. **Sort Tasks**
   - As a user, I want to sort tasks by priority or creation date
   - Acceptance Criteria:
     - Tasks can be sorted by priority (high → medium → low)
     - Tasks can be sorted by creation date (newest or oldest first)
     - Sorting can be applied after filtering

## Data Model

### Task Entity

| Property  | Type   | Description                          |
|-----------|--------|--------------------------------------|
| id        | string | Unique identifier for the task       |
| title     | string | Task title (required)                |
| description | string | Detailed task description (optional) |
| status    | enum   | "todo", "in-progress", or "done"    |
| priority  | enum   | "low", "medium", or "high"          |
| createdAt | date   | Task creation timestamp             |
| updatedAt | date   | Last modification timestamp         |

### Status Values
- `todo` - Task not yet started
- `in-progress` - Task currently being worked on
- `done` - Task completed

### Priority Values
- `low` - Low priority task
- `medium` - Medium priority task (default)
- `high` - High priority task

## Error Handling Conventions and Input Validation

### Input Validation Rules

#### Task Title
- **Required**: Title must be provided and cannot be empty
- **Validation**: Must be a non-empty string with 1-200 characters
- **Error Message**: "Error: Task title is required and must be 1-200 characters"

#### Task Description
- **Optional**: Description can be omitted or empty
- **Validation**: If provided, must be a string with 0-1000 characters
- **Error Message**: "Error: Task description must not exceed 1000 characters"

#### Task Status
- **Validation**: Must be one of: "todo", "in-progress", "done"
- **Default**: "todo" if not specified during creation
- **Error Message**: "Error: Invalid status. Must be one of: todo, in-progress, done"

#### Task Priority
- **Validation**: Must be one of: "low", "medium", "high"
- **Default**: "medium" if not specified during creation
- **Error Message**: "Error: Invalid priority. Must be one of: low, medium, high"

#### Task ID
- **Validation**: Must exist in the task collection for get, update, and delete operations
- **Format**: Unique string identifier (generated automatically or provided)
- **Error Message**: "Error: Task with ID '{id}' not found"

#### Filter and Sort Parameters
- **Status Filter**: Must be a valid status value or null for no filter
- **Priority Filter**: Must be a valid priority value or null for no filter
- **Sort Field**: Must be "priority" or "createdAt"
- **Sort Order**: Must be "asc" (ascending) or "desc" (descending)
- **Error Message**: "Error: Invalid filter/sort parameter: {parameter}"

### Error Handling Conventions

#### Error Classes
- Create custom error types (e.g., `ValidationError`, `NotFoundError`) for specific error scenarios
- Include error codes or types for programmatic error handling
- Provide descriptive error messages suitable for end users

#### Error Response Format
- All errors should follow a consistent format: `"Error: [message]"`
- Include context when relevant (e.g., task ID, invalid value)
- Avoid exposing internal implementation details

#### Validation Strategy
- Validate input at the CLI entry point (index.js) before delegating to TaskManager
- Validate again at the TaskManager level to ensure data integrity
- Use defensive programming: check for null/undefined values before processing
- Normalize input (trim whitespace, convert case) where applicable

#### Error Recovery
- Provide helpful suggestions in error messages (e.g., suggest valid enum values)
- Do not modify data if validation fails
- Display usage examples for command errors
- Exit with appropriate status codes (0 for success, non-zero for errors)

#### Specific Error Scenarios
- **Duplicate Operations**: Silently succeed if user attempts to create identical tasks (idempotent)
- **Empty Task List**: Display "No tasks found" for list operations when collection is empty
- **Invalid Arguments**: Show command usage and valid argument options
- **Missing Required Fields**: Specify which field is missing in the error message
- **Type Mismatches**: Convert where possible (e.g., string to number), reject if conversion fails

## File Structure

```
src/
├── index.js           # Main CLI entry point and command dispatcher
├── task.js            # Task class definition
├── taskManager.js     # TaskManager class for business logic
└── formatter.js       # Output formatting utilities
```

### File Responsibilities

- **index.js**: Parses command-line arguments, routes to appropriate handlers, and displays results
- **task.js**: Defines the Task class with properties and methods
- **taskManager.js**: Manages the in-memory task collection with CRUD operations, filtering, and sorting
- **formatter.js**: Formats task data for console output (tables, JSON, error messages)

## Implementation Phases

### Phase 1: Core Data Structure and CRUD Operations
- Implement Task class with required properties
- Implement TaskManager class with create, read, update, delete methods
- Add automatic timestamp management
- Add unique ID generation for tasks

### Phase 2: CLI Interface and Command Parsing
- Create main CLI entry point
- Implement command parsing (add, list, get, update, delete)
- Add basic argument validation
- Implement help/usage documentation

### Phase 3: Filtering and Sorting
- Add filter methods to TaskManager (by status, by priority)
- Add sort methods to TaskManager (by priority, by creation date)
- Integrate filtering and sorting into CLI commands
- Support combining filters and sorts

### Phase 4: Output Formatting and User Experience
- Create formatter utilities for readable output
- Implement table-based display for task lists
- Add JSON output option for programmatic use
- Add colored output for status and priority distinction
- Implement helpful error messages

### Phase 5: Testing and Documentation
- Test each command with various inputs
- Document CLI usage with examples
- Test edge cases (empty task list, invalid IDs, invalid status/priority values)
- Add inline code documentation
