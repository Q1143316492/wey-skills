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
- Identify all parameters and their roles
- Note the return value (if any) and what it represents

### 3. Write documentation

Add a comment block above each function that includes:

**Summary (required):**
- Describes what the function does in 1 sentence
- Uses clear, simple language
- Starts with a verb (e.g., "Calculates...", "Returns...", "Updates...")

**Parameters (if any):**
- Document each parameter's purpose
- Use format: `paramName - Brief description`

**Returns (if non-void):**
- Explain what the return value represents
- Use format: `Returns: Description`

### 4. Apply changes

Use the file editing tools to add the documentation comments to the code.

## Documentation format

Use the appropriate comment style for the language:

**C# / C++ / JavaScript:**
```
// Brief description of what the function does.
// param1 - Description of first parameter
// param2 - Description of second parameter
// Returns: Description of return value
```

**Python:**
```
"""
Brief description of what the function does.
Args:
    param1: Description of first parameter
    param2: Description of second parameter
Returns:
    Description of return value
"""
```

## Examples

### Simple function with no parameters
Before:
```csharp
public int GetScore() {
    return score;
}
```

After:
```csharp
// Returns the current player score.
// Returns: The current score value
public int GetScore() {
    return score;
}
```

### Function with parameters
Before:
```csharp
public void SpawnEnemy(string enemyType, Vector3 position, int health) {
    // implementation...
}
```

After:
```csharp
// Spawns an enemy at the specified position with given properties.
// enemyType - The type of enemy to spawn
// position - World position for the enemy
// health - Initial health value
public void SpawnEnemy(string enemyType, Vector3 position, int health) {
    // implementation...
}
```

### Function returning a calculated value
Before:
```csharp
public float CalculateDistance(Vector2 a, Vector2 b) {
    return Mathf.Sqrt((b.x - a.x) * (b.x - a.x) + (b.y - a.y) * (b.y - a.y));
}
```

After:
```csharp
// Calculates the Euclidean distance between two 2D points.
// a - First point
// b - Second point
// Returns: Distance between the two points
public float CalculateDistance(Vector2 a, Vector2 b) {
    return Mathf.Sqrt((b.x - a.x) * (b.x - a.x) + (b.y - a.y) * (b.y - a.y));
}
```

## Best practices

- Keep descriptions brief and focused — avoid lengthy explanations
- Use present tense
- Avoid obvious statements
- Focus on the "what", not the "how"
- For parameters, explain purpose/role, not data type (the signature shows that)
- For return values, explain meaning, not just "returns an int"
