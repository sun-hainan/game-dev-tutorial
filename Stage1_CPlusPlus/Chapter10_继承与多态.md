# Chapter 10：继承与多态

> 🎯 目标：掌握类的继承关系构建角色体系，理解多态的实现原理（虚函数），用抽象类和接口实现灵活的技能系统。

---

## 10.1 继承的概念

### 生活中的类比 🧬

> **继承 = 父子关系：**
> - 儿子继承了父亲的姓氏、房产
> - 儿子可以在父亲基础上**新增**自己的东西（车子、存款）
> - 儿子也可以**覆盖/重写**父亲的一些特征（不同的职业）
>
> **子类继承父类 = 在父类的基础上扩展自己的专属特性。**

### 游戏开发中的继承关系

```
        Character（角色基类）
        ├── name, hp, attack
        ├── attack(), takeDamage()
        │
        ├── Player（玩家）
        │   ├── level, mp, skills
        │   ├── useSkill(), levelUp()
        │   └── equipWeapon(), addItem()
        │
        ├── Enemy（敌人基类）
        │   ├── isAggressive, dropItems
        │   ├── patrol(), chase()
        │   │
        │   ├── Monster（普通怪物）
        │   │   ├── monsterType
        │   │   └── basicAI()
        │   │
        │   └── Boss（BOSS）
        │       ├── specialSkills
        │       └── bossAI()
        │
        └── NPC（路人）
            ├── dialogue
            └── questInfo
```

---

## 10.2 继承的基本语法

```cpp
// -------- 父类（基类）--------
class Character {
public:
    string name;
    int hp;
    int attack;

    void attackTarget(Character& target) {
        cout << name << " 攻击 " << target.name;
        cout << "，造成 " << attack << " 点伤害" << endl;
        target.hp -= attack;
    }
};

// -------- 子类（派生类）--------
class Player : public Character {
public:
    int level;
    int mp;

    // -------- 子类新增的方法 --------
    void useMagic(Character& target) {
        cout << name << " 施放魔法" << endl;
        target.hp -= attack * 2;
    }
};

// -------- 主函数 --------
int main()
{
    Player hero;
    hero.name = "孙悟空";    // 继承自 Character
    hero.hp = 5000;          // 继承自 Character
    hero.attack = 250;       // 继承自 Character
    hero.level = 85;         // Player 自有
    hero.mp = 1200;          // Player 自有

    hero.attackTarget(hero); // 继承的方法可用
    hero.useMagic(hero);     // 子类自己的方法

    return 0;
}
```

### 继承的访问权限

| 父类成员在子类的访问性 | `public` 继承 | `protected` 继承 | `private` 继承 |
|---------------------|-------------|-----------------|---------------|
| `public` 成员 | `public` | `protected` | `private` |
| `protected` 成员 | `protected` | `protected` | `private` |
| `private` 成员 | 不可访问 | 不可访问 | 不可访问 |

> **游戏开发中，`public` 继承最常用。**

---

## 10.3 构造函数的调用顺序

### 顺序规则

> **父类构造函数先于子类构造函数执行，析构函数顺序相反。**

```cpp
#include <iostream>
using namespace std;

class Base {
public:
    Base() {
        cout << "1. 父类构造函数" << endl;
    }
    ~Base() {
        cout << "5. 父类析构函数" << endl;
    }
};

class Derived : public Base {
public:
    Derived() {
        cout << "2. 子类构造函数" << endl;
    }
    ~Derived() {
        cout << "4. 子类析构函数" << endl;
    }
};

int main()
{
    cout << "=== 创建对象 ===" << endl;
    Derived obj;

    cout << "=== 销毁对象 ===" << endl;
    return 0;
}
```

**运行结果：**
```
=== 创建对象 ===
1. 父类构造函数
2. 子类构造函数
=== 销毁对象 ===
4. 子类析构函数
5. 父类析构函数
```

### 子类构造函数如何调用父类构造函数？

