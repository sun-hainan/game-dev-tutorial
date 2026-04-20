# Chapter 07：指针进阶与内存管理

> 🎯 目标：掌握 `new`/`delete` 动态内存管理，理解内存泄漏原因并避免，掌握智能指针，了解函数指针。

---

## 7.1 动态内存基础：new 与 delete

### 生活中的类比 🏗️

> **租房 vs 买房：**
> - **静态分配**（栈上）= 买房：程序启动时分配好，大小固定，不能改变
> - **动态分配**（堆上）= 租房：需要时申请，退房时释放，灵活但需要手动管理
>
> **动态内存 = 按需租房，手动退房。**

### new：申请内存

```cpp
int* p = new int;        // 申请一个 int 的空间
*p = 42;                 // 给这块空间写入数据
cout << *p << endl;      // 输出：42
```

### delete：释放内存

```cpp
delete p;                // 释放内存
p = nullptr;             // 避免悬空指针
```

### 完整的申请-使用-释放流程

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 申请一个整数 --------
    int* p = new int(100);   // 申请并初始化为 100
    cout << "值：" << *p << endl;

    // -------- 修改值 --------
    *p = 200;
    cout << "修改后：" << *p << endl;

    // -------- 释放内存（必须！）--------
    delete p;
    p = nullptr;

    cout << "内存已释放" << endl;
    return 0;
}
```

---

## 7.2 动态数组

### 为什么需要动态数组？

> 静态数组的大小必须是编译时常量。但游戏中的敌人数量、道具数量是**运行时**才知道的。

### 动态数组的创建与释放

```cpp
#include <iostream>
using namespace std;

int main()
{
    int n = 0;
    cout << "请输入敌人数量：";
    cin >> n;

    // -------- 动态申请一个数组 --------
    int* enemies = new int[n];   // n 是变量，大小在运行时决定

    // -------- 使用数组 --------
    for (int i = 0; i < n; i++) {
        enemies[i] = (i + 1) * 100;   // 初始化：100, 200, 300...
        cout << "敌人 " << (i + 1) << " 血量：" << enemies[i] << endl;
    }

    // -------- 释放动态数组（必须加 []）--------
    delete[] enemies;
    enemies = nullptr;

    return 0;
}
```

**运行结果：**
```
请输入敌人数量：3
敌人 1 血量：100
敌人 2 血量：200
敌人 3 血量：300
内存已释放
```

### ⚠️ 释放时 `delete` vs `delete[]`

| 类型 | 释放方式 | 说明 |
|-----|---------|------|
| `new int` | `delete p` | 单个对象 |
| `new int[n]` | `delete[] p` | 数组，必须加 `[]` |

```cpp
// ❌ 错误示例
int* p = new int[10];
delete p;           // 危险！只释放了第一个元素

// ✅ 正确示例
int* p = new int[10];
delete[] p;         // 释放整个数组
```

---

## 7.3 内存泄漏与避免

### 什么是内存泄漏？

> **内存泄漏 = 申请了内存但忘了释放，导致内存浪费。**
> 就像租了房子不交租也不退房，房东再也租不出去。

### 内存泄漏的常见场景

```cpp
// -------- 场景1：忘了 delete --------
void leakExample() {
    int* p = new int[100];
    // ... 使用 p ...
    // ❌ 函数结束前没有 delete
    return;   // 内存泄漏！
}

// -------- 场景2：指针被覆盖 --------
void leakExample2() {
    int* p = new int(10);
    p = new int(20);   // 第一个 int 的地址丢了，内存泄漏！
    delete p;
}

// -------- 场景3：在异常情况下没有释放 --------
void leakWithException() {
    int* p = new int[100];
    // if (something wrong) { return; }  // 如果这里return，内存泄漏
    delete[] p;
}
```

### 避免内存泄漏的方法

```cpp
// 方法1：手动确保每条 new 都有 delete
void safeExample() {
    int* p = new int[100];
    // ... 使用 p ...
    delete[] p;    // 确保释放
}

// 方法2：使用智能指针（推荐！）
#include <memory>

void smartPointerExample() {
    unique_ptr<int[]> p(new int[100]);
    // unique_ptr 在离开作用域时自动释放内存
    // 不需要手动 delete
}
```

---

## 7.4 智能指针

### 什么是智能指针？

> **智能指针 = 自动管理内存的指针。** 你只管用，不用记着释放。

### unique_ptr：独占所有权

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main()
{
    // -------- unique_ptr：独占，不能共享 --------
    unique_ptr<int> p1(new int(42));
    cout << "p1 值：" << *p1 << endl;

    // unique_ptr 离开作用域自动释放内存
    // 不需要 delete

    // -------- 可以用 make_unique 更安全 --------
    auto p2 = make_unique<int>(100);
    cout << "p2 值：" << *p2 << endl;

    return 0;
}
```

