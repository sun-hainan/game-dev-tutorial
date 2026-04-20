# Chapter 12：STL 算法

> 🎯 目标：掌握 `<algorithm>` 中最常用的算法——查找、排序、统计、变换、删除。配合 Lambda 让算法灵活百变。

---

## 12.1 算法概述

### 生活中的类比 🔍

> **算法 = 图书馆的检索系统：**
> - 你告诉管理员"找《C++ Primer》"
> - 管理员帮你**查找**（find）
> - 你说"把书架上的书按书名**排序**"（sort）
> - 你说"**统计**有多少本超过 500 页"（count_if）
>
> **STL 算法 = 已写好的标准检索函数，拿来就用。**

### 头文件

```cpp
#include <algorithm>   // 大部分算法
#include <numeric>     // 数值算法（accumulate, iota）
```

### 分类

| 类别 | 常用算法 |
|-----|---------|
| 查找 | `find`, `find_if`, `binary_search` |
| 排序 | `sort`, `stable_sort`, `reverse` |
| 统计 | `count`, `count_if`, `accumulate` |
| 变换 | `transform`, `copy`, `replace` |
| 删除 | `remove`, `remove_if`, `unique` |
| 最值 | `min_element`, `max_element` |

---

## 12.2 查找算法

### find：查找单个元素

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

int main()
{
    vector<int> scores = {85, 92, 78, 95, 88};

    // -------- find：找第一个匹配的元素 --------
    auto it = find(scores.begin(), scores.end(), 92);

    if (it != scores.end()) {
        cout << "找到了！位置索引：" << (it - scores.begin()) << endl;
        cout << "值：" << *it << endl;
    } else {
        cout << "未找到" << endl;
    }

    return 0;
}
```

### find_if：按条件查找

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

// -------- 查找条件：HP > 150 --------
struct Enemy {
    string name;
    int hp;
};

int main()
{
    vector<Enemy> enemies = {
        {"哥布林", 100},
        {"骷髅", 80},
        {"巨石魔像", 200},
        {"幽灵", 50}
    };

    // -------- 找第一个 HP > 150 的敌人 --------
    auto it = find_if(enemies.begin(), enemies.end(),
        [](const Enemy& e) {
            return e.hp > 150;
        });

    if (it != enemies.end()) {
        cout << "找到强力敌人：" << it->name << "，HP=" << it->hp << endl;
    }

    return 0;
}
```

**运行结果：** `找到强力敌人：巨石魔像，HP=200`

### binary_search：有序二分查找

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

int main()
{
    vector<int> levels = {10, 20, 30, 40, 50, 60, 70};

    // -------- 二分查找：前提是数组有序！--------
    if (binary_search(levels.begin(), levels.end(), 40)) {
        cout << "找到 40 级！" << endl;   // 速度快 O(logn)
    } else {
        cout << "未找到" << endl;
    }

    // -------- 如果数组无序，需要先 sort --------
    vector<int> unsorted = {50, 20, 80, 10, 40};
    sort(unsorted.begin(), unsorted.end());

    if (binary_search(unsorted.begin(), unsorted.end(), 40)) {
        cout << "无序数组排序后，也能二分查到 40" << endl;
    }

    return 0;
}
```

---

## 12.3 排序算法

### sort：快速排序（默认）

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
using namespace std;

// -------- 敌人结构体 --------
struct Enemy {
    string name;
    int hp;
};

int main()
{
    // -------- 整数排序 --------
    vector<int> scores = {85, 92, 78, 95, 88};
    sort(scores.begin(), scores.end());   // 默认从小到大

    cout << "从小到大：";
    for (int s : scores) cout << s << " ";
    cout << endl;

    // -------- 降序：使用 greater<int>() --------
    sort(scores.begin(), scores.end(), greater<int>());

    cout << "从大到小：";
    for (int s : scores) cout << s << " ";
    cout << endl;

    // -------- 按敌人 HP 排序 --------
    vector<Enemy> enemies = {
        {"哥布林", 100},
        {"巨龙", 500},
        {"骷髅", 50},
        {"牛魔王", 800}
    };

    sort(enemies.begin(), enemies.end(),
        [](const Enemy& a, const Enemy& b) {
            return a.hp < b.hp;   // HP 从小到大
        });

    cout << endl << "按 HP 排序：" << endl;
    for (const auto& e : enemies) {
        cout << e.name << " HP=" << e.hp << endl;
    }

    return 0;
}
```

**运行结果：**
```
从小到大：78 85 88 92 95 
从大到小：95 92 88 85 78 

按 HP 排序：
骷髅 HP=50
哥布林 HP=100
巨龙 HP=500
牛魔王 HP=800
```

### stable_sort：稳定排序

