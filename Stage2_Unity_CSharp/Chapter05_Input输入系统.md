# Chapter 05：Input 输入系统

> 🎯 目标：掌握 Unity 的输入系统——键盘、鼠标、触摸、手柄，支持 PC 和移动端。

---

## 5.1 Unity Input 系统概述

### 生活中的类比 🎮

> **输入 = 你的手柄/键盘。** 告诉游戏"玩家想做什么"。

### Unity Input 发展历程

| 系统 | 说明 | 推荐度 |
|-----|------|-------|
| **Input Manager** | 传统方式，Edit → Project Settings → Input | ⭐ 了解即可 |
| **Input.GetAxis/GetButton** | 基于 Input Manager 的封装 | ⭐⭐ |
| **New Input System** | Unity 2019+ 官方推荐 | ⭐⭐⭐⭐⭐ |

> 本章先讲传统 Input（GetAxis/GetButton），下一节讲 New Input System。

---

## 5.2 键盘输入

### GetKey / GetKeyDown / GetKeyUp

```csharp
void Update()
{
    // -------- 按住键持续触发 --------
    if (Input.GetKey(KeyCode.W))
    {
        Debug.Log("W 键按住中...");
    }

    // -------- 按下的那一帧触发 --------
    if (Input.GetKeyDown(KeyCode.Space))
    {
        Debug.Log("空格键按下！");
        // 适合：跳跃、开火、确认
    }

    // -------- 抬起的那一帧触发 --------
    if (Input.GetKeyUp(KeyCode.E))
    {
        Debug.Log("E 键抬起！");
        // 适合：停止冲刺、取消
    }
}
```

### 常用键码

```csharp
// 字母键
Input.GetKey(KeyCode.A)
Input.GetKey(KeyCode.W)
Input.GetKey(KeyCode.Alpha1)    // 数字 1

// 功能键
Input.GetKey(KeyCode.F1)
Input.GetKey(KeyCode.Escape)    // ESC
Input.GetKey(KeyCode.Return)     // 回车

// 特殊键
Input.GetKey(KeyCode.LeftShift)
Input.GetKey(KeyCode.Tab)
```

---

## 5.3 鼠标输入

### 鼠标状态

```csharp
void Update()
{
    // -------- 鼠标位置（屏幕像素坐标，原点左下角）--------
    Vector3 mousePos = Input.mousePosition;
    Debug.Log($"鼠标位置：{mousePos}");

    // -------- 鼠标按钮 --------
    if (Input.GetMouseButton(0))      // 0=左键, 1=右键, 2=中键
    {
        Debug.Log("左键按住");
    }
    if (Input.GetMouseButtonDown(0))  // 按下那一帧
    {
        Debug.Log("左键点击！");
    }
    if (Input.GetMouseButtonUp(0))    // 抬起那一帧
    {
        Debug.Log("左键抬起！");
    }

    // -------- 鼠标滚轮 --------
    float scroll = Input.GetAxis("Mouse ScrollWheel");
    if (scroll > 0f)
    {
        Debug.Log("滚轮向上，缩放");
    }
    if (scroll < 0f)
    {
        Debug.Log("滚轮向下，缩放");
    }
}
```

### 鼠标位置转换为世界坐标

```csharp
public class MouseToWorld : MonoBehaviour
{
    void Update()
    {
        // -------- 方法1：屏幕坐标 → 世界坐标 --------
        Vector3 screenPos = Input.mousePosition;
        screenPos.z = 10f;   // 相机到平面的距离
        Vector3 worldPos = Camera.main.ScreenToWorldPoint(screenPos);
        Debug.Log($"世界坐标：{worldPos}");

        // -------- 方法2：射线检测（更精确）--------
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit))
        {
            Debug.Log($"射线击中：{hit.collider.name}，位置：{hit.point}");
        }
    }
}
```

---

## 5.4 Input.GetAxis（轴输入）

### 虚拟轴概念

> **GetAxis = 带平滑的模拟输入，返回 -1 ~ 1 之间的浮点数。**