### shared_ptr：共享所有权

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main()
{
    // -------- shared_ptr：引用计数，共享所有权 --------
    auto player1 = make_shared<int>(50);
    {
        auto player2 = player1;   // 引用计数 +1
        cout << "player2：" << *player2 << endl;
        cout << "引用计数：" << player1.use_count() << endl;  // 2
    }   // player2 销毁，引用计数 -1

    cout << "player1：" << *player1 << endl;   // 50
    cout << "引用计数：" << player1.use_count() << endl;      // 1

    return 0;
}
```

### 智能指针用于动态数组

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main()
{
    // unique_ptr 管理动态数组
    unique_ptr<int[]> enemies(new int[5]);

    for (int i = 0; i < 5; i++) {
        enemies[i] = (i + 1) * 100;
        cout << "敌人 " << (i + 1) << " HP: " << enemies[i] << endl;
    }

    // unique_ptr 离开作用域自动 delete[]

    return 0;
}
```

### 游戏开发实战：敌人管理器

```cpp
#include <iostream>
#include <memory>
#include <vector>
using namespace std;

// -------- 敌人类 --------
class Enemy {
public:
    string name;
    int hp;
    Enemy(string n, int h) : name(n), hp(h) {
        cout << "敌人 [" << name << "] 生成，HP=" << hp << endl;
    }
    ~Enemy() {
        cout << "敌人 [" << name << "] 被销毁" << endl;
    }
};

int main()
{
    // -------- 用 unique_ptr 管理敌人列表 --------
    vector<unique_ptr<Enemy>> enemies;

    // 生成敌人
    enemies.push_back(make_unique<Enemy>("哥布林", 100));
    enemies.push_back(make_unique<Enemy>("骷髅", 150));
    enemies.push_back(make_unique<Enemy>("巨龙", 500));

    cout << "--- 战斗开始 ---" << endl;

    // 击败第一个敌人（unique_ptr 离开作用域自动销毁）
    enemies.erase(enemies.begin());

    cout << "--- 剩余敌人 ---" << endl;

    return 0;   // 离开 main，vector 清空，所有敌人自动销毁
}
```

**运行结果：**
```
敌人 [哥布林] 生成，HP=100
敌人 [骷髅] 生成，HP=150
敌人 [巨龙] 生成，HP=500
--- 战斗开始 ---
敌人 [哥布林] 被销毁
--- 剩余敌人 ---
敌人 [骷髅] 被销毁
敌人 [巨龙] 被销毁
```

---

## 7.5 二级指针

### 概念

> **二级指针 = 指向指针的指针。** 一级指针是变量的地址，二级指针是指针的地址。

### 图解

```
变量 a      值：100
地址：0x1000

一级指针 p  值：0x1000（指向 a）
地址：0x2000

二级指针 pp 值：0x2000（指向 p）
地址：0x3000
```

### 代码示例

```cpp
#include <iostream>
using namespace std;

int main()
{
    int a = 100;

    // -------- 一级指针 --------
    int* p = &a;
    cout << "a 的地址：" << p << endl;

    // -------- 二级指针 --------
    int** pp = &p;     // pp 指向 p 的地址
    cout << "p 的地址：" << pp << endl;
    cout << "通过 pp 访问 a：" << **pp << endl;   // 100

    // -------- 修改 p 的值（通过 pp）--------
    *p = 200;
    cout << "a 变为：" << a << endl;    // 200

    **pp = 300;
    cout << "a 变为：" << a << endl;    // 300

    return 0;
}
```

### 游戏开发实战：动态改变指针指向

```cpp
#include <iostream>
using namespace std;

// 当前目标
int main()
{
    int monster1HP = 100;
    int monster2HP = 200;

    int* currentTarget = &monster1HP;   // 当前目标是怪物1
    cout << "当前目标 HP：" << *currentTarget << endl;

    // -------- 切换目标（改指针的指向）--------
    int** targetPtr = &¤tTarget;   // 传入指针的地址
    *targetPtr = &monster2HP;       // 修改指针本身

    cout << "切换后目标 HP：" << *currentTarget << endl;  // 200

    return 0;
}
```

---

## 7.6 函数指针

### 什么是函数指针？

> **函数指针 = 指向函数的指针。** 可以把函数作为参数传递，也可以在运行时决定调用哪个函数。

### 基本语法

```cpp
// 函数原型
int add(int a, int b) { return a + b; }

// 函数指针定义
int (*funcPtr)(int, int);   // 指向返回 int、接受两个 int 的函数

// 赋值
funcPtr = add;              // 函数名就是函数的地址
// 或者
funcPtr = &add;             // & 也可以省略
```

### 游戏开发实战：技能系统

