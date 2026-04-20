# Chapter 10：预制体与对象池

> 🎯 目标：掌握预制体的进阶用法，理解对象池原理并实现，提升游戏性能。

---

## 10.1 预制体（Prefab）进阶

### Prefab 变体（Prefab Variant）

```
适用场景：
- 所有敌人有共同基础，但每种敌人有细微差异
- 创建 BaseEnemy.prefab（基础敌人）
- 创建 GoblinVariant.prefab（继承 BaseEnemy，增加哥布林特有属性）
```

### 嵌套 Prefab（Nested Prefab）

```csharp
// 在代码中实例化 Prefab
public class PrefabManager : MonoBehaviour
{
    public GameObject enemyPrefab;
    public Transform spawnPoint;

    public void SpawnEnemy()
    {
        // -------- 实例化 --------
        GameObject enemy = Instantiate(
            enemyPrefab,
            spawnPoint.position,
            spawnPoint.rotation
        );

        // -------- 设置父对象 --------
        enemy.transform.SetParent(spawnPoint);

        // -------- 设置同层级 --------
        enemy.transform.SetParent(null);   // 无父对象，顶级
    }
}
```

---

## 10.2 对象池（Object Pool）

### 为什么需要对象池？

| 方式 | 问题 |
|-----|------|
| Instantiate/Destroy | 大量对象时 GC 压力大，帧率抖动 |
| 对象池 | 预先创建，用时取，不用时回收复用 |

### 完整对象池实现

```csharp
using UnityEngine;
using System.Collections.Generic;

public class ObjectPool : MonoBehaviour
{
    // -------- 单例 --------
    public static ObjectPool instance;

    [System.Serializable]
    public class Pool
    {
        public string tag;          // 对象标签
        public GameObject prefab;   // 预制体
        public int size;            // 预创建数量
    }

    public List<Pool> pools;
    private Dictionary<string, Queue<GameObject>> poolDictionary;

    void Awake()
    {
        instance = this;
        poolDictionary = new Dictionary<string, Queue<GameObject>>();
    }

    void Start()
    {
        // -------- 预创建对象 --------
        foreach (Pool pool in pools)
        {
            Queue<GameObject> objectPool = new Queue<GameObject>();

            for (int i = 0; i < pool.size; i++)
            {
                GameObject obj = Instantiate(pool.prefab);
                obj.SetActive(false);
                obj.transform.SetParent(transform);   // 放在 Pool 管理器下
                objectPool.Enqueue(obj);
            }

            poolDictionary.Add(pool.tag, objectPool);
        }
    }

    // -------- 从池中取出对象 --------
    public GameObject Spawn(string tag, Vector3 position, Quaternion rotation)
    {
        if (!poolDictionary.ContainsKey(tag))
        {
            Debug.LogWarning($"Pool [{tag}] not found!");
            return null;
        }

        // -------- 取出一个对象 --------
        GameObject obj = poolDictionary[tag].Dequeue();
        obj.SetActive(true);
        obj.transform.position = position;
        obj.transform.rotation = rotation;

        // -------- 如果对象实现了 IPoolable 接口 --------
        IPoolable poolable = obj.GetComponent<IPoolable>();
        poolable?.OnSpawn();

        // -------- 用完归还到池 --------
        poolDictionary[tag].Enqueue(obj);

        return obj;
    }

    // -------- 归还到池 --------
    public void Despawn(GameObject obj)
    {
        obj.SetActive(false);
    }
}

// -------- 对象池接口 --------
public interface IPoolable
{
    void OnSpawn();    // 被取出时调用
    void OnDespawn();  // 被归还时调用
}
```

### 使用对象池

```csharp
public class Bullet : MonoBehaviour, IPoolable
{
    private Rigidbody rb;
    private float lifetime = 3f;
    private float spawnTime;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    public void OnSpawn()
    {
        spawnTime = Time.time;
        rb.velocity = Vector3.zero;
    }

    public void OnDespawn()
    {
        // 清理
    }

    void Update()
    {
        if (Time.time - spawnTime > lifetime)
        {
            ObjectPool.instance.Despawn(gameObject);
        }
    }

    public void Fire(Vector3 direction, float speed)
    {
        rb.velocity = direction * speed;
    }
}

public class Shooter : MonoBehaviour
{
    public void Shoot()
    {
        GameObject bullet = ObjectPool.instance.Spawn(
            "Bullet",
            firePoint.position,
            firePoint.rotation
        );

        bullet.GetComponent<Bullet>().Fire(firePoint.forward, 20f);
    }
}
```

---

## 10.3 章节小结

```
✅ 已掌握：
├── Prefab 变体与嵌套 Prefab
├── 对象池原理
├── 完整对象池实现
├── IPoolable 接口
└── 子弹对象池实战

🔜 下章预告：
第十一章：场景管理与存档系统 —— SceneManager、场景切换、UnityEngine.Experimental.SceneManagement。
```

---

_📚 参考资料：《Unity 对象池最佳实践》_
