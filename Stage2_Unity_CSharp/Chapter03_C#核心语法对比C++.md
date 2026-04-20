# Chapter 03：C# 核心语法（对比 C++）

> 🎯 目标：掌握 C# 的基本语法，对比 C++ 快速上手。Unity 使用 C# 作为主要脚本语言。

---

## 3.1 C++ vs C#：快速对比

### 为什么先学 C++ 再学 C#？

> **你已经理解了编程思想。** C++ 学的指针、内存管理、OOP 在 C# 里依然有用，只是语法略有不同。

### 核心差异一览

| 维度 | C++ | C# |
|-----|-----|-----|
| **内存管理** | 手动（new/delete）| 自动垃圾回收（GC）|
| **编译** | 编译成机器码 | 编译成 IL（中间语言），由 CLR 执行 |
| **指针** | 可以用 | 一般不推荐用，有安全托管指针 |
| **多继承** | 支持 | 不支持（用接口代替）|
| **头文件** | 需要 include | 不需要，用 using |
| **命名空间** | :: 作用域 | using namespace |
| **字符串** | `std::string` | `string`（内置类型）|
| **布尔** | `bool` | `bool` |
| **控制台输出** | `cout <<` | `Debug.Log()` 或 `Console.WriteLine()` |

---

## 3.2 第一个 C# 程序

### C++ 对比

```cpp
// C++ 版本
#include <iostream>
#include <string>
using namespace std;

int main()
{
    cout << "Hello World" << endl;
    return 0;
}
```

```csharp
// C# 版本（Unity 脚本）
using UnityEngine;

public class HelloWorld : MonoBehaviour  // Unity 中所有脚本继承 MonoBehaviour
{
    // -------- Unity 启动时自动调用一次 --------
    void Start()
    {
        Debug.Log("Hello World!");   // Unity 的输出方式
    }

    // -------- 每帧调用一次 --------
    void Update()
    {
        // 适合放持续执行的逻辑
    }
}
```

### C# 代码写在 Unity 里的步骤

```
1. Project 窗口 → Scripts 文件夹 → 右键 → Create → C# Script
2. 命名为 HelloWorld.cs
3. 双击打开（会用 VS2022/VSCode 打开）
4. 编写代码
5. 保存（Ctrl+S）
6. 把脚本拖到场景中的 GameObject 上
7. 点击播放运行
```

---

## 3.3 数据类型

### 基本类型对比

```csharp
// C# 内置类型（C++ 对比）

// 整数
int a = 10;           // C++: int a = 10;
long b = 100L;        // C++: long long b = 100;
short c = 5;          // C++: short c = 5;
byte d = 255;         // C++: unsigned char d = 255;（无符号字节）

// 浮点
float f = 3.14f;      // C++: float f = 3.14f;（必须加 f 后缀）
double g = 3.14;      // C++: double g = 3.14;
decimal h = 3.14m;    // C# 独有：高精度定点数（用于金融计算）

// 字符和字符串
char ch = 'A';        // C++: char ch = 'A';
string name = "孙悟空";  // C++: std::string name = "孙悟空";
                       // C# string 是内置类型，不用 include

// 布尔
bool isAlive = true;  // C++: bool isAlive = true;

// ---------- Unity 常用类型 ----------
Vector3 pos = new Vector3(1, 2, 3);  // Unity 特有：3D 向量
Vector2 v2 = new Vector2(1, 2);       // 2D 向量
Quaternion rot = Quaternion.identity;   // 四元数旋转
Color color = Color.red;               // 颜色
```

### Unity 常用 API

