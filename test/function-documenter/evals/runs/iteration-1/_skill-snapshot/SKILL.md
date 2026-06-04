---
name: function-documenter
description: Add documentation comments to functions that are missing them. Use when the user asks to document functions, add comments to code, or improve code documentation.
---

# Function Documenter

Automatically add clear and concise documentation comments to functions.

## When to use

- User asks to "add documentation" or "document this function"
- User wants to improve code readability
- Code review identifies missing function comments
- Preparing code for team sharing or public release

## Procedure

### 1. Identify functions to document

Read the file and identify all functions that lack documentation comments.

### 2. Analyze each function

For each undocumented function:
- Understand what the function does by reading its implementation
- Note the function's purpose and behavior

### 3. Write documentation

Add a comment block above each function that:
- Describes what the function does in 1-2 sentences
- Uses clear, simple language
- Starts with a verb (e.g., "Calculates...", "Returns...", "Updates...")

### 4. Apply changes

Use the file editing tools to add the documentation comments to the code.

## Documentation format

Use the appropriate comment style for the language:

**C# / C++ / JavaScript:**
```
// Brief description of what the function does.
```

**Python:**
```
"""Brief description of what the function does."""
```

## Example

Before:
```csharp
public int Add(int a, int b) {
    return a + b;
}
```

After:
```csharp
// Adds two integers and returns the result.
public int Add(int a, int b) {
    return a + b;
}
```

## Best practices

- Keep descriptions brief and focused
- Use present tense
- Avoid obvious statements
- Focus on the "what", not the "how"
