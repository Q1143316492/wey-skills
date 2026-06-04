```csharp
// Calculates the Euclidean distance between two Vector2 points.
public float CalculateDistance(Vector2 a, Vector2 b) {
    float dx = b.x - a.x;
    float dy = b.y - a.y;
    return Mathf.Sqrt(dx * dx + dy * dy);
}
```
