# Chapter 11 — STL 容器

> 参考资料：《C++ Primer（第5版）》《Effective STL》Scott Meyers  
> 适合读者：已掌握 C++ 基础语法、类与对象的学习者

---

## 11.1 STL 三大组件：容器、算法、迭代器

### 🍱 生活类比

把 STL 想象成一座**超级食堂**：
- **容器（Container）** = 各种餐盘和锅碗（存放食物/数据）
- **算法（Algorithm）** = 厨师的烹饪手法（排序、查找、变换）
- **迭代器（Iterator）** = 服务员的托盘（在容器和算法之间传递数据）

三者分工明确，却协作无间。

### 💡 核心思想

STL（Standard Template Library）是 C++ 标准库的核心，它通过**泛型编程**将数据结构和算法解耦：
- 容器管理数据的存储方式
- 算法处理数据的逻辑
- 迭代器充当二者之间统一的"接口"

```cpp
#include <iostream>
#include <vector>       // 容器
#include <algorithm>    // 算法
using namespace std;

int main() {
    vector<int> nums = {5, 3, 1, 4, 2};  // 容器：存储整数序列

    sort(nums.begin(), nums.end());        // 算法：排序
    // nums.begin() 和 nums.end() 就是迭代器

    for (auto it = nums.begin(); it != nums.end(); ++it) {
        cout << *it << " ";               // 通过迭代器访问元素
    }
    // 输出：1 2 3 4 5
    return 0;
}
```

---

## 11.2 vector（动态数组）

### 🍱 生活类比

`vector` 就像一条**可以自动加长的购物收据**：
- 从左到右按顺序记录每件商品（随机访问）
- 买的东西越来越多，收据自动变长（动态扩容）
- 想删中间某行很麻烦（插入/删除效率低）

### 💡 核心思想

`vector` 是最常用的 STL 容器，底层是**连续内存**，支持 O(1) 随机访问，尾部插入/删除高效，中间操作代价较高。

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    // --- 创建与初始化 ---
    vector<int> hp;                         // 空 vector
    vector<int> scores = {100, 90, 85, 70}; // 初始化列表

    // --- push_back：尾部追加 ---
    hp.push_back(200);  // hp = [200]
    hp.push_back(150);  // hp = [200, 150]
    hp.push_back(300);  // hp = [200, 150, 300]

    // --- size：元素个数 ---
    cout << "size: " << hp.size() << endl;  // 输出：3

    // --- [] 操作符：随机访问（不检查越界）---
    cout << "hp[0] = " << hp[0] << endl;    // 输出：200

    // --- at：安全访问（越界抛出 std::out_of_range）---
    cout << "hp.at(1) = " << hp.at(1) << endl; // 输出：150

    // --- 迭代器遍历 ---
    cout << "所有HP：";
    for (auto it = hp.begin(); it != hp.end(); ++it) {
        cout << *it << " ";  // 解引用迭代器得到值
    }
    cout << endl;

    // --- erase：按迭代器删除元素 ---
    hp.erase(hp.begin() + 1);  // 删除索引1的元素（150）
    // hp = [200, 300]

    // --- clear：清空所有元素 ---
    scores.clear();
    cout << "scores.empty() = " << scores.empty() << endl;  // 1 (true)

    // --- 范围 for 遍历（现代写法）---
    vector<string> items = {"剑", "盾", "药水"};
    for (const auto& item : items) {
        cout << item << " ";
    }
    cout << endl;

    return 0;
}
```

> ⚠️ **注意**：`[]` 不检查越界，`at()` 会抛异常——调试时优先用 `at()`。

---

## 11.3 list（双向链表）

### 🍱 生活类比

`list` 就像一列**火车车厢**：
- 每节车厢连接前后两节（双向链表）
- 可以随时在任意位置加挂/摘除车厢（高效插入/删除）
- 但要找第 N 节必须从头数（不支持随机访问）

### 💡 核心思想

`list` 底层是**双向链表**，任意位置插入/删除为 O(1)，但不支持随机访问（无 `[]`），适合频繁增删的场景。

```cpp
#include <iostream>
#include <list>
using namespace std;

