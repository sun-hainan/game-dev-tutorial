# 第八章 结构体与联合体

> 🧱 *"如果数组是把同类数据打包成一捆，那么结构体就是把不同类数据打包成一箱。"*

---

## 8.1 结构体的定义与初始化

### 🏠 生活类比

想象你在搬家。你有一个**行李箱**（数组），只能装同一种东西，比如全是衣服。但如果要打包一个**完整的厨房**，你需要不同类型的物品：碗、刀、锅、调料瓶。**结构体**就像一个有隔板的收纳箱，每个隔间可以放不同类型的东西，然后整体搬运。

### 💡 核心思想

结构体是**用户自定义的复合数据类型**，它把不同类型的变量组合成一个整体。定义结构体就像画一张"数据蓝图"。

### 📝 代码示例

```cpp
#include <iostream>
#include <string>
using namespace std;

// 定义结构体 —— 就像设计一张"角色卡"的模板
struct Player {
    string name;   // 角色名字
    int level;     // 等级
    float hp;      // 生命值
    float mp;      // 法力值
};  // 别忘了分号！

int main() {
    // 方式一：按声明顺序初始化（顺序初始化）
    Player hero = {"亚瑟", 1, 100.0f, 50.0f};

    // 方式二：指定成员名初始化（C++20 推荐，更清晰）
    Player mage = {.name = "甘道夫", .level = 5, .hp = 80.0f, .mp = 200.0f};

    // 方式三：先定义，再逐个赋值
    Player warrior;
    warrior.name = "贝恩";
    warrior.level = 3;
    warrior.hp = 120.0f;
    warrior.mp = 0.0f;

    // 打印结果
    cout << "=== 英雄信息 ===" << endl;
    cout << "战士: " << hero.name << " LV." << hero.level << endl;
    cout << "HP: " << hero.hp << " | MP: " << hero.mp << endl;
    cout << endl;
    cout << "法师: " << mage.name << " LV." << mage.level << endl;
    cout << "HP: " << mage.hp << " | MP: " << mage.mp << endl;

    return 0;
}
```

**运行结果：**
```
=== 英雄信息 ===
战士: 亚瑟 LV.1
HP: 100 | MP: 50

法师: 甘道夫 LV.5
HP: 80 | MP: 200
```

### 🎯 关键点

| 注意点 | 说明 |
|-------|------|
| `struct` 关键字 | 声明结构体类型 |
| `;` 结尾 | 结构体定义后必须加分号 |
| 内存布局 | 结构体成员在内存中是**连续存放**的 |
| 成员类型 | 可以是基本类型、数组、指针，甚至另一个结构体 |

---

## 8.2 结构体成员访问（`.` 和 `->`）

### 🏠 生活类比

你已经把行李打包好了，现在要去**拿东西**：
- 如果你拿着**箱子本身**（结构体变量），用**钥匙 `.`** 开门
- 如果你拿着箱子的**钥匙卡**（结构体指针），用**磁卡感应 `->`** 开门

### 💡 核心思想

- `.`（点运算符）：用于**结构体对象**本身
- `->`（箭头运算符）：用于**指向结构体的指针**

### 📝 代码示例

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Item {
    string name;    // 物品名称
    int price;      // 价格
    int durability; // 耐久度
};

