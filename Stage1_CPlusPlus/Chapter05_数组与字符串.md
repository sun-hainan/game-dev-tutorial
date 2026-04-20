# Chapter 05：数组与字符串

> 🎯 目标：学会用数组批量管理数据（敌人列表、背包道具、地图瓦片），掌握字符串处理技能，为游戏数据管理打下坚实基础。

---

## 5.1 数组：定义、初始化、访问

### 生活中的类比 🏠

> **公寓楼：**
> - 一栋公寓有很多房间，每个房间有房号（0, 1, 2, 3...）
> - 住户A住在101室，住户B住在102室...
> - 数组就像一栋公寓楼，数据按顺序排列，每个位置有固定编号

### 为什么需要数组？

```cpp
// -------- 不用数组：每个敌人血量单独存 --------
int enemy1HP = 500;
int enemy2HP = 300;
int enemy3HP = 800;
int enemy4HP = 200;
int enemy5HP = 600;

// 遍历？不存在的！只能一个个写...
cout << enemy1HP << endl;
cout << enemy2HP << endl;
cout << enemy3HP << endl;
cout << enemy4HP << endl;
cout << enemy5HP << endl;

// -------- 用数组：5个敌人血量，一个搞定 --------
int enemyHPs[5] = {500, 300, 800, 200, 600};

// 一个循环，遍历所有敌人！
for (int i = 0; i < 5; i++) {
    cout << "敌人" << (i+1) << "血量：" << enemyHPs[i] << endl;
}
```

### 数组的定义与初始化

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 定义数组的几种方式 --------

    // 方式1：指定大小，初始化所有元素
    int scores[5];           // 定义大小为5的数组（未初始化，值是随机的）
    scores[0] = 100;         // 手动赋值
    scores[1] = 90;
    scores[2] = 85;

    // 方式2：定义时直接初始化（最常用！）
    int levels[5] = {1, 2, 3, 4, 5};   // 初始化列表

    // 方式3：让编译器自动数个数
    int gold[3] = {100, 200, 300};     // 编译器自动知道大小是3

    // 方式4：部分初始化（未指定的元素默认为0）
    int arrows[5] = {10, 20};           // arrows = {10, 20, 0, 0, 0}

    // 方式5：定义时可以不写大小（编译器推断）
    int exp[] = {100, 200, 300, 400};   // 编译器自动推断大小为4

    // -------- 访问数组元素 --------
    cout << "第1个元素：" << levels[0] << endl;   // 输出 1
    cout << "第2个元素：" << levels[1] << endl;   // 输出 2
    cout << "第3个元素：" << levels[2] << endl;   // 输出 3

    return 0;
}
```

### 数组的下标

> **重要规则：数组下标从 0 开始，不是从 1 开始！**

```cpp
#include <iostream>
using namespace std;

int main()
{
    // 定义一个包含5个元素的数组
    int items[5] = {10, 20, 30, 40, 50};

    // -------- 下标从 0 开始 --------
    cout << "items[0] = " << items[0] << endl;   // 第1个元素 = 10
    cout << "items[1] = " << items[1] << endl;   // 第2个元素 = 20
    cout << "items[2] = " << items[2] << endl;   // 第3个元素 = 30
    cout << "items[3] = " << items[3] << endl;   // 第4个元素 = 40
    cout << "items[4] = " << items[4] << endl;   // 第5个元素 = 50

    // -------- 越界访问（危险！）--------
    // cout << items[5] << endl;   // 越界！数组只有 0-4，items[5] 是未定义的！
    // cout << items[-1] << endl;  // 负数下标也是越界！

    // -------- 用循环遍历数组 --------
    cout << "\n===== 遍历所有道具 =====" << endl;
    for (int i = 0; i < 5; i++) {
        cout << "道具" << (i+1) << "数量：" << items[i] << endl;
    }

    return 0;
}
```

---

## 5.2 多维数组（以二维数组为主）

### 生活中的类比 📊

> **Excel 表格：**
> - 有行有列，用 (行, 列) 定位单元格
> - B3 单元格 = 第3列第B行
> - 二维数组就像 Excel 表格，用两个下标定位

### 为什么需要多维数组？

> 游戏中常见的应用：地图数据（二维网格）、棋盘游戏、像素图...

```cpp
// -------- 一维数组：单排货架 --------
int singleRow[5] = {1, 2, 3, 4, 5};   // 1行，5列

