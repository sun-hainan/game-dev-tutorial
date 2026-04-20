# Chapter 04：Unity 脚本开发（MonoBehaviour）

> 🎯 目标：深入理解 MonoBehaviour 生命周期、常见 API、组件化开发思想。

---

## 4.1 MonoBehaviour 是什么？

### 生活中的类比 🎭

> **MonoBehaviour = 游戏对象的"灵魂"。**
> 一个 GameObject 本身只是个空壳，挂上脚本后才有"思想"。

### Unity 脚本的本质

```csharp
// 所有 Unity C# 脚本都继承自 MonoBehaviour
public class MyScript : MonoBehaviour
{
    // 你的代码...
}
```

### MonoBehaviour vs 普通类

| 特性 | MonoBehaviour | 普通 C# 类 |
|-----|--------------|-----------|
| 可以挂到 GameObject | ✅ | ❌ |
| 有生命周期函数 | ✅（Update/Awake...）| ❌ |
| 可以拖到 Inspector | ✅ | ❌ |
| 可以使用 Unity API | ✅ | ❌ |
| 可以使用 GetComponent | ✅ | ❌ |
| 可以使用 coroutine | ✅ | ❌ |

---

## 4.2 生命周期函数详解

### 完整调用顺序图

```
应用启动
    ↓
Awake()          ← 1. 脚本实例创建时（即使脚本未启用）
    ↓
OnEnable()       ← 2. 脚本启用时
    ↓
Start()          ← 3. 第一次 Update 前（脚本必须启用）
    ↓
┌─────────────────────────────────┐
│      每帧执行（游戏循环）         │
│                                   │
│  FixedUpdate()  ← 固定物理更新    │
│       ↓                          │
│  Update()       ← 每帧逻辑更新    │
│       ↓                          │
│  LateUpdate()   ← 所有 Update 后  │
└─────────────────────────────────┘
    ↓
OnDisable()      ← 脚本禁用时
    ↓
OnDestroy()      ← 脚本/对象销毁时
```

### Awake vs Start

```csharp
public class LifecycleDemo : MonoBehaviour
{
    void Awake()
    {
        Debug.Log("1. Awake - 脚本实例创建时调用，即使未启用");
    }

    void OnEnable()
    {
        Debug.Log("2. OnEnable - 脚本被启用时调用");
    }

    void Start()
    {
        Debug.Log("3. Start - 第一次 Update 前调用，脚本必须启用");
    }

    void Update()
    {
        // 适合：输入检测、游戏逻辑
        // 每帧调用，帧率不同则间隔不同
    }

    void FixedUpdate()
    {
        // 适合：物理相关（力、扭矩）
        // 固定时间间隔调用（默认 0.02 秒，即 50次/秒）
    }

    void LateUpdate()
    {
        // 适合：相机跟随、第三方观察者
        // 在所有 Update 调用后执行
    }

    void OnDisable()
    {
        Debug.Log("OnDisable - 脚本被禁用时调用");
    }

    void OnDestroy()
    {
        Debug.Log("OnDestroy - 脚本被销毁时调用");
    }
}
```

---

## 4.3 Update vs FixedUpdate

### 关键区别

| 函数 | 触发频率 | 适用场景 |
|-----|---------|---------|
| `Update()` | 每帧（帧率不固定，60FPS = 每 0.016 秒）| 输入检测、普通逻辑 |
| `FixedUpdate()` | 固定间隔（默认 50次/秒）| 物理模拟（Rigidbody）|
| `LateUpdate()` | 每帧最后 | 相机跟随 |

### 实际例子：移动玩家

```csharp
public class PlayerController : MonoBehaviour
{
    public float moveSpeed = 5f;

    void Update()
    {
        // -------- 移动逻辑在 Update --------
        // 帧率不同时，用 Time.deltaTime 保证速度一致
        float h = Input.GetAxis("Horizontal");  // A/D 或 ←/→
        float v = Input.GetAxis("Vertical");    // W/S 或 ↑/↓

        Vector3 move = new Vector3(h, 0, v);
        transform.position += move * moveSpeed * Time.deltaTime;
    }

    void FixedUpdate()
    {
        // -------- 物理力在 FixedUpdate --------
        // 物理计算需要稳定的时间步长
        // rb.AddForce() 放在这里
    }
}
```

### Time.deltaTime vs Time.fixedDeltaTime