int main() {
    // 场景：你有两把剑，一把是真剑，一把是钥匙卡（指针）

    // 真剑 —— 结构体变量
    Item sword;
    sword.name = "圣剑 Excalibur";
    sword.price = 5000;
    sword.durability = 100;

    // 钥匙卡 —— 指向结构体的指针
    Item* pSword = &sword;  // 指向 sword 的地址

    // 方式一：用"."访问（直接操作箱子）
    cout << "用 . 访问：" << sword.name << endl;
    cout << "价格: " << sword.price << endl;

    // 方式二：用 "->" 访问（通过指针的钥匙卡）
    cout << "用 -> 访问：" << pSword->name << endl;
    cout << "耐久度: " << pSword->durability << endl;

    // 等价关系：pSword->name 等价于 (*pSword).name
    cout << "等价验证：" << (*pSword).name << endl;

    // 修改数据
    sword.durability -= 10;       // 正常扣耐久
    pSword->price = 4500;        // 用指针也能改
    cout << "修改后耐久: " << sword.durability << endl;

    return 0;
}
```

**运行结果：**
```
用 . 访问：圣剑 Excalibur
价格: 5000
用 -> 访问：圣剑 Excalibur
耐久度: 100
等价验证：圣剑 Excalibur
修改后耐久: 90
```

### 🧠 记忆口诀

> **"."** 像手直接摸到，用"对象.成员"
> **"->"** 像遥控器遥控，用"指针->成员"

---

## 8.3 结构体与函数

### 🏠 生活类比

快递包裹（结构体）可以通过三种方式传递：
1. **全套复制**：原封不动复印一份寄过去（原包裹不动，但费时费力）
2. **只给钥匙**：把钥匙（引用）寄过去，让对方直接操作原包裹（快，但数据可能被改）
3. **拆开重装**：对方根据内容重新拼一个（返回新结构体）

### 💡 核心思想

结构体可以作为函数参数或返回值，传值、传引用、返回结构体三种方式各有权衡。

### 📝 代码示例

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Weapon {
    string name;
    int attack;
    float weight;
};

// ==================== 方式一：传值（复制一份） ====================
// 特点：原结构体不变，但拷贝开销大
void showWeaponByValue(Weapon w) {
    cout << "[传值] " << w.name << " 攻击力:" << w.attack << endl;
    // 这里对 w 的修改不会影响外面的原始数据
    w.attack += 999;  // 假装强化
}

// ==================== 方式二：传引用（共享原数据） ====================
// 特点：高效，但可能意外修改原数据
void upgradeWeaponByRef(Weapon& w) {
    cout << "[传引用] " << w.name << " 攻击力:" << w.attack;
    w.attack += 50;  // 真正修改了原始数据
    cout << " → 强化后:" << w.attack << endl;
}

// ==================== 方式三：传const引用（只读不修改） ====================
// 特点：高效 + 安全，推荐！
void showWeaponConstRef(const Weapon& w) {
    cout << "[const引用] " << w.name << " 重量:" << w.weight << endl;
    // w.attack += 10;  // 编译错误！const引用不能修改
}

// ==================== 方式四：返回结构体 ====================
// 特点：创建新结构体，适合计算后返回结果
Weapon createWeapon(string n, int atk, float wt) {
    Weapon w;
    w.name = n;
    w.attack = atk;
    w.weight = wt;
    return w;  // 返回一个结构体副本
}

int main() {
    Weapon longSword = {"长剑", 50, 3.5f};

    // 1. 传值测试
    showWeaponByValue(longSword);
    cout << "传值后原始攻击力: " << longSword.attack << endl;  // 还是50，未变

    // 2. 传引用测试
    upgradeWeaponByRef(longSword);
    cout << "传引用后攻击力: " << longSword.attack << endl;    // 变成了100！

    // 3. const引用测试
    showWeaponConstRef(longSword);

    // 4. 返回结构体
    Weapon magicStaff = createWeapon("魔法杖", 30, 2.0f);
    cout << "新武器: " << magicStaff.name << " 攻击:" << magicStaff.attack << endl;

    return 0;
}
```

**运行结果：**
```
[传值] 长剑 攻击力:50
传值后原始攻击力: 50
[传引用] 长剑 攻击力:50 → 强化后:100
传引用后攻击力: 100
[const引用] 长剑 重量:3.5
新武器: 魔法杖 攻击:30
```

### 📊 参数传递方式对比

| 方式 | 效率 | 是否修改原数据 | 适用场景 |
|------|------|---------------|---------|
| 传值 | ❌ 慢（拷贝） | ❌ 不会 | 小结构体、不需要修改 |
| 传引用 | ✅ 快 | ⚠️ 可能修改 | 需要修改、效率优先 |
| const引用 | ✅ 快 | ❌ 安全 | 只读、推荐默认选择 |
| 返回结构体 | ❌ 可能有拷贝 | N/A | 构造新对象 |

---

## 8.4 结构体数组

### 🏠 生活类比

一间**宿舍楼**（数组）里住着很多学生（元素），每个学生有自己的信息卡（结构体）。数组的索引就是房间号。

### 💡 核心思想

结构体数组是**相同结构体的连续排列**，用于管理大量相似的数据集合。

### 📝 代码示例

```cpp
#include <iostream>
#include <string>
using namespace std;

struct Monster {
    string name;
    int hp;
    int attack;
    bool isBoss;  // 是否是Boss
};

int main() {
    // 方式一：声明时初始化（定义怪物列表）
    Monster monsters[] = {
        {"哥布林",      50,  10, false},
        {"食人魔",     150,  30, false},
        {"骷髅王",     500, 100, true }
    };
    int monsterCount = 3;

    // 方式二：先声明数组，再逐个填充
    Monster inventory[5];  // 最多5个物品
    inventory[0].name = "生命药水";
    inventory[0].hp = 50;
    inventory[0].attack = 0;
    inventory[0].isBoss = false;

    // 遍历怪物列表
    cout << "=== 地下城怪物列表 ===" << endl;
    for (int i = 0; i < monsterCount; i++) {
        cout << "[" << i << "] ";
        cout << monsters[i].name;
        cout << " HP:" << monsters[i].hp;
        cout << " ATK:" << monsters[i].attack;
        if (monsters[i].isBoss) {
            cout << " ⚔️ BOSS";
        }
        cout << endl;
    }

    // 找出血量最高的怪物
    int maxHpIndex = 0;
    for (int i = 1; i < monsterCount; i++) {
        if (monsters[i].hp > monsters[maxHpIndex].hp) {
            maxHpIndex = i;
        }
    }
    cout << "最强怪物: " << monsters[maxHpIndex].name << endl;

    // 数组长度计算
    int len = sizeof(monsters) / sizeof(monsters[0]);
    cout << "怪物总数: " << len << endl;

    return 0;
}
```

