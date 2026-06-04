```csharp
// Returns the current player score.
// Returns: The current score value
public int GetScore() {
    return score;
}

// Adds points to the player's current score.
// points - Number of points to add to the score
public void AddScore(int points) {
    score += points;
}

// Resets the player's score to zero.
public void ResetScore() {
    score = 0;
}
```