```cpp
// stable_sort 保证相同元素的相对顺序不变
stable_sort(enemies.begin(), enemies.end(),
    [](const Enemy& a, const Enemy& b) {
        return a.hp < b.hp;
    });
```

### reverse：反转顺序

```cpp
vector<int> v = {1, 2, 3, 4, 5};
reverse(v.begin(), v.end());   // 变成 {5, 4, 3, 2, 1}
```

---

## 12.4 统计类算法

### count / count_if

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

int main()
{
    vector<int> enemyHPs = {100, 200, 100, 150, 100, 300};

    // -------- count：统计指定值的个数 --------
    int deadCount = count(enemyHPs.begin(), enemyHPs.end(), 100);
    cout << "HP=100 的敌人数量：" << deadCount << endl;

    // -------- count_if：按条件统计 --------
    int highHPCount = count_if(enemyHPs.begin(), enemyHPs.end(),
        [](int hp) {
            return hp > 150;
        });
    cout << "HP > 150 的敌人数量：" << highHPCount << endl;

    return 0;
}
```

**运行结果：**
```
HP=100 的敌人数量：3
HP > 150 的敌人数量：2
```

### accumulate：求和

```cpp
#include <iostream>
#include <numeric>
#include <vector>
using namespace std;

int main()
{
    vector<int> items = {100, 250, 80, 320, 150};

    // -------- accumulate：累加求和 --------
    int totalGold = accumulate(items.begin(), items.end(), 0);
    cout << "物品总价值：" << totalGold << " 金币" << endl;

    // -------- 自定义运算 --------
    int totalDamage = accumulate(items.begin(), items.end(), 0,
        [](int sum, int item) {
            return sum + item / 10;   // 每件物品价值/10 作为攻击力
        });
    cout << "总攻击力：" << totalDamage << endl;

    return 0;
}
```

---

## 12.5 变换类算法

### transform：批量转换

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
using namespace std;

int main()
{
    vector<int> baseDamages = {50, 80, 120, 200};

    vector<int> finalDamages(baseDamages.size());   // 预分配空间

    // -------- transform：一元变换（+50%暴击加成）--------
    transform(baseDamages.begin(), baseDamages.end(),
        finalDamages.begin(),
        [](int damage) {
            return (int)(damage * 1.5f);   // 暴击加成
        });

    cout << "基础伤害：";
    for (int d : baseDamages) cout << d << " ";
    cout << endl;

    cout << "暴击伤害：";
    for (int d : finalDamages) cout << d << " ";
    cout << endl;

    // -------- 两个序列的变换 --------
    vector<string> itemNames = {"铁剑", "皮甲", "血瓶"};
    vector<string> fullNames;
    fullNames.reserve(itemNames.size());

    transform(itemNames.begin(), itemNames.end(),
        back_inserter(fullNames),
        [](const string& name) {
            return "[装备] " + name;
        });

    cout << "完整名称：";
    for (const string& n : fullNames) cout << n << " ";
    cout << endl;

    return 0;
}
```

### replace / replace_if

```cpp
vector<int> hpList = {100, 0, 50, 0, 200};

// -------- replace：把所有 0 换成 1（死亡标记归一）--------
replace(hpList.begin(), hpList.end(), 0, 1);

// -------- replace_if：把 HP < 50 的标记为 1 --------
replace_if(hpList.begin(), hpList.end(),
    [](int hp) { return hp < 50; }, 1);
```

---

## 12.6 删除类算法

### remove / remove_if

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;

int main()
{
    // -------- 场景：删除已死亡的敌人 --------
    vector<int> enemyHPs = {100, 0, 50, 0, 200, 0, 150};

    cout << "删除前：";
    for (int hp : enemyHPs) cout << hp << " ";
    cout << endl;

    // -------- remove_if：把 hp <= 0 的"删除" --------
    // 注意：remove 返回新的"有效"结尾迭代器
    auto newEnd = remove_if(enemyHPs.begin(), enemyHPs.end(),
        [](int hp) {
            return hp <= 0;   // 死亡条件
        });

    // -------- 真正删除后面的元素 --------
    enemyHPs.erase(newEnd, enemyHPs.end());

    cout << "删除后：";
    for (int hp : enemyHPs) cout << hp << " ";
    cout << endl;

    return 0;
}
```

**运行结果：**
```
删除前：100 0 50 0 200 0 150 
删除后：100 50 200 150 
```

### unique：去除相邻重复

```cpp
vector<int> levels = {1, 1, 2, 2, 2, 3, 3, 3, 3};

// -------- unique：去除相邻重复（需先排序）--------
auto last = unique(levels.begin(), levels.end());
levels.erase(last, levels.end());

