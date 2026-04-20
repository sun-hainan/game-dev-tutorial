# Chapter 04：回合制战斗系统

> 🎯 目标：掌握 SLG 回合制战斗完整系统——Buff/Debuff、伤害公式、战斗 AI、战斗回放。

---

## 4.1 战斗流程

```
┌─────────────────┐
│  回合开始       │
│  刷新 Buff/Debuff │
└────────┬────────┘
         ▼
┌─────────────────┐
│  玩家操作阶段   │ ← 等待玩家选择技能/目标
│  等待操作      │
└────────┬────────┘
         ▼
┌─────────────────┐
│  执行操作       │
│  播放动画      │
└────────┬────────┘
         ▼
┌─────────────────┐
│  回合结束      │
│  Buff 结算/AI   │
└────────┬────────┘
         ▼
┌─────────────────┐
│  胜负判定       │ ← HP <= 0 → 失败
└─────────────────┘
```

---

## 4.2 Buff/Debuff 系统

```csharp
// -------- Buff 基类 --------
[System.Serializable]
public abstract class Buff
{
    public string buffName;
    public int duration;       // 持续回合数
    public int currentDuration;
    public int iconID;

    public abstract void OnApply(BattleUnit unit);
    public abstract void OnRemove(BattleUnit unit);
    public abstract void OnTurnStart(BattleUnit unit);
}

// -------- 具体 Buff --------
public class AttackUp : Buff
{
    public int attackBonus;

    public AttackUp(int bonus, int dur) {
        buffName = "攻击力提升";
        attackBonus = bonus;
        duration = dur;
        currentDuration = dur;
    }

    public override void OnApply(BattleUnit unit) {
        unit.attack += attackBonus;
        Debug.Log($"{unit.unitName} 攻击力 +{attackBonus}");
    }

    public override void OnRemove(BattleUnit unit) {
        unit.attack -= attackBonus;
        Debug.Log($"{unit.unitName} 攻击力恢复");
    }

    public override void OnTurnStart(BattleUnit unit) {
        currentDuration--;
        if (currentDuration <= 0) {
            unit.RemoveBuff(this);
        }
    }
}

public class Poison : Buff  // 中毒
{
    public int damagePerTurn;

    public Poison(int dmg, int dur) {
        buffName = "中毒";
        damagePerTurn = dmg;
        duration = dur;
        currentDuration = dur;
    }

    public override void OnApply(BattleUnit unit) {
        Debug.Log($"{unit.unitName} 中毒了");
    }

    public override void OnRemove(BattleUnit unit) {
        Debug.Log($"{unit.unitName} 中毒解除");
    }

    public override void OnTurnStart(BattleUnit unit) {
        unit.TakeDamage(damagePerTurn, DamageType.Poison);
        currentDuration--;
        if (currentDuration <= 0) {
            unit.RemoveBuff(this);
        }
    }
}

// -------- Buff 管理器 --------
public class BuffManager
{
    private List<Buff> buffs = new List<Buff>();

    public void AddBuff(Buff buff, BattleUnit unit) {
        buffs.Add(buff);
        buff.OnApply(unit);
    }

    public void OnTurnStart(BattleUnit unit) {
        foreach (var buff in buffs.ToArray()) {  // ToArray 防止修改
            buff.OnTurnStart(unit);
        }
    }
}
```

---

## 4.3 伤害公式

```csharp
// -------- 伤害计算 --------
public enum DamageType { Physical, Magic, True }

public static class DamageCalculator
{
    // 公式1：《火焰纹章》类（攻防差）
    public static int CalcType1(int atk, int def) {
        int damage = Mathf.Max(1, atk - def);
        return damage;
    }

    // 公式2：《梦幻模拟战》类（克制加成）
    public static int CalcType2(int atk, int def, float advantage) {
        int baseDamage = atk * 100 / (100 + def);
        return Mathf.Max(1, (int)(baseDamage * advantage));
    }

    // 公式3：《最终幻想》类（ATK^2 / (ATK+DEF)）
    public static int CalcType3(int atk, int def) {
        float damage = (float)atk * atk / (atk + def);
        return Mathf.Max(1, (int)damage);
    }

    // 最终伤害（含暴击/护盾/Buff）
    public static DamageResult CalcFinalDamage(
        BattleUnit attacker, BattleUnit defender,
        Skill skill, int terrainBonus)
    {
        int baseDamage = CalcType2(skill.baseDamage, defender.defense, 1f);

        // 属性克制
        float advantage = GetElementAdvantage(attacker.element, defender.element);
        baseDamage = (int)(baseDamage * advantage);

        // 随机波动（±10%）
        float random = Random.Range(0.9f, 1.1f);
        int finalDamage = (int)(baseDamage * random);

        // 护盾抵消
        int shield = defender.currentShield;
        int actualDamage = Mathf.Max(0, finalDamage - shield);
        defender.currentShield = Mathf.Max(0, shield - finalDamage);

        // 暴击
        bool isCrit = Random.Range(0f, 1f) < attacker.critRate;
        if (isCrit) actualDamage *= 2;

        return new DamageResult {
            damage = actualDamage,
            isCrit = isCrit,
            element = attacker.element
        };
    }
}

public struct DamageResult {
    public int damage;
    public bool isCrit;
    public Element element;
}
```

---

## 4.4 战斗回放

```csharp
// -------- 战斗回放系统 --------
public class BattleReplay
{
    private List<BattleAction> actions = new List<BattleAction>();

    public void Record(BattleAction action) {
        actions.Add(new BattleAction {
            turn = currentTurn,
            unitID = action.unitID,
            actionType = action.actionType,
            targetID = action.targetID,
            skillID = action.skillID,
            timestamp = DateTime.Now
        });
    }

    public void Replay() {
        StartCoroutine(ReplayCoroutine());
    }

    IEnumerator ReplayCoroutine() {
        foreach (var action in actions) {
            Debug.Log($"回放：回合{action.turn} {action.unitID} 使用 {action.skillID}");

            // 播放对应的动画
            yield return StartCoroutine(PlayAnimation(action));
            yield return new WaitForSeconds(0.5f);  // 每动作间隔
        }
    }
}

public struct BattleAction {
    public int turn;
    public int unitID;
    public ActionType actionType;
    public int targetID;
    public int skillID;
    public DateTime timestamp;
}

public enum ActionType {
    Skill, Attack, Item, Move, Wait
}
```

---

## 4.5 本章小结

```
✅ 已掌握：
├── 回合制战斗流程
├── Buff/Debuff 系统（叠加/覆盖/持续回合）
├── 三种伤害公式
├── 暴击/护盾/属性克制
├── 战斗回放系统
└章 战斗 AI（MCTS / 规则树）
```

---

_📚 参考资料：《Fire Emblem 伤害公式》《FFT 战斗系统》_