```cpp
class Player : public Character {
public:
    int level;

    // -------- 调用父类带参构造 --------
    Player(string n, int h, int atk, int lv)
        : Character(n, h, atk)   // 显式调用父类构造函数
        , level(lv)              // 初始化自己的成员
    {
        cout << "Player 构造函数" << endl;
    }
};

class Character {
public:
    string name;
    int hp;
    int attack;

    Character(string n, int h, int atk)
        : name(n), hp(h), attack(atk) {
        cout << "Character 构造函数：" << name << endl;
    }
};
```

---

## 10.4 多态：虚函数

### 为什么需要多态？

> **同一个方法调用，不同对象有不同行为。**

```cpp
// 不使用多态：每种敌人都单独处理
void attackAllMonsters(Monster& m, Boss& b, Ghost& g) {
    m.attack();   // 怪物攻击
    b.attack();   // Boss攻击
    g.attack();   // 幽灵攻击
    // 如果再加新敌人，又要改代码！
}

// 使用多态：一个函数处理所有敌人
void attackAll(Enemy& e) {   // 引用基类
    e.attack();   // 自动调用实际类型的 attack()
}
```

### 虚函数实现多态

```cpp
#include <iostream>
using namespace std;

// -------- 基类：敌人 --------
class Enemy {
public:
    string name;
    int hp;
    int attack;

    Enemy(string n, int h, int atk) : name(n), hp(h), attack(atk) {}

    // -------- 虚函数：允许子类重写 --------
    virtual void attackTarget(Character& target) {
        cout << name << " 进行普通攻击，造成 " << attack << " 点伤害" << endl;
        target.hp -= attack;
    }

    // -------- 虚析构函数（重要！）--------
    virtual ~Enemy() {
        cout << name << " 被销毁" << endl;
    }
};

// -------- 子类：普通怪物 --------
class Monster : public Enemy {
public:
    Monster(string n, int h, int atk)
        : Enemy(n, h, atk) {}

    // -------- 重写父类的 attackTarget --------
    void attackTarget(Character& target) override {
        cout << name << " 撕咬攻击！" << endl;
        target.hp -= attack;
    }
};

// -------- 子类：Boss --------
class Boss : public Enemy {
public:
    Boss(string n, int h, int atk)
        : Enemy(n, h, atk) {}

    // -------- 重写：Boss 有特殊攻击 --------
    void attackTarget(Character& target) override {
        cout << name << " 释放必杀技【天崩地裂】！" << endl;
        target.hp -= attack * 3;
    }
};

// -------- 子类：幽灵 --------
class Ghost : public Enemy {
public:
    Ghost(string n, int h, int atk)
        : Enemy(n, h, atk) {}

    void attackTarget(Character& target) override {
        cout << name << " 灵魂冲击，造成 " << (attack / 2) << " 物理 + " << (attack / 2) << " 魔法伤害" << endl;
        target.hp -= attack;
    }
};

// -------- 主角 --------
class Character {
public:
    string name;
    int hp;

    Character(string n, int h) : name(n), hp(h) {}
};

// -------- 统一的攻击函数（多态的关键！）--------
void attackAllEnemies(Enemy& enemy, Character& target) {
    // -------- 通过基类引用调用，实际执行的是子类的版本 --------
    enemy.attackTarget(target);
}

int main()
{
    Character hero("孙悟空", 5000);

    Monster goblin("哥布林", 100, 15);
    Boss dragon("巨龙", 1000, 100);
    Ghost spirit("幽灵", 80, 20);

    cout << "=== 攻击测试 ===" << endl;

    // -------- 同一个函数调用，不同行为 --------
    attackAllEnemies(goblin, hero);    // 哥布林攻击
    attackAllEnemies(dragon, hero);     // Boss 攻击
    attackAllEnemies(spirit, hero);     // 幽灵攻击

    cout << endl << "英雄剩余 HP：" << hero.hp << endl;

    return 0;
}
```

**运行结果：**
```
=== 攻击测试 ===
哥布林 撕咬攻击！
巨龙 释放必杀技【天崩地裂】！
幽灵 灵魂冲击，造成 10 物理 + 10 魔法伤害

英雄剩余 HP：4520
```

---

## 10.5 纯虚函数与抽象类

### 纯虚函数

> **`virtual 返回类型 函数名(参数) = 0;`**
> 有纯虚函数的类是**抽象类**，不能直接创建对象，只能作为基类被继承。

