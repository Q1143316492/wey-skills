```csharp
// Spawns an enemy at the specified position with given properties.
// enemyType - The type of enemy to spawn
// position - World position for the enemy
// health - Initial health value
// isBoss - Whether this enemy is a boss enemy
public void SpawnEnemy(string enemyType, Vector3 position, int health, bool isBoss) {
    GameObject enemy = Instantiate(enemyPrefab, position, Quaternion.identity);
    enemy.GetComponent<Enemy>().enemyType = enemyType;
    enemy.GetComponent<Enemy>().health = health;
    enemy.GetComponent<Enemy>().isBoss = isBoss;
}
```
