# Chapter 06：指针基础

> 🎯 目标：理解指针的本质——存储内存地址的变量。掌握 &（取地址）和 *（解引用）操作，为后续的动态内存管理和游戏对象管理打下基础。

---

## 6.1 指针：地址与引用的概念

### 生活中的类比 📬

> **快递柜：**
> - 快递柜有很多格子，每个格子有一个编号（地址）
> - 变量就像格子里的物品
> - 指针就像记录格子编号的纸条
>
> **关键理解：指针不存数据本身，而是存"数据放在哪里"**

### 内存地址的概念

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 变量在内存中有个地址 --------
    int playerLevel = 50;

    // -------- & 是"取地址运算符" --------
    cout << "playerLevel 的值：" << playerLevel << endl;
    cout << "playerLevel 的地址：" << &playerLevel << endl;
    // 地址是一个十六进制数，例如：0x61fe14

    // -------- 变量有地址，指针变量也有地址 --------
    int* ptr = &playerLevel;    // ptr 存储 playerLevel 的地址

    cout << "ptr 存储的地址：" << ptr << endl;
    cout << "ptr 自己的地址：" << &ptr << endl;   // ptr 变量自己的地址

    return 0;
}
```

### 指针的构成

```
┌────────────────────────────────────────────┐
│                  内存                        │
├──────────────┬─────────────────────────────┤
│  地址        │  内容                         │
├──────────────┼─────────────────────────────┤
│  0x61fe10    │  [ptr变量：存的是0x61fe14]    │
│  0x61fe14    │  50  ← playerLevel 的值      │
│  0x61fe18    │                              │
└──────────────┴─────────────────────────────┘

ptr 的类型是 int*（指向int的指针）
ptr 的值是 0x61fe14（playerLevel 的地址）
*ptr 的值是 50（解引用后得到实际数据）
```

---

## 6.2 指针的定义与基本操作

### 基本语法

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 定义指针 --------
    int* ptr1;          // 推荐：int* 表示"指向int的指针"
    int *ptr2;          // 也合法：风格偏好
    int * ptr3;        // 也合法，但容易混淆

    // -------- 基本类型 --------
    int* intPtr;        // 指向 int 的指针
    float* floatPtr;    // 指向 float 的指针
    char* charPtr;      // 指向 char 的指针
    double* doublePtr;  // 指向 double 的指针

    // -------- 定义时初始化（推荐）--------
    int playerHP = 100;
    int* hpPtr = &playerHP;    // 直接指向 playerHP

    // -------- & 取地址 --------
    cout << "playerHP 的地址：" << &playerHP << endl;
    cout << "hpPtr 的值：" << hpPtr << endl;
    // 两者相同！

    // -------- * 解引用（取值）--------
    cout << "*hpPtr = " << *hpPtr << endl;   // 输出 100

    // -------- 通过指针修改值 --------
    *hpPtr = 200;             // 把 hpPtr 指向的变量的值改成 200
    cout << "playerHP 现在是：" << playerHP << endl;   // 200

    return 0;
}
```

### 指针运算：解引用的含义

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 普通变量 vs 指针变量 --------
    int gold = 100;
    int* goldPtr = &gold;

    // -------- 用法对比 --------
    cout << "gold 的值：" << gold << endl;           // 100（直接读取）
    cout << "goldPtr 的值：" << goldPtr << endl;    // 地址（如 0x61fe14）
    cout << "*goldPtr 的值：" << *goldPtr << endl;  // 100（解引用=取值）

    // -------- 通过指针修改变量 --------
    cout << "\n===== 通过指针修改 =====" << endl;
    cout << "修改前 gold = " << gold << endl;
    *goldPtr = 500;            // 等价于 gold = 500
    cout << "修改后 gold = " << gold << endl;

    // -------- 多级指针 --------
    cout << "\n===== 多级指针 =====" << endl;
    int value = 42;
    int* p1 = &value;         // p1 是指向 value 的指针
    int** p2 = &p1;           // p2 是指向 p1 的指针（二级指针）
    int*** p3 = &p2;          // p3 是指向 p2 的指针（三级指针）

    cout << "value = " << value << endl;
    cout << "*p1 = " << *p1 << endl;
    cout << "**p2 = " << **p2 << endl;
    cout << "***p3 = " << ***p3 << endl;   // 都是 42

    return 0;
}
```

---

## 6.3 指针与数组的关系

### 核心概念

> **数组名就是数组首元素的地址！**

```cpp
#include <iostream>
using namespace std;

