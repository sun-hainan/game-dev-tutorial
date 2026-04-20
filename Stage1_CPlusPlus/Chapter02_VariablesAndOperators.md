# Chapter 02：变量、数据类型与运算符

> 🎯 目标：理解变量是"有名字的盒子"，掌握 C++ 的基本数据类型，学会用运算符做计算。

---

## 2.1 变量：数据的"盒子"

### 生活中的类比 🗃️

> 想象你有很多不同大小的收纳盒：
> - 装首饰用小盒子 💍
> - 装鞋子用大箱子 👟
> - 装文件用文件袋 📁
>
> **变量就是计算机里的收纳盒，每个盒子上贴了名字，里面装不同类型的东西。**

### 为什么需要变量？

```cpp
// 没有变量：写死数字，难理解
cout << 90 + 90 + 90;          // 这是什么？不知道！

// 有变量：见名知意，清晰易懂
int score = 90;
int bonus = 90;
int total = score + score + bonus;  // 一眼看出：分数 + 分数 + 奖励
cout << total;
```

### 变量的定义语法

```
类型 变量名;           // 只定义，不赋值（盒子存在，内容未知）
类型 变量名 = 初值;   // 定义并赋值（盒子存在，里面有东西）
```

### 完整示例

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 定义各种类型的变量 --------
    int playerScore = 0;       // 整数：玩家分数
    float moveSpeed = 5.5f;    // 单精度浮点数：移动速度
    double health = 100.0;     // 双精度浮点数：生命值
    char grade = 'A';          // 字符：评级
    bool isAlive = true;       // 布尔：是否存活

    // -------- 输出变量的值 --------
    cout << "玩家分数：" << playerScore << endl;
    cout << "移动速度：" << moveSpeed << endl;
    cout << "生命值：" << health << endl;
    cout << "评级：" << grade << endl;
    cout << "存活状态：" << isAlive << endl;

    return 0;
}
```

**运行结果：**
```
玩家分数：0
移动速度：5.5
生命值：100
评级：A
存活状态：1
```

---

## 2.2 基本数据类型

### 类型一览表

| 类型 | 中文名 | 占位 | 示例 | 游戏中的用途 |
|-----|-------|------|------|------------|
| `int` | 整数 | 4 字节 | `int level = 10;` | 等级、击杀数 |
| `float` | 单精度浮点 | 4 字节 | `float speed = 3.5f;` | 移动速度、角度 |
| `double` | 双精度浮点 | 8 字节 | `double pi = 3.14159;` | 物理计算 |
| `char` | 字符 | 1 字节 | `char rank = 'S';` | 评级、状态标识 |
| `bool` | 布尔 | 1 字节 | `bool isFire = true;` | 开关状态 |
| `string` | 字符串 | 可变 | `string name = "Hero";` | 角色名、物品名 |

### 游戏开发中的典型变量

```cpp
// 角色属性系统
string heroName = "孙悟空";      // 角色名称
int level = 85;                  // 等级
int hp = 5000;                   // 生命值
int mp = 1200;                   // 法力值
float attackPower = 256.5f;      // 攻击力
float moveSpeed = 8.3f;          // 移动速度
bool hasShield = true;           // 是否有护盾
char rating = 'S';               // 角色评级
```

### float 和 double 的区别

```cpp
#include <iostream>
using namespace std;

int main()
{
    // float 后缀 f，double 不需要
    float a = 3.14159f;       // 单精度
    double b = 3.14159;       // 双精度

    cout << "float:  " << a << endl;
    cout << "double: " << b << endl;

    // double 更精确，游戏开发中常用 double 做物理计算
    return 0;
}
```

---

## 2.3 变量的命名规则

### 命名规范 ✅

```
规则                          示例
─────────────────────────────────────────
必须以字母或下划线开头        score, _temp, player1
区分大小写                    Score 和 score 是不同变量
不能使用关键字                int、return、if 等
建议使用有意义的名称           playerScore > a > x
驼峰命名法（常用）            playerScore, moveSpeed
下划线命名法                  player_score, move_speed
```

### 常见命名风格对比

```cpp
// 驼峰法（camelCase）：首个单词小写，之后单词大写
int playerScore = 100;
float moveSpeed = 5.0f;

// 下划线法（snake_case）：全小写，单词间下划线分隔
int player_score = 100;
float move_speed = 5.0f;

// 游戏开发中两种都可以，关键是保持一致
```

---

## 2.4 运算符

### 算术运算符

```cpp
#include <iostream>
using namespace std;

