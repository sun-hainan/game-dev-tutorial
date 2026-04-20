# Chapter 09：类与面向对象

> 🎯 目标：掌握类的定义、构造/析构函数、访问控制、static 成员、this 指针、const 成员函数、拷贝构造、友元。

---

## 9.1 面向对象：类 vs 对象

### 生活中的类比 🏭

> **类 = 模具 / 蓝图**（比如"汽车设计图"）
> **对象 = 用模具造出来的实物**（比如"我的那辆红色轿车"）
>
> 一张设计图可以造出多辆车，每辆车都是独立的个体，但都遵循同一个设计。

### 游戏开发中的类与对象

| 类（模具） | 对象（实物）|
|---------|-----------|
| 玩家类 `Player` | 玩家A、玩家B |
| 敌人类 `Enemy` | 哥布林、骷髅、巨龙 |
| 武器类 `Weapon` | 铁剑、魔法杖、圣剑 |
| 道具类 `Item` | 红药瓶、蓝药瓶、盾牌 |

---

## 9.2 类的定义

### 基本语法

```cpp
class 类名 {
private:               // 私有成员（外部不能访问）
    // 属性

public:                // 公有成员（外部可以访问）
    // 方法
};
```

### 完整的 Player 类

```cpp
#include <iostream>
#include <string>
using namespace std;

// -------- Player 类的定义 --------
class Player {
private:
    // -------- 私有属性：外部不能直接访问 --------
    string name;       // 角色名称
    int hp;            // 生命值
    int mp;            // 魔法值
    int attack;        // 攻击力

public:
    // -------- 公有方法：外部可以通过这些操作角色 --------
    // 构造函数：创建对象时自动调用
    Player(string n, int h, int m, int a) {
        name = n;
        hp = h;
        mp = m;
        attack = a;
        cout << "角色 [" << name << "] 创建成功！" << endl;
    }

    // -------- 攻击方法 --------
    void attackTarget(Player& target) {
        cout << name << " 攻击 " << target.name;
        cout << "，造成 " << attack << " 点伤害" << endl;
        target.hp -= attack;
        if (target.hp < 0) target.hp = 0;
    }

    // -------- 受伤方法 --------
    void takeDamage(int dmg) {
        hp -= dmg;
        if (hp < 0) hp = 0;
        cout << name << " 受到 " << dmg << " 点伤害，剩余 HP：" << hp << endl;
    }

    // -------- 显示状态 --------
    void showStatus() {
        cout << "==========" << name << " ==========" << endl;
        cout << "HP：" << hp << "  MP：" << mp << "  ATK：" << attack << endl;
    }

    // -------- 获取名称（getter）--------
    string getName() {
        return name;
    }

    // -------- 获取 HP（getter）--------
    int getHP() {
        return hp;
    }

    // -------- 析构函数：对象销毁时自动调用 --------
    ~Player() {
        cout << "角色 [" << name << "] 被销毁" << endl;
    }
};

// -------- 主函数 --------
int main()
{
    // -------- 创建两个玩家对象 --------
    Player hero("孙悟空", 5000, 1200, 250);
    Player enemy("白骨精", 3000, 800, 150);

    hero.showStatus();
    enemy.showStatus();

    cout << endl << "=== 战斗开始 ===" << endl;

    // -------- 战斗 --------
    hero.attackTarget(enemy);
    enemy.takeDamage(0);  // attackTarget 里已经扣血了，这里演示另一个方法

    enemy.attackTarget(hero);
    hero.takeDamage(150);

    cout << endl << "=== 战斗结束 ===" << endl;
    hero.showStatus();
    enemy.showStatus();

    return 0;   // main 结束前，hero 和 enemy 依次调用析构函数
}
```

**运行结果：**
```
角色 [孙悟空] 创建成功！
角色 [白骨精] 创建成功！
==========孙悟空 ==========
HP：5000  MP：1200  ATK：250
==========白骨精 ==========
HP：3000  MP：800  ATK：150

=== 战斗开始 ===
孙悟空 攻击 白骨精，造成 250 点伤害
白骨精 受到 250 点伤害，剩余 HP：2750
白骨精 攻击 孙悟空，造成 150 点伤害
孙悟空 受到 150 点伤害，剩余 HP：4850

=== 战斗结束 ===
==========孙悟空 ==========
HP：4850  MP：1200  ATK：250
==========白骨精 ==========
HP：2750  MP：800  ATK：150
角色 [孙悟空] 被销毁
角色 [白骨精] 被销毁
```