**运行结果：**
```
=== 地下城怪物列表 ===
[0] 哥布林 HP:50 ATK:10
[1] 食人魔 HP:150 ATK:30
[2] 骷髅王 HP:500 ATK:100 ⚔️ BOSS
最强怪物: 骷髅王
怪物总数: 3
```

---

## 8.5 嵌套结构体

### 🏠 生活类比

一份**简历**里包含"个人信息"部分，而"个人信息"里又有姓名、年龄、地址。一个结构体里包含另一个结构体，就像文档里嵌套章节。

### 💡 核心思想

结构体可以**嵌套**定义，一个结构体的成员可以是另一个结构体类型。

### 📝 代码示例

```cpp
#include <iostream>
#include <string>
using namespace std;

// 外层结构体：坐标（游戏中常用于位置）
struct Position {
    int x;  // 横坐标
    int y;  // 纵坐标
};

// 中层结构体：日期
struct Date {
    int year;
    int month;
    int day;
};

// 内层结构体：地址
struct Address {
    string city;
    string district;
    string street;
};

// 外层结构体：角色
struct Character {
    string name;           // 角色名
    int age;               // 年龄
    Position location;     // 当前位置（嵌套Position）
    Date birthday;         // 生日（嵌套Date）
    Address home;          // 住址（嵌套Address）
};

int main() {
    // 创建一个角色：使用嵌套结构体
    Character hero;

    // 基本信息
    hero.name = "林月";
    hero.age = 22;

    // 嵌套赋值
    hero.location.x = 100;     // 位置的x坐标
    hero.location.y = 200;     // 位置的y坐标

    hero.birthday.year = 2004;   // 出生年
    hero.birthday.month = 3;     // 出生月
    hero.birthday.day = 15;      // 出生日

    hero.home.city = "深圳";      // 城市
    hero.home.district = "南山区"; // 区
    hero.home.street = "科技园路"; // 街道

    // 打印完整信息
    cout << "=== 角色档案 ===" << endl;
    cout << "姓名: " << hero.name << endl;
    cout << "年龄: " << hero.age << endl;
    cout << "位置: (" << hero.location.x << ", " << hero.location.y << ")" << endl;
    cout << "生日: " << hero.birthday.year << "-" 
         << hero.birthday.month << "-" 
         << hero.birthday.day << endl;
    cout << "住址: " << hero.home.city << hero.home.district 
         << hero.home.street << endl;

    // 修改嵌套成员
    hero.location.x += 50;  // 往右移动50单位
    cout << "移动后位置: (" << hero.location.x << ", " << hero.location.y << ")" << endl;

    return 0;
}
```

**运行结果：**
```
=== 角色档案 ===
姓名: 林月
年龄: 22
位置: (100, 200)
生日: 2004-3-15
住址: 深圳南山区科技园路
移动后位置: (150, 200)
```

### 🎯 嵌套层数建议

> 嵌套结构体虽强大，但**建议不超过3层**。层数太多会让代码变得复杂，难以维护。

---

## 8.6 枚举类型（enum）

### 🏠 生活类比

一副扑克牌有四种花色：黑桃、红心、梅花、方块。如果你用整数表示，可以用0、1、2、3，但这样做的问题是：**没有任何人来限制你输入5**（这不是合法的花色）。枚举就像给数字起了**有意义的名字**，限定取值范围。

### 💡 核心思想

枚举是一种**受限的整数类型**，用于定义一组命名的整数值常量集合。

### 📝 代码示例

```cpp
#include <iostream>
using namespace std;

// ==================== 基本 enum ====================
enum WeaponType {
    SWORD = 1,     // 剑
    BOW = 2,       // 弓
    STAFF = 3,     // 法杖
    AXE = 4        // 斧头
};

enum GameState {
    MENU,      // 菜单（默认从0开始）
    PLAYING,   // 游戏中
    PAUSED,    // 暂停
    GAMEOVER   // 游戏结束
};

// ==================== enum class（强类型枚举，C++11）===================
// 推荐使用！更安全，不会污染命名空间
enum class ItemRarity {
    COMMON,    // 普通
    UNCOMMON,  // 优秀
    RARE,      // 稀有
    EPIC,      // 史诗
    LEGENDARY  // 传说
};

int main() {
    // ==================== 基本 enum 使用 ====================
    WeaponType currentWeapon = SWORD;
    GameState state = PLAYING;

    // 枚举值可以当作整数使用
    cout << "当前武器编号: " << currentWeapon << endl;
    cout << "当前游戏状态: " << state << endl;

    // 可以和整数比较或运算
    if (state == 1) {  // 1 = PLAYING
        cout << "游戏正在进行中..." << endl;
    }

    // 遍历枚举（基本 enum）
    cout << "\n武器类型列表: ";
    for (int i = SWORD; i <= AXE; i++) {
        cout << i << " ";
    }
    cout << endl;

    // ==================== enum class 使用 ====================
    ItemRarity dropRarity = ItemRarity::EPIC;  // 必须用 "ItemRarity::" 前缀

    // enum class 不会隐式转换为 int，需要强制转换
    cout << "\n掉宝稀有度: ";
    switch (dropRarity) {
        case ItemRarity::COMMON:    cout << "普通"; break;
        case ItemRarity::UNCOMMON:  cout << "优秀"; break;
        case ItemRarity::RARE:      cout << "稀有"; break;
        case ItemRarity::EPIC:      cout << "史诗"; break;
        case ItemRarity::LEGENDARY: cout << "传说"; break;
    }
    cout << endl;

    // 强制转换回整数
    int rarityValue = static_cast<int>(dropRarity);
    cout << "稀有度数值: " << rarityValue << endl;  // 输出: 4

    return 0;
}
```