int main()
{
    int enemies[] = {100, 200, 300, 400, 500};

    // -------- 数组名 = 首元素地址 --------
    cout << "enemies 的值：" << enemies << endl;         // 数组首地址
    cout << "&enemies[0] 的值：" << &enemies[0] << endl;  // 也是首地址
    // 两者相等！

    // -------- 指针算术运算 --------
    int* ptr = enemies;           // 指针指向数组首元素

    cout << "\n===== 指针算术 =====" << endl;
    cout << "ptr 指向：" << *ptr << endl;        // 100（enemies[0]）

    ptr++;                        // 指针移动到下一个元素
    cout << "ptr++ 后指向：" << *ptr << endl;    // 200（enemies[1]）

    ptr += 2;                     // 指针再移动2个位置
    cout << "ptr+=2 后指向：" << *ptr << endl;   // 400（enemies[3]）

    // -------- 用指针遍历数组 --------
    cout << "\n===== 用指针遍历数组 =====" << endl;
    int* p = enemies;
    int length = sizeof(enemies) / sizeof(enemies[0]);

    for (int i = 0; i < length; i++) {
        cout << "enemies[" << i << "] = " << *(enemies + i) << endl;
        // *(enemies + i) 等价于 enemies[i]
    }

    return 0;
}
```

### 指针 vs 数组的对比

```cpp
#include <iostream>
using namespace std;

int main()
{
    int arr[] = {10, 20, 30, 40, 50};
    int* ptr = arr;           // 指针指向数组首元素

    // -------- [] 和 * 可以互换 --------
    cout << "arr[2] = " << arr[2] << endl;        // 30
    cout << "*(arr+2) = " << *(arr+2) << endl;    // 30
    cout << "ptr[2] = " << ptr[2] << endl;        // 30
    cout << "*(ptr+2) = " << *(ptr+2) << endl;    // 30

    // -------- 重要区别：sizeof --------
    cout << "\n===== 重要区别 =====" << endl;
    cout << "sizeof(arr) = " << sizeof(arr) << endl;   // 20（整个数组的大小）
    cout << "sizeof(ptr) = " << sizeof(ptr) << endl;   // 8（指针的大小，在64位系统上）

    // -------- ++ 的区别 --------
    cout << "\n===== ++ 的区别 =====" << endl;
    int arr2[] = {1, 2, 3};
    int* p1 = arr2;
    p1++;              // 指针移动，p1 指向 arr2[1]
    // arr2++;         // 错误！数组名是常量，不能 ++

    cout << "*p1 = " << *p1 << endl;   // 2

    return 0;
}
```

---

## 6.4 指针与函数（值传递 vs 指针传递）

### 值传递：副本不受影响

```cpp
#include <iostream>
using namespace std;

// -------- 值传递 --------
void tryChange(int num) {
    num = 999;
    cout << "函数内 num = " << num << endl;
}

int main()
{
    int hp = 100;
    cout << "调用前 hp = " << hp << endl;
    tryChange(hp);              // 传的是 hp 的副本
    cout << "调用后 hp = " << hp << endl;   // hp 不变！

    return 0;
}
```

**运行结果：**
```
调用前 hp = 100
函数内 num = 999
调用后 hp = 100    ← hp 没变！
```

### 指针传递：真正修改原数据

```cpp
#include <iostream>
using namespace std;

// -------- 指针传递 --------
void heal(int* hpPtr) {
    *hpPtr = *hpPtr + 50;    // 通过指针修改原数据
    cout << "治疗后 hp = " << *hpPtr << endl;
}

