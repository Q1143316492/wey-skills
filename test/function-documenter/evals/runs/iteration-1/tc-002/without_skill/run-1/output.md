```csharp
/// <summary>
/// Spawns an enemy at the specified position with the given properties.
/// </summary>
/// <param name="enemyType">The type of enemy to spawn.</param>
/// <param name="position">The world position where the enemy will be spawned.</param>
/// <param name="health">The initial health value for the enemy.</param>
/// <param name="isBoss">Whether the enemy is a boss enemy.</param>
public void SpawnEnemy(string enemyType, Vector3 position, int health, bool isBoss) {
    GameObject enemy = Instantiate(enemyPrefab, position, Quaternion.identity);
    enemy.GetComponent<Enemy>().enemyType = enemyType;
    enemy.GetComponent<Enemy>().health = health;
    enemy.GetComponent<Enemy>().isBoss = isBoss;
}
```