int main() {
    list<string> monsterQueue;

    // --- push_back：尾部插入 ---
    monsterQueue.push_back("哥布林");
    monsterQueue.push_back("骷髅兵");
    monsterQueue.push_back("火焰龙");

    // --- push_front：头部插入 ---
    monsterQueue.push_front("史莱姆");  // 优先级最高的怪物放最前

    // 当前：史莱姆 → 哥布林 → 骷髅兵 → 火焰龙

    // --- 遍历 ---
    cout << "怪物队列：";
    for (const auto& m : monsterQueue) {
        cout << m << " ";
    }
    cout << endl;

    // --- remove：按值删除所有匹配项 ---
    monsterQueue.remove("哥布林");  // 删除所有"哥布林"

    // --- sort：链表内置排序（不能用 std::sort）---
    list<int> levels = {5, 2, 8, 1, 9, 3};
    levels.sort();  // 升序
    // levels = [1, 2, 3, 5, 8, 9]

    levels.sort(greater<int>());  // 降序
    // levels = [9, 8, 5, 3, 2, 1]

    cout << "等级排序：";
    for (int lv : levels) cout << lv << " ";
    cout << endl;

    return 0;
}
```

---

## 11.4 deque（双端队列）

### 🍱 生活类比

`deque` 就像**地铁站台上可以两端上下车的队列**：
- 前门可以上/下客（头部 O(1) 操作）
- 后门也可以上/下客（尾部 O(1) 操作）
- 中间依然支持随机访问（类似 vector）

### 💡 核心思想

`deque`（double-ended queue）兼具 `vector`（随机访问）和 `list`（头部高效操作）的优点，底层是**分段连续内存**。`stack` 和 `queue` 默认以 `deque` 为底层容器。

```cpp
#include <iostream>
#include <deque>
using namespace std;

int main() {
    deque<int> taskList;

    // --- 两端插入 ---
    taskList.push_back(10);   // 尾部：[10]
    taskList.push_back(20);   // 尾部：[10, 20]
    taskList.push_front(5);   // 头部：[5, 10, 20]
    taskList.push_front(1);   // 头部：[1, 5, 10, 20]

    // --- 随机访问 ---
    cout << "第2个任务优先级：" << taskList[1] << endl;  // 5

    // --- 两端删除 ---
    taskList.pop_front();  // 删头：[5, 10, 20]
    taskList.pop_back();   // 删尾：[5, 10]

    // --- 遍历 ---
    for (int t : taskList) cout << t << " ";
    cout << endl;

    return 0;
}
```

---

## 11.5 stack（栈）

### 🍱 生活类比

`stack` 就像**叠放的餐盘**：
- 只能从顶部放盘子（push）
- 只能从顶部取盘子（pop/top）
- **后进先出**（LIFO）

### 💡 核心思想

`stack` 是**容器适配器**，只暴露 `push/pop/top/empty` 接口，完全封装了底层（默认 `deque`）。典型应用：函数调用栈、括号匹配、撤销操作。

```cpp
#include <iostream>
#include <stack>
using namespace std;

int main() {
    stack<string> undoStack;  // 撤销历史栈

    // --- push：压栈 ---
    undoStack.push("移动玩家");
    undoStack.push("拾取道具");
    undoStack.push("使用技能");

    // --- top：查看栈顶（不弹出）---
    cout << "最近操作：" << undoStack.top() << endl;  // "使用技能"

    // --- empty：判空 ---
    while (!undoStack.empty()) {
        cout << "撤销：" << undoStack.top() << endl;
        undoStack.pop();  // 弹栈
    }
    // 输出顺序：使用技能 → 拾取道具 → 移动玩家

    return 0;
}
```

---

## 11.6 queue（队列）

### 🍱 生活类比

`queue` 就像**银行排号系统**：
- 新客户从后面排队（push）
- 叫号从前面叫（front/pop）
- **先进先出**（FIFO）

### 💡 核心思想

`queue` 也是容器适配器，提供 `push/pop/front/back/empty` 接口。经典用途：任务调度、BFS 广度优先搜索、消息队列。

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main() {
    queue<string> spawnQueue;  // 怪物刷新队列

    // --- push：入队 ---
    spawnQueue.push("哥布林A");
    spawnQueue.push("哥布林B");
    spawnQueue.push("精英兽人");

    // --- front/back：查看头尾（不弹出）---
    cout << "下一个出生：" << spawnQueue.front() << endl;  // 哥布林A
    cout << "最后入队：" << spawnQueue.back() << endl;     // 精英兽人

    // --- empty + pop：依次处理 ---
    while (!spawnQueue.empty()) {
        cout << "刷新怪物：" << spawnQueue.front() << endl;
        spawnQueue.pop();  // 出队
    }

    return 0;
}
```

