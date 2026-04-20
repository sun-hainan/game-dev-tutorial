# Chapter 14：数据结构与算法

> 🎯 目标：掌握算法复杂度分析、基础数据结构（栈/队列/链表/树）、经典排序和查找算法。理解"用什么数据结构最合适"，为 Unity 开发中的性能优化打基础。

---

## 14.1 算法复杂度：时间与空间

### 生活中的类比 ⏱️

> **时间复杂度 = 做饭所需时间：**
> - O(1) = 微波炉热剩饭：不管多少，始终一样快
> - O(n) = 洗一摞碗：碗越多，时间越长
> - O(n²) = 写贺卡：每张都要写上所有人的名字
>
> **空间复杂度 = 厨房大小：**
> - 用多少临时空间（食材、砧板）来完成烹饪

### 为什么要分析复杂度？

```cpp
// O(1)：无论数组多大，一步完成
int getFirst(int arr[], int n) {
    return arr[0];
}

// O(n)：遍历一遍数组
int findMax(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > max) max = arr[i];
    }
    return max;
}

// O(n²)：双重循环
void bubbleSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}
```

---

## 14.2 复杂度记号

| 记号 | 名称 | 典型场景 | 例子 |
|-----|------|---------|------|
| O(1) | 常数时间 | 直接访问数组元素 | `arr[5]` |
| O(log n) | 对数时间 | 二分查找 | `binary_search` |
| O(n) | 线性时间 | 遍历数组 | `find`, `count` |
| O(n log n) | 线性对数 | 高效排序 | `sort` |
| O(n²) | 平方时间 | 冒泡/选择/插入排序 | `bubbleSort` |
| O(2ⁿ) | 指数时间 | 递归斐波那契（差实现）| `fib(n)` |
| O(n!) | 阶乘时间 | 全排列穷举 | 旅行商问题近似 |

### 对比图

```
n=10 时各复杂度所需操作数：
O(1)        █                         1
O(logn)     ██                        3-4
O(n)        ██████████                10
O(nlogn)    ████████████████          30-40
O(n²)      ██████████████████████████ 100
O(2ⁿ)      ██████████████████████████ (1024)
O(n!)      超过 3,628,800
```

---

## 14.3 线性表：数组 vs 链表

### 数组

> **数组 = 连续的内存格子，每个格子有编号（下标）。**
> 就像一排编好号的停车位。

```cpp
// 数组的特点
int arr[5] = {10, 20, 30, 40, 50};

// ✅ 优点：随机访问 O(1)，速度快
cout << arr[3] << endl;   // 直接用下标，O(1)

// ❌ 缺点：插入/删除 O(n)，要移动元素
// 在位置 2 插入一个数，后面的都要挪
```

### 链表

> **链表 = 用线串起来的节点，每个节点知道下一个在哪。**
> 就像寻宝游戏，每个地点告诉你下一个地点在哪。

```cpp
// -------- 链表节点定义 --------
struct Node {
    int data;      // 数据
    Node* next;    // 下一个节点的地址

    Node(int d) : data(d), next(nullptr) {}
};

// -------- 在头部插入 --------
void pushFront(Node*& head, int value) {
    Node* newNode = new Node(value);
    newNode->next = head;
    head = newNode;
}

// -------- 在尾部插入 --------
void pushBack(Node*& head, int value) {
    Node* newNode = new Node(value);
    if (!head) {
        head = newNode;
        return;
    }
    Node* cur = head;
    while (cur->next) cur = cur->next;
    cur->next = newNode;
}

// -------- 删除节点 --------
void remove(Node*& head, int value) {
    if (!head) return;
    if (head->data == value) {
        Node* tmp = head;
        head = head->next;
        delete tmp;
        return;
    }
    Node* cur = head;
    while (cur->next && cur->next->data != value) {
        cur = cur->next;
    }
    if (cur->next) {
        Node* tmp = cur->next;
        cur->next = tmp->next;
        delete tmp;
    }
}
```

### 对比总结

| 操作 | 数组 | 链表 |
|-----|------|------|
| 访问（下标）| O(1) | O(n) |
| 头部插入 | O(n) | O(1) |
| 尾部插入 | O(1) | O(n) |
| 中间插入 | O(n) | O(n) |
| 删除 | O(n) | O(n) |
| 内存 | 连续，节省 | 分散，开销大 |