```csharp
// -------- Vector3（最常用）--------
Vector3 pos = new Vector3(0, 5, 0);
Vector3 dir = pos.normalized;         // 单位化
float distance = Vector3.Distance(pos1, pos2);  // 距离
Vector3 lerpPos = Vector3.Lerp(pos1, pos2, 0.5f);  // 插值

// -------- Transform（控制位置/旋转/缩放）--------
transform.position = new Vector3(0, 1, 0);   // 世界坐标位置
transform.localPosition = new Vector3(0, 1, 0);  // 相对父物体位置
transform.rotation = Quaternion.Euler(0, 90, 0);  // 欧拉角旋转
transform.localScale = new Vector3(2, 2, 2);     // 缩放
transform.Translate(Vector3.forward * Time.deltaTime);  // 移动
transform.Rotate(0, 90 * Time.deltaTime, 0);     // 旋转

// -------- Time（时间相关）--------
float deltaTime = Time.deltaTime;   // 上一帧的时间间隔（秒）
float gameTime = Time.time;         // 游戏开始到现在的时间
```

---

## 3.4 变量与常量

### C# 变量声明

```csharp
public class Variables : MonoBehaviour
{
    // -------- Unity 序列化字段（可在 Inspector 中编辑）--------
    [SerializeField] private int speed = 10;    // SerializeField 也能序列化 private
    public float jumpForce = 5.0f;              // public 会在 Inspector 显示
    public string playerName = "Player";         // 字符串

    // -------- 常量（编译时确定，永不改变）--------
    public const int MAX_HEALTH = 100;          // const
    public const float GRAVITY = -9.81f;

    // -------- 只读字段（运行时确定，只能读不能改）--------
    public readonly int instanceID;
    private static readonly float PI = 3.14159f;

    // -------- Unity 特有属性 --------
    [Header("角色属性")]        // Inspector 中显示分组标题
    public int level = 1;
    public int maxHP = 100;
    public int attack = 10;

    [Header("移动设置")]
    public float moveSpeed = 5f;
    public float rotateSpeed = 10f;

    [Tooltip("是否二段跳")]      // 鼠标悬停时的提示
    public bool canDoubleJump = false;

    // -------- Awake：脚本实例创建时调用（比 Start 更早）--------
    void Awake()
    {
        instanceID = GetInstanceID();  // 每个对象有唯一ID
        Debug.Log("Awake called, instanceID=" + instanceID);
    }

    // -------- Start：第一次 Update 前调用 --------
    void Start()
    {
        Debug.Log("Start called, speed=" + speed);
    }
}
```

### Unity 序列化系统

```csharp
// SerializeField：让 private 字段在 Inspector 中可见可编辑
[SerializeField] private int hiddenValue = 100;

// HideInInspector：让 public 字段在 Inspector 中隐藏
[HideInInspector] public int debugValue;

// Range：限制数值在 Inspector 中的范围
[Range(0, 100)] public float health = 50f;

// Tooltip：鼠标悬停显示提示
[Tooltip("角色攻击力，数值越高伤害越大")]
public int attackPower = 10;
```

---

## 3.5 运算符与表达式

### 与 C++ 基本一致

```csharp
// 算术运算符：+, -, *, /, %（与 C++ 一样）
// 自增自减：++, --（与 C++ 一样）
// 比较运算符：==, !=, <, >, <=, >=（与 C++ 一样）
// 逻辑运算符：&&, ||, !（与 C++ 一样）
// 位运算符：&, |, ^, ~, <<, >>（与 C++ 一样）
// 三目运算符：condition ? value1 : value2（与 C++ 一样）

// -------- C# 特有：字符串拼接 --------
string a = "Hello";
string b = "World";
string c = a + " " + b;       // "Hello World"
string d = $"{a} {b}";        // 字符串插值（C# 6.0+，推荐）
string e = string.Format("{0} {1}", a, b);  // Format 方式

// -------- 字符串方法 --------
string s = "Hello World";
bool contains = s.Contains("World");    // true
bool starts = s.StartsWith("Hello");     // true
string upper = s.ToUpper();              // "HELLO WORLD"
string sub = s.Substring(6, 5);          // "World"
string[] parts = s.Split(' ');           // ["Hello", "World"]
```

---

## 3.6 控制流程

### if / else（与 C++ 完全一致）