// -------- 二维数组：Excel表格 --------
int gameMap[3][4] = {
    {1, 0, 1, 0},   // 第1行：1=墙，0=空地
    {0, 1, 0, 1},   // 第2行
    {1, 0, 0, 1}    // 第3行
};
// 3 行，4 列 = 3x4 = 12 个格子
```

### 二维数组的定义与初始化

```cpp
#include <iostream>
using namespace std;

int main()
{
    // -------- 定义二维数组 --------
    // 语法：类型 数组名[行数][列数];

    // 方式1：定义时初始化
    int matrix[2][3] = {
        {1, 2, 3},    // 第1行
        {4, 5, 6}     // 第2行
    };

    // 方式2：不写行数（编译器自动推断）
    int grid[][3] = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9}
    };   // 编译器推断为 3 行

    // 方式3：部分初始化
    int sparse[3][4] = {
        {1, 0, 0, 0},
        {0, 2, 0, 0},
        {0, 0, 0, 3}
    };   // 对角线之外都是0（稀疏矩阵）

    // -------- 访问二维数组元素 --------
    cout << "第1行第1列：" << matrix[0][0] << endl;   // 输出 1
    cout << "第2行第3列：" << matrix[1][2] << endl;   // 输出 6

    // -------- 遍历二维数组（双层循环）--------
    cout << "\n===== 打印 3x3 地图 =====" << endl;
    for (int row = 0; row < 3; row++) {        // 外层循环：行
        for (int col = 0; col < 3; col++) {    // 内层循环：列
            cout << grid[row][col] << " ";     // 打印元素和空格
        }
        cout << endl;                          // 换行
    }

    return 0;
}
```

**运行结果：**
```
第1行第1列：1
第2行第3列：6

===== 打印 3x3 地图 =====
1 2 3
4 5 6
7 8 9
```

### 游戏开发实战：地图系统（二维数组实现）

```cpp
#include <iostream>
#include <string>
using namespace std;

// -------- 地图符号说明 --------
/*
 * 0 = 空地（可以通行）
 * 1 = 墙壁（不可通行）
 * 2 = 玩家起始点
 * 3 = 敌人
 * 4 = 宝藏
 */

int main()
{
    // -------- 定义一个 8x8 的地牢地图 --------
    int dungeonMap[8][8] = {
        {1, 1, 1, 1, 1, 1, 1, 1},
        {1, 2, 0, 0, 1, 3, 0, 1},   // 2=玩家, 3=敌人
        {1, 0, 1, 0, 0, 0, 0, 1},
        {1, 0, 0, 0, 1, 1, 0, 1},
        {1, 1, 1, 0, 0, 0, 0, 1},
        {1, 3, 0, 0, 1, 0, 0, 1},
        {1, 0, 0, 0, 0, 0, 4, 1},   // 4=宝藏
        {1, 1, 1, 1, 1, 1, 1, 1}
    };

    // -------- 打印地图 --------
    cout << "===== 地牢地图 =====" << endl;
    cout << "图例：0=空地 1=墙 2=玩家 3=敌人 4=宝藏" << endl << endl;

    for (int row = 0; row < 8; row++) {
        for (int col = 0; col < 8; col++) {
            cout << dungeonMap[row][col] << " ";
        }
        cout << endl;
    }

    // -------- 查找玩家位置 --------
    cout << "\n===== 查找玩家位置 =====" << endl;
    for (int row = 0; row < 8; row++) {
        for (int col = 0; col < 8; col++) {
            if (dungeonMap[row][col] == 2) {
                cout << "玩家位于：(" << row << ", " << col << ")" << endl;
            }
        }
    }

    // -------- 统计敌人数量 --------
    cout << "\n===== 统计敌人 =====" << endl;
    int enemyCount = 0;
    for (int row = 0; row < 8; row++) {
        for (int col = 0; col < 8; col++) {
            if (dungeonMap[row][col] == 3) {
                enemyCount++;
                cout << "敌人在：(" << row << ", " << col << ")" << endl;
            }
        }
    }
    cout << "敌人总数：" << enemyCount << endl;

    return 0;
}
```

---

## 5.3 字符串：C风格字符串 vs string类

### 生活中的类比 📝

> **纸条 vs 笔记本：**
> - C风格字符串像一张纸条，写完就不能加字了（固定长度）
> - string类像笔记本，需要时可以撕掉旧页、添加新页（动态大小）

### C风格字符串（字符数组）

```cpp
#include <iostream>
#include <cstring>   // C风格字符串操作函数
using namespace std;

