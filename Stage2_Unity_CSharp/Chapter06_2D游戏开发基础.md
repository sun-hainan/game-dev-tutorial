# Chapter 06：2D 游戏开发基础

> 🎯 目标：掌握 Unity 2D 开发的核心——Sprite、Physics 2D、2D 角色控制。

---

## 6.1 Unity 2D vs 3D

### 核心区别

| 维度 | 3D | 2D |
|-----|-----|-----|
| 相机 | Perspective（透视）| Orthographic（正交）|
| 物理 | Physics | Physics 2D |
| 对象 | 3D Mesh | Sprite |
| 轴向 | X/Y/Z | X/Y（Z=前后）|

### 创建 2D 项目

```
1. Unity Hub → 新建项目 → 2D 模板
2. 自动设置：
   - 相机：Orthographic
   - 天空盒：纯色
   - 纹理导入：2D（无 mipmap）
```

---

## 6.2 Sprite（精灵）

### 什么是 Sprite？

> **Sprite = 2D 图片在 Unity 中的表示。**

### 导入 2D 图片

```
1. 把 PNG/JPG 拖入 Assets/Sprites 文件夹
2. 选中图片 → Inspector：
   - Texture Type → Sprite (2D and UI)
   - Sprite Mode → Single
   - Pixels Per Unit → 100（决定图片在 Unity 中的大小）
   - 点击 Apply
3. 拖入 Scene 或 Hierarchy，即可在 2D 场景中使用
```

### Sprite Renderer 组件

```csharp
// SpriteRenderer：控制 2D 图片的渲染
public class SpriteDemo : MonoBehaviour
{
    public SpriteRenderer spriteRenderer;
    public Sprite sprite1;
    public Sprite sprite2;

    void Start()
    {
        // -------- 显示图片 --------
        spriteRenderer.sprite = sprite1;

        // -------- 颜色 --------
        spriteRenderer.color = Color.red;
        spriteRenderer.color = new Color(1f, 0.5f, 0f);   // 橙色
        spriteRenderer.color = new Color(1f, 1f, 1f, 0.5f); // 半透明

        // -------- 水平翻转 --------
        spriteRenderer.flipX = true;   // 左右翻转
        spriteRenderer.flipY = true;   // 上下翻转

        // -------- 排序（类似 Z-index）--------
        spriteRenderer.sortingOrder = 10;   // 数字越大越靠前
        spriteRenderer.sortingLayerName = "Player";  // 用层管理
    }
}
```

---

## 6.3 2D 物理系统

### Physics 2D 设置

```
Edit → Project Settings → Physics 2D
- Gravity：重力（默认 -9.81，近似地球）
- 设置层之间的碰撞矩阵
```

### Rigidbody 2D

> 挂载 Rigidbody 2D 的对象才会受物理影响。

```csharp
public class Physics2DDemo : MonoBehaviour
{
    private Rigidbody2D rb;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        // -------- 移动（用 Velocity 更好）--------
        float h = Input.GetAxisRaw("Horizontal");
        rb.velocity = new Vector2(h * 5f, rb.velocity.y);

        // -------- 跳跃 --------
        if (Input.GetButtonDown("Jump"))
        {
            rb.AddForce(Vector2.up * 10f, ForceMode2D.Impulse);
        }
    }
}
```

### 常用 Rigidbody 2D 设置

| 属性 | 说明 | 推荐值 |
|-----|------|-------|
| Body Type | Kinematic/ Dynamic/ Static | Dynamic |
| Gravity Scale | 重力缩放 | 地面对象设为 0 |
| Mass | 质量 | 玩家默认 1 |
| Linear Drag | 移动阻力 | 角色 3-5 |
| Angular Drag | 旋转阻力 | 角色 5-10 |
| Collision Detection | 碰撞检测模式 | Continuous |
| Interpolate | 插值（防抖动）| Interpolate（移动快时）|

### Collider 2D

```
常用 2D 碰撞体：
- Box Collider 2D：矩形（箱子、地面）
- Circle Collider 2D：圆形（球、角色）
- Capsule Collider 2D：胶囊形（角色）
- Polygon Collider 2D：自定义多边形（不规则形状）
```

---

## 6.4 2D 角色控制器

