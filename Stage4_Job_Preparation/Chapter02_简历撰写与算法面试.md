# Chapter 02：简历撰写与算法面试

> 🎯 目标：写一份打动招聘者的简历，掌握游戏公司算法面试高频考点。

---

## 2.1 游戏开发简历模板

### 完整简历示例

```markdown
# [姓名]
🎮 游戏开发工程师 | C++ / Unity C#

📱 微信：xxxx | 📧 邮箱：xxx@example.com | 🐱 GitHub：github.com/username

---

## 技术栈

| 分类 | 技术 |
|-----|------|
| 游戏引擎 | Unity (C#)、Unreal (C++)、Cocos2d-x |
| 编程语言 | C++、C#、Lua、Python、JavaScript |
| 图形学 | OpenGL、DirectX、Shader (HLSL/GLSL) |
| 工具 | Git、Perforce、JIRA、Confluence |
| 其他 | 数据结构与算法、计算机网络、操作系统 |

---

## 项目经历

### 🎮 [项目名称] — 2D 平台跳跃游戏
**GitHub**: https://github.com/username/platformer  
**开发时间**: 2025.01 - 2025.03（个人项目）

**项目描述**:  
一款使用 Unity 开发的 2D 平台跳跃游戏，包含完整的玩家控制系统、敌人 AI、道具系统和关卡编辑器。

**技术实现**:
- 实现了基于物理的 2D 玩家控制器，支持跳跃、蹬墙跳等操作
- 使用对象池管理子弹和敌人，GC 降低 80%，帧率稳定 60FPS
- 设计并实现了 A* 寻路算法，支持动态障碍物更新
- 开发了可视化关卡编辑器，支持拖拽式关卡设计

**技术栈**: Unity 2022.3 / C# / UGUI / 对象池 / A* 寻路

---

### 🎮 [项目名称] — 3D 动作游戏
**GitHub**: https://github.com/username/3d-action  
**开发时间**: 2024.09 - 2024.12（团队项目，担任主程序）

**项目描述**:  
3D 俯视角动作游戏，支持近战和远程武器、敌人 AI、道具系统和存档。

**技术实现**:
- 负责玩家控制器和武器系统，支持 WASD 移动 + 鼠标瞄准
- 实现敌人 AI（巡逻/追击/攻击状态机），支持 5 种敌人类型
- 开发了基于 JSON 的存档系统，支持玩家数据持久化
- 优化 Draw Call，合并相同材质物体，从 200+ 降至 30

**技术栈**: Unity 2022.3 / C# / Shader / Physics / JSON

---

### 📚 [项目名称] — C++ 算法学习仓库
**GitHub**: https://github.com/username/algorithms  
**描述**: 系统整理了常用数据结构与算法，包含详细中文注释。

**内容**:
- 排序算法：冒泡/选择/插入/归并/快速/堆排序
- 查找算法：顺序/二分/哈希
- 数据结构：数组/链表/栈/队列/树/图
- 动态规划：斐波那契/背包/LIS/编辑距离

**技术栈**: C++17 / STL / 算法复杂度分析

---

## 工作经历

### [公司名称] — 游戏开发实习生
2024.06 - 2024.08

- 负责活动模块开发，完成了签到系统和排行榜功能
- 优化了游戏启动速度，从 15s 降至 8s
- 修复了 20+线上 bug，协助上线 2 个活动

---

## 教育背景

**[学校名称]** — 计算机科学与技术（本科）2019.09 - 2023.06

- GPA: 3.5/4.0（专业前 20%）
- 奖学金：校级二等奖学金（2021-2022）
- 竞赛：ACM-ICPC 省级铜牌

---

## 自我评价

热爱游戏开发，有良好的编码习惯和代码规范意识。熟悉游戏开发流程，具备独立解决问题的能力。乐于分享，在 CSDN 发布技术博客 20+ 篇。

---

## 📌 简历关键词速查表

| 类别 | 高频关键词 |
|-----|-----------|
| 引擎 | Unity / Unreal / Cocos / Godot |
| 语言 | C++ / C# / Lua / GDScript |
| 图形学 | Shader / HLSL / GLSL / PBR / 光照模型 |
| 物理 | Rigidbody / 碰撞检测 / 物理优化 |
| AI | 状态机 / 行为树 / A* / NavMesh |
| 网络 | TCP/UDP / KCP / 帧同步 / 状态同步 |
| 工具 | Git / Perforce / JIRA / Confluence |
| 优化 | DrawCall / Batching / LOD / GC 优化 |
```