int main()
{
    // -------- C风格字符串：字符数组 --------
    char name1[] = "Hero";        // 自动在末尾加 '\0'（字符串结束符）
    char name2[10] = "Luffy";     // 预留空间，可以容纳10个字符

    // -------- 字符串长度 --------
    cout << "name1 长度：" << strlen(name1) << endl;   // 输出 4

    // -------- 访问字符 --------
    cout << "name1[0] = " << name1[0] << endl;   // 输出 'H'
    cout << "name1[1] = " << name1[1] << endl;   // 输出 'e'

    // -------- 拼接（需要 strcpy + strcat）--------
    char dest[50] = "Hello ";     // 目标字符串，要有足够空间
    char src[] = "World";
    strcat(dest, src);            // 拼接：dest = "Hello World"
    cout << "拼接后：" << dest << endl;

    // -------- 复制 --------
    char copy[20];
    strcpy(copy, name1);          // 复制：copy = "Hero"
    cout << "复制后：" << copy << endl;

    // -------- 比较 --------
    char str1[] = "apple";
    char str2[] = "banana";
    cout << "strcmp(str1, str2) = " << strcmp(str1, str2) << endl;  // 负数：str1 < str2

    return 0;
}
```

### string类（C++标准库）

```cpp
#include <iostream>
#include <string>      // string 类头文件
using namespace std;

int main()
{
    // -------- 定义 string --------
    string playerName = "Luffy";    // 直接赋值，简洁！
    string emptyStr;                // 空字符串

    // -------- 赋值、拼接（比C风格简单100倍）--------
    string s1 = "Hello";
    string s2 = "World";
    string s3 = s1 + " " + s2;      // 直接用 + 拼接！
    cout << "拼接结果：" << s3 << endl;

    // -------- 长度 --------
    cout << "s3 长度：" << s3.length() << endl;   // 10
    cout << "s3 长度：" << s3.size() << endl;     // 同 length()

    // -------- 访问字符 --------
    cout << "s3[0] = " << s3[0] << endl;   // 'H'
    cout << "s3[6] = " << s3[6] << endl;   // 'W'

    // -------- 比较（直接用 ==）--------
    string a = "apple";
    string b = "apple";
    string c = "banana";
    cout << "a == b ? " << (a == b) << endl;   // 1（真）
    cout << "a == c ? " << (a == c) << endl;   // 0（假）

    // -------- 判断是否为空 --------
    cout << "emptyStr.empty() ? " << emptyStr.empty() << endl;  // 1（空）

    return 0;
}
```

---

## 5.4 字符串常见操作

### 核心操作一览

| 操作 | string方法 | 示例 |
|-----|-----------|------|
| 拼接 | `+` | `s1 + s2` |
| 长度 | `.length()` / `.size()` | `s.length()` |
| 比较 | `==`, `!=`, `<`, `>` | `s1 == s2` |
| 子串 | `.substr(pos, len)` | `s.substr(0, 3)` |
| 查找 | `.find(str)` | `s.find("abc")` |
| 替换 | `.replace(pos, len, str)` | `s.replace(0, 3, "xyz")` |
| 插入 | `.insert(pos, str)` | `s.insert(0, "Hello")` |
| 删除 | `.erase(pos, len)` | `s.erase(0, 3)` |

### 游戏开发实战：字符串处理

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;

int main()
{
    // -------- 1. 拼接：生成玩家完整信息 --------
    string playerName = "Luffy";
    string title = "海贼王";
    string fullTitle = playerName + " - " + title;   // 拼接
    cout << "完整称号：" << fullTitle << endl;

    // -------- 2. 子串：提取武器名称 --------
    string weaponCode = "[传说]火之高兴-炎帝";
    string quality = weaponCode.substr(0, 4);          // 取前4个字符
    string name = weaponCode.substr(5, 4);            // 从位置5开始，取4个字符
    cout << "品质：" << quality << endl;
    cout << "名称：" << name << endl;

    // -------- 3. 查找：检查字符串是否包含关键词 --------
    string itemDesc = "传说级武器，攻击力+500，暴击率+20%";
    if (itemDesc.find("传说") != string::npos) {
        cout << "这是传说级装备！" << endl;
    }
    if (itemDesc.find("攻击力") != string::npos) {
        cout << "有攻击力加成属性" << endl;
    }

    // -------- 4. 替换：敏感词过滤（聊天系统）--------
    string chatMessage = "你这个白痴玩家！";
    string filtered = chatMessage;
    if (filtered.find("白痴") != string::npos) {
        filtered.replace(filtered.find("白痴"), 2, "**");   // 把"白痴"替换成"**"
    }
    cout << "原消息：" << chatMessage << endl;
    cout << "过滤后：" << filtered << endl;

    // -------- 5. 分割：解析CSV格式道具数据 --------
    string csvData = "1001,铁剑,50,10";
    vector<string> tokens;
    string temp = "";
    for (int i = 0; i < csvData.length(); i++) {
        if (csvData[i] == ',') {
            tokens.push_back(temp);    // 遇到逗号，保存当前token
            temp = "";
        } else {
            temp += csvData[i];        // 累加字符
        }
    }
    tokens.push_back(temp);           // 最后一个token

    cout << "\n===== 解析道具CSV =====" << endl;
    cout << "ID：" << tokens[0] << endl;
    cout << "名称：" << tokens[1] << endl;
    cout << "价格：" << tokens[2] << endl;
    cout << "数量：" << tokens[3] << endl;

    return 0;
}
```