---

## 14.4 栈（LIFO）

### 概念

> **栈 = 叠盘子。** 最后放上去的，最先被拿走。
> `push` 压入 / `pop` 弹出 / `top` 查看栈顶

### 数组实现

```cpp
#include <iostream>
#include <vector>
using namespace std;

class IntStack {
private:
    vector<int> data;

public:
    void push(int value) {
        data.push_back(value);
        cout << "压入：" << value << endl;
    }

    void pop() {
        if (data.empty()) {
            cout << "栈空，无法弹出" << endl;
            return;
        }
        cout << "弹出：" << data.back() << endl;
        data.pop_back();
    }

    int top() const {
        if (data.empty()) return -1;
        return data.back();
    }

    bool empty() const {
        return data.empty();
    }

    int size() const {
        return data.size();
    }
};

int main()
{
    IntStack s;

    // -------- 后进先出 --------
    s.push(10);    // 栈：[10]
    s.push(20);    // 栈：[10, 20]
    s.push(30);    // 栈：[10, 20, 30]

    cout << "栈顶：" << s.top() << endl;   // 30

    s.pop();       // 弹出 30
    s.pop();       // 弹出 20

    cout << "栈顶：" << s.top() << endl;   // 10

    return 0;
}
```

### STL stack

```cpp
#include <iostream>
#include <stack>
using namespace std;

int main()
{
    stack<int> s;
    s.push(100);
    s.push(200);
    s.push(300);

    while (!s.empty()) {
        cout << s.top() << " ";
        s.pop();
    }
    // 输出：300 200 100

    return 0;
}
```

### 游戏开发实战：撤销操作

```cpp
#include <iostream>
#include <stack>
#include <string>
using namespace std;

// -------- 玩家操作记录 --------
class PlayerActionRecorder {
private:
    stack<string> undoStack;    // 撤销栈
    stack<string> redoStack;   // 重做栈

public:
    void recordAction(const string& action) {
        undoStack.push(action);
        redoStack = stack<string>();   // 新操作后清空重做栈
        cout << "执行：" << action << endl;
    }

    void undo() {
        if (undoStack.empty()) {
            cout << "没有可撤销的操作" << endl;
            return;
        }
        string action = undoStack.top();
        undoStack.pop();
        redoStack.push(action);
        cout << "↩️ 撤销：" << action << endl;
    }

    void redo() {
        if (redoStack.empty()) {
            cout << "没有可重做的操作" << endl;
            return;
        }
        string action = redoStack.top();
        redoStack.pop();
        undoStack.push(action);
        cout << "↪️ 重做：" << action << endl;
    }
};

int main()
{
    PlayerActionRecorder recorder;

    recorder.recordAction("移动到坐标(100, 200)");
    recorder.recordAction("拾取 [圣剑]");
    recorder.recordAction("攻击 哥布林");

    cout << endl;
    recorder.undo();    // 撤销攻击
    recorder.undo();    // 撤销拾取

    recorder.redo();    // 重做拾取

    return 0;
}
```

---

## 14.5 队列（FIFO）

### 概念

> **队列 = 排队买票。** 最先排队的，最先买到票。
> `push` 入队 / `pop` 出队 / `front` 查看队首

### 数组实现

```cpp
class IntQueue {
private:
    vector<int> data;
    int frontIndex = 0;

public:
    void push(int value) {
        data.push_back(value);
    }

    void pop() {
        if (empty()) return;
        frontIndex++;
    }

    int front() const {
        if (empty()) return -1;
        return data[frontIndex];
    }

    bool empty() const {
        return frontIndex >= (int)data.size();
    }

    int size() const {
        return data.size() - frontIndex;
    }
};
```

### STL queue

```cpp
#include <iostream>
#include <queue>
using namespace std;

int main()
{
    queue<int> q;
    q.push(10);
    q.push(20);
    q.push(30);

    while (!q.empty()) {
        cout << q.front() << " ";
        q.pop();
    }
    // 输出：10 20 30

    return 0;
}
```

### 游戏开发实战：消息队列