```csharp
if (hp <= 0)
{
    Debug.Log("角色死亡！");
}
else if (hp < 30)
{
    Debug.Log("危险！HP 很低！");
}
else
{
    Debug.Log("HP 充足");
}
```

### switch（与 C++ 一致，但 C# 7+ 支持更多用法）

```csharp
// -------- 基本 switch --------
switch (itemType)
{
    case ItemType.Weapon:
        Debug.Log("武器");
        break;
    case ItemType.Armor:
        Debug.Log("护甲");
        break;
    case ItemType.Potion:
        Debug.Log("药水");
        break;
    default:
        Debug.Log("未知类型");
        break;
}

// -------- C# 7+ 模式匹配（switch 表达式）--------
string result = itemType switch
{
    ItemType.Weapon => "武器",
    ItemType.Armor => "护甲",
    ItemType.Potion => "药水",
    _ => "未知"
};
```

### for / while / foreach

```csharp
// -------- for（与 C++ 完全一致）--------
for (int i = 0; i < 10; i++)
{
    Debug.Log("第 " + i + " 次");
}

// -------- while（与 C++ 完全一致）--------
while (hp > 0)
{
    hp -= 10;
}

// -------- do...while（与 C++ 完全一致）--------
do
{
    hp -= 10;
} while (hp > 0);

// -------- foreach：C# 特有，用于遍历集合（比 for 更简洁）--------
int[] scores = { 85, 92, 78, 95 };

foreach (int score in scores)
{
    Debug.Log("分数：" + score);
}

// 在 Unity 中遍历子对象
foreach (Transform child in transform)
{
    Debug.Log("子对象：" + child.name);
}
```

---

## 3.7 数组与集合

### C# 数组（与 C++ 相似）

```csharp
// -------- 一维数组 --------
int[] scores = new int[5];           // 创建长度为 5 的数组
int[] numbers = { 1, 2, 3, 4, 5 };   // 声明并初始化

// -------- 多维数组（矩阵）--------
int[,] matrix = new int[3, 3];       // 3x3 矩阵
int[,] grid = { { 1, 2, 3 }, { 4, 5, 6 } };

// -------- 交错数组（数组的数组）--------
int[][] jagged = new int[3][];
jagged[0] = new int[] { 1, 2 };
jagged[1] = new int[] { 3, 4, 5, 6 };

// -------- Unity 中最常用的 List --------
using System.Collections.Generic;

List<int> hpList = new List<int>();   // 不需要预先知道大小
hpList.Add(100);
hpList.Add(200);
hpList.Add(150);
hpList.Remove(200);                   // 按值删除
hpList.RemoveAt(0);                   // 按索引删除
int count = hpList.Count;             // .Count 不是 .length
hpList.Sort();
```

### 字典（Dictionary）

```csharp
// -------- Dictionary：键值对（类似 unordered_map）--------
using System.Collections.Generic;

Dictionary<string, int> playerScores = new Dictionary<string, int>();

playerScores["孙悟空"] = 8500;
playerScores["猪八戒"] = 7200;
playerScores["沙和尚"] = 6800;

// -------- 查找 --------
if (playerScores.ContainsKey("孙悟空"))
{
    int score = playerScores["孙悟空"];
    Debug.Log("孙悟空分数：" + score);
}

// -------- 遍历 --------
foreach (KeyValuePair<string, int> kvp in playerScores)
{
    Debug.Log(kvp.Key + " = " + kvp.Value);
}

// -------- Unity 中的实用场景 --------
Dictionary<string, GameObject> enemyPrefabs = new Dictionary<string, GameObject>();
enemyPrefabs["goblin"] = goblinPrefab;
enemyPrefabs["dragon"] = dragonPrefab;

// 动态生成敌人
GameObject.Instantiate(enemyPrefabs["dragon"], spawnPos, Quaternion.identity);
```

---

## 3.8 函数（方法）

### 基本语法（与 C++ 类似，但不需要声明）