```csharp
void Update()
{
    // deltaTime：上一帧的时间间隔（秒）
    // 60FPS → 0.016秒
    // 30FPS → 0.033秒
    float distance = speed * Time.deltaTime;   // 每帧移动的距离
}

void FixedUpdate()
{
    // fixedDeltaTime：物理更新间隔（秒）
    float force = Physics.gravity.magnitude * Time.fixedDeltaTime;
}
```

---

## 4.4 获取组件（GetComponent）

### 最常用的 API

```csharp
public class ComponentDemo : MonoBehaviour
{
    void Start()
    {
        // -------- 获取自身组件 --------
        Rigidbody rb = GetComponent<Rigidbody>();
        if (rb != null)
        {
            rb.AddForce(Vector3.up * 10f);
        }

        // -------- 获取子对象组件 --------
        Collider col = GetComponentInChildren<Collider>();
        Renderer rend = GetComponentInChildren<Renderer>();

        // -------- 获取父对象组件 --------
        Transform parent = transform.parent;
        if (parent != null)
        {
            Rigidbody parentRB = parent.GetComponent<Rigidbody>();
        }

        // -------- 获取多个组件 --------
        Collider[] cols = GetComponents<Collider>();
        foreach (var c in cols)
        {
            c.enabled = false;   // 禁用所有碰撞体
        }
    }
}
```

### CompareTag：检查对象标签

```csharp
void OnCollisionEnter(Collision collision)
{
    // -------- 方法1：CompareTag（推荐，更快）--------
    if (collision.gameObject.CompareTag("Enemy"))
    {
        Debug.Log("撞到了敌人！");
    }

    // -------- 方法2：tag 属性 --------
    if (collision.gameObject.tag == "Enemy")
    {
        Debug.Log("撞到了敌人！");
    }

    // -------- 方法3：.layer 检测 --------
    if (collision.gameObject.layer == LayerMask.NameToLayer("Enemy"))
    {
        Debug.Log("撞到敌人层！");
    }
}
```

---

## 4.5 实例化与销毁

### 创建和销毁对象

```csharp
public class Bullet : MonoBehaviour
{
    public float speed = 10f;
    public float lifetime = 3f;

    void Start()
    {
        // -------- 发射方向移动 --------
        GetComponent<Rigidbody>().velocity = transform.forward * speed;

        // -------- 延迟销毁 --------
        Destroy(gameObject, lifetime);   // 3秒后自动销毁
    }

    void OnCollisionEnter(Collision col)
    {
        // -------- 撞到敌人时立即销毁 --------
        if (col.gameObject.CompareTag("Enemy"))
        {
            Destroy(gameObject);   // 销毁子弹
        }
    }
}

// -------- 生成子弹 --------
public class Shooter : MonoBehaviour
{
    public GameObject bulletPrefab;   // 拖入 Inspector
    public Transform firePoint;       // 枪口位置
    public float fireRate = 0.2f;
    private float nextFireTime = 0f;

    void Update()
    {
        if (Input.GetButton("Fire1") && Time.time > nextFireTime)
        {
            // -------- 实例化（克隆） --------
            GameObject bullet = Instantiate(bulletPrefab, firePoint.position, firePoint.rotation);

            nextFireTime = Time.time + fireRate;
        }
    }
}
```

### Instantiate 的三种形式

```csharp
// -------- 形式1：只克隆（不指定父对象）--------
GameObject obj = Instantiate(prefab);

// -------- 形式2：指定位置和旋转 --------
GameObject obj = Instantiate(prefab, position, Quaternion.identity);

// -------- 形式3：指定父对象 --------
GameObject obj = Instantiate(prefab, parentTransform);

// -------- 形式4：完整参数 --------
GameObject obj = Instantiate(prefab, position, rotation, parentTransform);

// -------- 使用返回的克隆对象 --------
GameObject enemy = Instantiate(enemyPrefab, spawnPos, Quaternion.identity);
enemy.GetComponent<Enemy>().Initialize(difficulty);   // 给克隆对象设置参数
```

---

## 4.6 访问其他脚本

### 同对象上的脚本

```csharp
// Enemy.cs 挂载在同一个 GameObject 上
public class Enemy : MonoBehaviour
{
    public void TakeDamage(int damage)
    {
        hp -= damage;
    }
}

// PlayerAttack.cs 也挂载在同一对象上
public class PlayerAttack : MonoBehaviour
{
    public int attackPower = 50;

    void Attack()
    {
        // -------- GetComponent 获取同对象的其他脚本 --------
        Enemy enemy = GetComponent<Enemy>();
        if (enemy != null)
        {
            enemy.TakeDamage(attackPower);
        }
    }
}
```