---

## 5.5 游戏开发实战：背包系统、怪物列表、地图数据

### 实战1：背包系统（数组 + string）

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <iomanip>
using namespace std;

// -------- 道具结构体 --------
struct Item {
    string name;     // 名称
    int quantity;    // 数量
    int price;       // 单价
};

// -------- 显示背包 --------
void showInventory(Item inventory[], int itemCount) {
    cout << "\n========== 背包 ==========" << endl;
    cout << setw(5) << "序号" << setw(15) << "名称" << setw(10) << "数量" << setw(10) << "总价" << endl;
    cout << string(40, '-') << endl;

    int totalValue = 0;
    for (int i = 0; i < itemCount; i++) {
        cout << setw(5) << (i+1)
             << setw(15) << inventory[i].name
             << setw(10) << inventory[i].quantity
             << setw(10) << (inventory[i].quantity * inventory[i].price) << endl;
        totalValue += inventory[i].quantity * inventory[i].price;
    }
    cout << string(40, '-') << endl;
    cout << "背包总价值：" << totalValue << " 金币" << endl;
}

// -------- 购买道具 --------
void buyItem(Item inventory[], int& itemCount, string itemName, int quantity, int price) {
    // 先查找背包中是否已有该道具
    for (int i = 0; i < itemCount; i++) {
        if (inventory[i].name == itemName) {
            inventory[i].quantity += quantity;   // 叠加数量
            cout << "购买成功！" << itemName << " x" << quantity << endl;
            return;
        }
    }
    // 背包中没有，新增道具
    inventory[itemCount].name = itemName;
    inventory[itemCount].quantity = quantity;
    inventory[itemCount].price = price;
    itemCount++;
    cout << "购买成功！新增道具 " << itemName << " x" << quantity << endl;
}