---

## 2.2 STAR 法则写项目

### STAR 法则

| 维度 | 内容 | 示例 |
|-----|------|------|
| **S**ituation | 背景/场景 | "游戏需要支持多人联机" |
| **T**ask | 任务/挑战 | "我负责实现网络同步系统" |
| **A**ction | 行动/做法 | "采用帧同步方案，优化了带宽占用" |
| **R**esult | 结果/成果 | "延迟从 200ms 降至 50ms，支持 10 人同屏" |

### 示例改写

```markdown
# ❌ 改写前（太笼统）
"实现了玩家移动同步功能"

# ✅ 改写后（STAR 法则）
"针对 10 人同屏卡顿问题（Situation），
负责网络同步模块开发（Task），
采用状态同步 + 增量压缩方案（Action），
最终延迟降低 60%，同屏流畅运行（Result）"
```

---

## 2.3 算法面试高频考点

### 考察比例

| 类别 | 占比 | 难度 |
|-----|------|------|
| 数组/链表 | 30% | ⭐~⭐⭐ |
| 栈/队列 | 15% | ⭐⭐ |
| 二叉树/图 | 20% | ⭐⭐~⭐⭐⭐ |
| 动态规划 | 20% | ⭐⭐⭐ |
| 排序/查找 | 10% | ⭐~⭐⭐ |
| 其他 | 5% | ⭐⭐⭐⭐ |

---

### 高频真题 1：两数之和

```csharp
// 题目：给定数组和目标值，找出和为目标值的两个数的索引
// LeetCode 1

// 方法1：暴力 O(n²)
int[] TwoSum_Brute(int[] nums, int target)
{
    for (int i = 0; i < nums.Length; i++)
        for (int j = i + 1; j < nums.Length; j++)
            if (nums[i] + nums[j] == target)
                return new[] { i, j };
    return Array.Empty<int>();
}

// 方法2：哈希表 O(n) ✅ 推荐
int[] TwoSum_Hash(int[] nums, int target)
{
    Dictionary<int, int> dict = new Dictionary<int, int>();

    for (int i = 0; i < nums.Length; i++)
    {
        int complement = target - nums[i];
        if (dict.ContainsKey(complement))
            return new[] { dict[complement], i };

        dict[nums[i]] = i;
    }

    return Array.Empty<int>();
}
```

### 高频真题 2：有效的括号

```csharp
// 题目：判断括号是否匹配
// LeetCode 20

bool IsValid(string s)
{
    Stack<char> stack = new Stack<char>();

    foreach (char c in s)
    {
        if (c == '(' || c == '[' || c == '{')
        {
            stack.Push(c);
        }
        else
        {
            if (stack.Count == 0) return false;

            char top = stack.Pop();
            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{'))
            {
                return false;
            }
        }
    }

    return stack.Count == 0;
}
```

### 高频真题 3：合并两个有序链表

```csharp
// LeetCode 21

public class ListNode
{
    public int val;
    public ListNode next;
}

ListNode MergeTwoLists(ListNode l1, ListNode l2)
{
    ListNode dummy = new ListNode(0);
    ListNode cur = dummy;

    while (l1 != null && l2 != null)
    {
        if (l1.val <= l2.val)
        {
            cur.next = l1;
            l1 = l1.next;
        }
        else
        {
            cur.next = l2;
            l2 = l2.next;
        }
        cur = cur.next;
    }

    cur.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

### 高频真题 4：爬楼梯（DP）

```csharp
// LeetCode 70：每次爬 1 或 2 级，求到第 N 级的方法数