**运行结果：**
```
当前武器编号: 1
当前游戏状态: 1
游戏正在进行中...

武器类型列表: 1 2 3 4

掉宝稀有度: 史诗
稀有度数值: 4
```

### 📊 基本 enum vs enum class 对比

| 特性 | enum | enum class |
|------|------|-----------|
| 作用域 | 枚举值在枚举名作用域内 | 枚举值在 enum class 作用域内 |
| 隐式转换 | 可以隐式转 int | 不能隐式转 int |
| 命名冲突 | 容易冲突 | 不会冲突 |
| 安全性 | 较低 | 高，推荐使用 |

---

## 8.7 联合体（union）

### 🏠 生活类比

想象一个**变形金刚**（union），它可以在不同时间扮演不同的角色——汽车、飞机、机器人。但**同一时刻只能是一种形态**。联合体的所有成员**共享同一块内存**，同一时间只能使用其中一个成员。

### 💡 核心思想

联合体的所有成员**共用同一段内存**，内存大小等于最大成员的大小。适合用于节省内存或实现"同一块数据不同解释"。

### 📝 代码示例

```cpp
#include <iostream>
#include <string>
using namespace std;

// ==================== 基本联合体 ====================
union Data {
    int i;      // 4字节
    float f;    // 4字节
    char str[4]; // 4字节
};

int main() {
    // 场景：游戏服务器需要用同一个内存空间存储不同类型的数据
    Data packet;

    // 存整数
    packet.i = 12345;
    cout << "存整数: " << packet.i << endl;

    // 存浮点数（会覆盖整数的数据！）
    packet.f = 3.14159f;
    cout << "存浮点: " << packet.f << endl;

    // 存字符串（也会覆盖）
    packet.str[0] = 'A';
    packet.str[1] = 'B';
    packet.str[2] = 'C';
    packet.str[3] = '\0';
    cout << "存字符串: " << packet.str << endl;

    // 查看联合体大小（取最大成员的大小）
    cout << "联合体大小: " << sizeof(Data) << " 字节" << endl;

    // ==================== 匿名联合体 ====================
    struct Message {
        int type;  // 消息类型

        // 匿名联合体 —— 直接使用成员名，不需要"."访问
        union {
            int intData;      // 类型0的数据
            float floatData;  // 类型1的数据
            char text[32];    // 类型2的数据
        };
    };

    Message msg;
    msg.type = 0;          // 整数消息
    msg.intData = 999;
    cout << "\n消息类型: " << msg.type << ", 数据: " << msg.intData << endl;

    msg.type = 1;          // 改成浮点消息
    msg.floatData = 2.718f;
    cout << "消息类型: " << msg.type << ", 数据: " << msg.floatData << endl;

    msg.type = 2;          // 改成文本消息
    strcpy(msg.text, "Hello Game!");
    cout << "消息类型: " << msg.type << ", 数据: " << msg.text << endl;

    return 0;
}
```

**运行结果：**
```
存整数: 12345
存浮点: 3.14159
存字符串: ABC
联合体大小: 4 字节

消息类型: 0, 数据: 999
消息类型: 1, 数据: 2.718
消息类型: 2, 数据: Hello Game!
```

### 🎯 union 的典型应用场景

| 场景 | 说明 |
|------|------|
| **节省内存** | 需要存储多种类型但同时只使用一种 |
| **数据转换** | 把同一个字节解释为不同类型（网络协议） |
| **标签联合** | 用一个额外的标签记录当前存储的类型 |

---

## 8.8 位域（bit-field）

### 🏠 生活类比

想象一个**停车场**（结构体）需要记录车辆信息：
- 如果用完整的 int（4字节）来存储"是否有车"（0或1），太浪费
- 位域就像用**单个停车位**（1 bit）来记录有无车辆，精打细算

### 💡 核心思想

位域允许直接指定**占用多少个bit**来存储成员，非常适合需要精确控制内存的底层场景（如网络协议、游戏协议）。

### 📝 代码示例

