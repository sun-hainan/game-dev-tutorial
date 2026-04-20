# Chapter 03：流程控制（条件语句 + 循环）

> 🎯 目标：掌握程序的"判断力"（条件语句）和"重复执行"（循环）。这是游戏 AI、事件系统、动画循环的核心。

---

## 3.1 什么是流程控制？

### 生活中的类比 🚦

> **十字路口：**
> - 红灯 → 停车等待
> - 绿灯 → 继续前进
> - 黄灯 → 减速停车
>
> **程序也是一样：满足条件 A 就执行 A，不满足就执行 B。**

### 两种类型

| 类型 | 作用 | 关键词 |
|-----|------|-------|
| **选择结构** | 根据条件决定走哪条路 | `if / else`, `switch` |
| **循环结构** | 重复执行一段代码 | `for`, `while`, `do...while` |

---

## 3.2 if / else 条件语句

### 基本语法

```
if (条件) {
    // 条件为真时执行
} else {
    // 条件为假时执行
}
```

### 逐行解释

```cpp
int hp = 30;

// -------- 第1步：判断条件 --------
if (hp <= 0) {
    // -------- 第2步：条件为真 --------
    cout << "角色死亡！" << endl;
} else {
    // -------- 第3步：条件为假 --------
    cout << "角色还活着！" << endl;
}
```

**运行结果：** `角色还活着！`（因为 hp = 30 > 0）

### 多条件分支：else if

```cpp
int score = 85;

if (score >= 90) {
    cout << "评级：S" << endl;      // 90分以上
} else if (score >= 80) {
    cout << "评级：A" << endl;      // 80-89分
} else if (score >= 60) {
    cout << "评级：B" << endl;      // 60-79分
} else {
    cout << "评级：C" << endl;      // 60分以下
}
```

### 游戏开发实战：检查是否能攻击

```cpp
#include <iostream>
using namespace std;

int main()
{
    int playerHP = 100;       // 玩家血量
    int enemyHP = 50;         // 敌人血量
    int playerAmmo = 0;       // 玩家弹药
    bool isReloading = false; // 是否正在装填

    // -------- 攻击判断系统 --------
    if (playerHP <= 0) {
        cout << "❌ 无法攻击：你已经死亡" << endl;
    } else if (enemyHP <= 0) {
        cout << "❌ 无法攻击：敌人已经死亡" << endl;
    } else if (playerAmmo <= 0) {
        cout << "⚠️ 无法攻击：弹药不足！" << endl;
    } else if (isReloading) {
        cout << "⚠️ 无法攻击：正在装填中..." << endl;
    } else {
        cout << "✅ 执行攻击！" << endl;
        enemyHP -= 25;           // 造成 25 点伤害
        playerAmmo -= 1;         // 消耗 1 发弹药
        cout << "敌人剩余血量：" << enemyHP << endl;
        cout << "剩余弹药：" << playerAmmo << endl;
    }

    return 0;
}
```

---

## 3.3 switch 多分支

### 适用场景

> 当判断的是**一个变量的多个离散值**时，`switch` 比 `if / else if` 更清晰。

### 基本语法

```cpp
switch (变量或表达式) {
    case 值1:
        // 等于值1时执行的代码
        break;           // 跳出 switch
    case 值2:
        // 等于值2时执行的代码
        break;
    default:
        // 所有 case 都不匹配时执行
        break;
}
```

### 游戏开发实战：道具系统

```cpp
#include <iostream>
using namespace std;

int main()
{
    char item = 'H';   // 'H'=红药瓶, 'M'=蓝药瓶, 'S'=盾牌, 'W'=武器

    switch (item) {
        case 'H':   // 红药瓶
            cout << "使用红药瓶，回复 50 HP" << endl;
            break;
        case 'M':   // 蓝药瓶
            cout << "使用蓝药瓶，回复 30 MP" << endl;
            break;
        case 'S':   // 盾牌
            cout << "装备盾牌，获得临时护甲" << endl;
            break;
        case 'W':   // 武器
            cout << "更换武器，攻击力提升" << endl;
            break;
        default:
            cout << "未知道具！" << endl;
            break;
    }

    return 0;
}
```

**运行结果：** `使用红药瓶，回复 50 HP`

### ⚠️ 忘记 break 的坑