// 方法1：DP O(n) ✅ 推荐
int ClimbStairs(int n)
{
    if (n <= 2) return n;

    int[] dp = new int[n + 1];
    dp[1] = 1;
    dp[2] = 2;

    for (int i = 3; i <= n; i++)
    {
        dp[i] = dp[i - 1] + dp[i - 2];   // 第 i 级 = 第 i-1 级跳上来 + 第 i-2 级跳上来
    }

    return dp[n];
}

// 方法2：空间优化 O(1)
int ClimbStairs_Optimized(int n)
{
    if (n <= 2) return n;

    int prev2 = 1, prev1 = 2, curr = 2;

    for (int i = 3; i <= n; i++)
    {
        curr = prev1 + prev2;   // 当前 = 前1步 + 前2步
        prev2 = prev1;
        prev1 = curr;
    }

    return curr;
}
```

### 高频真题 5：BFS/DFS 岛屿数量

```csharp
// LeetCode 200：统计岛屿数量（'1'=陆地，'0'=水）

int NumIslands(char[][] grid)
{
    if (grid == null || grid.Length == 0) return 0;

    int count = 0;
    int m = grid.Length, n = grid[0].Length;

    for (int i = 0; i < m; i++)
    {
        for (int j = 0; j < n; j++)
        {
            if (grid[i][j] == '1')
            {
                count++;
                DFS(grid, i, j);   // 把相邻的 1 都变成 0
            }
        }
    }

    return count;
}

void DFS(char[][] grid, int i, int j)
{
    if (i < 0 || i >= grid.Length ||
        j < 0 || j >= grid[0].Length ||
        grid[i][j] == '0')
    {
        return;
    }

    grid[i][j] = '0';   // 标记已访问
    DFS(grid, i + 1, j);
    DFS(grid, i - 1, j);
    DFS(grid, i, j + 1);
    DFS(grid, i, j - 1);
}
```

---

## 2.4 游戏开发专业面试题

### 碰撞检测

```csharp
// AABB（轴对齐包围盒）碰撞检测
bool AABBCollision(Vector3 posA, Vector3 sizeA, Vector3 posB, Vector3 sizeB)
{
    return (Mathf.Abs(posA.x - posB.x) <= (sizeA.x + sizeB.x) / 2f) &&
           (Mathf.Abs(posA.y - posB.y) <= (sizeA.y + sizeB.y) / 2f) &&
           (Mathf.Abs(posA.z - posB.z) <= (sizeA.z + sizeB.z) / 2f);
}

// 点到线段的最近点
Vector3 ClosestPointOnSegment(Vector3 p, Vector3 a, Vector3 b)
{
    Vector3 ab = b - a;
    float t = Vector3.Dot(p - a, ab) / Vector3.Dot(ab, ab);
    t = Mathf.Clamp01(t);
    return a + t * ab;
}
```

### A* 寻路

```csharp
// A* 寻路核心：f(n) = g(n) + h(n)
// g(n) = 从起点到 n 的实际代价
// h(n) = 从 n 到终点的估计代价（启发函数）