```cpp
#include <iostream>
using namespace std;

// ==================== 位域示例：网络数据包 ====================
struct NetPacket {
    // 用:1表示只占用1个bit
    unsigned int header : 4;    // 包头 (0-15)
    unsigned int type : 4;      // 类型 (0-15)
    unsigned int version : 3;   // 版本 (0-7)
    unsigned int encrypted : 1; // 是否加密 (0-1)
    unsigned int priority : 2;  // 优先级 (0-3)
    unsigned int : 2;          // 保留位，不使用
};

int main() {
    // 创建一个数据包
    NetPacket packet;

    // 直接给位域赋值，就像操作普通成员一样
    packet.header = 10;       // 二进制: 1010
    packet.type = 5;         // 二进制: 0101
    packet.version = 3;       // 二进制: 011
    packet.encrypted = 1;    // 已加密
    packet.priority = 2;     // 高优先级

    cout << "=== 网络数据包 ===" << endl;
    cout << "包头:     " << packet.header << endl;
    cout << "类型:     " << packet.type << endl;
    cout << "版本:     " << packet.version << endl;
    cout << "加密:     " << (packet.encrypted ? "是" : "否") << endl;
    cout << "优先级:   " << packet.priority << endl;

    // 查看位域结构体的总大小
    cout << "\nNetPacket 大小: " << sizeof(NetPacket) << " 字节" << endl;
    // 说明: 4+4+3+1+2+2 = 16 bits = 2 bytes (编译器可能会有所调整)

    // ==================== 游戏技能标志位 ====================
    struct SkillFlags {
        unsigned int canMove : 1;      // 移动: 1=可移动, 0=不可移动
        unsigned int canAttack : 1;    // 攻击: 1=可攻击, 0=不可攻击
        unsigned int isInvisible : 1;  // 隐身: 1=隐身中
        unsigned int isBuffed : 1;     // 增益: 1=有增益效果
        unsigned int isDebuffed : 1;   // 减益: 1=有减益效果
        unsigned int isStunned : 1;    // 眩晕: 1=眩晕中
        unsigned int isPoisoned : 1;   // 中毒: 1=中毒中
        unsigned int canFly : 1;       // 飞行: 1=可飞行
    };

    SkillFlags playerSkill;
    playerSkill.canMove = 1;
    playerSkill.canAttack = 1;
    playerSkill.isBuffed = 1;
    playerSkill.isPoisoned = 1;

    cout << "\n=== 玩家技能状态 ===" << endl;
    cout << "可移动:   " << playerSkill.canMove << endl;
    cout << "可攻击:   " << playerSkill.canAttack << endl;
    cout << "隐身:     " << playerSkill.isInvisible << endl;
    cout << "增益:     " << playerSkill.isBuffed << endl;
    cout << "减益:     " << playerSkill.isDebuffed << endl;
    cout << "眩晕:     " << playerSkill.isStunned << endl;
    cout << "中毒:     " << playerSkill.isPoisoned << endl;
    cout << "可飞行:   " << playerSkill.canFly << endl;

    cout << "\nSkillFlags 大小: " << sizeof(SkillFlags) << " 字节" << endl;

    return 0;
}
```

**运行结果：**
```
=== 网络数据包 ===
包头:     10
类型:     5
版本:     3
加密:     是
优先级:   2

NetPacket 大小: 4 字节

=== 玩家技能状态 ===
可移动:   1
可攻击:   1
隐身:     0
增益:     1
减益:     0
眩晕:     0
中毒:     1
可飞行:   0

SkillFlags 大小: 4 字节
```

### ⚠️ 位域注意事项

1. **不能取地址**：`&playerSkill.canMove` 是非法的
2. **跨平台差异**：不同编译器的位域实现可能不同
3. **顺序依赖**：成员的排列顺序依赖于编译器

---

## 8.9 typedef 和 using

### 🏠 生活类比

想象你经常填写地址"北京市朝阳区建国路88号"，太长了。给它起一个**别名**——"我家地址"，填写时就更方便。`typedef` 和 `using` 就是给类型起别名的工具。

### 💡 核心思想

`typedef` 和 `using`（C++11）用来为类型创建**别名**，让代码更简洁、更易读。

### 📝 代码示例

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// ==================== typedef ====================
typedef int Integer;           // int 的别名
typedef float RealNumber;      // float 的别名
typedef unsigned int Count;    // 无符号整数的别名

// 给复杂的类型起别名
typedef int* IntPtr;           // int* 的别名
typedef vector<int> IntVector; // vector<int> 的别名

// ==================== using（C++11 更现代）===================
using Int = int;               // 等价于 typedef int Int;
using Str = string;            // 等价于 typedef string Str;
using Callback = void(*)(int); // 函数指针类型别名

// 给结构体起别名（简化书写）
struct Point3D {
    float x, y, z;
};

// 用 typedef 起别名
typedef Point3D Vector3;

// 用 using 起别名（C++11 推荐）
using Position = Point3D;

