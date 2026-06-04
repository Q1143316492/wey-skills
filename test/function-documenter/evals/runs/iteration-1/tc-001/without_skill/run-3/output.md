```csharp
/// <summary>
/// Calculates the Euclidean distance between two 2D points.
/// </summary>
/// <param name="a">The first point.</param>
/// <param name="b">The second point.</param>
/// <returns>The distance between points a and b.</returns>
public float CalculateDistance(Vector2 a, Vector2 b) {
    float dx = b.x - a.x;
    float dy = b.y - a.y;
    return Mathf.Sqrt(dx * dx + dy * dy);
}
```