### 访问其他对象

```csharp
// -------- 通过标签找对象 --------
GameObject player = GameObject.FindGameObjectWithTag("Player");

// -------- 通过名字找对象（慢，少用）--------
GameObject enemy = GameObject.Find("Enemy_Boss");

// -------- 找多个对象 --------
GameObject[] enemies = GameObject.FindGameObjectsWithTag("Enemy");

// -------- 通过路径找（Editor 专用，不推荐游戏运行时用）--------
GameObject target = GameObject.Find("/Level1/Enemies/Boss");
```

### SendMessage（跨语言调用）

```csharp
// -------- 发送消息给目标对象的所有脚本 --------
target.SendMessage("TakeDamage", 50);
// 等价于在 target 上的所有脚本中查找名为 TakeDamage 的方法并调用

// -------- BroadcastMessage：包括子对象 --------
target.BroadcastMessage("TakeDamage", 50);
```

---

## 4.7 协程（Coroutine）基础

### 什么是协程？

> **协程 = 能在中间暂停、之后恢复的函数。**
> 用于"等一会儿再做某事"或"每帧做一点点"。

```csharp
// -------- 普通函数：一次性完成 --------
void DelayedAction()
{
    Debug.Log("等待2秒...");
    // 没有真正的等待，代码会立即继续执行
    Debug.Log("继续！");   // 这行会立即执行
}

// -------- 协程：可以在中间暂停 --------
IEnumerator DelayedActionCoroutine()
{
    Debug.Log("等待2秒...");
    yield return new WaitForSeconds(2f);   // 暂停 2 秒！
    Debug.Log("2秒后继续！");

    yield return new WaitForSeconds(1f);   // 再等 1 秒
    Debug.Log("又等了1秒");
}

// -------- 启动协程 --------
void Start()
{
    StartCoroutine(DelayedActionCoroutine());
}

// -------- 停止协程 --------
void StopCoroutine("DelayedActionCoroutine");  // 停止指定的
StopAllCoroutines();                           // 停止所有
```

### 协程实战：淡入淡出

```csharp
public class FadeUI : MonoBehaviour
{
    public CanvasGroup canvasGroup;   // 拖入 CanvasGroup 组件

    // -------- 淡入 --------
    public IEnumerator FadeIn(float duration)
    {
        canvasGroup.alpha = 0f;
        float elapsed = 0f;

        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            canvasGroup.alpha = Mathf.Lerp(0f, 1f, elapsed / duration);
            yield return null;   // 等待下一帧
        }

        canvasGroup.alpha = 1f;
    }

    // -------- 淡出 --------
    public IEnumerator FadeOut(float duration)
    {
        canvasGroup.alpha = 1f;
        float elapsed = 0f;

        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            canvasGroup.alpha = Mathf.Lerp(1f, 0f, elapsed / duration);
            yield return null;
        }

        canvasGroup.alpha = 0f;
    }

    // -------- 使用 --------
    void Start()
    {
        StartCoroutine(FadeIn(1f));
    }
}
```

### 协程 vs Invoke

```csharp
// -------- Invoke（简单定时，简单用）--------
void Start()
{
    Invoke("Explode", 2f);    // 2秒后调用 Explode()
    InvokeRepeating("Spawn", 1f, 0.5f);   // 1秒后开始，每0.5秒重复
}

void Explode()
{
    Debug.Log("爆炸！");
}

// -------- 协程（复杂定时，用协程）--------
IEnumerator SpawnLoop()
{
    yield return new WaitForSeconds(1f);   // 等待1秒
    while (true)
    {
        Spawn();
        yield return new WaitForSeconds(0.5f);   // 每0.5秒
    }
}

void Start()
{
    StartCoroutine(SpawnLoop());
}
```

---

## 4.8 常用 Unity API

### 变换操作

