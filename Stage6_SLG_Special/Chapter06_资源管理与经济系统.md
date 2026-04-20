# Chapter 06：资源管理与经济系统

> 🎯 目标：掌握 SLG 完整经济系统——产出/消费/通货膨胀/抽卡/生产链。

---

## 6.1 资源产出循环

```
采集（农民/矿骥）
    ↓
资源 → 仓库（上限）
    ↓
消费（建造/训练/研究）
    ↓
积累 → 再次采集（良性循环）
    ↓
通货膨胀（产出 > 消费 → 贬值）
```

```csharp
// -------- 资源管理器 --------
[System.Serializable]
public class Resource
{
    public string name;
    public int current;   // 当前储量
    public int maxCapacity; // 仓库上限
    public int perMinute;   // 每分钟产量
}

public class EconomyManager : MonoBehaviour
{
    public Resource gold = new Resource { name = "金币", current = 100, maxCapacity = 10000, perMinute = 100 };
    public Resource wood = new Resource { name = "木材", current = 50, maxCapacity = 5000, perMinute = 50 };
    public Resource food = new Resource { name = "食物", current = 200, maxCapacity = 5000, perMinute = 80 };

    void Update() {
        AccumulateResources(Time.deltaTime);
    }

    void AccumulateResources(float deltaTime) {
        float ratio = deltaTime / 60f;

        gold.current = Mathf.Min(gold.current + (int)(gold.perMinute * ratio), gold.maxCapacity);
        wood.current = Mathf.Min(wood.current + (int)(wood.perMinute * ratio), wood.maxCapacity);
        food.current = Mathf.Min(food.current + (int)(food.perMinute * ratio), food.maxCapacity);
    }

    // -------- 消耗资源（返回是否成功）--------
    public bool Consume(string resName, int amount) {
        Resource res = GetResource(resName);
        if (res.current < amount) return false;
        res.current -= amount;
        return true;
    }

    Resource GetResource(string name) {
        switch (name) {
            case "金币": return gold;
            case "木材": return wood;
            case "食物": return food;
            default: return null;
        }
    }
}
```

---

## 6.2 通货膨胀控制

```csharp
// -------- 通货膨胀控制 --------
public class InflationControl
{
    // 产出上限
    public int GetMaxOutput(string resource) {
        float inflationRate = CalculateInflation();
        return (int)(BASE_OUTPUT * inflationRate);
    }

    float CalculateInflation() {
        // 资源越多 → 产出越低（通货膨胀）
        // goldPerDay / MAX_GOLD_PER_DAY 比例超过 0.8 → 开始通胀
        float ratio = totalDailyOutput / MAX_GOLD_PER_DAY;
        return ratio > 0.8f ? 1f - (ratio - 0.8f) : 1f;
    }
}
```

---

## 6.3 抽卡概率系统

```csharp
// -------- 抽卡 --------
public class GachaSystem
{
    [Header("奖池")]
    public int[] unitIDs;       // 抽中的 unitID
    public float[] probabilities; // 概率（必须加起来 = 1）
    public int pityCount = 90;  // 保底次数
    public int pityUnitID = 9999;  // 保底必出

    private int currentPulls = 0;  // 当前抽数

    public int Pull() {
        currentPulls++;

        int result;

        // -------- 保底 --------
        if (currentPulls >= pityCount) {
            result = pityUnitID;
            currentPulls = 0;
        } else {
            // -------- 随机抽取 --------
            float rand = Random.Range(0f, 1f);
            float cumulative = 0f;
            result = unitIDs[0];

            for (int i = 0; i < probabilities.Length; i++) {
                cumulative += probabilities[i];
                if (rand <= cumulative) {
                    result = unitIDs[i];
                    break;
                }
            }
        }

        // 记录抽卡日志（用于公示）
        RecordPull(result);
        return result;
    }
}
```

---

## 6.4 生产链

```csharp
// -------- 生产建筑 --------
[System.Serializable]
public class ProductionBuilding
{
    public string buildingName;
    public Resource input;       // 消耗什么
    public Resource output;       // 产出什么
    public int craftTime;        // 生产周期（秒）
    public int level = 1;
}

public class ProductionManager
{
    public List<ProductionBuilding> buildings = new List<ProductionBuilding>();

    public IEnumerator Produce(ProductionBuilding building) {
        // 1. 扣除原料
        if (!economy.Consume(building.input.name, building.input.current))
            yield break;

        // 2. 生产动画
        float elapsed = 0f;
        while (elapsed < building.craftTime) {
            float progress = elapsed / building.craftTime;
            // UpdateProgressUI(progress);
            yield return null;
            elapsed += Time.deltaTime;
        }

        // 3. 产出
        economy.AddResource(building.output.name, building.output.current);
        Debug.Log($"{building.buildingName} 生产完成，获得 {building.output.current} {building.output.name}");
    }
}
```

---

## 6.5 本章小结

```
✅ 已掌握：
├── 资源产出循环（采集/仓库/消费）
├── 通货膨胀控制（产出上限）
├── 抽卡概率系统（保底机制）
├── 生产链（前置建筑/产出）
└章 付费点设计（加速/直购/抽卡）
```

---

_📚 参考资料：《经济系统设计》《Gacha 概率设计》_