### 完整的 2D 玩家控制器

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody2D))]
[RequireComponent(typeof(Collider2D))]
public class Player2DController : MonoBehaviour
{
    [Header("移动设置")]
    public float moveSpeed = 5f;
    public float jumpForce = 10f;

    [Header("地面检测")]
    public LayerMask groundLayer;     // 拖入地面层
    public float groundCheckRadius = 0.2f;
    public Transform groundCheck;    // 在角色脚下创建一个空对象

    private Rigidbody2D rb;
    private bool isGrounded = false;
    private float horizontalInput;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        // -------- 水平输入 --------
        horizontalInput = Input.GetAxisRaw("Horizontal");

        // -------- 地面检测（2D 用 CircleCast）--------
        isGrounded = Physics2D.CircleCast(
            groundCheck.position,
            groundCheckRadius,
            Vector2.down,
            groundCheckRadius,
            groundLayer
        );

        // -------- 跳跃 --------
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
        }
    }

    void FixedUpdate()
    {
        // -------- 水平移动（FixedUpdate 中设置速度更物理正确）--------
        rb.velocity = new Vector2(horizontalInput * moveSpeed, rb.velocity.y);

        // -------- 翻转角色（朝向移动方向）--------
        if (horizontalInput > 0)
        {
            transform.localScale = new Vector3(1, 1, 1);   // 朝右
        }
        else if (horizontalInput < 0)
        {
            transform.localScale = new Vector3(-1, 1, 1);  // 朝左
        }
    }
}
```

---

## 6.5 碰撞检测（2D）

### 碰撞响应函数

```csharp
// 2D 碰撞响应（物理接触）
void OnCollisionEnter2D(Collision2D col)
{
    Debug.Log($"碰撞开始：{col.collider.name}");
}

void OnCollisionStay2D(Collision2D col)
{
    Debug.Log($"碰撞中：{col.collider.name}");
}

void OnCollisionExit2D(Collision2D col)
{
    Debug.Log($"碰撞结束：{col.collider.name}");
}

// 触发响应（穿透检测，不产生物理碰撞）
void OnTriggerEnter2D(Collider2D col)
{
    Debug.Log($"触发进入：{col.name}");
}

void OnTriggerStay2D(Collider2D col)
{
    Debug.Log($"触发中：{col.name}");
}

void OnTriggerExit2D(Collider2D col)
{
    Debug.Log($"触发离开：{col.name}");
}
```

### Collision2D 信息

```csharp
void OnCollisionEnter2D(Collision2D col)
{
    // -------- 接触点 --------
    Vector2 contactPoint = col.contacts[0].point;
    Debug.Log($"接触点：{contactPoint}");

    // -------- 碰撞对象的标签 --------
    if (col.gameObject.CompareTag("Enemy"))
    {
        Debug.Log("撞到敌人！");
    }

    // -------- 反弹方向 --------
    Vector2 normal = col.contacts[0].normal;
    Debug.Log($"法线：{normal}");
}
```

---

## 6.6 2D 摄像机跟随

```csharp
using UnityEngine;

public class Camera2DFollow : MonoBehaviour
{
    public Transform target;           // 跟随目标（玩家）
    public float smoothSpeed = 0.125f; // 平滑速度
    public Vector3 offset;             // 偏移量

    void LateUpdate()
    {
        // -------- 计算目标位置 --------
        Vector3 desiredPosition = target.position + offset;

        // -------- 平滑移动（线性插值）--------
        Vector3 smoothedPosition = Vector3.Lerp(
            transform.position,
            desiredPosition,
            smoothSpeed
        );

        transform.position = smoothedPosition;
        // -------- 相机朝向保持 --------
        transform.position = new Vector3(
            transform.position.x,
            transform.position.y,
            -10   // 2D 相机 Z 轴固定 -10
        );
    }
}
```

### 带边界的摄像机跟随

```csharp
public class Camera2DBoundedFollow : MonoBehaviour
{
    public Transform target;
    public float smoothSpeed = 0.125f;
    public float minX, maxX;   // 限制相机 X 范围
    public float minY, maxY;  // 限制相机 Y 范围