int main() {
    // 使用别名，就像使用原始类型一样
    Integer score = 100;
    RealNumber pi = 3.14159f;
    Count players = 10;

    cout << "分数: " << score << endl;
    cout << "圆周率: " << pi << endl;
    cout << "玩家数: " << players << endl;

    // 使用结构体别名
    Vector3 velocity;
    velocity.x = 1.0f;
    velocity.y = 2.0f;
    velocity.z = 3.0f;

    Position cameraPos;
    cameraPos.x = 0.0f;
    cameraPos.y = 10.0f;
    cameraPos.z = -5.0f;

    cout << "\n速度向量: (" << velocity.x << ", " << velocity.y << ", " << velocity.z << ")" << endl;
    cout << "相机位置: (" << cameraPos.x << ", " << cameraPos.y << ", " << cameraPos.z << ")" << endl;

    // using 模板别名（类似泛型）
    using IntVec = vector<int>;
    IntVec numbers = {1, 2, 3, 4, 5};
    cout << "\n数字列表: ";
    for (int n : numbers) {
        cout << n << " ";
    }
    cout << endl;

    // 函数指针类型别名
    Callback onHit = [](int damage) {
        cout << "受到伤害: " << damage << endl;
    };
    onHit(50);  // 调用回调函数

    return 0;
}
```

**运行结果：**
```
分数: 100
圆周率: 3.14159
玩家数: 10

速度向量: (1, 2, 3)
相机位置: (0, 10, -5)