---

## 11.7 set / multiset

### 🍱 生活类比

`set` 就像**图书馆的分类书架**：
- 书按字母/编号**自动排好序**
- **同一本书只放一本**（不重复）
- 查找特定书很快（二叉搜索树，O(log n)）

`multiset` 类似，但允许放多本相同的书（可重复）。

### 💡 核心思想

`set` 底层是**红黑树**，元素自动有序，所有操作 O(log n)。适合需要有序且唯一的集合。

```cpp
#include <iostream>
#include <set>
using namespace std;

int main() {
    // --- set：不重复，自动排序 ---
    set<int> levels;

    // --- insert：插入 ---
    levels.insert(10);
    levels.insert(5);
    levels.insert(20);
    levels.insert(5);   // 重复，忽略
    levels.insert(15);

    // set 自动升序：5 10 15 20
    for (int lv : levels) cout << lv << " ";
    cout << endl;

    // --- find：查找（返回迭代器，找不到返回 end()）---
    auto it = levels.find(15);
    if (it != levels.end()) {
        cout << "找到等级：" << *it << endl;
    }

    // --- erase：按值删除 ---
    levels.erase(10);

    // --- multiset：可重复 ---
    multiset<string> tags;
    tags.insert("火属性");
    tags.insert("火属性");  // 允许重复
    tags.insert("冰属性");

    cout << "火属性数量：" << tags.count("火属性") << endl;  // 2

    return 0;
}
```

---

## 11.8 map / multimap

### 🍱 生活类比

`map` 就像**新华字典**：
- 每个词条（key）对应一个释义（value）
- 词条按字母**自动排序**
- 同一个词条只出现一次（key 唯一）

`multimap` 允许同一 key 对应多个 value。

### 💡 核心思想

`map` 是**键值对容器**，底层红黑树，key 自动有序唯一，查找/插入/删除均 O(log n)。

```cpp
#include <iostream>
#include <map>
using namespace std;

int main() {
    map<string, int> playerScore;  // 玩家名 → 分数

    // --- insert：插入键值对 ---
    playerScore.insert({"Alice", 1500});
    playerScore.insert({"Bob", 2000});
    playerScore.insert(make_pair("Charlie", 1800));

    // --- [] 操作符：访问或新建 ---
    playerScore["Dave"] = 1200;       // 新建
    playerScore["Alice"] = 1600;      // 修改（覆盖）

    // --- at：安全访问（key 不存在抛异常）---
    cout << "Bob 的分数：" << playerScore.at("Bob") << endl;

    // --- find：查找 ---
    auto it = playerScore.find("Charlie");
    if (it != playerScore.end()) {
        cout << it->first << " 的分数：" << it->second << endl;
    }

    // --- 遍历（按 key 升序）---
    cout << "\n--- 排行榜 ---\n";
    for (const auto& [name, score] : playerScore) {  // C++17 结构化绑定
        cout << name << ": " << score << endl;
    }

    // --- erase：按 key 删除 ---
    playerScore.erase("Dave");

    // --- multimap：一个 key 对应多个 value ---
    multimap<string, string> itemDrop;
    itemDrop.insert({"史莱姆", "史莱姆凝胶"});
    itemDrop.insert({"史莱姆", "经验球"});     // 同一怪物掉落多种物品
    itemDrop.insert({"骷髅", "骨头"});

    cout << "\n史莱姆掉落物：";
    auto range = itemDrop.equal_range("史莱姆");
    for (auto it2 = range.first; it2 != range.second; ++it2) {
        cout << it2->second << " ";
    }
    cout << endl;

    return 0;
}
```

---

## 11.9 unordered_map / unordered_set

### 🍱 生活类比

`unordered_map` 就像**超市的条形码扫描仪**：
- 扫码（哈希函数）直接定位到货架位置
- **O(1) 平均查找**，极快
- 但货物不按顺序排列（无序）