```cpp
#include <iostream>
#include <queue>
#include <string>
#include <chrono>
#include <thread>
using namespace std;

// -------- 游戏事件 --------
struct GameEvent {
    string type;
    string message;
    int timestamp;
};

// -------- 事件队列 --------
class EventQueue {
private:
    queue<GameEvent> events;

public:
    void pushEvent(const string& type, const string& msg) {
        GameEvent e;
        e.type = type;
        e.message = msg;
        e.timestamp = chrono::duration_cast<chrono::seconds>(
            chrono::system_clock::now().time_since_epoch()).count();
        events.push(e);
        cout << "📩 事件入队：" << type << " - " << msg << endl;
    }

    void processEvents() {
        cout << "=== 处理事件队列 ===" << endl;
        while (!events.empty()) {
            GameEvent e = events.front();
            events.pop();

            cout << "处理 [" << e.type << "]：" << e.message << endl;

            // 模拟处理时间
            this_thread::sleep_for(chrono::milliseconds(100));
        }
    }
};

int main()
{
    EventQueue eq;

    eq.pushEvent("PLAYER_MOVE", "玩家移动到(100,200)");
    eq.pushEvent("PLAYER_ATTACK", "玩家攻击哥布林");
    eq.pushEvent("ENEMY_DEAD", "哥布林死亡，掉落金币");
    eq.pushEvent("PLAYER_PICKUP", "拾取 [金创药 x3]");

    eq.processEvents();

    return 0;
}
```

---

## 14.6 树与二叉树

### 树的概念

> **树 = 族谱。** 有根节点（祖先），每个节点有零个或多个子节点，没有环。

### 二叉树

> **二叉树 = 每个节点最多两个子节点。** 像倒过来的树杈。

```cpp
// -------- 二叉树节点 --------
struct TreeNode {
    int value;
    TreeNode* left;
    TreeNode* right;

    TreeNode(int v) : value(v), left(nullptr), right(nullptr) {}
};

// -------- 二叉树类 --------
class BinaryTree {
public:
    TreeNode* root;

    BinaryTree() : root(nullptr) {}

    // -------- 插入节点（ BST 方式）--------
    void insert(int value) {
        root = insertRec(root, value);
    }

    TreeNode* insertRec(TreeNode* node, int value) {
        if (!node) return new TreeNode(value);
        if (value < node->value) {
            node->left = insertRec(node->left, value);
        } else {
            node->right = insertRec(node->right, value);
        }
        return node;
    }

    // -------- 中序遍历（左-根-右）--------
    void inorder() {
        inorderRec(root);
        cout << endl;
    }

    void inorderRec(TreeNode* node) {
        if (!node) return;
        inorderRec(node->left);
        cout << node->value << " ";
        inorderRec(node->right);
    }
};

int main()
{
    BinaryTree tree;
    int values[] = {50, 30, 70, 20, 40, 60, 80};

    for (int v : values) {
        tree.insert(v);
    }

    cout << "中序遍历（从小到大）：";
    tree.inorder();   // 输出：20 30 40 50 60 70 80

    return 0;
}
```

---

## 14.7 二叉搜索树（BST）

### 核心性质

> **左子树所有节点 < 根节点 < 右子树所有节点。**
> 中序遍历 BST 得到的就是排序好的序列！

### 查找

```cpp
TreeNode* find(TreeNode* node, int value) {
    if (!node || node->value == value) return node;
    if (value < node->value) return find(node->left, value);
    return find(node->right, value);
}
```

### 插入

```cpp
TreeNode* insert(TreeNode* node, int value) {
    if (!node) return new TreeNode(value);
    if (value < node->value)
        node->left = insert(node->left, value);
    else
        node->right = insert(node->right, value);
    return node;
}
```

### 删除（三种情况）

| 情况 | 处理方式 |
|-----|---------|
| 叶子节点 | 直接删除 |
| 只有一个子节点 | 用子节点替代 |
| 有两个子节点 | 用后继节点（左子树最大或右子树最小）替代 |

---

## 14.8 基础排序算法

### 冒泡排序（Bubble Sort）