```csharp
void Update()
{
    // -------- 水平轴：←/→ 或 A/D --------
    float h = Input.GetAxis("Horizontal");
    Debug.Log($"水平：{h}");   // ← 为负，→ 为正，中间有平滑

    // -------- 垂直轴：↑/↓ 或 W/S --------
    float v = Input.GetAxis("Vertical");
    Debug.Log($"垂直：{v}");   // ↓ 为负，↑ 为正

    // -------- 鼠标 X --------
    float mouseX = Input.GetAxis("Mouse X");
    Debug.Log($"鼠标X：{mouseX}");

    // -------- 鼠标 Y --------
    float mouseY = Input.GetAxis("Mouse Y");
    Debug.Log($"鼠标Y：{mouseY}");
}
```

### 自定义轴（Edit → Project Settings → Input Manager）

```
每个轴包含：
- Name：轴名称（代码中用这个名字）
- Positive Button：正向按键（如 D）
- Negative Button：负向按键（如 A）
- Gravity：松开时回中速度
- Sensitivity：按键响应灵敏度
```

---

## 5.5 手柄（游戏手柄）

```csharp
void Update()
{
    // -------- 手柄按钮 --------
    if (Input.GetButtonDown("Fire1"))     // 手柄 A 键
    {
        Debug.Log("手柄 A 按下");
    }

    if (Input.GetButton("Fire2"))         // 手柄 B 键
    {
        Debug.Log("手柄 B 按住");
    }

    // -------- 手柄摇杆 --------
    float leftX = Input.GetAxis("Horizontal");   // 左摇杆 X
    float leftY = Input.GetAxis("Vertical");    // 左摇杆 Y

    float rightX = Input.GetAxis("Mouse X");   // 右摇杆 X
    float rightY = Input.GetAxis("Mouse Y");   // 右摇杆 Y

    // -------- 手柄振动（需要 XboxOneController 或类似）--------
    if (Input.GetButtonDown("Fire3"))
    {
        StartCoroutine(Vibrate(0.5f, 1f, 0.5f));
    }
}

IEnumerator Vibrate(float duration, float lowFreq, float highFreq)
{
    // Unity 5.x 开始手柄振动
    // Handheld.Vibrate() 仅对移动设备有效
    // PC 手柄需要插件或 Input System
    yield return new WaitForSeconds(duration);
}
```

---

## 5.6 New Input System（Unity 2019.4+）

### 为什么需要 New Input System？

| 传统 Input | New Input System |
|-----------|----------------|
| 只能在运行时配置 | 可在编辑器可视化配置 |
| PC/移动端分开处理 | 统一处理所有平台 |
| 不支持多点触控 | 支持多点触控 |
| 不支持手柄交互 | 原生支持手柄 |

### 安装 New Input System

```
1. Window → Package Manager
2. 搜索 "Input System"
3. 点击 Install
4. 弹窗提示切换，点击 Yes
5. 等待安装完成
```

### 代码使用

```csharp
// 需要在脚本顶部 using
using UnityEngine.InputSystem;

public class NewInputDemo : MonoBehaviour
{
    // -------- 创建 Input Action 资产后拖入 --------
    public InputActionAsset actions;

    private InputAction moveAction;
    private InputAction fireAction;

    void OnEnable()
    {
        // -------- 获取 Action --------
        moveAction = actions["Move"];     // Move 是你在 Input Actions 资产里定义的 Action 名称
        fireAction = actions["Fire"];

        // -------- 启用 --------
        moveAction.Enable();
        fireAction.Enable();

        // -------- 绑定回调 --------
        fireAction.performed += OnFire;
    }

    void OnDisable()
    {
        moveAction.Disable();
        fireAction.Disable();

        fireAction.performed -= OnFire;
    }

    void Update()
    {
        // -------- 读取 Move（每个 Update 轮询）--------
        Vector2 move = moveAction.ReadValue<Vector2>();
        Debug.Log($"移动：{move}");
    }

    void OnFire(InputAction.CallbackContext context)
    {
        Debug.Log("开火！");
        Debug.Log($"触发相位：{context.phase}");   // Started/Performed/Canceled
    }
}
```