### 💡 核心思想

`unordered_map/set` 底层是**哈希表**，平均 O(1) 操作，适合对速度要求极高、不需要有序的场景。最坏情况（哈希冲突严重）退化到 O(n)。

```cpp
#include <iostream>
#include <unordered_map>
#include <unordered_set>
using namespace std;

int main() {
    // --- unordered_map：哈希键值对 ---
    unordered_map<int, string> itemDB;  // 物品ID → 物品名

    itemDB[1001] = "铁剑";
    itemDB[1002] = "皮甲";
    itemDB[2001] = "回血药";
    itemDB[2002] = "魔法水";

    // O(1) 查找！
    cout << "ID 1001: " << itemDB[1001] << endl;

    // --- find ---
    auto it = itemDB.find(2001);
    if (it != itemDB.end()) {
        cout << "找到：" << it->second << endl;
    }

    // --- 遍历（无序）---
    for (const auto& [id, name] : itemDB) {
        cout << id << " -> " << name << endl;
    }

    // --- unordered_set：哈希集合，O(1) 去重查找 ---
    unordered_set<string> visitedAreas;  // 已探索区域
    visitedAreas.insert("新手村");
    visitedAreas.insert("黑森林");
    visitedAreas.insert("新手村");  // 重复，忽略

    if (visitedAreas.count("黑森林")) {
        cout << "已探索：黑森林" << endl;
    }

    return 0;
}
```

> 📊 **性能对比**：
> | 容器 | 查找 | 有序 | 适用场景 |
> |------|------|------|---------|
> | `map` | O(log n) | ✅ | 排行榜、字典 |
> | `unordered_map` | O(1) 均摊 | ❌ | 高频 ID 查询 |

---

## 11.10 迭代器用法

### 🍱 生活类比

迭代器就像**书签+翻页功能**：
- 书签指向当前位置（迭代器指向某个元素）
- 向后翻一页（`++it`）
- 读当前内容（`*it`）

### 💡 核心思想

迭代器是**容器和算法之间的桥梁**，统一了不同容器的遍历接口。主要迭代器类型：
- **正向迭代器**：`begin()` → `end()`
- **反向迭代器**：`rbegin()` → `rend()`
- **常量迭代器**：`cbegin()` → `cend()`

```cpp
#include <iostream>
#include <vector>
#include <list>
using namespace std;

int main() {
    vector<int> hp = {100, 80, 60, 40, 20};

    // --- begin/end 正向遍历 ---
    cout << "正向：";
    for (auto it = hp.begin(); it != hp.end(); ++it) {
        cout << *it << " ";  // 解引用获取值
    }
    cout << endl;

    // --- rbegin/rend 反向遍历 ---
    cout << "反向：";
    for (auto rit = hp.rbegin(); rit != hp.rend(); ++rit) {
        cout << *rit << " ";
    }
    cout << endl;

    // --- 迭代器算术（仅随机访问迭代器支持）---
    auto it = hp.begin() + 2;  // 指向第3个元素
    cout << "第3个HP：" << *it << endl;  // 60

    // --- 修改元素 ---
    for (auto it2 = hp.begin(); it2 != hp.end(); ++it2) {
        *it2 += 10;  // 所有HP +10
    }

    // --- cbegin/cend 只读迭代器 ---
    for (auto cit = hp.cbegin(); cit != hp.cend(); ++cit) {
        // *cit = 0;  // ❌ 编译错误，常量迭代器不可修改
        cout << *cit << " ";
    }
    cout << endl;

    return 0;
}
```

---

## 11.11 游戏开发实战

### 🎮 实战1：vector 背包道具管理

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
using namespace std;

struct Item {
    int id;
    string name;
    int quantity;
};

class Backpack {
private:
    vector<Item> items;
    int maxSlots = 10;  // 最大格数

public:
    // 添加道具（若已有则叠加数量）
    void addItem(const Item& newItem) {
        for (auto& item : items) {
            if (item.id == newItem.id) {
                item.quantity += newItem.quantity;
                cout << "[背包] " << item.name << " +1，共 " << item.quantity << " 个\n";
                return;
            }
        }
        if ((int)items.size() >= maxSlots) {
            cout << "[背包] 背包已满！\n";
            return;
        }
        items.push_back(newItem);
        cout << "[背包] 获得新道具：" << newItem.name << "\n";
    }