// -------- 出售道具 --------
bool sellItem(Item inventory[], int& itemCount, string itemName, int quantity) {
    // 查找道具
    for (int i = 0; i < itemCount; i++) {
        if (inventory[i].name == itemName) {
            if (inventory[i].quantity >= quantity) {
                inventory[i].quantity -= quantity;
                int earnings = quantity * inventory[i].price / 2;   // 卖出价是原价一半
                cout << "出售成功！获得 " << earnings << " 金币" << endl;
                // 如果数量为0，移除道具
                if (inventory[i].quantity == 0) {
                    for (int j = i; j < itemCount - 1; j++) {
                        inventory[j] = inventory[j + 1];
                    }
                    itemCount--;
                }
                return true;
            } else {
                cout << "数量不足！" << endl;
                return false;
            }
        }
    }
    cout << "背包中没有 " << itemName << "！" << endl;
    return false;
}

int main()
{
    // -------- 初始背包 --------
    const int MAX_ITEMS = 20;    // 背包最大容量
    Item inventory[MAX_ITEMS];
    int itemCount = 3;

    // -------- 初始道具 --------
    inventory[0] = {"生命药水", 5, 50};
    inventory[1] = {"魔法药水", 3, 80};
    inventory[2] = {"铁剑", 1, 500};

    // -------- 显示初始背包 --------
    showInventory(inventory, itemCount);

    // -------- 玩家操作 --------
    cout << "\n===== 商店 =====" << endl;
    buyItem(inventory, itemCount, "火球术卷轴", 2, 300);
    buyItem(inventory, itemCount, "生命药水", 3, 50);   // 叠加到现有道具
    buyItem(inventory, itemCount, "凤凰羽毛", 1, 5000);

    showInventory(inventory, itemCount);

    // -------- 出售道具 --------
    cout << "\n===== 出售 =====" << endl;
    sellItem(inventory, itemCount, "生命药水", 4);
    sellItem(inventory, itemCount, "铁剑", 1);

    showInventory(inventory, itemCount);

    return 0;
}
```

### 实战2：怪物列表管理

```cpp
#include <iostream>
#include <string>
#include <iomanip>
using namespace std;

// -------- 怪物结构体 --------
struct Monster {
    string name;      // 名称
    int hp;           // 血量
    int attack;       // 攻击力
    int defense;      // 防御力
    string type;      // 类型：野兽/亡灵/恶魔/元素
};

// -------- 打印怪物列表 --------
void printMonsterList(Monster monsters[], int count) {
    cout << "\n========== 怪物图鉴 ==========" << endl;
    cout << setw(4) << "ID" << setw(12) << "名称" << setw(8) << "血量"
         << setw(10) << "攻击" << setw(10) << "防御" << setw(8) << "类型" << endl;
    cout << string(52, '-') << endl;

    for (int i = 0; i < count; i++) {
        cout << setw(4) << (i+1)
             << setw(12) << monsters[i].name
             << setw(8) << monsters[i].hp
             << setw(10) << monsters[i].attack
             << setw(10) << monsters[i].defense
             << setw(8) << monsters[i].type << endl;
    }
}

// -------- 计算总战斗力 --------
int totalBattlePower(Monster monsters[], int count) {
    int total = 0;
    for (int i = 0; i < count; i++) {
        total += monsters[i].attack + monsters[i].defense + monsters[i].hp / 10;
    }
    return total;
}

// -------- 查找最强敌人 --------
Monster findStrongest(Monster monsters[], int count) {
    Monster strongest = monsters[0];
    int maxPower = 0;

    for (int i = 0; i < count; i++) {
        int power = monsters[i].attack + monsters[i].defense;
        if (power > maxPower) {
            maxPower = power;
            strongest = monsters[i];
        }
    }
    return strongest;
}

// -------- 主函数 --------
int main()
{
    // -------- 定义怪物数组 --------
    Monster monsterList[5] = {
        {"哥布林", 100, 15, 5, "野兽"},
        {"骷髅战士", 200, 25, 15, "亡灵"},
        {"火焰元素", 150, 35, 5, "元素"},
        {"暗影刺客", 120, 40, 10, "恶魔"},
        {"巨魔", 500, 30, 25, "野兽"}
    };

    int monsterCount = 5;

    // -------- 打印怪物列表 --------
    printMonsterList(monsterList, monsterCount);

    // -------- 计算相关信息 --------
    cout << "\n===== 统计信息 =====" << endl;
    cout << "怪物总数：" << monsterCount << endl;
    cout << "队伍总战斗力：" << totalBattlePower(monsterList, monsterCount) << endl;

    Monster boss = findStrongest(monsterList, monsterCount);
    cout << "最强敌人：" << boss.name << "（攻击" << boss.attack
         << "，防御" << boss.defense << "）" << endl;

    return 0;
}
```

---

## 5.6 数组作为函数参数

### 核心概念

> 数组作为函数参数时，传递的是**数组首元素的地址**（即引用传递），而不是整个数组的副本。

### 示例

```cpp
#include <iostream>
using namespace std;