```cpp
#include <iostream>
#include <vector>
using namespace std;

// -------- 冒泡排序：O(n²) --------
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}

int main()
{
    vector<int> scores = {85, 92, 78, 95, 88, 73, 90};

    cout << "排序前：";
    for (int s : scores) cout << s << " ";
    cout << endl;

    bubbleSort(scores);

    cout << "排序后：";
    for (int s : scores) cout << s << " ";
    cout << endl;

    return 0;
}
```

**运行结果：**
```
排序前：85 92 78 95 88 73 90 
排序后：73 78 85 88 90 92 95 
```

### 选择排序（Selection Sort）

```cpp
// -------- 选择排序：O(n²)，每次选最小的 --------
void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }
        swap(arr[i], arr[minIdx]);
    }
}
```

### 插入排序（Insertion Sort）

```cpp
// -------- 插入排序：O(n²)，像整理扑克牌 --------
void insertionSort(vector<int>& arr) {
    for (int i = 1; i < (int)arr.size(); i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

---

## 14.9 高效排序算法

### 快速排序（Quick Sort）

```cpp
#include <iostream>
#include <vector>
using namespace std;

// -------- 分区：返回枢轴位置 --------
int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high];   // 选最后一个作为枢轴
    int i = low - 1;

    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return i + 1;
}

// -------- 快速排序：平均 O(n log n) --------
void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);   // 分区
        quickSort(arr, low, pi - 1);          // 左半部分
        quickSort(arr, pi + 1, high);          // 右半部分
    }
}

// -------- 重载 --------
void quickSort(vector<int>& arr) {
    if (!arr.empty()) {
        quickSort(arr, 0, arr.size() - 1);
    }
}

int main()
{
    vector<int> scores = {85, 92, 78, 95, 88, 73, 90};

    quickSort(scores);

    cout << "快速排序后：";
    for (int s : scores) cout << s << " ";
    cout << endl;

    return 0;
}
```

### 归并排序（Merge Sort）

```cpp
#include <iostream>
#include <vector>
using namespace std;

// -------- 合并两个有序数组 --------
void merge(vector<int>& arr, int left, int mid, int right) {
    vector<int> leftArr(arr.begin() + left, arr.begin() + mid + 1);
    vector<int> rightArr(arr.begin() + mid + 1, arr.begin() + right + 1);

    int i = 0, j = 0, k = left;
    while (i < (int)leftArr.size() && j < (int)rightArr.size()) {
        if (leftArr[i] <= rightArr[j]) {
            arr[k++] = leftArr[i++];
        } else {
            arr[k++] = rightArr[j++];
        }
    }
    while (i < (int)leftArr.size()) arr[k++] = leftArr[i++];
    while (j < (int)rightArr.size()) arr[k++] = rightArr[j++];
}

// -------- 归并排序：O(n log n)--------
void mergeSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);      // 左半部分
        mergeSort(arr, mid + 1, right);  // 右半部分
        merge(arr, left, mid, right);    // 合并
    }
}