    // 使用道具（消耗1个）
    void useItem(int id) {
        for (auto it = items.begin(); it != items.end(); ++it) {
            if (it->id == id) {
                it->quantity--;
                cout << "[背包] 使用 " << it->name << "，剩余 " << it->quantity << " 个\n";
                if (it->quantity <= 0) {
                    items.erase(it);  // 数量为0则移除
                    cout << "[背包] 道具耗尽，已移除\n";
                }
                return;
            }
        }
        cout << "[背包] 道具不存在！\n";
    }

    // 显示背包
    void show() const {
        cout << "\n=== 背包 (" << items.size() << "/" << maxSlots << ") ===\n";
        for (const auto& item : items) {
            cout << "  [" << item.id << "] " << item.name
                 << " x" << item.quantity << "\n";
        }
    }
};

int main() {
    Backpack bag;
    bag.addItem({1001, "回血药", 1});
    bag.addItem({1002, "魔法水", 3});
    bag.addItem({1001, "回血药", 1});  // 叠加
    bag.show();

    bag.useItem(1001);
    bag.useItem(1001);
    bag.show();

    return 0;
}
```

---

### 🎮 实战2：list 怪物刷新列表

```cpp
#include <iostream>
#include <list>
#include <string>
using namespace std;

struct Monster {
    string name;
    int hp;
    int spawnPriority;  // 刷新优先级（越小越先刷）
};

int main() {
    list<Monster> spawnList;

    // 加入刷新队列
    spawnList.push_back({"史莱姆", 50, 3});
    spawnList.push_back({"骷髅兵", 120, 2});
    spawnList.push_front({"精英哥布林", 200, 1});  // 高优先级放前面

    // 按优先级排序刷新
    spawnList.sort([](const Monster& a, const Monster& b) {
        return a.spawnPriority < b.spawnPriority;
    });

    cout << "=== 怪物刷新顺序 ===\n";
    for (const auto& m : spawnList) {
        cout << "[优先级" << m.spawnPriority << "] "
             << m.name << " (HP:" << m.hp << ")\n";
    }

    // 模拟玩家消灭怪物，从列表移除
    spawnList.remove_if([](const Monster& m) {
        return m.hp < 100;  // 移除所有HP<100的弱小怪物
    });

    cout << "\n消灭弱小怪物后，剩余：\n";
    for (const auto& m : spawnList) {
        cout << "  " << m.name << "\n";
    }

    return 0;
}
```

---

### 🎮 实战3：map 分数排行榜

```cpp
#include <iostream>
#include <map>
#include <vector>
#include <algorithm>
using namespace std;

class Leaderboard {
private:
    map<string, int> scores;  // 玩家名 → 分数

public:
    // 更新分数（取最高分）
    void updateScore(const string& player, int score) {
        if (scores.find(player) == scores.end() || scores[player] < score) {
            scores[player] = score;
            cout << "[排行榜] " << player << " 分数更新为 " << score << "\n";
        }
    }

    // 显示 Top N
    void showTop(int n) {
        // map 按 key 排序，我们需要按 value 排序
        vector<pair<string, int>> ranking(scores.begin(), scores.end());
        sort(ranking.begin(), ranking.end(),
             [](const auto& a, const auto& b) { return a.second > b.second; });

        cout << "\n=== TOP " << n << " 排行榜 ===\n";
        for (int i = 0; i < min(n, (int)ranking.size()); i++) {
            cout << "#" << (i+1) << " " << ranking[i].first
                 << " - " << ranking[i].second << "分\n";
        }
    }
};