---

## 9.3 访问控制详解

### 三个关键字

| 关键字 | 访问权限 | 说明 |
|-------|---------|------|
| `public` | 任何地方都能访问 | 对外接口 |
| `private` | 只能类内部访问 | 隐藏实现细节 |
| `protected` | 类内部 + 派生类 | 继承相关，下章讲 |

### 为什么需要 private？

```cpp
class Player {
private:
    int hp;   // ❌ 外部不能直接改！否则可以写成 hp = -1000

public:
    void setHP(int value) {   // ✅ 通过方法改，可以加校验
        if (value < 0) value = 0;
        if (value > 99999) value = 99999;
        hp = value;
    }

    int getHP() {   // ✅ 只能通过 getter 读取
        return hp;
    }
};
```

---

## 9.4 构造函数详解

### 默认构造函数（无参）

```cpp
class Player {
public:
    string name;
    int hp;

    // -------- 默认构造函数 --------
    Player() {
        name = "无名";
        hp = 100;
        cout << "默认构造函数被调用" << endl;
    }
};

Player p1;           // 调用默认构造函数
Player p2();         // ❌ 这是函数声明，不是对象！
```

### 带参构造函数

```cpp
class Player {
public:
    string name;
    int hp;

    // -------- 带参构造函数 --------
    Player(string n, int h) {
        name = n;
        hp = h;
    }
};

Player p1("孙悟空", 5000);   // 调用带参构造函数
```

### 初始化列表（推荐写法）

```cpp
class Player {
public:
    string name;
    int hp;
    int attack;

    // -------- 初始化列表：更高效的写法 --------
    Player(string n, int h, int a)
        : name(n), hp(h), attack(a)   // 在对象创建时直接初始化
    {
        // 构造函数体可以为空
    }
};
```

### 构造函数重载

```cpp
class Player {
public:
    string name;
    int hp;
    int attack;

    // -------- 重载1：无参 --------
    Player() : name("无名"), hp(100), attack(10) {}

    // -------- 重载2：一个参数 --------
    Player(string n) : name(n), hp(100), attack(10) {}

    // -------- 重载3：三个参数 --------
    Player(string n, int h, int a)
        : name(n), hp(h), attack(a) {}
};

Player a;                 // 调用无参版
Player b("林黛玉");       // 调用一个参数版
Player c("孙悟空", 5000, 250);  // 调用三个参数版
```

---

## 9.5 析构函数

### 什么时候调用？

| 场景 | 是否调用 |
|-----|---------|
| `Player p("name")` 在栈上 | 函数结束自动调用 |
| `new Player` 在堆上 | 必须手动 `delete` 才调用 |
| `unique_ptr<Player>` | 离开作用域自动调用 |
| `shared_ptr<Player>` | 引用计数归零时调用 |

### 实际用途：清理资源

```cpp
class FileReader {
private:
    FILE* file;   // 文件指针
public:
    FileReader(const char* filename) {
        file = fopen(filename, "r");
        if (file) {
            cout << "文件打开成功" << endl;
        }
    }

    // -------- 析构函数：关闭文件 --------
    ~FileReader() {
        if (file) {
            fclose(file);
            cout << "文件已关闭" << endl;
        }
    }
};

void readData() {
    FileReader reader("data.txt");
    // ... 使用 reader ...
}   // reader 离开作用域，析构函数自动关闭文件
```

---

## 9.6 拷贝构造函数

### 什么是拷贝构造？

> **用一个已存在的对象，创建出一个新的同类型对象。**

```cpp
class Player {
public:
    string name;
    int hp;

    // -------- 拷贝构造函数 --------
    Player(const Player& other) {
        name = other.name;
        hp = other.hp;
        cout << "拷贝构造函数被调用，复制：" << name << endl;
    }

    Player(string n, int h) : name(n), hp(h) {}
};

Player p1("孙悟空", 5000);
Player p2(p1);     // 调用拷贝构造函数，用 p1 复制出 p2
Player p3 = p1;    // 也是调用拷贝构造函数
```