// 伪代码
void AStar(Node start, Node goal)
{
    PriorityQueue openSet = new PriorityQueue();  // 按 f 值排序
    Dictionary<Node, float> gScore = new Dictionary<Node, float>();
    Dictionary<Node, Node> cameFrom = new Dictionary<Node, Node>();

    gScore[start] = 0;
    openSet.Enqueue(start, Heuristic(start, goal));

    while (!openSet.IsEmpty())
    {
        Node current = openSet.Dequeue();

        if (current == goal)
            return ReconstructPath(cameFrom, current);

        foreach (Node neighbor in current.neighbors)
        {
            float tentativeG = gScore[current] + Distance(current, neighbor);

            if (tentativeG < gScore.GetValueOrDefault(neighbor, float.MaxValue))
            {
                cameFrom[neighbor] = current;
                gScore[neighbor] = tentativeG;
                float f = tentativeG + Heuristic(neighbor, goal);
                openSet.Enqueue(neighbor, f);
            }
        }
    }

    return null;  // 没找到路径
}
```

### 排序算法复杂度对比

| 算法 | 时间复杂度（平均）| 空间复杂度 | 稳定性 |
|-----|----------------|-----------|-------|
| 冒泡排序 | O(n²) | O(1) | ✅ 稳定 |
| 选择排序 | O(n²) | O(1) | ❌ 不稳定 |
| 插入排序 | O(n²) | O(1) | ✅ 稳定 |
| 快速排序 | O(n log n) | O(log n) | ❌ 不稳定 |
| 归并排序 | O(n log n) | O(n) | ✅ 稳定 |
| 堆排序 | O(n log n) | O(1) | ❌ 不稳定 |

---

## 2.5 面试技巧

### 5 分钟法则

```
遇到没思路的题：
1. 0-2 分钟：分析题目，clarify 边界条件
2. 2-5 分钟：尝试暴力解，先跑通再说
3. 5 分钟后仍无思路：主动沟通
   - "我想不到最优解，能提示一下方向吗？"
   - "从暴力解优化，我想到可以用 XXX"
   
不要硬撑 10 分钟不说话！
```

### 代码先说思路

```
正确顺序：
1. 先说思路（"我想用哈希表，遍历一次..."）
2. 等面试官确认后再写
3. 边写边说（"现在遍历数组..."）
4. 写完解释复杂度（"时间 O(n)，空间 O(n)"）
```

### 测试用例

```
写完代码后，主动检查：
1. 空数组 [] → 返回什么？
2. 单元素 [1] → 返回正确吗？
3. 负数 / 0 → 边界情况
4. 大数组 → 性能考虑
```

---

## 2.6 刷题平台推荐

| 平台 | 特点 | 适合 |
|-----|------|------|
| **LeetCode** | 题库全，有中文站 | 算法基础 + 按公司刷 |
| **LeetCode 中文站** | leetcode.cn，题目有中文 | 国内面试 |
| **牛客网** | 国内公司真题多 | 字节/腾讯/阿里面经 |
| **Codeforces** | 竞赛，难度高 | 算法竞赛/ACM |
| **洛谷** | 中文，有详细题解 | 入门 |

### 按公司标签刷题

```
牛客网 → 标签 → 选择公司 → 刷高频题
例如：字节跳动高频 TOP50
```

---

## 2.7 动手练习 🧪

### 练习 1：写简历 ⭐⭐⭐
```
按模板写一份完整简历
包含：个人信息/技术栈/2-3个项目/工作经历
用 STAR 法则描述项目
```

### 练习 2：两数之和变体 ⭐⭐
```
LeetCode 167：有序数组两数之和（双指针）
LeetCode 15：三数之和（排序 + 双指针）
```

### 练习 3：DP 专项 ⭐⭐⭐
```
LeetCode 70：爬楼梯
LeetCode 198：打家劫舍
LeetCode 322：零钱兑换
LeetCode 300：最长递增子序列
```

### 练习 4：实现 AABB 碰撞 ⭐⭐
```
用 C# 实现 AABB 碰撞检测
包含两个 Box 的碰撞和点在线段上的最近点
```

### 练习 5：模拟面试 ⭐⭐⭐
```
找同学或自己对着镜子
模拟 45 分钟算法面试
练习边说边写
```

---

## 2.8 本章小结

```
✅ 已掌握：
├── 游戏开发简历标准模板
├── STAR 法则描述项目
├── 技术关键词速查表
├── 高频算法题（两数之和/括号/链表/DP/BFS）
├── AABB 碰撞检测算法
├── A* 寻路算法核心
├── 排序算法复杂度
├── 面试技巧（5分钟法则/边说边写）
└── 刷题平台推荐

🔜 下章预告：
第三章：求职渠道与内推 —— 找到目标公司，找到内推人，拿到 offer。
```

---

_📚 参考资料：《LeetCode 中文站》《简历写这个，HR看了贼开心》《游戏程序员面试指南》_