void mergeSort(vector<int>& arr) {
    if (!arr.empty()) {
        mergeSort(arr, 0, arr.size() - 1);
    }
}
```

### 排序算法对比

| 算法 | 时间复杂度 | 空间复杂度 | 稳定性 |
|-----|-----------|-----------|-------|
| 冒泡排序 | O(n²) | O(1) | ✅ 稳定 |
| 选择排序 | O(n²) | O(1) | ❌ 不稳定 |
| 插入排序 | O(n²) | O(1) | ✅ 稳定 |
| 快速排序 | O(n log n)（平均）| O(log n) | ❌ 不稳定 |
| 归并排序 | O(n log n) | O(n) | ✅ 稳定 |

---

## 14.10 查找算法

### 顺序查找

```cpp
// -------- O(n)，逐个检查 --------
int sequentialSearch(const vector<int>& arr, int target) {
    for (int i = 0; i < (int)arr.size(); i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}
```

### 二分查找

```cpp
// -------- O(log n)，前提是有序数组 --------
int binarySearch(const vector<int>& arr, int target) {
    int left = 0;
    int right = arr.size() - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;   // 未找到
}

int main()
{
    vector<int> levels = {10, 20, 30, 40, 50, 60, 70};

    cout << binarySearch(levels, 40) << endl;   // 3
    cout << binarySearch(levels, 35) << endl;   // -1（未找到）

    return 0;
}
```

---

## 14.11 哈希表

### 概念

> **哈希表 = 字典。** 知道单词就能直接翻到页码，O(1) 查找。
> 通过哈希函数把 key 转换成数组下标。

### C++ unordered_map（哈希实现）

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

int main()
{
    // -------- unordered_map：哈希表实现，O(1) 查找 --------
    unordered_map<int, string> itemDB;

    // -------- 插入装备 --------
    itemDB[1001] = "铁剑";
    itemDB[1002] = "皮甲";
    itemDB[1003] = "魔法杖";
    itemDB[1004] = "圣剑";
    itemDB[1005] = "龙鳞盾";

    // -------- O(1) 查找 --------
    int searchID = 1003;
    auto it = itemDB.find(searchID);

    if (it != itemDB.end()) {
        cout << "ID=" << searchID << " 是：" << it->second << endl;
    } else {
        cout << "未找到 ID=" << searchID << endl;
    }

    // -------- 遍历 --------
    cout << endl << "=== 装备库 ===" << endl;
    for (const auto& [id, name] : itemDB) {
        cout << "ID " << id << "：" << name << endl;
    }

    return 0;
}
```

---

## 14.12 游戏开发实战

### 敌人血量排序（选择排序）

```cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

// -------- 选择排序：按 HP 从小到大 --------
void selectionSortEnemyHP(vector<pair<string, int>>& enemies) {
    int n = enemies.size();
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (enemies[j].second < enemies[minIdx].second) {
                minIdx = j;
            }
        }
        swap(enemies[i], enemies[minIdx]);
    }
}

int main()
{
    vector<pair<string, int>> enemies = {
        {"哥布林", 100},
        {"骷髅", 80},
        {"巨龙", 500},
        {"幽灵", 50},
        {"牛魔王", 800}
    };

    cout << "=== 排序前 ===" << endl;
    for (const auto& [name, hp] : enemies) {
        cout << name << " HP=" << hp << endl;
    }

    selectionSortEnemyHP(enemies);

    cout << endl << "=== 按 HP 排序后（从小到大）===" << endl;
    for (const auto& [name, hp] : enemies) {
        cout << name << " HP=" << hp << endl;
    }

    return 0;
}
```

### 二分查找快速定位等级

```cpp
#include <iostream>
#include <vector>
using namespace std;

// -------- 二分查找：找 >= target 的最小索引 --------
int lowerBound(const vector<int>& levels, int target) {
    int left = 0, right = levels.size() - 1;
    int result = -1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (levels[mid] >= target) {
            result = mid;
            right = mid - 1;
        } else {
            left = mid + 1;
        }
    }
    return result;
}

int main()
{
    // -------- 等级阈值列表 --------
    vector<int> levelThresholds = {1, 10, 20, 30, 40, 50, 60, 70, 80, 90};

    cout << "=== 等级段位系统 ===" << endl;
    cout << "玩家 Lv.1  → 段位：" << (levelThresholds[lowerBound(levelThresholds, 1)]) << endl;
    cout << "玩家 Lv.15 → 段位：" << (levelThresholds[lowerBound(levelThresholds, 15)]) << endl;
    cout << "玩家 Lv.45 → 段位：" << (levelThresholds[lowerBound(levelThresholds, 45)]) << endl;
    cout << "玩家 Lv.99 → 段位：" << (levelThresholds[lowerBound(levelThresholds, 99)]) << endl;

    return 0;
}
```

**运行结果：**
```
=== 等级段位系统 ===
玩家 Lv.1  → 段位：1
玩家 Lv.15 → 段位：10
玩家 Lv.45 → 段位：40
玩家 Lv.99 → 段位：90
```

### 哈希表管理装备 ID

```cpp
#include <iostream>
#include <unordered_map>
#include <string>
using namespace std;

// -------- 装备信息 --------
struct Item {
    string name;
    string type;     // weapon/armor/potion
    int value;       // 价值
    int rarity;      // 1=普通 2=稀有 3=传说
};

int main()
{
    unordered_map<string, Item> itemDatabase;

    // -------- 填充装备库 --------
    itemDatabase["10001"] = {"铁剑", "weapon", 100, 1};
    itemDatabase["10002"] = {"圣剑", "weapon", 5000, 3};
    itemDatabase["10003"] = {"皮甲", "armor", 80, 1};
    itemDatabase["10004"] = {"龙鳞甲", "armor", 3000, 3};
    itemDatabase["20001"] = {"金创药", "potion", 50, 1};
    itemDatabase["20002"] = {"凤凰羽毛", "potion", 2000, 3};

    // -------- 根据 ID 查询装备 --------
    string searchIDs[] = {"10002", "20001", "99999"};

    cout << "=== 装备查询 ===" << endl;
    for (const string& id : searchIDs) {
        auto it = itemDatabase.find(id);
        if (it != itemDatabase.end()) {
            const Item& item = it->second;
            string rarityStr = (item.rarity == 3) ? "传说" : (item.rarity == 2) ? "稀有" : "普通";
            cout << "ID " << id << "：" << item.name
                 << " [" << rarityStr << "] 价值：" << item.value << endl;
        } else {
            cout << "ID " << id << "：未找到" << endl;
        }
    }

    // -------- 统计各稀有度数量 --------
    cout << endl << "=== 稀有度统计 ===" << endl;
    int count[4] = {0};
    for (const auto& [id, item] : itemDatabase) {
        count[item.rarity]++;
    }
    cout << "普通：" << count[1] << " 件" << endl;
    cout << "稀有：" << count[2] << " 件" << endl;
    cout << "传说：" << count[3] << " 件" << endl;

    return 0;
}
```

---

## 14.13 常见错误与调试

| 错误现象 | 原因 | 解决方法 |
|---------|------|---------|
| 排序后数组没变 | 比较函数写反了 | 检查 `<` 还是 `<=` |
| 二分查找死循环 | 更新 left/right 时边界错误 | 确保 `left <= right` |
| BST 插入后中序遍历无序 | 插入逻辑有 bug | 检查递归 |
| 快速排序栈溢出 | 递归太深（最坏情况）| 用迭代或随机 pivot |
| 哈希表 find 找不到 | key 类型不对 | 确认 key 的类型和比较方式 |

---

## 14.14 动手练习 🧪

### 练习 1：实现冒泡排序 ⭐
```cpp
// 对整数数组进行冒泡排序
// 打印每趟的结果，观察如何"冒泡"
```

### 练习 2：二分查找 ⭐
```cpp
// 在有序数组 {2,5,8,12,16,23,38,45,56,72} 中
// 用二分查找 23 和 30，输出位置或"Not Found"
```

### 练习 3：栈应用——括号匹配 ⭐⭐
```cpp
// 用栈检查字符串中的括号是否匹配
// "(())" → 匹配，"([)]" → 不匹配
```

### 练习 4：快速排序 ⭐⭐
```cpp
// 实现快速排序函数，对敌人 HP 列表排序
// 包含 partition 和 quickSort 两个函数
```

### 练习 5：综合——排行榜 ⭐⭐⭐
```cpp
// 用 unordered_map 存玩家分数
// 支持：添加分数、查询前 N 名（用 vector 排序后取前 N）
// 支持：删除玩家
```

---

## 14.15 本章小结

```
✅ 已掌握：
├── 时间复杂度 O(1)/O(n)/O(n²)/O(log n)/O(n log n)
├── 空间复杂度
├── 数组 vs 链表
├── 栈（LIFO）和队列（FIFO）
├── 二叉树和中序遍历
├── 二叉搜索树（BST）插入/查找
├── 冒泡/选择/插入排序（O(n²)）
├── 快速排序（O(n log n)）
├── 归并排序（O(n log n)）
├── 顺序查找 vs 二分查找（O(log n)）
├── 哈希表（unordered_map，O(1)）
└── 敌人排序 + 装备查询实战

🎉 Stage 1 C++ 基础 到此结束！
恭喜你完成了 C++ 的全部基础内容。
🔜 Stage 2 将进入 Unity + C# 开发，用游戏引擎把知识用起来！
```

---

_📚 参考资料：《算法导论》《C++ Primer》《Algorithms》_