// -------- 打印数组（常量指针，不能修改）--------
void printArray(const int arr[], int size) {
    cout << "数组元素：";
    for (int i = 0; i < size; i++) {
        cout << arr[i] << " ";
    }
    cout << endl;
}

// -------- 求数组总和 --------
int sumArray(int arr[], int size) {
    int total = 0;
    for (int i = 0; i < size; i++) {
        total += arr[i];
    }
    return total;
}

// -------- 求数组最大值 --------
int maxArray(int arr[], int size) {
    int maxVal = arr[0];
    for (int i = 1; i < size; i++) {
        if (arr[i] > maxVal) {
            maxVal = arr[i];
        }
    }
    return maxVal;
}

// -------- 修改数组（引用传递的本质）--------
void doubleArray(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        arr[i] = arr[i] * 2;    // 真的改了原数组！
    }
}

// -------- 主函数 --------
int main()
{
    int numbers[] = {10, 20, 30, 40, 50};
    int size = 5;

    // -------- 打印 --------
    printArray(numbers, size);

    // -------- 求和 & 求最大 --------
    cout << "总和：" << sumArray(numbers, size) << endl;   // 150
    cout << "最大：" << maxArray(numbers, size) << endl;   // 50

    // -------- 修改数组 --------
    doubleArray(numbers, size);
    cout << "加倍后：";
    printArray(numbers, size);   // 输出 20 40 60 80 100

    return 0;
}
```

---

## 5.7 动态数组 vector 简介

### 为什么需要 vector？

> 普通数组大小固定，创建后不能改变。`vector` 是**动态数组**，可以随时添加/删除元素。

### 基本用法

```cpp
#include <iostream>
#include <vector>     // vector 头文件
using namespace std;

int main()
{
    // -------- 定义 vector --------
    vector<int> scores;           // 空 vector
    vector<int> levels(5);        // 大小为5，初始值为0
    vector<int> items{1, 2, 3};   // 初始化列表（C++11）

    // -------- 添加元素 --------
    scores.push_back(100);        // 末尾添加
    scores.push_back(90);
    scores.push_back(85);

    // -------- 访问元素（和数组一样）--------
    cout << "scores[0] = " << scores[0] << endl;
    cout << "scores.at(1) = " << scores.at(1) << endl;   // at() 会检查越界

    // -------- 大小 --------
    cout << "元素个数：" << scores.size() << endl;

    // -------- 判断空 --------
    vector<int> emptyVec;
    cout << "emptyVec.empty() = " << emptyVec.empty() << endl;   // 1（真）

    // -------- 删除元素 --------
    scores.pop_back();            // 删除末尾元素
    cout << "删除后大小：" << scores.size() << endl;

    // -------- 遍历 --------
    cout << "所有元素：";
    for (int i = 0; i < scores.size(); i++) {
        cout << scores[i] << " ";
    }
    cout << endl;

    // -------- 范围for循环（C++11）--------
    cout << "范围for：";
    for (int s : scores) {
        cout << s << " ";
    }
    cout << endl;

    return 0;
}
```

### vector 在游戏开发中的应用

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// -------- 动态背包系统 --------
int main()
{
    vector<string> backpack;       // 动态背包（开始为空）

    // -------- 添加道具 --------
    backpack.push_back("铁剑");
    backpack.push_back("生命药水 x3");
    backpack.push_back("魔法钥匙");
    backpack.push_back("火焰戒指");

    cout << "===== 初始背包 =====" << endl;
    for (int i = 0; i < backpack.size(); i++) {
        cout << i+1 << ". " << backpack[i] << endl;
    }

    // -------- 删除道具（使用魔法钥匙后）--------
    cout << "\n===== 使用魔法钥匙 =====" << endl;
    for (auto it = backpack.begin(); it != backpack.end(); ) {
        if (*it == "魔法钥匙") {
            cout << "使用：" << *it << endl;
            it = backpack.erase(it);    // erase 返回下一个迭代器
        } else {
            ++it;
        }
    }

    cout << "\n===== 当前背包 =====" << endl;
    for (int i = 0; i < backpack.size(); i++) {
        cout << i+1 << ". " << backpack[i] << endl;
    }

    // -------- 清空背包 --------
    cout << "\n===== 清空背包 =====" << endl;
    backpack.clear();
    cout << "背包大小：" << backpack.size() << endl;

    return 0;
}
```