// -------- 指针传递：交换两个数 --------
void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main()
{
    int playerHP = 100;
    cout << "治疗前 hp = " << playerHP << endl;
    heal(&playerHP);              // 传地址
    cout << "治疗后 hp = " << playerHP << endl;   // 变了！

    cout << "\n===== 交换测试 =====" << endl;
    int x = 10, y = 20;
    cout << "交换前：x = " << x << ", y = " << y << endl;
    swap(&x, &y);
    cout << "交换后：x = " << x << ", y = " << y << endl;

    return 0;
}
```

**运行结果：**
```
治疗前 hp = 100
治疗后 hp = 150

===== 交换测试 =====
交换前：x = 10, y = 20
交换后：x = 20, y = 10
```

### 游戏开发实战：攻击函数（指针版）

```cpp
#include <iostream>
#include <string>
#include <cstdlib>
#include <ctime>
using namespace std;

// -------- 造成伤害（指针传递）--------
void dealDamage(int* hpPtr, int damage) {
    *hpPtr -= damage;        // 扣血
    if (*hpPtr < 0) *hpPtr = 0;
    cout << "💥 造成 " << damage << " 点伤害！" << endl;
    cout << "敌人剩余 HP：" << *hpPtr << endl;
}

// -------- 判断是否死亡 --------
bool isDead(int* hpPtr) {
    return (*hpPtr <= 0);
}

// -------- 主函数 --------
int main()
{
    srand(time(0));

    // -------- 定义敌人 --------
    string enemyName = "暗影巨龙";
    int enemyHP = 500;
    int enemyDefense = 30;

    // -------- 玩家攻击 --------
    int playerAttack = 80;

    cout << "===== 战斗开始 =====" << endl;
    cout << enemyName << " 出现！HP = " << enemyHP << endl;
    cout << "玩家攻击力：" << playerAttack << endl;

    int round = 1;
    while (!isDead(&enemyHP)) {
        cout << "\n--- 第 " << round << " 回合 ---" << endl;

        // 计算实际伤害（考虑防御）
        int damage = playerAttack - enemyDefense + rand() % 20;
        if (damage < 0) damage = 0;

        dealDamage(&enemyHP, damage);   // 传指针，真正扣血

        round++;
        if (round > 20) break;   // 防止无限循环
    }

    if (isDead(&enemyHP)) {
        cout << "\n🎉 " << enemyName << " 被击败！" << endl;
    }

    return 0;
}
```

---

## 6.5 指针的常见用法

### NULL 指针

> **NULL 指针：** 指针不指向任何有效地址，是一种"空状态"的表示。

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- NULL 指针 --------
    int* nullPtr = NULL;          // C 风格
    int* nullPtr2 = nullptr;      // C++11 推荐！

    // -------- 检查是否为空 --------
    if (nullPtr == nullptr) {
        cout << "nullPtr 是空指针" << endl;
    }

    // -------- 使用前必须检查！--------
    int value = 100;
    int* ptr = nullptr;

    // 安全检查
    if (ptr != nullptr) {
        cout << *ptr << endl;   // 只有非空时才解引用
    } else {
        cout << "指针为空，不能解引用" << endl;
    }

    // -------- 指针作为函数返回值表示错误 --------
    int* findEnemy(string name) {
        if (name == "Boss") {
            static int bossHP = 1000;   // 静态变量，函数结束后不销毁
            return &bossHP;
        }
        return nullptr;   // 找不到返回空指针
    }

    int* result = findEnemy("Boss");
    if (result != nullptr) {
        cout << "找到 Boss，HP = " << *result << endl;
    } else {
        cout << "敌人不存在" << endl;
    }

    return 0;
}
```

### void 指针

> **void*：** 通用指针，可以指向任意类型，但不能直接解引用（需要强制转换）。

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- void 指针：可以存任意类型的地址 --------
    int intVal = 42;
    float floatVal = 3.14f;
    char charVal = 'A';

    void* genericPtr;

    // void 指针可以指向 int
    genericPtr = &intVal;
    cout << "void* 指向 int：" << *(static_cast<int*>(genericPtr)) << endl;

    // void 指针可以指向 float
    genericPtr = &floatVal;
    cout << "void* 指向 float：" << *(static_cast<float*>(genericPtr)) << endl;

    // -------- 游戏开发中的用法 --------
    // 游戏引擎中常用 void* 存储任意类型的对象指针
    // 然后根据需要强制转换回具体类型

    cout << "\n===== 游戏开发示例 =====" << endl;
    cout << "void* 常用于回调函数、事件系统" << endl;
    cout << "可以存储任何指针，但使用时必须知道实际类型" << endl;

    return 0;
}
```

### const 与指针

```cpp
#include <iostream>
using namespace std;

