# Chapter 01：SLG 游戏类型与核心机制

> 🎯 目标：了解 SLG 游戏的全貌——从 4X 策略到即时战术，从回合制到塔防，搞清楚 SLG 的分类体系和核心机制设计。

---

## 1.1 SLG 游戏是什么？

### 生活中的类比 🏰

> **SLG = 经营 + 打仗。**
> 就像你同时是市长和将军：既要规划城市（资源管理），又要指挥部队（战斗决策）。

### SLG 全称

> **Simulation Strategy / Strategy Game** — 策略游戏/模拟策略游戏

### SLG 子类型一览

| 类型 | 代表游戏 | 核心特点 |
|-----|---------|---------|
| **4X 策略** | 《文明》《无尽帝国》 | 探索(eXplore)、扩张(eXpand)、开发(eXploit)、消灭(eXterminate) |
| **RTS 即时战略** | 《星际争霸》《红警》 | 实时操控多个单位，兼顾经济+军事 |
| **回合制策略** | 《文明》《高级战争》 | 每回合决策，无时间压力 |
| **塔防 TD** | 《植物大战僵尸》 | 建造防御，阻止敌人路径 |
| **Auto Battler** | 《云顶之弈》《刀塔霸业》 | 放置单位自动战斗，策略在于布阵 |
| **战术 RPG** | 《火焰纹章》《高级战争》 | 回合制+角色扮演，格子移动 |
| **RTT 实时战术** | 《盟军敢死队》《影子战术》 | 实时操作小队，非正面作战 |

---

## 1.2 核心机制拆解

### SLG 通用核心机制

```
SLG 核心 = 资源系统 + 单位系统 + 地图系统 + 战斗系统
            ↓             ↓            ↓           ↓
         采集/生产      建造/升级     地形/移动    伤害/胜负
```

### 资源系统

```csharp
// -------- SLG 资源系统 --------
public class ResourceManager : MonoBehaviour
{
    // -------- 资源类型 --------
    public enum ResourceType
    {
        Gold,      // 金币（通用）
        Wood,       // 木材（建筑）
        Stone,      // 石料（防御）
        Food,       // 食物（维持人口）
        Iron,       // 铁矿（高级单位）
        GoldCoined  // 点券（付费）
    }

    // -------- 资源数据 --------
    [System.Serializable]
    public class ResourceData
    {
        public ResourceType type;
        public int currentAmount;   // 当前储量
        public int maxCapacity;      // 仓库上限
        public float incomeRate;     // 每分钟产量
        public float expenseRate;    // 每分钟消耗
    }

    private Dictionary<ResourceType, ResourceData> resources = new Dictionary<ResourceType, ResourceData>();

    // -------- 更新资源 --------
    public void UpdateResources(float deltaTime)
    {
        foreach (var kvp in resources)
        {
            int change = (int)(kvp.Value.incomeRate * deltaTime / 60f);
            change -= (int)(kvp.Value.expenseRate * deltaTime / 60f);

            kvp.Value.currentAmount = Mathf.Clamp(
                kvp.Value.currentAmount + change,
                0,
                kvp.Value.maxCapacity
            );
        }
    }

    // -------- 消耗资源（返回是否成功）--------
    public bool Consume(ResourceType type, int amount)
    {
        if (!resources.ContainsKey(type)) return false;
        if (resources[type].currentAmount < amount) return false;

        resources[type].currentAmount -= amount;
        return true;
    }

    // -------- 获取资源 --------
    public int GetAmount(ResourceType type)
    {
        return resources.ContainsKey(type) ? resources[type].currentAmount : 0;
    }
}
```

### 单位系统

```csharp
// -------- SLG 单位基类 --------
public abstract class SLGUnit : MonoBehaviour
{
    public string unitName;
    public int level = 1;

    [Header("属性")]
    public int maxHP;
    public int currentHP;
    public int attack;
    public int defense;
    public float moveSpeed;        // 格/秒
    public float attackRange;       // 攻击范围（格）
    public float attackInterval;   // 攻击间隔（秒）
    public int attackCount = 1;     // 一次攻击次数

    [Header("消耗")]
    public int foodConsume;         // 人口消耗
    public int goldCost;           // 建造费用

    [Header("移动")]
    public GridPosition gridPosition;  // 当前格子位置
    public GridPosition[] path;        // 移动路径
    private bool isMoving = false;

    // -------- 寻路 --------
    public virtual void MoveTo(GridPosition target)
    {
        path = AStarPathfinding.FindPath(gridPosition, target);
        if (path != null && path.Length > 0)
        {
            isMoving = true;
            StartCoroutine(MoveAlongPath());
        }
    }

    // -------- 沿路径移动 --------
    IEnumerator MoveAlongPath()
    {
        foreach (GridPosition waypoint in path)
        {
            Vector3 worldPos = GridManager.Instance.GridToWorld(waypoint);
            while (Vector3.Distance(transform.position, worldPos) > 0.1f)
            {
                transform.position = Vector3.Lerp(
                    transform.position, worldPos, moveSpeed * Time.deltaTime * 5f);
                yield return null;
            }
            gridPosition = waypoint;
        }
        isMoving = false;
    }

    // -------- 攻击 --------
    public virtual void Attack(SLGUnit target)
    {
        if (isMoving) return;
        if (Vector3.Distance(transform.position, target.transform.position) > attackRange)
            return;

        // 伤害计算
        int damage = Mathf.Max(1, attack - target.defense / 2);
        target.TakeDamage(damage);
    }

    public virtual void TakeDamage(int damage)
    {
        currentHP -= damage;
        if (currentHP <= 0)
        {
            Die();
        }
    }

    protected virtual void Die()
    {
        // 播放死亡动画，移除单位
        Destroy(gameObject);
    }
}
```