---

## 5.8 常见错误与调试

### 常见错误一览

| 错误 | 示例 | 后果 |
|-----|------|-----|
| 数组越界 | `arr[5]`，但数组大小只有5 | 程序崩溃或数据错乱 |
| 未初始化 | `int arr[10];` 直接用 | 值是随机的 |
| 下标负数 | `arr[-1]` | 访问未知内存 |
| 字符串忘记`\0` | `char s[3] = "Hi"` | 越界读取 |
| `sizeof`误解 | `sizeof(arr)` vs `strlen(s)` | 得到意外结果 |

### 调试技巧

```cpp
#include <iostream>
using namespace std;

int main()
{
    int scores[5] = {100, 90, 85, 70, 60};

    // -------- 调试技巧1：打印下标和值 --------
    cout << "===== 调试：打印下标 =====" << endl;
    for (int i = 0; i < 5; i++) {
        cout << "scores[" << i << "] = " << scores[i] << endl;
    }

    // -------- 调试技巧2：用 assert 检查边界（C++11）--------
    // #include <cassert>
    // assert(i >= 0 && i < 5);  // 如果条件不满足，程序崩溃并报错

    // -------- 调试技巧3：用 vector 代替原始数组 --------
    // vector 会自动管理大小，减少越界风险
    // vector<int> safeScores = {100, 90, 85, 70, 60};
    // cout << safeScores.at(10);   // 越界会抛出异常，而不是崩溃

    // -------- 常见错误示例：数组越界 --------
    cout << "\n===== 越界测试（危险！）=====" << endl;
    int small[3] = {1, 2, 3};
    cout << "small[0] = " << small[0] << endl;   // 安全
    cout << "small[2] = " << small[2] << endl;   // 安全（最后一个）
    // cout << "small[3] = " << small[3] << endl; // 越界！可能输出垃圾值
    // cout << "small[100] = " << small[100] << endl; // 严重越界！可能崩溃

    return 0;
}
```

---

## 5.9 动手练习 🧪

### 练习 1：数组初始化
定义一个包含5个元素的数组，存储5个怪物的血量：`300, 150, 500, 200, 400`，然后打印所有元素。⭐

### 练习 2：二维数组地图
创建一个 4x4 的棋盘地图，用 0 表示空地，1 表示障碍物，打印出棋盘并统计障碍物数量。⭐

### 练习 3：字符串操作
定义玩家称号 "Lv.99 刺客"，用 substr 分别提取等级（Lv.99）和职业（刺客）。⭐⭐

### 练习 4：背包系统
用 struct 定义道具（名称、数量、价格），用数组存储5种道具，实现显示背包和计算总价的功能。⭐⭐

### 练习 5：vector 动态数组
用 vector 存储游戏中的敌人名字，实现添加敌人、移除指定敌人、统计敌人总数的功能。⭐⭐⭐

---

## 本章小结

```
✅ 已掌握：
├── 数组的定义与初始化
├── 数组下标（从0开始）
├── 多维数组（二维数组）
├── C风格字符串（字符数组）
├── string 类常用操作
├── 数组作为函数参数
├── vector 动态数组
└── 常见错误与调试

🔜 下章预告：
第六章：指针基础 —— 揭开内存地址的神秘面纱，理解指针与数组的关系，掌握游戏开发中敌人状态切换的技巧。
```

---

_📚 参考资料：《C++ Primer》《Effective C++》《游戏编程模式》_