int main() {
    Leaderboard lb;
    lb.updateScore("Alice", 1500);
    lb.updateScore("Bob", 2100);
    lb.updateScore("Charlie", 1800);
    lb.updateScore("Alice", 2000);   // Alice 刷新了更高分
    lb.updateScore("Dave", 2500);
    lb.showTop(3);

    return 0;
}
```

---

### 🎮 实战4：unordered_map 物品 ID 查询（O(1)）

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

struct ItemInfo {
    string name;
    string type;     // "weapon" / "armor" / "consumable"
    int value;       // 金币价值
};

class ItemDatabase {
private:
    unordered_map<int, ItemInfo> db;  // ID → 物品信息

public:
    void registerItem(int id, const ItemInfo& info) {
        db[id] = info;
    }

    // O(1) 查询
    const ItemInfo* query(int id) const {
        auto it = db.find(id);
        if (it != db.end()) return &it->second;
        return nullptr;
    }

    void printInfo(int id) const {
        auto* info = query(id);
        if (info) {
            cout << "[ID:" << id << "] " << info->name
                 << " (" << info->type << ") 价值:" << info->value << "金\n";
        } else {
            cout << "[警告] 未知物品 ID: " << id << "\n";
        }
    }
};

int main() {
    ItemDatabase idb;

    // 注册物品（通常从配置文件加载）
    idb.registerItem(1001, {"铁剑",   "weapon",    150});
    idb.registerItem(1002, {"皮甲",   "armor",     200});
    idb.registerItem(2001, {"回血药", "consumable", 50});
    idb.registerItem(2002, {"传送卷", "consumable", 300});

    // 游戏中玩家拾取时，快速查询
    int droppedItems[] = {1001, 2001, 9999, 2002};
    for (int id : droppedItems) {
        idb.printInfo(id);
    }

    return 0;
}
```

---

## 11.12 动手练习

> 📝 完成以下练习，巩固 STL 容器技能

---

**⭐ 练习1（基础）**  
创建一个 `vector<int>` 存储 10 个玩家的生命值，遍历并打印出所有 HP > 50 的玩家索引和值。

---

**⭐⭐ 练习2（初级）**  
用 `stack<string>` 实现一个简单的"游戏操作撤销"功能：
- 玩家执行操作时 push 到栈
- 按 Ctrl+Z（模拟）时 pop 并打印撤销的操作

---

**⭐⭐ 练习3（初级）**  
用 `map<string, vector<string>>` 表示怪物掉落表：
- key = 怪物名
- value = 可能掉落的物品列表（允许多个）
- 实现"查询某怪物所有掉落物"的函数

---

**⭐⭐⭐ 练习4（中级）**  
实现一个技能冷却系统：
- 用 `unordered_map<string, int>` 存储技能名 → 剩余冷却时间
- 每帧调用 `tick()` 将所有冷却值减1
- 提供 `canUse(string skill)` 方法判断技能是否就绪

---

**⭐⭐⭐⭐ 练习5（挑战）**  
结合 `priority_queue`（优先队列，STL 扩展）实现一个**敌人AI行动队列**：
- 每个敌人有一个行动速度属性
- 速度越高越先行动
- 用 `priority_queue` 保证每次取出速度最快的敌人

---

## 本章小结

| 容器 | 底层结构 | 时间复杂度 | 特点 |
|------|---------|------------|------|
| `vector` | 连续数组 | O(1)随机访问 | 最常用，尾部高效 |
| `list` | 双向链表 | O(1)任意插删 | 不支持随机访问 |
| `deque` | 分段数组 | O(1)两端操作 | 兼顾两端和随机访问 |
| `stack` | deque适配 | O(1) | LIFO，适合撤销 |
| `queue` | deque适配 | O(1) | FIFO，适合调度 |
| `set` | 红黑树 | O(log n) | 有序唯一 |
| `map` | 红黑树 | O(log n) | 有序键值对 |
| `unordered_map` | 哈希表 | O(1)均摊 | 无序，高速查询 |

**选容器的黄金法则：**
1. 需要随机访问 → `vector`
2. 频繁中间插删 → `list`
3. 需要有序+唯一 → `set/map`
4. 需要极速查找 → `unordered_map`
5. LIFO → `stack`，FIFO → `queue`

---

## 下章预告

**Chapter 12 — STL 算法**  
光有"容器"还不够，真正让 STL 发光的是内置的**80多个算法**！下章我们将学习：
- `sort`、`find`、`count` 等常用算法
- Lambda 表达式如何与 STL 无缝结合
- 游戏中用算法处理敌人列表、道具属性的实战技巧

敬请期待！🚀

---

> 📚 **参考资料**  
> - 《C++ Primer（第5版）》第9–11章：顺序容器、关联容器  
> - 《Effective STL》Scott Meyers — 50条高效使用 STL 的建议  
> - cppreference.com — STL 容器在线文档