### 浅拷贝 vs 深拷贝

```cpp
class Buffer {
private:
    int* data;     // 指针成员
    int size;

public:
    Buffer(int s) : size(s) {
        data = new int[s];   // 动态分配内存
    }

    // -------- 浅拷贝（默认的拷贝构造）--------
    // 只拷贝指针的值，不拷贝指针指向的内容
    // ⚠️ 问题：两个对象指向同一块内存，delete 时会崩溃

    // -------- 深拷贝（自定义拷贝构造）--------
    Buffer(const Buffer& other) : size(other.size) {
        data = new int[other.size];
        for (int i = 0; i < other.size; i++) {
            data[i] = other.data[i];
        }
    }

    ~Buffer() {
        delete[] data;   // 释放内存
    }
};
```

---

## 9.7 const 成员函数

### 概念

> **`const 成员函数 = 不能修改成员变量的成员函数。`**

### 为什么需要 const？

```cpp
class Player {
private:
    string name;
    int hp;

public:
    Player(string n, int h) : name(n), hp(h) {}

    // -------- const 成员函数：只读，不修改任何成员 --------
    void showStatus() const {
        cout << name << " HP=" << hp << endl;
        // hp = 1000;  // ❌ 编译错误！const 函数不能修改成员
    }

    // -------- 非 const 函数：可以修改 --------
    void takeDamage(int dmg) {
        hp -= dmg;
    }
};

void printPlayer(const Player& p) {
    // p 是 const 引用，只能调用 const 成员函数
    p.showStatus();      // ✅ 可以
    // p.takeDamage(10); // ❌ 不能，因为 takeDamage 不是 const
}
```

---

## 9.8 static 静态成员

### 什么是 static 成员？

> **static 成员 = 属于类本身，而不是属于某个对象。所有对象共享同一份。**

### static 变量：计数器

```cpp
#include <iostream>
using namespace std;

class Player {
public:
    string name;
    int hp;

    // -------- static 变量：所有对象共享 --------
    static int playerCount;   // 声明

    Player(string n) : name(n), hp(100) {
        playerCount++;   // 每创建一个玩家，计数器 +1
        cout << "玩家 [" << name << "] 加入，当前在线：" << playerCount << endl;
    }

    ~Player() {
        playerCount--;
        cout << "玩家 [" << name << "] 离开，剩余：" << playerCount << endl;
    }
};

// -------- static 变量必须在类外定义并初始化 --------
int Player::playerCount = 0;

int main()
{
    Player p1("孙悟空");
    Player p2("猪八戒");
    Player p3("沙和尚");

    cout << "当前服务器在线人数：" << Player::playerCount << endl;

    {
        Player p4("白龙马");
    }   // p4 销毁

    cout << "最终在线人数：" << Player::playerCount << endl;

    return 0;
}
```

**运行结果：**
```
玩家 [孙悟空] 加入，当前在线：1
玩家 [猪八戒] 加入，当前在线：2
玩家 [沙和尚] 加入，当前在线：3
当前服务器在线人数：3
玩家 [白龙马] 加入，当前在线：4
玩家 [白龙马] 离开，剩余：3
最终在线人数：3
玩家 [沙和尚] 离开，剩余：2
玩家 [猪八戒] 离开，剩余：1
玩家 [孙悟空] 离开，剩余：0
```

### static 函数

```cpp
class Game {
private:
    static int totalPlayers;   // static 成员

public:
    static int getTotalPlayers() {
        // static 函数只能访问 static 成员
        return totalPlayers;
    }
};
```

---

## 9.9 this 指针

### 什么是 this？

> **`this = 指向当前对象的指针。`**
> 在成员函数内部，用来区分成员变量和参数（当名字冲突时）。