// levels 现在是：{1, 2, 3}
```

---

## 12.7 最值算法

### min_element / max_element

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
using namespace std;

int main()
{
    vector<int> enemyHPs = {100, 50, 200, 80, 150};

    // -------- max_element：找最大 --------
    auto maxIt = max_element(enemyHPs.begin(), enemyHPs.end());
    cout << "最大 HP：" << *maxIt << "，位置：" << (maxIt - enemyHPs.begin()) << endl;

    // -------- min_element：找最小 --------
    auto minIt = min_element(enemyHPs.begin(), enemyHPs.end());
    cout << "最小 HP：" << *minIt << "，位置：" << (minIt - enemyHPs.begin()) << endl;

    // -------- minmax_element：同时找最大最小 --------
    auto [minIt2, maxIt2] = minmax_element(enemyHPs.begin(), enemyHPs.end());
    cout << "范围：" << *minIt2 << " ~ " << *maxIt2 << endl;

    return 0;
}
```

---

## 12.8 Lambda 条件谓词实战

### 完整技能 CD 管理示例

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
using namespace std;

// -------- 技能结构体 --------
struct Skill {
    string name;
    int damage;
    int mpCost;
    bool onCooldown;   // 是否在冷却中
};

int main()
{
    vector<Skill> skills = {
        {"火球术", 80, 30, false},
        {"冰冻术", 50, 20, true},
        {"雷击", 120, 50, false},
        {"治疗术", -50, 25, false},
        {"圣光术", 200, 80, true}
    };

    cout << "=== 所有技能 ===" << endl;
    for (const auto& s : skills) {
        cout << s.name << " 伤害=" << s.damage << " MP=" << s.mpCost;
        cout << (s.onCooldown ? " [冷却中]" : " [可用]") << endl;
    }

    // -------- 1. 找所有可用的技能（count_if + Lambda）--------
    int availableCount = count_if(skills.begin(), skills.end(),
        [](const Skill& s) {
            return !s.onCooldown;
        });
    cout << endl << "可用技能数量：" << availableCount << endl;

    // -------- 2. 找最高伤害技能（max_element + Lambda）--------
    auto maxDmgSkill = max_element(skills.begin(), skills.end(),
        [](const Skill& a, const Skill& b) {
            return a.damage < b.damage;
        });
    cout << "最高伤害技能：" << maxDmgSkill->name << "，伤害=" << maxDmgSkill->damage << endl;

    // -------- 3. 过滤出魔法攻击技能（伤害 > 0 且不在冷却）--------
    vector<Skill> attackSkills;
    copy_if(skills.begin(), skills.end(), back_inserter(attackSkills),
        [](const Skill& s) {
            return s.damage > 0 && !s.onCooldown;
        });

    cout << endl << "可用的攻击技能：" << endl;
    for (const auto& s : attackSkills) {
        cout << "  - " << s.name << " (伤害=" << s.damage << ")" << endl;
    }

    // -------- 4. 给所有技能加上"强化"效果（transform）--------
    transform(attackSkills.begin(), attackSkills.end(),
        attackSkills.begin(),
        [](Skill s) {
            s.damage = (int)(s.damage * 1.2f);   // 伤害 +20%
            s.name = "[强化]" + s.name;
            return s;
        });

    cout << endl << "强化后：" << endl;
    for (const auto& s : attackSkills) {
        cout << "  - " << s.name << " (伤害=" << s.damage << ")" << endl;
    }

    return 0;
}
```

**运行结果：**
```
=== 所有技能 ===
火球术 伤害=80 MP=30 [可用]
冰冻术 伤害=50 MP=20 [冷却中]
雷击 伤害=120 MP=50 [可用]
治疗术 伤害=-50 MP=25 [可用]
圣光术 伤害=200 MP=80 [冷却中]

可用技能数量：3
最高伤害技能：圣光术，伤害=200

可用的攻击技能：
  - 火球术 (伤害=80)
  - 雷击 (伤害=120)

强化后：
  - [强化]火球术 (伤害=96)
  - [强化]雷击 (伤害=144)