```cpp
int day = 1;
switch (day) {
    case 1:
        cout << "星期一" << endl;
        // 忘记 break！会"落入"下一个 case
    case 2:
        cout << "星期二" << endl;
        break;
}
// 输出：星期一 星期二（不是预期的结果）
```

---

## 3.4 for 循环

### 生活中的类比 🔄

> 广播体操喊操：喊 8 遍"齐步走"，每遍之间稍作休息。
>
> **for 循环 = 知道要重复多少次**

### 基本语法

```
for (初始化; 条件判断; 更新) {
    // 循环体
}
```

### 逐行解析

```cpp
// -------- 打印 5 次"第 X 遍" --------
for (int i = 1; i <= 5; i++) {
    // i = 1：初始化，从 1 开始数
    // i <= 5：条件判断，没到 5 就继续
    // i++：每次循环结束后 i 加 1
    cout << "第 " << i << " 遍" << endl;
}
```

**运行结果：**
```
第 1 遍
第 2 遍
第 3 遍
第 4 遍
第 5 遍
```

### for 循环的三个阶段

```
  初始化 ──── 只在开始执行一次
     │
     ▼
┌─────────┐    是    ┌─────────┐
│ 条件判断 │ ──────▶ │ 执行循环体│
└─────────┘         └────┬────┘
     │                    │
     │ 否                 │
     ▼                    ▼
   结束              更新 (i++)
                           │
                           └─────▶ 回到条件判断
```

### 游戏开发实战：敌人列表遍历

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 敌人血量数组 --------
    int enemyHPs[5] = {100, 200, 150, 300, 80};

    cout << "=== 敌人状态检查 ===" << endl;

    // -------- 遍历所有敌人 --------
    for (int i = 0; i < 5; i++) {
        cout << "敌人 " << (i + 1) << "：";

        // -------- 根据血量判断状态 --------
        if (enemyHPs[i] <= 0) {
            cout << "已死亡" << endl;
        } else if (enemyHPs[i] < 100) {
            cout << "重伤" << endl;
        } else {
            cout << "正常" << endl;
        }
    }

    return 0;
}
```

**运行结果：**
```
=== 敌人状态检查 ===
敌人 1：正常
敌人 2：正常
敌人 3：正常
敌人 4：重伤
敌人 5：已死亡
```

---

## 3.5 while 循环

### 生活中的类比 🎮

> 游戏中的"暂停界面"：
> - 等待玩家按任意键继续
> - 玩家没按就继续等
> - **不知道要等多少次，只知道要等到条件满足**
>
> **while 循环 = 不知道要循环多少次，但知道结束条件**

### 基本语法

```
while (条件) {
    // 循环体
}
```

### 循环猜数字游戏

```cpp
#include <iostream>
using namespace std;

int main()
{
    int secret = 7;        // 正确答案
    int guess = 0;         // 玩家输入

    cout << "=== 猜数字游戏 ===" << endl;
    cout << "请输入一个 1-10 的数字：" << endl;

    // -------- 循环：直到猜对才退出 --------
    while (guess != secret) {
        cout << "你的猜测：";
        cin >> guess;      // cin：读取用户输入

        if (guess > secret) {
            cout << "太大了！再试试。" << endl;
        } else if (guess < secret) {
            cout << "太小了！再试试。" << endl;
        } else {
            cout << "🎉 恭喜猜对了！答案是 " << secret << endl;
        }
    }

    return 0;
}
```

### while 和 for 的对比

| 维度 | for | while |
|-----|-----|-------|
| 适用场景 | 已知循环次数 | 未知循环次数 |
| 语法结构 | 初始化+条件+更新一体 | 条件单独出现 |
| 示例 | 遍历数组 | 等待用户输入 |

```cpp
// 这两个写法效果完全一样：
for (int i = 0; i < 5; i++) { ... }
int i = 0;
while (i < 5) { ...; i++; }
```

---

## 3.6 do...while 循环

### 特点

> **至少执行一次**，再判断条件。

### 基本语法

```
do {
    // 先执行一次
} while (条件);
```

### 实战：菜单系统

```cpp
#include <iostream>
using namespace std;