```csharp
// -------- 位置 --------
transform.position = new Vector3(0, 1, 0);     // 设置世界坐标位置
transform.localPosition = new Vector3(0, 1, 0); // 设置相对父物体位置
Vector3 pos = transform.position;               // 获取世界坐标

// -------- 移动 --------
transform.Translate(Vector3.forward * speed * Time.deltaTime);  // 沿 forward 移动
transform.Translate(Vector3.up * jumpForce * Time.deltaTime);   // 向上跳

// -------- 旋转 --------
transform.Rotate(0, 90 * Time.deltaTime, 0);    // 旋转
transform.rotation = Quaternion.Euler(0, 90, 0);  // 欧拉角设置
transform.LookAt(target);                       // 朝向目标

// -------- 缩放 --------
transform.localScale = new Vector3(2, 2, 2);    // 放大2倍

// -------- 父子关系 --------
transform.SetParent(newParent);                  // 设置父对象
transform.SetParent(null);                      // 解除父子关系（成为根对象）
transform.DetachChildren();                     // 解除所有子对象

// -------- 方向 --------
Vector3 forward = transform.forward;           // transform 的 Z 轴正方向
Vector3 up = transform.up;                      // transform 的 Y 轴正方向
Vector3 right = transform.right;                // transform 的 X 轴正方向
```

### 计时器

```csharp
void Update()
{
    // -------- Time 类 --------
    float delta = Time.deltaTime;          // 上一帧的时间
    float time = Time.time;               // 游戏开始到现在的时间
    float realtime = Time.realtimeSinceStartup;  // 实际墙钟时间
    float unscaled = Time.unscaledDeltaTime;     // 不受 Time.timeScale 影响的 delta

    // -------- 暂停/加速 --------
    // Time.timeScale = 0;   // 暂停
    // Time.timeScale = 0.5f; // 0.5倍速
    // Time.timeScale = 2f;  // 2倍速
}
```

### 随机数

```csharp
// -------- Unity 随机 --------
int r1 = Random.Range(1, 10);       // 整数：返回 1~9（不包括10）
float r2 = Random.Range(0f, 1f);    // 浮点：返回 0~1
Vector3 r3 = Random.insideUnitSphere;   // 单位球内随机点
Vector3 r4 = Random.onUnitSphere;        // 单位球面上随机点
Vector2 r5 = Random.insideUnitCircle;    // 单位圆内随机点
Quaternion r6 = Random.rotation;         // 随机四元数旋转
```

---

## 4.9 常见错误与调试

### Debug.Log 调试

```csharp
void Update()
{
    Debug.Log("每帧打印，太多了！");     // 每帧都打印，会卡
    Debug.LogWarning("警告信息");         // 黄色警告
    Debug.LogError("错误信息");           // 红色错误
}
```

### 运行时修改 Inspector 数值

```csharp
[ContextMenu("Reset HP")]   // 在 Inspector 右键菜单添加选项
void ResetHP()
{
    currentHP = maxHP;
    Debug.Log("HP 已重置");
}
```

### 调试断点

```
1. 在 VS 中打开 C# 脚本
2. 在代码行号左侧点击（红点）
3. 在 Unity 中点击 Play
4. VS 会停在断点处
5. 可以单步调试（F10/F11）
```

---

## 4.10 动手练习 🧪

### 练习 1：生命周期顺序观察 ⭐
```
新建脚本 LifecycleCheck
在 Awake/Start/OnEnable/Update 里分别 Debug.Log
观察各函数的调用顺序
```

### 练习 2：玩家移动控制 ⭐⭐
```
用 Input.GetAxis 控制玩家 Cube 在地面上移动
用 W/A/S/D 和方向键都能控制
速度适中，用 deltaTime 保证一致性
```

### 练习 3：射击子弹 ⭐⭐
```
创建 Bullet 预制体
每点击鼠标左键，从玩家位置发射一颗子弹
子弹向前飞行，3秒后自动销毁
```

### 练习 4：延迟消失的道具 ⭐⭐
```
生成一个道具后 5 秒自动消失
用协程实现，可以显示倒计时
```

### 练习 5：敌人 AI 简单版 ⭐⭐⭐
```
敌人自动朝玩家方向移动
每 3 秒攻击一次
距离 < 2 时停止移动
```

---

## 4.11 本章小结

```
✅ 已掌握：
├── MonoBehaviour 生命周期（Awake/Start/Update/FixedUpdate/LateUpdate）
├── Update vs FixedUpdate 的选择
├── GetComponent 获取组件
├── CompareTag vs tag
├── Instantiate 实例化
├── Destroy 销毁
├── 访问其他脚本（GetComponent/SendMessage/FindGameObjectWithTag）
├── 协程（Coroutine）基础
├── 常用 Unity API（Transform/Time/Random）
└── Debug 调试

🔜 下章预告：
第五章：Input 输入系统 —— 键盘/鼠标/手柄输入，完整玩家控制器。
```

---

_📚 参考资料：《Unity 脚本入门》《Unity 官方文档 - MonoBehaviour》_
