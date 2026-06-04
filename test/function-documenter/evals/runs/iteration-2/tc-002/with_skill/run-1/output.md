```csharp
// Spawns an enemy at the specified position with given properties.
// enemyType - The type of enemy to spawn
// position - World position where the enemy will be spawned
// health - Initial health value for the enemy
// isBoss - Whether the enemy is a boss
public void SpawnEnemy(string enemyType, Vector3 position, int health, bool isBoss) {
    GameObject enemy = Instantiate(enemyPrefab, position, Quaternion.identity);
    enemy.GetComponent<Enemy>().enemyType = enemyType;
    enemy.GetComponent<Enemy>().health = health;
    enemy.GetComponent<Enemy>().isBoss = isBoss;
}
```
