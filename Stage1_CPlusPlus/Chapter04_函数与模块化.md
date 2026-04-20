# Chapter 04：函数与模块化编程

> 🎯 目标：学会把重复代码封装成函数，让代码复用、逻辑清晰。游戏中的攻击函数、伤害计算函数都可以抽成独立模块。

---

## 4.1 什么是函数？

### 生活中的类比 🏭

> **工厂流水线：**
> - 原材料进入 → 流水线加工 → 成品出来
> - 同一种产品，重复使用同一条流水线
>
> **函数就是程序里的"流水线"：数据进去 → 函数处理 → 结果出来。**

### 为什么需要函数？

```cpp
// -------- 没有函数：重复代码 --------
cout << "=== 攻击造成 25 点伤害 ===" << endl;
cout << "敌人剩余血量：" << (200 - 25) << endl;

cout << "=== 攻击造成 25 点伤害 ===" << endl;
cout << "敌人剩余血量：" << (175 - 25) << endl;

cout << "=== 攻击造成 25 点伤害 ===" << endl;
cout << "敌人剩余血量：" << (150 - 25) << endl;

// -------- 有函数：一行搞定 --------
void showAttackResult(int enemyHP, int damage) {
    cout << "=== 攻击造成 " << damage << " 点伤害 ===" << endl;
    cout << "敌人剩余血量：" << (enemyHP - damage) << endl;
}

showAttackResult(200, 25);  // 第1次攻击
showAttackResult(175, 25);  // 第2次攻击
showAttackResult(150, 25);  // 第3次攻击
```

---

## 4.2 函数的定义与调用

### 基本语法

```
返回类型 函数名(参数列表) {
    // 函数体
    return 结果;   // 根据返回类型决定是否需要 return
}
```

### 逐行解析

```cpp
// -------- 定义一个计算攻击力的函数 --------
int calculateDamage(int attackPower, float critRate) {
    // 参数：attackPower 攻击力，critRate 暴击倍率
    int baseDamage = attackPower;        // 基础伤害 = 攻击力
    int finalDamage = baseDamage * critRate;  // 最终伤害 = 基础 * 暴击倍率
    return finalDamage;                   // 返回结果
}

// -------- 主函数 --------
int main()
{
    int damage = calculateDamage(100, 2.0f);  // 调用：100攻击力，2倍暴击
    cout << "造成的伤害：" << damage << endl;   // 输出：200
    return 0;
}
```

### 函数的组成要素

```
┌──────────────────────────────────────┐
│  返回类型  │  函数名  │  参数列表     │  ← 函数签名
├──────────────────────────────────────┤
│                                      │
│         函数体：具体实现              │
│                                      │
└──────────────────────────────────────┘
```

---

## 4.3 无返回值函数（void）

### 什么时候用 void？

> 当函数只是**执行一系列操作，不需要返回结果**时，用 `void`。

### 游戏开发实战：显示角色状态

```cpp
#include <iostream>
#include <string>
using namespace std;

// -------- 无返回值函数：显示角色状态 --------
void showPlayerStatus(string name, int hp, int mp, int level) {
    cout << "========== 角色状态 ==========" << endl;
    cout << "名称：" << name << endl;
    cout << "等级：" << level << endl;
    cout << "生命值：" << hp << endl;
    cout << "魔法值：" << mp << endl;
    cout << "================================" << endl;
}

// -------- 主函数 --------
int main()
{
    // -------- 初始状态 --------
    showPlayerStatus("孙悟空", 5000, 1200, 85);

    cout << endl;

    // -------- 升级后状态 --------
    showPlayerStatus("孙悟空", 5500, 1400, 86);

    return 0;
}
```

**运行结果：**
```
========== 角色状态 ==========
名称：孙悟空
等级：85
生命值：5000
魔法值：1200
================================

========== 角色状态 ==========
名称：孙悟空
等级：86
生命值：5500
魔法值：1400
================================
```

---

## 4.4 带返回值函数

### 典型场景