```cpp
// -------- 抽象类：所有角色都有攻击行为，但具体怎么攻击由子类决定 --------
class Character {
public:
    string name;
    int hp;

    Character(string n, int h) : name(n), hp(h) {}

    // -------- 纯虚函数：子类必须实现 --------
    virtual void attack(Character& target) = 0;

    // -------- 普通虚函数：子类可以选择重写 --------
    virtual void showInfo() {
        cout << name << " HP=" << hp << endl;
    }

    // -------- 虚析构函数 --------
    virtual ~Character() {}
};
```

### 抽象类实例：技能系统

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

// -------- 抽象类：技能基类 --------
class Skill {
public:
    string name;
    int mpCost;

    Skill(string n, int mp) : name(n), mpCost(mp) {}

    // -------- 纯虚函数：每个技能效果不同 --------
    virtual void execute(class Character& caster, class Character& target) = 0;

    // -------- 虚析构 --------
    virtual ~Skill() {}
};

// -------- 具体技能：火球术 --------
class FireballSkill : public Skill {
private:
    int damage;

public:
    FireballSkill() : Skill("火球术", 30), damage(80) {}

    void execute(Character& caster, Character& target) override {
        cout << "🔥 " << caster.name << " 施展【火球术】！" << endl;
        target.hp -= damage;
        cout << "造成 " << damage << " 点伤害" << endl;
    }
};

// -------- 具体技能：治疗术 --------
class HealSkill : public Skill {
private:
    int healAmount;

public:
    HealSkill() : Skill("治疗术", 20), healAmount(50) {}

    void execute(Character& caster, Character& target) override {
        cout << "💚 " << caster.name << " 施展【治疗术】！" << endl;
        target.hp += healAmount;
        if (target.hp > 1000) target.hp = 1000;   // 上限
        cout << "恢复 " << healAmount << " HP" << endl;
    }
};

// -------- 具体技能：冰冻术 --------
class FreezeSkill : public Skill {
private:
    int damage;

public:
    FreezeSkill() : Skill("冰冻术", 40), damage(30) {}

    void execute(Character& caster, Character& target) override {
        cout << "❄️ " << caster.name << " 施展【冰冻术】！" << endl;
        target.hp -= damage;
        cout << "造成 " << damage << " 点伤害，并附加冰冻效果" << endl;
    }
};

// -------- 角色类 --------
class Character {
public:
    string name;
    int hp;
    vector<Skill*> skills;

    Character(string n, int h) : name(n), hp(h) {}

    void addSkill(Skill* s) {
        skills.push_back(s);
        cout << name << " 学会了 [" << s->name << "]" << endl;
    }

    void useSkill(int index, Character& target) {
        if (index < 0 || index >= (int)skills.size()) {
            cout << "无效的技能索引" << endl;
            return;
        }
        skills[index]->execute(*this, target);
    }

    void showInfo() {
        cout << "【" << name << "】HP=" << hp << endl;
    }
};

int main()
{
    Character hero("孙悟空", 500);
    Character monster("牛魔王", 800);

    // -------- 创建技能 --------
    FireballSkill fireball;
    HealSkill heal;
    FreezeSkill freeze;

    // -------- 角色学习技能 --------
    hero.addSkill(&fireball);
    hero.addSkill(&heal);
    monster.addSkill(&freeze);

    cout << endl;

    // -------- 战斗 --------
    hero.showInfo();
    monster.showInfo();

    cout << endl << "=== 战斗开始 ===" << endl;

    hero.useSkill(0, monster);   // 火球术攻击牛魔王
    hero.useSkill(1, hero);       // 治疗术给自己
    monster.useSkill(0, hero);    // 冰冻术攻击孙悟空

    cout << endl;
    hero.showInfo();
    monster.showInfo();

    return 0;
}
```

**运行结果：**
```
孙悟空 学会了 [火球术]
孙悟空 学会了 [治疗术]
牛魔王 学会了 [冰冻术]

【孙悟空】HP=500
【牛魔王】HP=800

=== 战斗开始 ===
🔥 孙悟空 施展【火球术】！
造成 80 点伤害
💚 孙悟空 施展【治疗术】！
恢复 50 HP
❄️ 牛魔王 施展【冰冻术】！
造成 30 点伤害，并附加冰冻效果