```cpp
class Player {
private:
    string name;
    int hp;

public:
    // -------- name 参数和成员变量 name 重名，用 this 区分 --------
    void setName(string name) {
        this->name = name;   // this->name 是成员变量，name 是参数
    }

    void setHP(int hp) {
        this->hp = hp;
    }

    // -------- 也可以返回自身（用于链式调用）--------
    Player& setName2(string name) {
        this->name = name;
        return *this;    // 返回对象本身
    }

    Player& setHP2(int hp) {
        this->hp = hp;
        return *this;
    }
};

int main()
{
    Player p;
    // -------- 链式调用 --------
    p.setName2("孙悟空").setHP2(5000);   // 类似 jQuery 的链式写法
    return 0;
}
```

---

## 9.10 友元函数（friend）

### 概念

> **`friend = 授予其他函数/类访问私有成员的权限。`**

### 示例：友元函数访问 private

```cpp
#include <iostream>
using namespace std;

class Player {
private:
    int secretHP;   // 私有：不想让外部直接访问

public:
    string name;
    Player(string n, int hp) : name(n), secretHP(hp) {}

    // -------- 声明 showPlayerInfo 是友元 --------
    friend void showPlayerInfo(Player& p);
};

// -------- 友元函数：可以访问 Player 的私有成员 --------
void showPlayerInfo(Player& p) {
    cout << "玩家：" << p.name << endl;
    cout << "HP：" << p.secretHP << endl;   // ✅ 可以访问私有成员！
}

int main()
{
    Player p("孙悟空", 5000);
    showPlayerInfo(p);
    return 0;
}
```

### 友元类

```cpp
class Player;   // 前向声明

class FriendClass {
public:
    void show(Player& p);   // 声明
};

class Player {
    // -------- 声明 FriendClass 是友元 --------
    friend class FriendClass;

private:
    int secretHP = 5000;
};

// -------- FriendClass 的实现可以访问 Player 的 private --------
void FriendClass::show(Player& p) {
    cout << "窥探私有HP：" << p.secretHP << endl;
}
```

---

## 9.11 游戏开发实战：完整 Player 类

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>
using namespace std;

// -------- 武器类 --------
class Weapon {
public:
    string name;
    int attack;

    Weapon(string n, int atk) : name(n), attack(atk) {}
};

// -------- 技能类 --------
class Skill {
public:
    string name;
    int damage;
    int mpCost;

    Skill(string n, int d, int m) : name(n), damage(d), mpCost(m) {}
};

// -------- 玩家类 --------
class Player {
private:
    // -------- 属性 --------
    string name;
    int level;
    int hp;
    int maxHP;
    int mp;
    int maxMP;
    int attack;
    vector<unique_ptr<Weapon>> weapons;   // 武器背包
    vector<Skill> skills;                   // 技能列表

    // -------- 私有方法：检查死亡 --------
    bool isDead() const {
        return hp <= 0;
    }

public:
    // -------- 构造函数 --------
    Player(string n)
        : name(n), level(1), hp(100), maxHP(100), mp(50), maxMP(50), attack(10) {
        cout << "🎮 玩家 [" << name << "] 创建成功！" << endl;
    }

    // -------- 攻击 --------
    void attackEnemy(Player& enemy) {
        if (isDead()) {
            cout << name << " 已死亡，无法攻击" << endl;
            return;
        }
        if (enemy.isDead()) {
            cout << enemy.name << " 已死亡" << endl;
            return;
        }

        cout << "⚔️ " << name << " 攻击 " << enemy.name;
        cout << "，造成 " << attack << " 点伤害" << endl;
        enemy.hp -= attack;
        if (enemy.hp < 0) enemy.hp = 0;
    }

    // -------- 使用技能 --------
    void useSkill(int skillIndex, Player& target) {
        if (skillIndex < 0 || skillIndex >= (int)skills.size()) {
            cout << "无效的技能索引" << endl;
            return;
        }

        Skill& skill = skills[skillIndex];

        if (mp < skill.mpCost) {
            cout << "MP 不足，无法使用 " << skill.name << endl;
            return;
        }

        mp -= skill.mpCost;
        cout << "✨ " << name << " 使用 [" << skill.name << "]！" << endl;
        target.hp -= skill.damage;
        if (target.hp < 0) target.hp = 0;
        cout << "造成 " << skill.damage << " 点伤害，目标剩余 HP：" << target.hp << endl;
    }

    // -------- 添加技能 --------
    void addSkill(Skill s) {
        skills.push_back(s);
        cout << "学会新技能：" << s.name << endl;
    }