```csharp
// -------- 基本函数 --------
int Add(int a, int b)
{
    return a + b;
}

// -------- Unity 常用返回类型 --------
Vector3 GetForward()
{
    return transform.forward;  // transform.forward = Vector3(0,0,1)
}

// -------- out 参数（C++ 中用指针）--------
void GetMinMax(int[] numbers, out int min, out int max)
{
    min = numbers[0];
    max = numbers[0];
    foreach (int n in numbers)
    {
        if (n < min) min = n;
        if (n > max) max = n;
    }
}

// 调用
int[] nums = { 3, 1, 4, 1, 5, 9, 2, 6 };
GetMinMax(nums, out int minVal, out int maxVal);
Debug.Log("最小=" + minVal + " 最大=" + maxVal);

// -------- ref 参数：引用传递（类似 C++ 的 &）--------
void DoubleValue(ref int value)
{
    value *= 2;
}

// -------- 可选参数（C++ 没有）--------
void CreateEnemy(string name = "小怪", int hp = 100, int attack = 10)
{
    Debug.Log($"创建敌人：{name} HP={hp} ATK={attack}");
}

CreateEnemy();                         // 使用默认值
CreateEnemy("哥布林");                  // 只传 name
CreateEnemy("巨龙", 500, 50);          // 全部传参

// -------- 方法重载（与 C++ 一样）--------
int Add(int a, int b) { return a + b; }
float Add(float a, float b) { return a + b; }   // 不同参数类型
int Add(int a, int b, int c) { return a + b + c; }  // 不同参数个数
```

---

## 3.9 面向对象（类）

### 类的定义（与 C++ 有差异）

```csharp
// C# 中默认访问修饰符是 private
// C# 中继承用 : （与 C++ 一样）

// -------- 基本类 --------
public class Player
{
    // -------- 字段（字段名建议加 _ 或 m_ 前缀）--------
    private int _hp;
    private string _name;
    private int _attack;

    // -------- 属性（Property）：封装字段，比 C++ 的 getter/setter 更简洁 --------
    public int HP
    {
        get { return _hp; }
        set { _hp = Mathf.Clamp(value, 0, 99999); }   // value 是 set 的参数
    }

    public string Name
    {
        get { return _name; }
        set { _name = value; }
    }

    // -------- 自动属性（编译器自动生成 backing field）--------
    public int Attack { get; set; }

    // -------- 只读属性 --------
    public bool IsAlive { get { return _hp > 0; } }

    // -------- 构造函数 --------
    public Player(string name, int hp, int attack)
    {
        _name = name;
        _hp = hp;
        _attack = attack;
        Debug.Log($"玩家 {_name} 创建，HP={_hp}");
    }

    // -------- 方法 --------
    public void AttackEnemy(Enemy enemy)
    {
        Debug.Log($"{_name} 攻击，造成 {_attack} 点伤害");
        enemy.TakeDamage(_attack);
    }

    public void TakeDamage(int damage)
    {
        _hp -= damage;
        if (_hp < 0) _hp = 0;
        Debug.Log($"{_name} 受伤，剩余 HP={_hp}");
    }
}

// -------- 使用类 --------
void Start()
{
    Player hero = new Player("孙悟空", 5000, 250);
    Debug.Log("存活状态：" + hero.IsAlive);
}
```

### 属性 vs 字段（C# 特有）

```csharp
// 字段（Field）：直接存储数据
public int hp;   // 可以直接访问，但无法控制读写

// 属性（Property）：受保护的访问方式
private int _hp;
public int HP
{
    get { return _hp; }    // 读取时的逻辑
    set { _hp = value; }   // 写入时的逻辑，value 是新值
}

// 自动属性（最常用）
public int MP { get; set; }   // 编译器自动生成 private backing field

//-------- 为什么用属性？--------
private int _hp;
public int HP
{
    get { return _hp;; }
    set
    {
        // 可以在这里加逻辑：限制范围、写日志、触发事件
        _hp = Mathf.Clamp(value, 0, maxHP);
        OnHPChanged?.Invoke(_hp);  // 血量变化时通知 UI
    }
}
```