int main()
{
    int a = 10, b = 3;

    // -------- 算术运算 --------
    cout << "加法 a + b = " << a + b << endl;    // 13
    cout << "减法 a - b = " << a - b << endl;    // 7
    cout << "乘法 a * b = " << a * b << endl;    // 30
    cout << "除法 a / b = " << a / b << endl;    // 3（整数除法，取整）
    cout << "取余 a % b = " << a % b << endl;    // 1（余数）

    // -------- 自增自减 --------
    int c = 5;
    cout << "c++ = " << c++ << endl;   // 先输出 c，再加1 → 输出 5
    cout << "c = " << c << endl;       // 此时 c = 6

    int d = 5;
    cout << "++d = " << ++d << endl;   // 先加1，再输出 → 输出 6
    return 0;
}
```

### 赋值运算符

```cpp
int x = 10;

// 常见赋值运算符
x += 5;    // 等价于 x = x + 5，结果 x = 15
x -= 3;    // 等价于 x = x - 3，结果 x = 12
x *= 2;    // 等价于 x = x * 2，结果 x = 24
x /= 4;    // 等价于 x = x / 4，结果 x = 6
x %= 3;    // 等价于 x = x % 3，结果 x = 0
```

### 比较运算符（结果为 bool）

```cpp
cout << (5 > 3)  << endl;   // 1（真）
cout << (5 < 3)  << endl;   // 0（假）
cout << (5 == 5) << endl;   // 1（真，注意是两个等号）
cout << (5 != 3) << endl;   // 1（真，不等于）
cout << (5 >= 5) << endl;   // 1（真）
```

### 逻辑运算符

```cpp
// && 逻辑与（AND）：两边都为真才为真
// || 逻辑或（OR）：至少一边为真就为真
// !  逻辑非（NOT）：取反

bool a = true, b = false;

cout << (a && b) << endl;   // 0（与：假）
cout << (a || b) << endl;   // 1（或：真）
cout << (!a)     << endl;   // 0（非：假）

// 游戏中的用法示例：是否可以攻击
bool hasEnemy = true;
bool hasAmmo = true;
bool canAttack = hasEnemy && hasAmmo;  // 有敌人且有弹药才能攻击
```

---

## 2.5 类型转换

### 隐式转换（自动）

```cpp
int a = 10;
double b = a;        // int → double，自动转换，结果 b = 10.0

double c = 3.14;
int d = c;           // double → int，可能丢失小数部分，结果 d = 3
```

### 显式转换（强制）

```cpp
double pi = 3.14159;
int n = (int)pi;           // C 风格，结果 n = 3
int m = static_cast<int>(pi);  // C++ 风格，推荐，结果 m = 3
```

### 游戏开发中的类型转换

```cpp
// 射击游戏中：角度转弧度
int angleDegrees = 45;
double angleRadians = angleDegrees * 3.14159 / 180.0;

// 伤害计算中：float 转 int
float damage = 89.6f;
int finalDamage = (int)damage;   // 取整，结果 89
```

---

## 2.6 const：不变的值

### 什么是 const？

> `const` = constant（常量），定义后**不能改变**的值。

### 为什么需要 const？

```cpp
// 不用 const：危险！不小心改了重要值
float GRAVITY = 9.8f;
GRAVITY = 10.0f;    // ❌ 容易误改，导致物理逻辑出错

// 用 const：安全！编译器会阻止修改
const float GRAVITY = 9.8f;
// GRAVITY = 10.0f;  // ❌ 编译错误！无法修改常量
```

### const 在游戏中的应用

```cpp
const int MAX_PLAYERS = 4;          // 最大玩家数
const int MAX_LEVEL = 100;          // 最大等级
const float PI = 3.14159f;          // 圆周率
const string GAME_TITLE = "我的游戏";  // 游戏标题
```

---

## 2.7 动手练习 🧪

### 练习 1：变量定义
定义以下变量并输出：
- 角色名：`"林黛玉"`
- 年龄：`18`
- 生命值：`95.5f`
- 是否在战斗：`true`

### 练习 2：计算器
```cpp
// 完成以下代码，计算并输出：
int a = 15, b = 4;
// 求：a + b, a - b, a * b, a / b, a % b
```

### 练习 3：游戏伤害计算
```cpp
// 已知：
float baseDamage = 50.0f;   // 基础伤害
float critRate = 2.0f;      // 暴击倍率
bool isCrit = true;         // 是否暴击

// 计算最终伤害（如果暴击则 *2，否则不变）
// 用 ? : 三目运算符简化
```

### 练习 4：温度转换
华氏度转摄氏度公式：
```
C = (F - 32) * 5 / 9
```
输入 `float fahrenheit = 98.6;`，输出摄氏度。

---

## 2.8 本章小结

```
✅ 已掌握：
├── 变量的定义与赋值
├── 6 种基本数据类型（int, float, double, char, bool, string）
├── 算术、赋值、比较、逻辑运算符
├── 类型转换（隐式 + 显式）
├── const 常量的使用
└── 变量的命名规范

🔜 下章预告：
第三章：流程控制 —— 学会用 if/else、switch、for/while 让程序做判断和重复执行。
   这是游戏 AI、状态机、循环任务的核心基础！
```

---

_📚 参考资料：《C++ Primer》《Game Programming Patterns》_
