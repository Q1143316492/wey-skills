```csharp
// Calculates the Euclidean distance between two 2D points.
// a - First point
// b - Second point
// Returns: Distance between the two points
public float CalculateDistance(Vector2 a, Vector2 b) {
    float dx = b.x - a.x;
    float dy = b.y - a.y;
    return Mathf.Sqrt(dx * dx + dy * dy);
}
```