    void LateUpdate()
    {
        float clampedX = Mathf.Clamp(target.position.x, minX, maxX);
        float clampedY = Mathf.Clamp(target.position.y, minY, maxY);

        Vector3 targetPos = new Vector3(clampedX, clampedY, -10);
        transform.position = Vector3.Lerp(transform.position, targetPos, smoothSpeed);
    }
}
```

---

## 6.7 2D 平台搭建

### 平台碰撞体配置

```
Layer 设置：
1. Edit → Project Settings → Tags and Layers
2. 新建 Layer：Ground, Player, Enemy, Collectible

碰撞矩阵：
Edit → Project Settings → Physics 2D → Layer Collision Matrix
- Player 与 Ground：✅ 碰撞
- Player 与 Enemy：✅ 碰撞
- Player 与 Collectible：✅ 触发（Is Trigger）
- Enemy 与 Ground：✅ 碰撞
```

### 平台移动（移动平台）

```csharp
public class MovingPlatform : MonoBehaviour
{
    public float speed = 2f;
    public Transform pointA;
    public Transform pointB;

    private Vector3 target;

    void Start()
    {
        target = pointB.position;
    }

    void Update()
    {
        // -------- 平台在两点之间移动 --------
        transform.position = Vector3.MoveTowards(
            transform.position,
            target,
            speed * Time.deltaTime
        );

        // -------- 到达目标点后切换方向 --------
        if (Vector3.Distance(transform.position, target) < 0.1f)
        {
            target = (target == pointA.position) ? pointB.position : pointA.position;
        }
    }
}
```

---

## 6.8 2D 背包拾取

```csharp
public class Collectible : MonoBehaviour
{
    public string itemName = "金币";
    public int value = 10;
    public AudioClip pickupSound;

    // -------- 触发检测（收集品设为 Is Trigger）--------
    void OnTriggerEnter2D(Collider2D col)
    {
        if (col.CompareTag("Player"))
        {
            // -------- 播放音效 --------
            if (pickupSound != null)
            {
                AudioSource.PlayClipAtPoint(pickupSound, transform.position);
            }

            // -------- 收集反馈 --------
            Debug.Log($"拾取了 {itemName}，价值 {value}");

            // -------- 销毁对象 --------
            Destroy(gameObject);
        }
    }
}
```

---

## 6.9 2D 相机设置

```
1. 选中 Main Camera
2. Projection → Orthographic（必须是这个）
3. Size = 5（视口高度为 10 单位）
4. Culling Mask：只渲染需要的层
```

### 像素完美相机

```csharp
// 适配像素游戏（像素风）
void Start()
{
    Camera.main.orthographicSize = Screen.height / 200f;  // 像素完美
}
```

---

## 6.10 动手练习 🧪

### 练习 1：2D 移动 + 翻转 ⭐
```
创建 2D 项目
用 Sprite Renderer 显示一个角色
WASD 或方向键控制左右移动
角色朝向移动方向（翻转）
```

### 练习 2：2D 跳跃 ⭐⭐
```
实现 2D 角色跳跃
地面检测（用 CircleCast）
空中不能二段跳
```

### 练习 3：2D 摄像机跟随 ⭐⭐
```
实现平滑相机跟随
限制相机不超出边界
```

### 练习 4：2D 收集金币 ⭐⭐⭐
```
创建金币预制体
玩家碰到金币时：播放音效 + 增加分数 + 金币消失
```

### 练习 5：2D 移动平台 ⭐⭐⭐
```
实现移动平台
玩家站在平台上时跟随移动
```

---

## 6.11 本章小结

```
✅ 已掌握：
├── 2D vs 3D 项目的核心区别
├── Sprite Renderer 设置（颜色、翻转、Order）
├── Physics 2D（重力、Rigidbody 2D、Collider 2D）
├── 2D 角色控制器（移动+跳跃+翻转）
├── Collision2D vs Trigger2D
├── 2D 摄像机跟随（平滑 + 边界）
├── 2D 平台搭建与碰撞矩阵
├── 收集品系统
└── Layer 碰撞配置

🔜 下章预告：
第七章：物理引擎与碰撞检测 —— 深入 Rigidbody、碰撞体、关节、力场，打造真实物理效果。
```

---

_📚 参考资料：《Unity 2D Game Development Cookbook》《Unity 官方文档 - Physics 2D》_