【孙悟空】HP=520
【牛魔王】HP=720
```

---

## 10.6 override 与 final

### override：确保是重写

```cpp
class Parent {
public:
    virtual void show() {}
};

class Child : public Parent {
public:
    void show() override {   // ✅ 明确表示要重写父类方法
        cout << "Child show" << endl;
    }

    // void display() override { }  // ❌ 编译错误：display 不是虚函数
};
```

### final：禁止继续重写

```cpp
class Base {
public:
    virtual void run() final {   // ✅ 不能被重写
        cout << "Base run" << endl;
    }
};

class Derived : public Base {
public:
    // void run() override { }    // ❌ 编译错误：run 已标记为 final
};
```

---

## 10.7 多继承

### 概念

> **一个子类继承多个父类。** 游戏开发中较少用，但某些场景很有用。

```cpp
class Flyable {
public:
    virtual void fly() = 0;
};

class Swimmable {
public:
    virtual void swim() = 0;
};

// -------- 同时继承飞行和游泳能力 --------
class Duck : public Flyable, public Swimmable {
public:
    void fly() override {
        cout << "鸭子飞翔" << endl;
    }
    void swim() override {
        cout << "鸭子游泳" << endl;
    }
};
```

### 钻石继承与虚继承

```
        Animal
       /      \
   Bird     Flyer
      \      /
       Dragon
```

```cpp
class Animal {
public:
    int age;
};

class Bird : virtual public Animal {};   // virtual：虚继承
class Flyer : virtual public Animal {};  // virtual：虚继承

class Dragon : public Bird, public Flyer {
    // Dragon 只有一个 Animal 子对象
};
```

---

## 10.8 虚析构函数

### 为什么需要虚析构函数？

> **通过基类指针删除子类对象时，如果析构函数不是虚函数，只会调用基类的析构函数，子类的资源泄漏！**

```cpp
class Enemy {
public:
    Enemy() { cout << "Enemy 构造" << endl; }

    // -------- 虚析构函数：确保正确析构 --------
    virtual ~Enemy() {
        cout << "Enemy 析构" << endl;
    }
};

class Boss : public Enemy {
public:
    int* specialAttack;   // 动态分配的资源

    Boss() {
        specialAttack = new int[100];   // 模拟动态资源
        cout << "Boss 构造" << endl;
    }

    ~Boss() {
        delete[] specialAttack;   // 如果基类析构不是虚函数，这行不会执行！
        cout << "Boss 析构" << endl;
    }
};