---

## 1.3 回合制 vs 即时制

### 回合制特点

| 维度 | 回合制 | 即时制 |
|-----|--------|--------|
| 决策时间 | 充裕，可暂停思考 | 实时，快速决策 |
| 单位控制 | 逐个或批量选择 | 实时同时控制 |
| 策略深度 | ✅ 更深 | ⚠️ 较浅（操作压力大）|
| 社交 | 适合异步 | 实时同步 |
| 代表 | 文明、火焰纹章 | 星际、红警 |

### 回合制实现

```csharp
// -------- 回合制管理器 --------
public class TurnBasedManager : MonoBehaviour
{
    public enum Phase
    {
        PlayerTurn,
        EnemyTurn,
        Animating,
        GameOver
    }

    public Phase currentPhase = Phase.PlayerTurn;
    private int currentTurn = 1;

    public void StartPlayerTurn()
    {
        currentPhase = Phase.PlayerTurn;
        UIManager.Instance.ShowTurnInfo($"第 {currentTurn} 回合 - 你的回合");

        // 允许玩家操作
        foreach (var unit in playerUnits)
        {
            unit.EnableAction();
        }
    }

    public void EndPlayerTurn()
    {
        // 禁用玩家操作
        foreach (var unit in playerUnits)
        {
            unit.DisableAction();
        }

        currentPhase = Phase.EnemyTurn;
        StartCoroutine(RunEnemyTurn());
    }

    IEnumerator RunEnemyTurn()
    {
        currentPhase = Phase.Animating;
        UIManager.Instance.ShowTurnInfo("敌方回合...");

        // -------- AI 决策 --------
        yield return new WaitForSeconds(0.5f);

        foreach (var enemy in enemyUnits)
        {
            yield return StartCoroutine(enemy.TakeTurn());
            yield return new WaitForSeconds(0.3f);
        }

        // -------- 进入下一回合 --------
        currentTurn++;
        StartPlayerTurn();
    }
}
```

---

## 1.4 地图系统

### 格子地图

```csharp
// -------- 格子管理器 --------
public class GridManager : MonoBehaviour
{
    public static GridManager Instance;

    [Header("地图配置")]
    public int width = 20;
    public int height = 15;
    public float cellSize = 1f;

    private GridCell[,] grid;

    void Awake()
    {
        Instance = this;
        InitializeGrid();
    }

    void InitializeGrid()
    {
        grid = new GridCell[width, height];

        for (int x = 0; x < width; x++)
        {
            for (int y = 0; y < height; y++)
            {
                grid[x, y] = new GridCell
                {
                    x = x,
                    y = y,
                    terrainType = TerrainType.Plain,
                    isWalkable = true,
                    defenseBonus = 0
                };
            }
        }
    }

    public GridCell GetCell(int x, int y)
    {
        if (x < 0 || x >= width || y < 0 || y >= height) return null;
        return grid[x, y];
    }

    public Vector3 GridToWorld(GridPosition gridPos)
    {
        return new Vector3(gridPos.x * cellSize, 0, gridPos.y * cellSize);
    }

    public GridPosition WorldToGrid(Vector3 worldPos)
    {
        return new GridPosition(
            Mathf.RoundToInt(worldPos.x / cellSize),
            Mathf.RoundToInt(worldPos.z / cellSize)
        );
    }
}

// -------- 格子数据 --------
[System.Serializable]
public class GridCell
{
    public int x, y;
    public TerrainType terrainType;
    public bool isWalkable;
    public int defenseBonus;      // 防御加成（地形）
    public float moveCost;        // 移动消耗
    public GameObject occupant;    // 格子上的单位
    public GameObject structure;  // 格子上的建筑
}

public enum TerrainType
{
    Plain,    // 平原（普通）
    Forest,    // 森林（掩护+防御）
    Mountain,  // 山地（高防御，不可通行）
    Water,    // 水域（部分单位不可通行）
    Road,     // 道路（移动消耗减半）
    Desert    // 沙漠（移动消耗加倍）
}
```

---

## 1.5 建造与科技树