```cpp
#include <iostream>
#include <vector>
using namespace std;

// -------- 技能函数 --------
void skillFireball(int& targetHP) {
    cout << "🔥 释放火球术！" << endl;
    targetHP -= 50;
}

void skillIcebolt(int& targetHP) {
    cout << "❄️ 释放寒冰箭！" << endl;
    targetHP -= 30;
}

void skillHeal(int& playerHP) {
    cout << "💚 释放治疗术！" << endl;
    playerHP += 40;
}

// -------- 技能类型：参数是一个引用 int --------
typedef void (*SkillFunc)(int&);

int main()
{
    // -------- 技能列表 --------
    vector<SkillFunc> skills;
    skills.push_back(skillFireball);
    skills.push_back(skillIcebolt);
    skills.push_back(skillHeal);

    int playerHP = 100;
    int enemyHP = 200;

    cout << "=== 玩家行动 ===" << endl;

    // -------- 使用函数指针执行技能 --------
    skills[0](enemyHP);    // 火球术
    cout << "敌人 HP：" << enemyHP << endl;

    skills[2](playerHP);   // 治疗术
    cout << "玩家 HP：" << playerHP << endl;

    return 0;
}
```

---

## 7.7 游戏开发实战：对象池概念

### 什么是对象池？

> **对象池 = 预先创建好一批对象，用时取，不用时还，避免频繁 new/delete。**

```cpp
#include <iostream>
#include <memory>
#include <vector>
using namespace std;

// -------- 子弹类 --------
class Bullet {
public:
    bool active;   // 是否在使用中
    int damage;

    Bullet() : active(false), damage(10) {}

    void fire(int d) {
        damage = d;
        active = true;
        cout << "子弹发射，伤害：" << damage << endl;
    }

    void recycle() {
        active = false;
        cout << "子弹回收" << endl;
    }
};

// -------- 对象池 --------
class BulletPool {
private:
    vector<unique_ptr<Bullet>> bullets;
    const int POOL_SIZE = 10;

public:
    BulletPool() {
        // -------- 预先创建子弹 --------
        for (int i = 0; i < POOL_SIZE; i++) {
            bullets.push_back(make_unique<Bullet>());
        }
        cout << "对象池初始化，创建 " << POOL_SIZE << " 发子弹" << endl;
    }

    Bullet* getBullet() {
        // -------- 找一个空闲的子弹 --------
        for (auto& bullet : bullets) {
            if (!bullet->active) {
                return bullet.get();
            }
        }
        cout << "对象池耗尽！" << endl;
        return nullptr;
    }
};

int main()
{
    BulletPool pool;

    // -------- 从对象池取出子弹 --------
    Bullet* b1 = pool.getBullet();
    b1->fire(20);

    Bullet* b2 = pool.getBullet();
    b2->fire(15);

    // -------- 回收子弹 --------
    b1->recycle();
    b2->recycle();

    return 0;
}
```

**运行结果：**
```
对象池初始化，创建 10 发子弹
子弹发射，伤害：20
子弹发射，伤害：15
子弹回收
子弹回收
```

---

## 7.8 常见错误与调试

| 错误现象 | 原因 | 解决方法 |
|---------|------|---------|
| 程序崩溃/内存错误 | `delete` 多次或 `delete` 后继续使用 | `delete` 后立即 `= nullptr` |
| 内存持续增长 | 内存泄漏（`new` 后未 `delete`）| 使用智能指针 |
| `delete[]` 写成 `delete` | 释放数组时忘记 `[]` | 记住：数组必须 `delete[]` |
| 悬空指针 | `delete` 后继续访问 | `delete` 后立即 `= nullptr` |
| `use_count` 异常 | `shared_ptr` 循环引用 | 使用 `weak_ptr` 打破循环 |

---

## 7.9 动手练习 🧪

### 练习 1：动态创建角色数组 ⭐
```cpp
// 输入 N，创建 N 个敌人的动态数组
// 每敌人 HP = 100 * (i+1)
// 打印后释放内存
```

### 练习 2：用 unique_ptr 改写 ⭐
将以上代码改用 `unique_ptr<int[]>` 管理，不再手动 delete。

### 练习 3：shared_ptr 引用计数 ⭐⭐
```cpp
// 创建两个 shared_ptr 指向同一个对象
// 打印引用计数
// 然后一个 reset，再打印引用计数
```

### 练习 4：函数指针实现技能系统 ⭐⭐
```cpp
// 定义至少 3 个技能函数
// 写一个 useSkill(skillFunc, targetHP) 函数
// 在 main 中让玩家选择使用哪个技能
```

### 练习 5：简化版对象池 ⭐⭐⭐
实现一个简单的整数对象池，预先分配 5 个整数，用 `get()` 获取，`release()` 归还。

---

## 7.10 本章小结

```
✅ 已掌握：
├── new / delete 动态内存分配
├── 动态数组的创建与释放（delete[]）
├── 内存泄漏的原因与避免
├── unique_ptr（独占所有权）
├── shared_ptr（引用计数）
├── 二级指针的概念
├── 函数指针
└── 对象池概念

🔜 下章预告：
第八章：结构体与联合体 —— 把不同类型的数据组织在一起，构建 RPG 角色属性、背包道具等复杂数据结构。
```

---

_📚 参考资料：《C++ Primer》《Effective C++》《Game Programming Patterns》_