| 场景 | 函数名 | 返回值 |
|-----|-------|-------|
| 计算伤害 | `calculateDamage` | `int` 伤害值 |
| 判断是否暴击 | `isCriticalHit` | `bool` 是否暴击 |
| 获取角色等级 | `getPlayerLevel` | `int` 等级 |
| 计算金币 | `calculateGold` | `int` 金币数量 |

### 游戏开发实战：暴击判断 + 伤害计算

```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>
using namespace std;

// -------- 判断是否暴击（随机）--------
bool isCriticalHit(float critChance) {
    // critChance: 暴击率，例如 0.2 表示 20% 暴击率
    int random = rand() % 100;          // 生成 0-99 的随机数
    return (random < critChance * 100);  // 如果随机数 < 暴击率*100，则暴击
}

// -------- 计算最终伤害 --------
int calculateFinalDamage(int attackPower, float critChance) {
    if (isCriticalHit(critChance)) {
        cout << "💥 暴击！" << endl;
        return attackPower * 2;          // 暴击伤害翻倍
    } else {
        return attackPower;              // 普通伤害
    }
}

// -------- 主函数 --------
int main()
{
    srand(time(0));   // 初始化随机数种子

    int playerAttack = 150;      // 玩家攻击力
    float critChance = 0.25f;    // 25% 暴击率

    cout << "===== 攻击测试（5次）=====" << endl;
    for (int i = 1; i <= 5; i++) {
        int damage = calculateFinalDamage(playerAttack, critChance);
        cout << "第 " << i << " 次攻击，伤害：" << damage << endl;
    }

    return 0;
}
```

---

## 4.5 参数传递：值传递 vs 引用传递

### 值传递（按值传递）

> **概念：** 把数据的**副本**传进去，函数里怎么改都不影响原数据。

```cpp
#include <iostream>
using namespace std;

// -------- 值传递 --------
void addTen(int num) {
    num = num + 10;   // 只改了副本
    cout << "函数内 num = " << num << endl;
}

int main()
{
    int score = 100;
    cout << "调用前 score = " << score << endl;
    addTen(score);                    // 把 score 的副本传进去
    cout << "调用后 score = " << score << endl;  // score 不变！
    return 0;
}
```

**运行结果：**
```
调用前 score = 100
函数内 num = 110
调用后 score = 100    ← 原值不变！
```

### 引用传递（按引用传递）

> **概念：** 把数据的**地址**传进去，函数里改了会影响原数据。

```cpp
#include <iostream>
using namespace std;

// -------- 引用传递 --------
void addTen(int& num) {   // 注意多了 &
    num = num + 10;       // 直接改原数据
    cout << "函数内 num = " << num << endl;
}

int main()
{
    int score = 100;
    cout << "调用前 score = " << score << endl;
    addTen(score);                    // 把 score 的地址传进去
    cout << "调用后 score = " << score << endl;  // score 变了！
    return 0;
}
```

**运行结果：**
```
调用前 score = 100
函数内 num = 110
调用后 score = 110    ← 原值被改了！
```

### 游戏开发实战：扣血 vs 回血

```cpp
#include <iostream>
using namespace std;

// -------- 扣血（引用传递）--------
void takeDamage(int& hp, int damage) {
    hp = hp - damage;
    if (hp < 0) hp = 0;
    cout << "💥 受到 " << damage << " 点伤害，剩余 HP：" << hp << endl;
}

// -------- 回血（引用传递）--------
void heal(int& hp, int healAmount) {
    hp = hp + healAmount;
    cout << "💚 回复 " << healAmount << " 点血量，HP：" << hp << endl;
}

int main()
{
    int playerHP = 100;

    takeDamage(playerHP, 30);  // 扣血：引用传递
    takeDamage(playerHP, 50);  // 扣血：引用传递
    heal(playerHP, 20);        // 回血：引用传递

    return 0;
}
```

**运行结果：**
```
💥 受到 30 点伤害，剩余 HP：70
💥 受到 50 点伤害，剩余 HP：20
💚 回复 20 点血量，HP：40
```

---

## 4.6 函数的声明与定义分离

### 为什么需要分离？

> 当项目变大，多个文件互相调用函数时，需要**先声明函数存在，再使用**。