### 建筑系统

```csharp
// -------- 建筑基类 --------
public abstract class Structure : MonoBehaviour
{
    public string structureName;
    public int level = 1;
    public float buildTime = 10f;   // 建造时间
    public bool isBuilt = false;

    protected GridPosition gridPosition;

    public abstract void OnBuilt();
    public abstract void OnDestroyed();

    public IEnumerator Build(GridPosition pos)
    {
        gridPosition = pos;
        float elapsed = 0f;

        while (elapsed < buildTime)
        {
            // 显示建造进度
            UIManager.Instance.ShowBuildProgress(elapsed / buildTime);
            elapsed += Time.deltaTime;
            yield return null;
        }

        isBuilt = true;
        OnBuilt();
        GridManager.Instance.GetCell(pos.x, pos.y).structure = gameObject;
    }
}

// -------- 具体建筑 --------
public class Barracks : Structure  // 兵营
{
    public GameObject[] trainableUnits;  // 可训练的兵种

    public override void OnBuilt()
    {
        Debug.Log("兵营建造完成，可以训练单位");
    }

    public override void OnDestroyed()
    {
        Debug.Log("兵营被摧毁，无法训练兵种");
    }
}
```

### 科技树

```csharp
// -------- 科技节点 --------
[System.Serializable]
public class TechNode
{
    public string techID;
    public string techName;
    public string description;
    public int researchTime;       // 研究时间（秒）

    public string[] prerequisiteTechs;  // 前置科技
    public ResourceType[] cost;         // 花费
    public int[] costAmounts;

    public enum TechStatus { Locked, Researching, Completed }
    public TechStatus status = TechStatus.Locked;

    public bool CanResearch()
    {
        // 检查前置科技是否完成
        foreach (string prereq in prerequisiteTechs)
        {
            if (!TechTree.Instance.IsCompleted(prereq))
                return false;
        }
        return true;
    }
}

// -------- 科技树 --------
public class TechTree : MonoBehaviour
{
    public static TechTree Instance;

    private Dictionary<string, TechNode> techs = new Dictionary<string, TechNode>();

    public bool IsCompleted(string techID)
    {
        return techs.ContainsKey(techID) && techs[techID].status == TechNode.TechStatus.Completed;
    }

    public void StartResearch(string techID)
    {
        if (!techs.ContainsKey(techID)) return;
        var tech = techs[techID];

        if (!tech.CanResearch()) return;

        tech.status = TechNode.TechStatus.Researching;
        StartCoroutine(ResearchTech(tech));
    }

    IEnumerator ResearchTech(TechNode tech)
    {
        float elapsed = 0f;
        while (elapsed < tech.researchTime)
        {
            elapsed += Time.deltaTime;
            yield return null;
        }

        tech.status = TechNode.TechStatus.Completed;
        OnTechCompleted(tech);
    }

    void OnTechCompleted(TechNode tech)
    {
        Debug.Log($"科技解锁：{tech.techName}");

        // 解锁对应单位/建筑/技能
        // 通知 UI 更新
    }
}
```

---

## 1.6 SLG 游戏类型代表

### 4X 策略游戏（《文明》系列）

```
核心循环：
1. 探索：地图探索，开图
2. 扩张：殖民新城市
3. 开发：建设城市，发展科技
4. 消灭：军事征服/外交胜利/科技胜利
```

### RTS 即时战略（《星际争霸》系列）

```
核心循环：
1. 运营：采集资源（农民/矿骥）
2. 建造：生产建筑/单位
3. 战术：控制部队
4. 攻防节奏：进攻/防守/骚扰
```

### Auto Battler（《云顶之弈》）

```
核心循环：
1. 布阵：放置单位（8人桌）
2. 回合自动战斗
3. 装备系统：给单位分配装备
4. 经济管理：存钱吃利息
5. 升级：提升等级解锁槽位
```

### 塔防 TD（《植物大战僵尸》）

```
核心循环：
1. 收集阳光
2. 放置防御单位
3. 敌人从路径进攻
4. 消灭敌人获得奖励
```

---

## 1.7 本章小结

```
✅ 已掌握：
├── SLG 定义与子类型（4X/RTS/回合制/塔防/Auto Battler/战术RPG）
├── 资源系统设计（多种资源类型/产量/消耗）
├── 单位系统基类（属性/移动/攻击/寻路）
├── 回合制实现（阶段管理/玩家/敌方回合）
├── 格子地图系统（GridManager/地形/格子数据）
├── 建造系统（Structure 基类/兵营）
├── 科技树系统（前置依赖/研究时间/解锁效果）
└章 SLG 各大代表游戏的核心机制解析

🔜 下章预告：
第二章：SLG 地图系统与寻路 —— 深入格子地图的生成、地形属性、移动消耗、A* 寻路。
```

---

_📚 参考资料：《游戏设计艺术》《Civilization 系列设计分析》《Game Programming Patterns》_