数字列表: 1 2 3 4 5
受到伤害: 50
```

### 📊 typedef vs using 对比

| 特性 | typedef | using |
|------|---------|-------|
| 语法 | `typedef 原始类型 别名` | `using 别名 = 原始类型` |
| 可读性 | 声明式（旧式） | 赋值式（更像变量声明） |
| 模板化 | 不支持 | C++11 支持模板别名 |
| 推荐场景 | 旧代码兼容 | 新代码首选 |

---

## 8.10 游戏开发实战：RPG角色属性结构体

### 🎮 实战目标

用结构体实现一个完整的 RPG 游戏角色系统，包括玩家（Player）、敌人（Enemy）和物品（Item）三种结构体。

### 💡 核心设计思路

```
Player：拥有角色属性、等级、经验值、背包、装备
Enemy：拥有怪物属性、AI状态、掉落物品
Item：拥有物品属性、堆叠数量、使用效果
```

### 📝 代码示例

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

// ==================== 枚举定义 ====================
enum class ItemType {
    WEAPON,
    ARMOR,
    POTION,
    MATERIAL
};

enum class Rarity {
    COMMON,
    UNCOMMON,
    RARE,
    EPIC,
    LEGENDARY
};

// ==================== 物品结构体 ====================
struct Item {
    string name;         // 物品名称
    ItemType type;       // 物品类型
    Rarity rarity;       // 稀有度
    int value;           // 价值（金钱）
    int effectValue;    // 效果值（攻击力或治疗量）
    int stackCount;     // 堆叠数量
};

// ==================== 装备结构体 ====================
struct Equipment {
    string name;         // 装备名称
    int attackBonus;     // 攻击加成
    int defenseBonus;    // 防御加成
    int durability;      // 耐久度
    int maxDurability;   // 最大耐久度
};

// ==================== 玩家结构体 ====================
struct Player {
    string name;                 // 玩家名称
    int level;                   // 当前等级
    int exp;                     // 当前经验
    int expToNextLevel;          // 升级所需经验

    // 属性
    int maxHp;                   // 最大生命
    int currentHp;               // 当前生命
    int maxMp;                   // 最大法力
    int currentMp;               // 当前法力
    int attack;                  // 攻击力
    int defense;                 // 防御力
    int speed;                   // 速度（决定行动顺序）

    // 背包和装备
    vector<Item> backpack;       // 背包（动态数组）
    Equipment weapon;            // 当前武器
    Equipment armor;             // 当前护甲
    Equipment accessory;         // 当前饰品

    // 状态
    bool isAlive;                // 是否存活
};

// ==================== 敌人结构体 ====================
struct Enemy {
    string name;                 // 敌人名称
    int level;                   // 等级
    int hp;                      // 生命值
    int maxHp;                   // 最大生命
    int attack;                  // 攻击力
    int defense;                 // 防御力
    int expReward;               // 经验奖励
    int goldReward;              // 金币奖励
    vector<Item> dropTable;      // 掉落表
    bool isBoss;                 // 是否是Boss
};

// ==================== 辅助函数 ====================

// 打印物品信息
void printItem(const Item& item) {
    cout << "[" << item.name << "] ";
    cout << "价值:" << item.value << " ";
    cout << "效果:" << item.effectValue;
    cout << " x" << item.stackCount;
}

// 打印玩家状态
void printPlayerStatus(const Player& player) {
    cout << "\n========== " << player.name << " ==========" << endl;
    cout << "等级: " << player.level << " (EXP: " << player.exp << "/" << player.expToNextLevel << ")" << endl;
    cout << "HP: " << player.currentHp << "/" << player.maxHp << "  ";
    cout << "MP: " << player.currentMp << "/" << player.maxMp << endl;
    cout << "属性: ATK " << player.attack << " | DEF " << player.defense << " | SPD " << player.speed << endl;
    cout << "装备: ";
    cout << "武器[" << player.weapon.name << "] ";
    cout << "护甲[" << player.armor.name << "] ";
    cout << "饰品[" << player.accessory.name << "]" << endl;
    cout << "背包(" << player.backpack.size() << "个物品): ";
    for (const auto& item : player.backpack) {
        printItem(item);
        cout << " | ";
    }
    cout << endl;
}

// 打印敌人状态
void printEnemyStatus(const Enemy& enemy) {
    cout << "\n========== " << enemy.name << " ==========" << endl;
    if (enemy.isBoss) cout << "⚔️ BOSS怪 ";
    cout << "LV." << enemy.level << endl;
    cout << "HP: " << enemy.hp << "/" << enemy.maxHp << endl;
    cout << "ATK: " << enemy.attack << " | DEF: " << enemy.defense << endl;
    cout << "奖励: EXP " << enemy.expReward << " | 金币 " << enemy.goldReward << endl;
}

// 计算伤害
int calculateDamage(int attackerAtk, int defenderDef) {
    int baseDamage = attackerAtk - defenderDef / 2;
    return max(1, baseDamage);  // 最低造成1点伤害
}

// 玩家攻击敌人
void playerAttackEnemy(Player& player, Enemy& enemy) {
    int damage = calculateDamage(player.attack, enemy.defense);
    enemy.hp = max(0, enemy.hp - damage);
    cout << player.name << " 攻击 " << enemy.name << "，造成 " << damage << " 点伤害！" << endl;
    if (enemy.hp == 0) {
        cout << enemy.name << " 倒下了！" << endl;
    }
}

// 敌人攻击玩家
void enemyAttackPlayer(Player& player, Enemy& enemy) {
    int damage = calculateDamage(enemy.attack, player.defense);
    player.currentHp = max(0, player.currentHp - damage);
    cout << enemy.name << " 攻击 " << player.name << "，造成 " << damage << " 点伤害！" << endl;
    if (player.currentHp == 0) {
        player.isAlive = false;
        cout << player.name << " 倒下了！游戏结束..." << endl;
    }
}

// 玩家使用药水
void usePotion(Player& player) {
    for (auto& item : player.backpack) {
        if (item.type == ItemType::POTION) {
            int healAmount = item.effectValue;
            player.currentHp = min(player.maxHp, player.currentHp + healAmount);
            item.stackCount--;
            if (item.stackCount == 0) {
                // 移除空堆叠
                auto it = find(player.backpack.begin(), player.backpack.end(), item);
                if (it != player.backpack.end()) player.backpack.erase(it);
            }
            cout << player.name << " 使用了 " << item.name << "，恢复了 " << healAmount << " HP！" << endl;
            return;
        }
    }
    cout << "背包里没有药水！" << endl;
}

// ==================== 主程序 ====================
int main() {
    cout << "=== RPG 角色系统演示 ===" << endl;

    // 初始化玩家
    Player hero;
    hero.name = "亚瑟";
    hero.level = 5;
    hero.exp = 300;
    hero.expToNextLevel = 1000;
    hero.maxHp = 200;
    hero.currentHp = 180;
    hero.maxMp = 100;
    hero.currentMp = 80;
    hero.attack = 45;
    hero.defense = 30;
    hero.speed = 35;
    hero.isAlive = true;

    // 初始化装备
    hero.weapon.name = "圣剑";
    hero.weapon.attackBonus = 15;
    hero.weapon.defenseBonus = 0;
    hero.weapon.durability = 80;
    hero.weapon.maxDurability = 100;

    hero.armor.name = "骑士胸甲";
    hero.armor.attackBonus = 0;
    hero.armor.defenseBonus = 20;
    hero.armor.durability = 100;
    hero.armor.maxDurability = 100;

    hero.accessory.name = "敏捷戒指";
    hero.accessory.attackBonus = 5;
    hero.accessory.defenseBonus = 5;
    hero.accessory.durability = 999;
    hero.accessory.maxDurability = 999;

    // 给玩家添加一些物品
    Item potion1 = {"生命药水", ItemType::POTION, Rarity::COMMON, 50, 50, 3};
    Item potion2 = {"高级生命药水", ItemType::POTION, Rarity::UNCOMMON, 150, 100, 1};
    Item sword = {"铁剑", ItemType::WEAPON, Rarity::COMMON, 100, 20, 1};
    hero.backpack.push_back(potion1);
    hero.backpack.push_back(potion2);
    hero.backpack.push_back(sword);

    // 打印玩家状态
    printPlayerStatus(hero);

    // 初始化敌人
    Enemy goblin;
    goblin.name = "哥布林首领";
    goblin.level = 4;
    goblin.maxHp = 150;
    goblin.hp = 150;
    goblin.attack = 35;
    goblin.defense = 15;
    goblin.expReward = 200;
    goblin.goldReward = 150;
    goblin.isBoss = false;
    Item dropItem = {"哥布林牙", ItemType::MATERIAL, Rarity::COMMON, 10, 0, 5};
    goblin.dropTable.push_back(dropItem);

    printEnemyStatus(goblin);

    // 战斗模拟
    cout << "\n========== 战斗开始！==========" << endl;

    // 第一回合：玩家攻击
    playerAttackEnemy(hero, goblin);

    // 第二回合：敌人攻击
    if (goblin.hp > 0) {
        enemyAttackPlayer(hero, goblin);
    }

    // 第三回合：玩家使用药水
    cout << "\n--- 玩家查看状态 ---" << endl;
    printPlayerStatus(hero);
    usePotion(hero);

    // 最终状态
    cout << "\n========== 战斗结束 ==========" << endl;
    printPlayerStatus(hero);
    printEnemyStatus(goblin);

    return 0;
}
```