### Input Actions 资产配置

```
1. Project 窗口右键 → Create → Input Actions
2. 双击打开可视化编辑器
3. 创建 Action Map（如 PlayerControls）
4. 在 Action Map 里创建 Actions（如 Move, Fire, Jump）
5. 为每个 Action 绑定按键/手柄/触摸
6. 把 .inputactions 资产拖到脚本的 InputActionAsset 字段上
```

---

## 5.7 完整玩家控制器

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("移动设置")]
    public float moveSpeed = 5f;
    public float jumpForce = 8f;

    [Header("组件引用")]
    private Rigidbody rb;
    private bool isGrounded = false;

    void Awake()
    {
        rb = GetComponent<Rigidbody>();
    }

    void Update()
    {
        // -------- 移动输入 --------
        float h = Input.GetAxisRaw("Horizontal");   // GetAxisRaw：无平滑，0 或 ±1
        float v = Input.GetAxisRaw("Vertical");

        Vector3 moveDir = new Vector3(h, 0, v).normalized;
        rb.velocity = new Vector3(moveDir.x * moveSpeed, rb.velocity.y, moveDir.z * moveSpeed);

        // -------- 跳跃 --------
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
            isGrounded = false;
        }

        // -------- 射击 --------
        if (Input.GetMouseButtonDown(0))
        {
            Shoot();
        }
    }

    // -------- 地面检测 --------
    void OnCollisionEnter(Collision col)
    {
        if (col.gameObject.CompareTag("Ground"))
        {
            isGrounded = true;
        }
    }

    void Shoot()
    {
        Debug.Log("射击！");
        // 实例化子弹...
    }
}
```

---

## 5.8 移动端触摸

```csharp
void Update()
{
    // -------- 单点触控 --------
    if (Input.touchCount > 0)
    {
        Touch touch = Input.GetTouch(0);

        switch (touch.phase)
        {
            case TouchPhase.Began:
                Debug.Log("触摸开始");
                break;
            case TouchPhase.Moved:
                Debug.Log($"移动：{touch.deltaPosition}");
                break;
            case TouchPhase.Stationary:
                Debug.Log("按住不动");
                break;
            case TouchPhase.Ended:
                Debug.Log("触摸结束");
                break;
            case TouchPhase.Canceled:
                Debug.Log("触摸取消");
                break;
        }
    }

    // -------- 多点触控 --------
    if (Input.touchCount == 2)
    {
        Touch touch1 = Input.GetTouch(0);
        Touch touch2 = Input.GetTouch(1);

        Vector2 delta1 = touch1.deltaPosition;
        Vector2 delta2 = touch2.deltaPosition;

        float pinch = (delta1.magnitude + delta2.magnitude) / 2f;
        Debug.Log($"捏合 pinch：{pinch}");
    }
}
```

---

## 5.9 动手练习 🧪

### 练习 1：WASD 移动 ⭐
```
用 WASD 控制 Cube 移动
速度 5，方向正确
```

### 练习 2：鼠标点击移动 ⭐⭐
```
点击屏幕，将 Cube 移动到点击位置
使用 Lerp 平滑移动
```

### 练习 3：虚拟摇杆（2D）⭐⭐
```
使用 New Input System
实现虚拟摇杆控制角色移动
```

### 练习 4：玩家控制器完整版 ⭐⭐⭐
```
WASD 移动 + 空格跳跃 + 鼠标左键射击
地面检测
限制不超出边界
```

---

## 5.10 本章小结

```
✅ 已掌握：
├── GetKey / GetKeyDown / GetKeyUp
├── 鼠标位置、按钮、滚轮
├── GetAxis（平滑）和 GetAxisRaw（无平滑）
├── 鼠标位置转世界坐标
├── 手柄摇杆输入
├── New Input System 安装和使用
├── 移动端触摸输入
└── 完整玩家控制器

🔜 下章预告：
第六章：2D 游戏开发基础 —— Sprite、Physics 2D、2D 角色控制、精灵动画。
```

---

_📚 参考资料：《Unity 官方文档 - Input》《New Input System 文档》_