```

---

## 12.9 游戏开发实战：综合

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
#include <map>
using namespace std;

// -------- 敌人 --------
struct Enemy {
    string name;
    string type;
    int hp;
    int attack;
};

int main()
{
    vector<Enemy> enemies = {
        {"哥布林", "兽类", 100, 10},
        {"骷髅", "不死族", 80, 15},
        {"巨石魔像", "机械", 300, 30},
        {"幽灵", "不死族", 50, 25},
        {"毒蛇", "兽类", 40, 20},
        {"远古巨龙", "龙类", 1000, 100},
        {"食尸鬼", "不死族", 90, 18}
    };

    cout << "=== 敌人管理系统 ===" << endl;

    // -------- 按 HP 从大到小排序 --------
    sort(enemies.begin(), enemies.end(),
        [](const Enemy& a, const Enemy& b) {
            return a.hp > b.hp;
        });

    cout << "按 HP 排序（高→低）：" << endl;
    for (const auto& e : enemies) {
        cout << e.name << " HP=" << e.hp << " ATK=" << e.attack << endl;
    }

    // -------- 统计不死族敌人数量 --------
    int undeadCount = count_if(enemies.begin(), enemies.end(),
        [](const Enemy& e) {
            return e.type == "不死族";
        });
    cout << endl << "不死族敌人数量：" << undeadCount << endl;

    // -------- 找 HP 最高的龙类敌人 --------
    auto dragonIt = max_element(enemies.begin(), enemies.end(),
        [](const Enemy& a, const Enemy& b) {
            if (a.type != "龙类") return true;
            if (b.type != "龙类") return false;
            return a.hp < b.hp;
        });

    // 更清晰的写法：
    auto dragonMax = max_element(enemies.begin(), enemies.end(),
        [](const Enemy& a, const Enemy& b) {
            if (a.type == "龙类" && b.type != "龙类") return false;
            if (a.type != "龙类" && b.type == "龙类") return true;
            return a.hp < b.hp;
        });

    cout << "龙类敌人中 HP 最高：";
    if (dragonMax->type == "龙类") {
        cout << dragonMax->name << " HP=" << dragonMax->hp << endl;
    }

    // -------- 删除所有 HP <= 0 的敌人（模拟死亡）--------
    enemies.erase(
        remove_if(enemies.begin(), enemies.end(),
            [](const Enemy& e) { return e.hp <= 0; }),
        enemies.end());

    // -------- 用 map 统计各类型敌人数量 --------
    map<string, int> typeCount;
    for (const auto& e : enemies) {
        typeCount[e.type]++;
    }

    cout << endl << "各类型敌人数量：" << endl;
    for (const auto& [type, count] : typeCount) {
        cout << "  " << type << "：" << count << " 只" << endl;
    }

    return 0;
}
```

---

## 12.10 常见错误与调试

| 错误现象 | 原因 | 解决方法 |
|---------|------|---------|
| `sort` 后容器乱了 | 自定义比较函数有 bug（未定义严格弱序）| 检查比较函数 |
| `binary_search` 找不到 | 数组无序 | 先 `sort` 再二分 |
| `remove` 后 size 没变 | 需要配合 `erase` | `v.erase(remove(...), v.end())` |
| `find_if` 返回 `end()` 但元素存在 | 条件写错了 | 检查 Lambda 条件 |
| `accumulate` 初始值类型不对 | `int` 和 `float` 混用 | 统一类型或用 `0.0f` |

---

## 12.11 动手练习 🧪

### 练习 1：查找玩家 ⭐
```cpp
// vector<string> players = {"Alice", "Bob", "Charlie", "Diana"};
// 用 find 查找 "Charlie"，用 find_if 查找名字长度 > 4 的第一个玩家
```

### 练习 2：排序分数排行榜 ⭐
```cpp
// map<string, int> scores = {{"Alice", 8500}, {"Bob", 9200}, {"Charlie", 7800}};
// 按分数从高到低排序并输出
```

### 练习 3：统计敌人 ⭐⭐
```cpp
// vector<Enemy> enemies（参考实战代码）
// 用 count_if 统计 HP > 100 的敌人数量
// 用 max_element 找攻击力最高的敌人
```

### 练习 4：技能过滤 ⭐⭐
```cpp
// 用 remove_if 删除所有在冷却中的技能
// 用 transform 给所有技能伤害 +50
// 用 copy_if 把符合条件的技能复制到新容器
```

### 练习 5：综合应用 ⭐⭐⭐
```cpp
// 实现一个道具筛选系统：
// - 用 vector 存储道具（name, type, rarity, value）
// - 用 sort 按 value 降序排序
// - 用 count_if 统计各稀有度的道具数量
// - 用 remove_if 删除价值 < 100 的道具
// - 用 transform 把所有道具名称加上 "[★]" 前缀
```

---

## 12.12 本章小结

```
✅ 已掌握：
├── find / find_if：查找
├── binary_search：有序二分查找
├── sort / stable_sort：排序
├── reverse：反转
├── count / count_if：统计
├── accumulate：累加
├── transform：变换
├── replace / replace_if：替换
├── remove / remove_if：删除
├── unique：去重
├── min/max_element：最值
└── Lambda 在算法中的使用

🔜 下章预告：
第十三章：文件操作 —— 学会读写存档、配置文件，让游戏数据永久保存！
```

---

_📚 参考资料：《C++ Primer》《Effective STL》《算法导论》_