int main()
{
    int choice = 0;

    do {
        // -------- 先显示菜单 --------
        cout << "=== 游戏菜单 ===" << endl;
        cout << "1. 开始游戏" << endl;
        cout << "2. 加载存档" << endl;
        cout << "3. 退出游戏" << endl;
        cout << "请选择：";
        cin >> choice;

        // -------- 再根据选择执行 --------
        switch (choice) {
            case 1:
                cout << ">>> 游戏开始！" << endl;
                break;
            case 2:
                cout << ">>> 加载存档中..." << endl;
                break;
            case 3:
                cout << ">>> 退出游戏。再见！" << endl;
                break;
            default:
                cout << "无效选项，请重新选择。" << endl;
                break;
        }

    } while (choice != 3);  // 用户选择 3 才退出循环

    return 0;
}
```

---

## 3.7 break 和 continue

| 关键词 | 作用 | 类比 |
|-------|------|------|
| `break` | 跳出**整个循环** | 逃出大楼（彻底结束）|
| `continue` | 跳过**本次循环**，继续下一次 | 跳过这首歌的副歌部分 |

### break 示例：找到第一个存活的敌人

```cpp
int enemyHPs[5] = {0, 200, 0, 150, 0};  // 0 = 死亡

for (int i = 0; i < 5; i++) {
    if (enemyHPs[i] > 0) {
        cout << "找到第一个存活的敌人，编号：" << (i + 1) << endl;
        break;     // 找到了，立即退出循环，不再继续找
    }
}
```

### continue 示例：跳过死亡敌人

```cpp
int enemyHPs[5] = {0, 200, 0, 150, 0};

for (int i = 0; i < 5; i++) {
    if (enemyHPs[i] <= 0) {
        continue;  // 死亡了，跳过这次循环，不执行后面的攻击代码
    }
    cout << "敌人 " << (i + 1) << " 受到攻击！" << endl;
}
```

---

## 3.8 嵌套循环：打印游戏地图

### 5x5 地牢地图

```cpp
#include <iostream>
using namespace std;

int main()
{
    cout << "=== 5x5 地牢地图 ===" << endl;

    // -------- 外层循环：行 --------
    for (int row = 0; row < 5; row++) {
        // -------- 内层循环：列 --------
        for (int col = 0; col < 5; col++) {
            // 特殊位置：玩家位置 (2,2)，出口 (4,4)
            if (row == 2 && col == 2) {
                cout << " P ";   // 玩家
            } else if (row == 4 && col == 4) {
                cout << " E ";   // 出口
            } else {
                cout << " # ";   // 墙壁
            }
        }
        cout << endl;   // 换行
    }

    return 0;
}
```

**运行结果：**
```
=== 5x5 地牢地图 ===
 #  #  #  #  # 
 #  #  #  #  # 
 #  #  P  #  # 
 #  #  #  #  # 
 #  #  #  #  E 
```

---

## 3.9 动手练习 🧪

### 练习 1：成绩评级
输入一个分数（0-100），用 if / else if 输出等级：
- 90+ → S
- 80-89 → A
- 60-79 → B
- < 60 → C

### 练习 2：1 到 100 求和
用 for 循环计算 1+2+3+...+100 的和，输出结果。

### 练习 3：九九乘法表
用嵌套 for 循环打印九九乘法表。

### 练习 4：FizzBuzz 游戏规则
遍历 1-20：
- 能被 3 整除 → 输出 "Fizz"
- 能被 5 整除 → 输出 "Buzz"
- 能被 15 整除 → 输出 "FizzBuzz"
- 其他 → 输出数字

### 练习 5：RPG 战斗循环
写一个简单战斗循环：
```cpp
int playerHP = 100, enemyHP = 50;
// 每回合玩家攻击 15 点，敌人攻击 10 点
// 直到一方血量 <= 0，输出胜负结果
```

---

## 3.10 本章小结

```
✅ 已掌握：
├── if / else 条件判断
├── else if 多条件分支
├── switch 多分支选择
├── for 循环（已知次数）
├── while 循环（未知次数）
├── do...while 循环（至少执行一次）
├── break 跳出循环
├── continue 跳过本次循环
└── 嵌套循环（打印地图等二维结构）

🔜 下章预告：
第四章：函数与模块化编程 —— 把重复代码封装成函数，让代码复用、逻辑清晰。
   游戏中的攻击函数、伤害计算函数都可以抽成独立模块！
```

---

_📚 参考资料：《C++ Primer》《Game Programming Patterns》《游戏设计梦工厂》_