int main()
{
    Enemy* e = new Boss();
    delete e;   // 必须基类析构是虚函数，Boss 的析构才会被调用
    return 0;
}
```

**运行结果（正确）：**
```
Enemy 构造
Boss 构造
Boss 析构
Enemy 析构
```

---

## 10.9 游戏开发实战：完整角色继承树

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

// -------- 抽象基类：所有角色 --------
class Entity {
protected:
    string name;
    int hp;
    int attack;

public:
    Entity(string n, int h, int atk) : name(n), hp(h), attack(atk) {}
    virtual ~Entity() {}

    virtual void showInfo() = 0;   // 纯虚
    virtual void attackTarget(Entity& target) = 0;

    string getName() const { return name; }
    int getHP() const { return hp; }
    void takeDamage(int dmg) {
        hp -= dmg;
        if (hp < 0) hp = 0;
    }
};

// -------- 玩家角色 --------
class Player : public Entity {
private:
    int level;
    int mp;

public:
    Player(string n, int h, int atk, int lv)
        : Entity(n, h, atk), level(lv), mp(100) {}

    void attackTarget(Entity& target) override {
        cout << name << " [Lv." << level << "] 发起攻击！" << endl;
        target.takeDamage(attack);
        cout << "造成 " << attack << " 点伤害，";
        cout << target.getName() << " 剩余 HP=" << target.getHP() << endl;
    }

    void showInfo() override {
        cout << "【玩家】" << name << " Lv." << level;
        cout << " HP=" << hp << " ATK=" << attack << " MP=" << mp << endl;
    }
};

// -------- 怪物基类 --------
class Monster : public Entity {
protected:
    string type;

public:
    Monster(string n, int h, int atk, string t)
        : Entity(n, h, atk), type(t) {}

    void showInfo() override {
        cout << "【怪物-" << type << "】" << name;
        cout << " HP=" << hp << " ATK=" << attack << endl;
    }
};

// -------- BOSS --------
class Boss : public Monster {
private:
    int specialAttackPower;

public:
    Boss(string n, int h, int atk, int special)
        : Monster(n, h, atk, "BOSS"), specialAttackPower(special) {}

    void attackTarget(Entity& target) override {
        cout << "⚠️ BOSS【" << name << "】释放必杀技！" << endl;
        cout << "造成 " << (attack + specialAttackPower) << " 点伤害！" << endl;
        target.takeDamage(attack + specialAttackPower);
    }

    void showInfo() override {
        cout << "【BOSS】" << name << " HP=" << hp;
        cout << " ATK=" << attack << " 必杀=" << specialAttackPower << endl;
    }
};

// -------- 主函数 --------
int main()
{
    vector<Entity*> entities;

    Player hero("孙悟空", 5000, 250, 85);
    Monster slime("史莱姆", 100, 10, "普通");
    Monster wolf("狼", 200, 25, "普通");
    Boss dragon("巨龙", 2000, 150, 200);

    entities.push_back(&hero);
    entities.push_back(&slime);
    entities.push_back(&wolf);
    entities.push_back(&dragon);

    cout << "=== 角色状态 ===" << endl;
    for (auto e : entities) {
        e->showInfo();
    }

    cout << endl << "=== 战斗演示 ===" << endl;
    hero.attackTarget(slime);
    dragon.attackTarget(hero);

    cout << endl << "=== 最终状态 ===" << endl;
    for (auto e : entities) {
        e->showInfo();
    }

    return 0;
}
```

---

## 10.10 常见错误与调试

| 错误现象 | 原因 | 解决方法 |
|---------|------|---------|
| 子类方法没被调用 | 基类方法不是 `virtual` | 加 `virtual` 关键字 |
| 内存泄漏 | 基类析构不是 `virtual` | 基类析构加 `virtual` |
| 抽象类不能创建对象 | 类中有 `= 0` 的纯虚函数 | 不要直接实例化抽象类 |
| `override` 报错 | 方法签名不匹配 | 检查父类虚函数签名 |

---

## 10.11 动手练习 🧪

### 练习 1：实现继承树 ⭐
```
基类 Shape（name, area()）
派生类 Circle（radius）
派生类 Rectangle（width, height）
实现 area() 并在 main 中测试
```

### 练习 2：虚函数 + 多态 ⭐⭐
```
基类 Animal（speak()）
派生类 Dog（重写 speak：输出"汪汪"）
派生类 Cat（重写 speak：输出"喵喵"）
用 Animal* 数组管理不同动物，循环调用 speak()
```

### 练习 3：纯虚函数 + 技能系统 ⭐⭐
```
Skill 基类（纯虚 execute）
FireSkill / IceSkill / LightningSkill 派生
Character 有 vector<Skill*>
实现战斗演示
```

### 练习 4：虚析构函数 ⭐⭐
```
基类 Enemy（有虚析构）
派生类 Boss（动态分配一个数组）
main 中用 Enemy* 指向 Boss，delete 验证析构正确调用
```

### 练习 5：完整继承树 + 多态 ⭐⭐⭐
```
RPG 角色继承树（至少 4 层）
使用 override 和 final
通过基类指针数组操作不同角色
```

---

## 10.12 本章小结

```
✅ 已掌握：
├── 继承的概念（基类 vs 派生类）
├── public 继承
├── 构造函数调用顺序
├── 虚函数（virtual）实现多态
├── 纯虚函数（= 0）与抽象类
├── override 和 final 关键字
├── 多继承
├── 虚析构函数（必须！）
├── 完整角色继承树实战
└── 技能多态系统实战

🔜 下章预告：
第十一章：STL 容器 —— 用 vector/list/set/map 高效管理游戏中的道具、敌人、排行榜。
   不用手写链表和红黑树，直接用现成的工业级数据结构！
```

---

_📚 参考资料：《C++ Primer》《Effective C++》《Design Patterns》_