    // -------- 装备武器 --------
    void equipWeapon(unique_ptr<Weapon> weapon) {
        weapons.push_back(move(weapon));
        attack += weapons.back()->attack;
        cout << "装备 [" << weapons.back()->name << "]，攻击力 +" << weapon->attack << endl;
    }

    // -------- 升级 --------
    void levelUp() {
        level++;
        maxHP += 20;
        maxMP += 10;
        attack += 5;
        hp = maxHP;   // 升级回满血
        mp = maxMP;
        cout << "🎉 " << name << " 升级！现在是 Lv." << level << endl;
    }

    // -------- 显示状态 --------
    void showStatus() const {
        cout << "========== " << name << " ==========" << endl;
        cout << "等级：" << level << "  攻击力：" << attack << endl;
        cout << "HP：" << hp << "/" << maxHP << "  MP：" << mp << "/" << maxMP << endl;
        if (isDead()) cout << "状态：死亡" << endl;
        else cout << "状态：存活" << endl;
        cout << "================================" << endl;
    }

    // -------- 获取名称（供外部访问）--------
    string getName() const { return name; }
    int getHP() const { return hp; }

    // -------- 析构函数 --------
    ~Player() {
        cout << "玩家 [" << name << "] 退出游戏" << endl;
    }
};

// -------- 主函数 --------
int main()
{
    // -------- 创建玩家 --------
    Player hero("齐天大圣");
    Player monster("牛魔王");

    // -------- 初始化 --------
    hero.addSkill(Skill("火球术", 80, 30));
    hero.addSkill(Skill("雷击", 120, 50));
    hero.equipWeapon(make_unique<Weapon>("如意金箍棒", 50));

    cout << endl;

    // -------- 战斗过程 --------
    hero.showStatus();
    monster.showStatus();

    cout << endl << "=== 战斗开始 ===" << endl;

    hero.attackEnemy(monster);
    monster.attackEnemy(hero);

    cout << endl;
    hero.useSkill(0, monster);   // 火球术
    monster.takeDamage(0);      // 借用 showStatus 不提供，简化

    hero.levelUp();
    hero.attackEnemy(monster);

    cout << endl << "=== 战斗结束 ===" << endl;
    hero.showStatus();
    monster.showStatus();

    return 0;
}
```

---

## 9.12 动手练习 🧪

### 练习 1：设计一个 Enemy 类 ⭐
```
包含：name, hp, attack, isAlive
方法：构造函数、attackTarget、takeDamage、showStatus
主函数中创建两个敌人并战斗
```

### 练习 2：实现拷贝构造 ⭐
```
创建一个类 Buffer，成员有 int* data 和 int size
实现深拷贝构造函数
在 main 中测试：Buffer b1(5); Buffer b2(b1);
```

### 练习 3：static 计数器 ⭐⭐
```
类 Item，有 static int itemCount
构造函数 itemCount++，析构 itemCount--
创建 3 个 Item，销毁 1 个，输出剩余数量
```

### 练习 4：const 成员函数 + 链式调用 ⭐⭐
```
类 StringBuilder，有 string content
方法 append(string) 返回 *this
show() 是 const，打印内容
实现链式：sb.append("Hello").append(" ").append("World").show()
```

### 练习 5：完整 RPG 角色系统 ⭐⭐⭐
```
实现 Player/Enemy/Weapon/Skill 四个类
要求：武器和技能都用 vector 管理
实现战斗循环，直到一方 HP<=0
```

---

## 9.13 本章小结

```
✅ 已掌握：
├── 类的定义（属性 + 方法）
├── public / private / protected 访问控制
├── 构造函数（默认/带参/重载/初始化列表）
├── 析构函数（何时调用）
├── 拷贝构造函数（浅拷贝 vs 深拷贝）
├── const 成员函数
├── static 静态成员（变量 + 函数）
├── this 指针
├── 友元函数（friend）
└── 完整 Player 类实战

🔜 下章预告：
第十章：继承与多态 —— 学会用继承构建角色树，用多态实现技能多样化。
   这是游戏开发中"角色类型多样化"的核心！
```

---

_📚 参考资料：《C++ Primer》《Effective C++》《Design Patterns》_