### 静态类和方法

```csharp
// -------- 静态类（不能实例化）--------
public static class GameMath
{
    public static float Distance(Vector3 a, Vector3 b)
    {
        return Vector3.Distance(a, b);
    }

    public static int Max(int a, int b)
    {
        return a > b ? a : b;
    }
}

// 调用（不需要 new）
float d = GameMath.Distance(pos1, pos2);

// -------- 静态变量（所有实例共享）--------
public class Enemy
{
    public static int enemyCount = 0;   // 所有敌人共享计数器

    public Enemy()
    {
        enemyCount++;
        Debug.Log("敌人生成，当前总数：" + enemyCount);
    }
}

// 调用
Debug.Log("敌人总数：" + Enemy.enemyCount);
```

---

## 3.10 接口（Interface）

### 什么是接口？

> **接口 = 约定"有什么方法"，但不规定"怎么实现"。**
> C# 不支持多继承，接口是实现多功能的替代方案。

### 定义和实现接口

```csharp
// -------- 定义接口 --------
public interface IDamageable
{
    void TakeDamage(int damage);   // 只声明，不实现
    int HP { get; }               // 只声明属性
    bool IsAlive { get; }
}

public interface IAttackable
{
    void Attack(IDamageable target);
}

// -------- 实现接口 --------
public class Enemy : MonoBehaviour, IDamageable, IAttackable
{
    private int _hp = 100;

    public int HP => _hp;   // 属性表达式体（C# 6.0+）
    public bool IsAlive => _hp > 0;

    public void TakeDamage(int damage)
    {
        _hp -= damage;
        Debug.Log($"受伤，HP={_hp}");
    }

    public void Attack(IDamageable target)
    {
        Debug.Log("发起攻击");
        target.TakeDamage(50);
    }
}

// -------- 使用接口 --------
public class Test : MonoBehaviour
{
    void Start()
    {
        Enemy e = new Enemy();
        IDamageable d = e;   // 可以隐式转换

        // 通过接口调用
        d.TakeDamage(30);    // 只能调用接口中定义的方法
        // d.Attack() 无法调用，因为 IDamageable 没有 Attack
    }
}
```

---

## 3.11 继承与多态

### 类继承

```csharp
// -------- 基类 --------
public class LivingThing
{
    public int HP { get; set; }
    public bool IsAlive => HP > 0;

    public virtual void TakeDamage(int damage)   // virtual：允许子类重写
    {
        HP -= damage;
        Debug.Log($"受伤，HP={HP}");
    }
}

// -------- 派生类 --------
public class Player : LivingThing
{
    public int MP { get; set; }

    public override void TakeDamage(int damage)   // override：重写基类方法
    {
        // 调用父类实现
        base.TakeDamage(damage);

        // 额外逻辑：护盾减伤
        Debug.Log("玩家受伤触发受伤动画");
    }
}

public class Enemy : LivingThing
{
    public int attackPower;

    public override void TakeDamage(int damage)
    {
        HP -= damage;
        Debug.Log($"敌人受伤，HP={HP}");

        if (!IsAlive)
        {
            Die();
        }
    }

    void Die()
    {
        Debug.Log("敌人死亡，播放死亡动画");
    }
}
```

### 抽象类

```csharp
// -------- 抽象类：不能实例化，只能被继承 --------
public abstract class Character : MonoBehaviour
{
    public int HP { get; set; }
    public int Attack { get; set; }

    // -------- 抽象方法：子类必须实现 --------
    public abstract void AttackTarget(Character target);

    // -------- 普通虚方法：子类可以选择重写 --------
    public virtual void Die()
    {
        Debug.Log("角色死亡");
        Destroy(gameObject);   // Unity 中销毁对象
    }
}

public class Goblin : Character
{
    public override void AttackTarget(Character target)
    {
        Debug.Log("哥布林攻击");
        target.HP -= Attack;
    }
}
```

