```csharp
/// <summary>
/// Spawns an enemy at the specified position with the given attributes.
/// </summary>
/// <param name="enemyType">The type of enemy to spawn (e.g., "Goblin", "Dragon").</param>
/// <param name="position">The world position where the enemy should be instantiated.</param>
/// <param name="health">The initial health value for the spawned enemy.</param>
/// <param name="isBoss">Whether this enemy is a boss enemy with special properties.</param>
public void SpawnEnemy(string enemyType, Vector3 position, int health, bool isBoss) {
    GameObject enemy = Instantiate(enemyPrefab, position, Quaternion.identity);
    enemy.GetComponent<Enemy>().enemyType = enemyType;
    enemy.GetComponent<Enemy>().health = health;
    enemy.GetComponent<Enemy>().isBoss = isBoss;
}
```
