```csharp
/// <summary>
/// Gets the current score value.
/// </summary>
/// <returns>The current score as an integer.</returns>
public int GetScore() {
    return score;
}

/// <summary>
/// Adds the specified number of points to the current score.
/// </summary>
/// <param name="points">The number of points to add to the score.</param>
public void AddScore(int points) {
    score += points;
}

/// <summary>
/// Resets the score to zero.
/// </summary>
public void ResetScore() {
    score = 0;
}
```