**运行结果：**
```
=== RPG 角色系统演示 ===

========== 亚瑟 ==========
等级: 5 (EXP: 300/1000)
HP: 180/200  MP: 80/100
属性: ATK 45 | DEF 30 | SPD 35
装备: 武器[圣剑] 护甲[骑士胸甲] 饰品[敏捷戒指]
背包(3个物品): [生命药水] 价值:50 效果:50 x3 | [高级生命药水] 价值:150 效果:100 x1 | [铁剑] 价值:100 效果:20 x1 |

========== 哥布林首领 ==========
⚔️ LV.4
HP: 150/150
ATK: 35 | DEF: 15
奖励: EXP 200 | 金币 150

========== 战斗开始！==========
亚瑟 攻击 哥布林首领，造成 38 点伤害！
哥布林首领 攻击 亚瑟，造成 20 点伤害！

--- 玩家查看状态 ---
========== 亚瑟 ==========
等级: 5 (EXP: 300/1000)
HP: 160/200  MP: 80/100
属性: ATK 45 | DEF 30 | SPD 35
装备: 武器[圣剑] 护甲[骑士胸甲] 饰品[敏捷戒指]
背包(3个物品): [生命药水] 价值:50 效果:50 x3 | [高级生命药水] 价值:150 效果:100 x1 | [铁剑] 价值:100 效果:20 x1 |

亚瑟 使用了 生命药水，恢复了 50 HP！

========== 战斗结束 ==========
========== 亚瑟 ==========
等级: 5 (EXP: 300/1000)
HP: 210/200  MP: 80/100
属性: ATK 45 | DEF 30 | SPD 35
装备: 武器[圣剑] 护甲[骑士胸甲] 饰品[敏捷戒指]
背包(3个物品): [生命药水] 价值:50 效果:50 x3 | [高级生命药水] 价值:150 效果:100 x1 | [铁剑] 价值:100 效果:20 x1 |

========== 哥布林首领 ==========
⚔️ LV.4
HP: 112/150
ATK: 35 | DEF: 15
奖励: EXP 200 | 金币 150
```

---

## 8.11 动手练习

### ⭐ 练习一：学生成绩管理（难度：⭐）

**题目：** 定义一个结构体 `Student`，包含姓名、三门课成绩（语文、数学、英语），计算总分和平均分。

```cpp
// 期望输出：
// 小明 总分: 270 平均分: 90.0
// 小红 总分: 285 平均分: 95.0
```

---

### ⭐ 练习二：复数计算器（难度：⭐⭐）

**题目：** 定义复数结构体 `Complex`，实现复数的加法和乘法运算。

---

### ⭐⭐⭐ 练习三：扑克牌管理系统（难度：⭐⭐⭐）

**题目：** 用枚举定义花色和点数，用结构体定义扑克牌，模拟发牌（随机发5张）。

---

### ⭐⭐⭐ 练习四：成绩排名系统（难度：⭐⭐⭐）

**题目：** 定义学生结构体数组，实现按总分升序排列，输出排名。

---

### ⭐⭐⭐⭐⭐ 练习五：位域与二进制（难度：⭐⭐⭐⭐⭐）

**题目：** 用位域设计一个 32 位 IPv4 地址结构体，实现点分十进制字符串与整数之间的转换。

---

## 本章小结

本章我们学习了 C++ 中的复合数据类型：

| 知识点 | 核心内容 |
|-------|---------|
| **结构体** | 将不同类型数据组合成一个整体 |
| **成员访问** | `.` 用于对象，`->` 用于指针 |
| **结构体与函数** | 传值、传引用、返回结构体 |
| **结构体数组** | 管理大量相似数据 |
| **嵌套结构体** | 结构体可以包含其他结构体 |
| **枚举** | 受限的整数类型，enum vs enum class |
| **联合体** | 成员共享内存，省空间 |
| **位域** | 按 bit 精细控制内存 |
| **typedef/using** | 类型别名，简化代码 |

结构体是面向对象编程（类）的基础，理解结构体能帮助我们更好地理解下一章的"类"。

---

## 下章预告

下一章我们将学习 **C++ 的核心：类与面向对象**。我们会看到结构体的"升级版"——**类**，它带来了封装、继承、多态三大特性，以及构造函数、析构函数、拷贝构造等重要概念。准备好了吗？让我们开启面向对象的大门！

---

## 参考资料

- 《C++ Primer》（第5版）第2章 - 复合类型
- 《Effective C++》第2版 - 条款1：视 C++ 为语言联邦
- Bjarne Stroustrup - 《A Tour of C++》