int main()
{
    int playerHP = 100;
    const int MAX_HP = 9999;

    // -------- 指向常量的指针（不能通过它修改数据）--------
    const int* ptr1 = &playerHP;   // 不能：*ptr1 = 200;
    // 但可以直接修改 playerHP = 200;

    // -------- 常量指针（指针本身不能改变）--------
    int* const ptr2 = &playerHP;  // 不能：ptr2 = &otherVar;
    *ptr2 = 200;                   // 但可以修改值

    // -------- 指向常量的常量指针（都不能改）--------
    const int* const ptr3 = &MAX_HP;   // 都不能改

    cout << "playerHP = " << playerHP << endl;

    return 0;
}
```

---

## 6.6 游戏开发实战：敌人状态切换

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <ctime>
#include <cstdlib>
using namespace std;

// -------- 敌人状态枚举 --------
enum EnemyState {
    IDLE,       // 空闲
    PATROL,     // 巡逻
    CHASE,      // 追击
    ATTACK,     // 攻击
    DEAD        // 死亡
};

// -------- 敌人结构体 --------
struct Enemy {
    string name;
    int hp;
    int maxHP;
    int x, y;               // 位置
    EnemyState state;       // 当前状态
};

// -------- 获取状态名称 --------
string getStateName(EnemyState state) {
    switch (state) {
        case IDLE:   return "空闲";
        case PATROL: return "巡逻";
        case CHASE:  return "追击";
        case ATTACK: return "攻击";
        case DEAD:   return "死亡";
        default:     return "未知";
    }
}

// -------- 切换状态 --------
void changeState(Enemy* enemy, EnemyState newState) {
    if (enemy == nullptr || enemy->state == DEAD) {
        return;   // 死人不能变状态
    }

    cout << enemy->name << "：";
    cout << getStateName(enemy->state) << " → " << getStateName(newState) << endl;
    enemy->state = newState;
}

// -------- 扣血 --------
void takeDamage(Enemy* enemy, int damage) {
    if (enemy == nullptr || enemy->state == DEAD) return;

    enemy->hp -= damage;
    cout << "💥 " << enemy->name << " 受到 " << damage << " 点伤害！" << endl;
    cout << "   HP：" << enemy->hp << "/" << enemy->maxHP << endl;

    if (enemy->hp <= 0) {
        enemy->hp = 0;
        changeState(enemy, DEAD);
    } else if (enemy->hp < enemy->maxHP * 0.3) {
        // 血量低于30%，进入恐惧状态（逃跑）
        if (enemy->state != CHASE && enemy->state != DEAD) {
            changeState(enemy, CHASE);
        }
    }
}

// -------- 显示敌人信息 --------
void showEnemy(Enemy* enemy) {
    cout << "===== " << enemy->name << " =====" << endl;
    cout << "HP：" << enemy->hp << "/" << enemy->maxHP << endl;
    cout << "状态：" << getStateName(enemy->state) << endl;
    cout << "位置：(" << enemy->x << ", " << enemy->y << ")" << endl;
}

// -------- 主函数 --------
int main()
{
    srand(time(0));

    // -------- 创建敌人 --------
    Enemy boss;
    boss.name = "暗影巨龙";
    boss.hp = 1000;
    boss.maxHP = 1000;
    boss.x = 10;
    boss.y = 20;
    boss.state = IDLE;

    // -------- 初始状态 --------
    cout << "===== 战斗模拟 =====" << endl;
    showEnemy(&boss);

    // -------- 玩家攻击 --------
    cout << "\n----- 玩家攻击 ----- " << endl;
    int damage = rand() % 200 + 100;   // 100-300 随机伤害
    takeDamage(&boss, damage);
    changeState(&boss, ATTACK);        // 切换到攻击状态

    // -------- 继续攻击 --------
    cout << "\n----- 再次攻击 ----- " << endl;
    damage = rand() % 200 + 100;
    takeDamage(&boss, damage);

    if (boss.state != DEAD) {
        // 敌人反击
        cout << "\n----- 敌人反击 ----- " << endl;
        cout << boss.name << " 发动攻击！" << endl;
    }

    // -------- 总攻 --------
    cout << "\n----- 总攻！----- " << endl;
    damage = 800;
    takeDamage(&boss, damage);

    showEnemy(&boss);

    return 0;
}
```