### 示例：头文件 + 实现文件

```cpp
// -------- Player.h（头文件：声明）--------
// 声明这个函数存在，告诉编译器它的签名
int calculateDamage(int attackPower, float critRate);
void showPlayerStatus(string name, int hp, int mp);

// -------- Player.cpp（源文件：定义）--------
#include <iostream>
#include "Player.h"
using namespace std;

int calculateDamage(int attackPower, float critRate) {
    return attackPower * critRate;
}

void showPlayerStatus(string name, int hp, int mp) {
    cout << "名称：" << name << endl;
    cout << "HP：" << hp << "  MP：" << mp << endl;
}

// -------- main.cpp（使用函数）--------
#include <iostream>
#include "Player.h"
using namespace std;

int main()
{
    int damage = calculateDamage(100, 2.0f);
    cout << "伤害：" << damage << endl;

    showPlayerStatus("孙悟空", 5000, 1200);
    return 0;
}
```

### 游戏项目中的典型结构

```
GameProject/
├── include/          ← 头文件 (.h)
│   ├── Player.h
│   ├── Enemy.h
│   └── Item.h
├── src/              ← 源文件 (.cpp)
│   ├── Player.cpp
│   ├── Enemy.cpp
│   └── main.cpp
└── build/            ← 编译输出
```

---

## 4.7 函数重载：同名不同参

### 概念

> **函数重载（Overload）：** 同名函数，但参数列表不同（个数不同 or 类型不同）。

### 为什么有用？

> 同样叫"攻击"，但可以对单个敌人攻击，也可以对一群敌人攻击。

```cpp
#include <iostream>
using namespace std;

// -------- 单体攻击 --------
int attack(int damage) {
    cout << "单体攻击，造成 " << damage << " 点伤害" << endl;
    return damage;
}

// -------- 范围攻击（参数不同：多一个范围）--------
int attack(int damage, int range) {
    cout << "范围攻击，" << range << " 米内，造成 " << damage << " 点伤害" << endl;
    return damage * range;  // 范围攻击总伤害更高
}

// -------- 主函数 --------
int main()
{
    attack(100);           // 调用第一个 attack
    attack(50, 3);         // 调用第二个 attack（参数个数不同）
    return 0;
}
```

**运行结果：**
```
单体攻击，造成 100 点伤害
范围攻击，3 米内，造成 50 点伤害
```

---

## 4.8 递归函数

### 生活中的类比 🔄

> **俄罗斯套娃：**
> - 打开一个大娃娃，里面有个中娃娃
> - 打开中娃娃，里面有个小娃娃
> - 直到打开最小的小娃娃（终止条件）
>
> **递归就是函数调用自己，但要有终止条件，否则无限循环。**

### 递归求阶乘

```cpp
#include <iostream>
using namespace std;

// -------- 递归：计算 n!（n 的阶乘）--------
int factorial(int n) {
    // -------- 终止条件 --------
    if (n <= 1) {
        return 1;
    }
    // -------- 递归调用 --------
    return n * factorial(n - 1);
}

// -------- 主函数 --------
int main()
{
    cout << "5!  = " << factorial(5) << endl;   // 5*4*3*2*1 = 120
    cout << "10! = " << factorial(10) << endl;   // 3628800
    return 0;
}
```

**执行过程：**
```
factorial(5)
= 5 * factorial(4)
= 5 * 4 * factorial(3)
= 5 * 4 * 3 * factorial(2)
= 5 * 4 * 3 * 2 * factorial(1)
= 5 * 4 * 3 * 2 * 1
= 120
```

### 游戏开发实战：递归遍历文件夹（伪代码）

```cpp
// 递归遍历一个目录下的所有文件
void listFiles(string path) {
    files = getFilesInDirectory(path);  // 获取目录下所有文件

    for each file in files {
        if (file is directory) {
            listFiles(file.path);  // 如果是目录，递归进入
        } else {
            print(file.name);      // 如果是文件，打印名称
        }
    }
}
```

---

## 4.9 内联函数（inline）

### 概念