---

## 3.12 委托与事件

### 委托（Delegate）

```csharp
// -------- 委托：方法的"容器" --------
public delegate void OnDamageDelegate(int damage);

public class Player : MonoBehaviour
{
    // -------- 声明委托变量 --------
    public OnDamageDelegate onDamage;

    private int _hp = 100;

    public void TakeDamage(int damage)
    {
        _hp -= damage;

        // -------- 调用委托 --------
        onDamage?.Invoke(damage);   // ?. 空值检查调用
    }
}

// -------- 使用委托 --------
public class Test : MonoBehaviour
{
    public Player player;

    void Start()
    {
        // -------- 订阅方法 --------
        player.onDamage += HandleDamage;

        // -------- 取消订阅 --------
        player.onDamage -= HandleDamage;
    }

    void HandleDamage(int damage)
    {
        Debug.Log($"受到 {damage} 点伤害！");
    }
}
```

### UnityEvent（推荐用法）

```csharp
using UnityEngine.Events;

public class Player : MonoBehaviour
{
    // -------- UnityEvent：比普通 delegate 更安全（支持 Inspector 绑定）--------
    [SerializeField]
    private UnityEvent<int> onDamage;

    public void TakeDamage(int damage)
    {
        Debug.Log("受伤：" + damage);
        onDamage?.Invoke(damage);
    }
}

// 在 Inspector 中：
// 1. 选中挂载 Player 脚本的对象
// 2. 在 Inspector 中找到 onDamage
// 3. 点击 + 添加监听
// 4. 拖入想要响应的对象的方法
```

---

## 3.13 动手练习 🧪

### 练习 1：对比 C++ 和 C# ⭐
```
用 C# 实现 C++ Chapter 02 的计算器程序
（加减乘除、取余）对比两者的语法差异
```

### 练习 2：属性练习 ⭐
```
写一个 Enemy 类
HP 属性：设置时自动限制在 0 到 maxHP 之间
IsAlive 属性：只读，返回 HP > 0
TakeDamage 方法：扣血并触发事件
```

### 练习 3：接口练习 ⭐⭐
```
定义 ICollectable 接口（Collect() 方法）
定义 IPickable 接口（Pick() 方法）
写一个 Item 类实现两个接口
在 Start 中创建 Item 实例并调用方法
```

### 练习 4：事件系统 ⭐⭐
```
写一个 GameManager 单例
定义 onGameOver UnityEvent
敌人死亡时触发 onGameOver
在 UI 脚本中订阅，Game Over 时显示 UI
```

### 练习 5：完整角色系统 ⭐⭐⭐
```
Player 类：HP/MP/Attack，有 Attack 方法
Enemy 类：继承 Player（用不同实现）
Weapon 类：实现 IAttackable
在 Start 中让它们互相攻击
用 foreach 遍历 List<Player> 打印存活状态
```

---

## 3.14 本章小结

```
✅ 已掌握：
├── C++ 与 C# 的核心语法差异
├── C# 数据类型（int, float, string, Vector3）
├── Unity 序列化字段（[SerializeField], [Header], [Range]）
├── 运算符（+、-、*、/、字符串插值 $""）
├── 控制流程（if/for/while/foreach/switch）
├── 数组、List、Dictionary
├── 函数/方法（out/ref/可选参数）
├── 类（字段、属性、自动属性）
├── 继承（override/virtual/abstract）
├── 接口（Interface）
├── 委托与事件（Delegate/UnityEvent）
└── 在 Unity 中运行 C# 脚本

🔜 下章预告：
第四章：Unity 脚本开发（MonoBehaviour）—— 深入理解 Start/Update/FixedUpdate/Awake/LateUpdate 等生命周期函数。
```

---

_📚 参考资料：《C# 入门经典》《Unity 脚本入门》《Microsoft C# 文档》_