---

## 6.7 常见错误

### 错误清单

| 错误类型 | 示例 | 后果 |
|---------|------|------|
| 未初始化指针 | `int* ptr; *ptr = 100;` | 程序崩溃（访问未知内存） |
| NULL 解引用 | `int* ptr = NULL; *ptr;` | 程序崩溃 |
| 野指针 | `int* ptr = (int*)100;` | 访问非法地址 |
| 内存泄漏 | `new int;` 没 delete | 内存浪费 |
| 重复 delete | `delete p; delete p;` | 程序崩溃 |
| 指针类型不匹配 | `float* fp = &intVar;` | 编译错误 |

### 正确 vs 错误示例

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 错误1：未初始化指针 --------
    // int* badPtr;           // 危险！ptr 的值是随机的
    // *badPtr = 100;         // 崩溃！访问未知内存

    // -------- 正确：初始化为 NULL --------
    int* safePtr = nullptr;   // 安全！明确为空

    // -------- 使用前检查 --------
    if (safePtr != nullptr) {
        cout << *safePtr << endl;
    } else {
        cout << "指针为空" << endl;
    }

    // -------- 错误2：指针类型不匹配 --------
    double price = 99.9;
    // int* intPtr = &price;  // 编译错误！类型不匹配

    // -------- 正确：类型匹配 --------
    double* doublePtr = &price;   // 正确！

    // -------- 数组指针的注意事项 --------
    int scores[] = {1, 2, 3, 4, 5};
    int* arrPtr = scores;          // 正确：指向数组首元素

    cout << "scores[0] = " << *arrPtr << endl;   // 1
    cout << "scores[2] = " << *(arrPtr + 2) << endl;   // 3

    return 0;
}
```

---

## 6.8 动手练习 🧪

### 练习 1：指针基础
定义一个 int 变量 `level = 50`，定义一个指针 `ptr` 指向它，用指针打印变量的值。⭐

### 练习 2：指针交换
写一个函数 `swap(int* a, int* b)`，交换两个整数的值，验证交换是否成功。⭐

### 练习 3：指针遍历数组
定义一个 `int arr[] = {10, 20, 30, 40, 50}`，用指针遍历并打印所有元素（不使用 []）。⭐⭐

### 练习 4：敌人血量管理
用指针实现：定义多个敌人血量变量，编写函数 `takeDamage(int* hp, int damage)` 和 `heal(int* hp, int amount)`，演示指针传递修改原值。⭐⭐

### 练习 5：状态机
用 enum 和指针实现一个简单的游戏AI状态机，定义状态转换函数 `changeState(Enemy* enemy, State newState)`，根据HP阈值自动切换状态。⭐⭐⭐

---

## 本章小结

```
✅ 已掌握：
├── 指针的概念（存储地址的变量）
├── & 取地址运算符
├── * 解引用运算符
├── 指针与数组的关系（数组名=首地址）
├── 指针算术（++、+、-）
├── 值传递 vs 指针传递
├── NULL 指针与 nullptr
├── void* 指针
├── const 与指针
└── 指针实现状态切换实战

🔜 下章预告：
第七章：指针进阶与内存管理 —— 学会使用 new/delete 动态分配内存，理解内存泄漏的危害，掌握智能指针，让游戏对象管理更安全。
```

---

_📚 参考资料：《C++ Primer》《Effective C++》《C++ Memory》《游戏编程模式》_