> **内联函数：** 编译器直接把函数体**插入**到调用处，省去函数调用的开销。
>
> 适合**短小、频繁调用**的函数（如 getter/setter）。

### 示例

```cpp
#include <iostream>
using namespace std;

// -------- 普通函数 --------
int add(int a, int b) {
    return a + b;
}

// -------- 内联函数 --------
inline int addInline(int a, int b) {
    return a + b;
}

int main()
{
    // 普通函数调用：有函数调用开销
    cout << add(1, 2) << endl;

    // 内联函数调用：编译时直接展开，无调用开销
    cout << addInline(1, 2) << endl;
    return 0;
}
```

### 什么时候用 inline？

| 适合 | 不适合 |
|-----|-------|
| 函数体只有 1-3 行 | 函数体很长 |
| 被频繁调用 | 包含循环、递归 |
| 不需要 this 指针 | 虚函数 |

---

## 4.10 Lambda 匿名函数

### 概念

> **Lambda：** 没有名字的函数，适合临时用一次的场景。

### 基本语法

```
[capture](parameters) -> return_type {
    // 函数体
}
```

### 示例

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    // -------- 普通方式：定义一个函数 --------
    auto isEven = [](int n) -> bool {
        return n % 2 == 0;
    };

    cout << "10 是偶数？" << isEven(10) << endl;  // 1（真）

    // -------- 在算法中直接用 Lambda --------
    vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    // 统计偶数个数
    int evenCount = count_if(numbers.begin(), numbers.end(),
        [](int n) { return n % 2 == 0; });

    cout << "偶数个数：" << evenCount << endl;  // 5

    return 0;
}
```

### 游戏开发实战：过滤敌人列表

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    vector<int> enemyHPs = {100, 0, 200, 0, 150, 50, 0};

    // -------- 用 Lambda 过滤：只保留活着的敌人（HP > 0）--------
    vector<int> aliveEnemies;

    for (int i = 0; i < enemyHPs.size(); i++) {
        if (enemyHPs[i] > 0) {
            aliveEnemies.push_back(enemyHPs[i]);
        }
    }

    // -------- 也可以用 remove_if + Lambda --------
    enemyHPs.erase(
        remove_if(enemyHPs.begin(), enemyHPs.end(),
            [](int hp) { return hp <= 0; }),
        enemyHPs.end()
    );

    cout << "存活的敌人数量：" << enemyHPs.size() << endl;
    for (int hp : enemyHPs) {
        cout << hp << " ";
    }
    cout << endl;

    return 0;
}
```

---

## 4.11 动手练习 🧪

### 练习 1：写一个攻击函数
定义函数 `void attack(int damage, string enemyName)`，调用示例：
```cpp
attack(50, "哥布林");
// 输出：>>> 对哥布林发起攻击，造成 50 点伤害！
```

### 练习 2：引用传递改写扣血
用引用传递改写以下代码，使 `playerHP` 真正被修改：
```cpp
void reduceHP(int hp, int damage);  // 改成 int& hp
```

### 练习 3：函数重载
写两个同名函数 `max`：
- `max(int a, int b)` 返回较大值
- `max(int a, int b, int c)` 返回三个数中的最大值

### 练习 4：递归实现斐波那契数列
```cpp
int fibonacci(int n);
// fibonacci(6) = 8  (1,1,2,3,5,8,...)
```

### 练习 5：Lambda 筛选装备
```cpp
vector<string> items = {"铁剑", "布甲", "魔法杖", "皮靴", "圣剑"};
// 用 Lambda 筛选名字长度 > 2 的装备
```

---

## 4.12 本章小结

```
✅ 已掌握：
├── 函数的定义与调用
├── void 无返回值函数
├── 带返回值函数
├── 值传递 vs 引用传递
├── 头文件 + 源文件分离
├── 函数重载（Overload）
├── 递归函数
├── 内联函数（inline）
└── Lambda 匿名函数

🔜 下章预告：
第五章：数组与字符串 —— 学会批量管理数据，如敌人列表、背包道具、角色属性数组。
```

---

_📚 参考资料：《C++ Primer》《Effective C++》《Game Programming Patterns》_
