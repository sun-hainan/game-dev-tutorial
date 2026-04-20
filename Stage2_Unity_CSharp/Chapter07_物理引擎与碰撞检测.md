# Chapter 07：物理引擎与碰撞检测

> 🎯 目标：深入理解 Unity 3D 物理引擎（Rigidbody、Collider、Joint、Force），掌握真实碰撞检测。

---

## 7.1 Unity 物理引擎概述

### 3D 物理系统组件

| 组件 | 作用 |
|-----|------|
| **Rigidbody** | 让对象受物理引擎控制（重力、力、碰撞）|
| **Collider** | 定义对象的碰撞形状 |
| **Joint** | 连接两个对象 |
| **Character Controller** | 角色专用，不完全受物理控制 |

---

## 7.2 Rigidbody 详解

### 核心属性

```csharp
// -------- 获取 Rigidbody --------
Rigidbody rb = GetComponent<Rigidbody>();

// -------- 基本属性 --------
rb.mass = 5f;                     // 质量（影响力的效果）
rb.drag = 0.5f;                   // 移动阻力
rb.angularDrag = 0.5f;            // 旋转阻力
rb.useGravity = true;              // 是否受重力
rb.isKinematic = false;           // true=不受物理控制（动画驱动）
rb.interpolation = RigidbodyInterpolation.Interpolate;  // 插值（防抖动）
rb.collisionDetectionMode = CollisionDetectionMode.Continuous;  // 高速碰撞检测
```

### 移动对象

```csharp
// -------- 方法1：直接设置速度（最常用）--------
rb.velocity = Vector3.forward * 5f;   // 向前移动，速度 5

// -------- 方法2：AddForce（施加力）--------
rb.AddForce(Vector3.up * 10f, ForceMode.Impulse);   // 跳
// ForceMode.Force：持续力（F=ma）
// ForceMode.Impulse：瞬时力（冲量）
// ForceMode.Acceleration：忽略质量的加速
// ForceMode.VelocityChange：忽略质量的瞬时速度改变

// -------- 方法3：MovePosition（平滑移动）--------
rb.MovePosition(rb.position + Vector3.forward * speed * Time.deltaTime);

// -------- 方法4：velocity（直接设置）--------
rb.velocity = new Vector3(h * speed, rb.velocity.y, v * speed);

// -------- 停止对象 --------
rb.velocity = Vector3.zero;
rb.angularVelocity = Vector3.zero;   // 停止旋转
```

### 旋转对象

```csharp
// -------- 施加扭矩 --------
rb.AddTorque(Vector3.up * 10f);

// -------- 看向目标（瞬间）--------
transform.LookAt(target.position);

// -------- 平滑朝向（插值）--------
Vector3 direction = (target.position - transform.position).normalized;
Quaternion targetRotation = Quaternion.LookRotation(direction);
transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, 5f * Time.deltaTime);
```

---

## 7.3 Collider 碰撞体

### 碰撞体类型

| 碰撞体 | 适用场景 |
|-------|---------|
| **Box Collider** | 箱子、房间、门 |
| **Sphere Collider** | 球形物体 |
| **Capsule Collider** | 角色（垂直）|
| **Mesh Collider** | 复杂形状（消耗高）|
| **Wheel Collider** | 车辆轮胎（专用）|

### Collider 属性

```csharp
// -------- 获取碰撞体 --------
Collider col = GetComponent<Collider>();

// -------- 常用属性 --------
col.isTrigger = false;        // 是否为触发器（穿透检测）
col.material = physicsMaterial;  // 物理材质
col.enabled = true;           // 启用/禁用碰撞

// -------- 调整碰撞体大小 --------
BoxCollider box = GetComponent<BoxCollider>();
box.center = new Vector3(0, 1, 0);   // 中心偏移
box.size = new Vector3(1, 2, 1);      // 大小
```

### 物理材质（Physic Material）

```
创建：Project 窗口右键 → Create → Physic Material
属性：
- Dynamic Friction：移动中的摩擦力（0-1）
- Static Friction：静止时的摩擦力
- Bounciness：弹性（0=不弹，1=完全弹性）
- Friction Combine：摩擦力组合模式（平均/最小/相乘）
```

---

## 7.4 碰撞检测（Collision）

### 碰撞响应函数

```csharp
// -------- 碰撞进入 --------
void OnCollisionEnter(Collision collision)
{
    Debug.Log($"碰撞开始：{collision.gameObject.name}");

    // -------- 获取碰撞信息 --------
    ContactPoint contact = collision.contacts[0];   // 第一个接触点
    Vector3 point = contact.point;                  // 接触位置
    Vector3 normal = contact.normal;               // 接触面法线
    Vector3 relVelocity = collision.relativeVelocity; // 相对速度

    Debug.Log($"接触点：{point}，法线：{normal}");
}

// -------- 碰撞中（每帧）--------
void OnCollisionStay(Collision collision)
{
    Debug.Log($"碰撞中...");
}

// -------- 碰撞结束 --------
void OnCollisionExit(Collision collision)
{
    Debug.Log($"碰撞结束");
}
```

### 碰撞过滤（Layer Collision Matrix）

```
设置路径：Edit → Project Settings → Physics
Layer Collision Matrix：
- 控制哪些 Layer 之间可以碰撞
- 更高效，不用代码判断
```

```csharp
// -------- 代码中检测 Layer --------
if (collision.gameObject.layer == LayerMask.NameToLayer("Enemy"))
{
    Debug.Log("撞到敌人！");
}
```

---

## 7.5 触发检测（Trigger）

### 触发 vs 碰撞

| 类型 | 物理效果 | 响应函数 | 适用场景 |
|-----|---------|---------|---------|
| **碰撞（Collision）** | 有物理反弹 | OnCollisionEnter | 地面、墙壁 |
| **触发（Trigger）** | 穿透，不反弹 | OnTriggerEnter | 传送门、道具 |

### 触发器使用

```csharp
// -------- 触发进入 --------
void OnTriggerEnter(Collider other)
{
    Debug.Log($"触发进入：{other.name}");

    // -------- 检测标签 --------
    if (other.CompareTag("Coin"))
    {
        Destroy(other.gameObject);  // 拾取金币
    }

    // -------- 检测层级 --------
    if (other.gameObject.layer == LayerMask.NameToLayer("Collectible"))
    {
        // 收集
    }
}

// -------- 触发中（持续）--------
void OnTriggerStay(Collider other)
{
    // 比如：站在治疗区域内持续回血
}

// -------- 触发离开 --------
void OnTriggerExit(Collider other)
{
    Debug.Log($"触发离开：{other.name}");
}
```

---

## 7.6 Character Controller

### 特点

> **Character Controller = 不完全受物理控制，适合玩家角色。**

```csharp
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class CharacterControllerDemo : MonoBehaviour
{
    public float moveSpeed = 5f;
    public float jumpSpeed = 8f;
    public float gravity = -20f;

    private CharacterController controller;
    private Vector3 velocity;

    void Start()
    {
        controller = GetComponent<CharacterController>();
    }

    void Update()
    {
        // -------- 地面检测 --------
        bool isGrounded = controller.isGrounded;

        // -------- 水平移动 --------
        float h = Input.GetAxis("Horizontal");
        float v = Input.GetAxis("Vertical");
        Vector3 move = transform.right * h + transform.forward * v;
        controller.Move(move * moveSpeed * Time.deltaTime);

        // -------- 跳跃 --------
        if (Input.GetButton("Jump") && isGrounded)
        {
            velocity.y = jumpSpeed;
        }

        // -------- 重力 --------
        if (!isGrounded)
        {
            velocity.y += gravity * Time.deltaTime;
        }

        controller.Move(velocity * Time.deltaTime);
    }
}
```

### Character Controller vs Rigidbody

| 维度 | Character Controller | Rigidbody |
|-----|---------------------|-----------|
| 爬坡 | 自动处理 | 需要配置 |
| 碰撞 | 胶囊形状 | 各种形状 |
| 推力/爆炸 | 无法被推动 | 会被推开 |
| 代码复杂度 | 低 | 中 |
| 性能 | 较高 | 较低 |

---

## 7.7 关节（Joint）

### Hinge Joint（铰链关节）

```csharp
// -------- 把两个对象用铰链连接 --------
HingeJoint hinge = gameObject.AddComponent<HingeJoint>();
hinge.connectedBody = otherRigidbody;   // 连接的对象
hinge.anchor = Vector3.zero;            // 本地锚点
hinge.axis = Vector3.up;                // 旋转轴

// -------- 限制角度 --------
JointLimits limits = hinge.limits;
limits.min = -45;
limits.max = 45;
hinge.limits = limits;
hinge.useLimits = true;

// -------- 弹簧（反弹效果）--------
JointSpring spring = hinge.spring;
spring.spring = 100;
spring.damper = 10;
hinge.spring = spring;
hinge.useSpring = true;
```

### Spring Joint（弹簧关节）

```csharp
SpringJoint spring = gameObject.AddComponent<SpringJoint>();
spring.connectedBody = connectedRB;
spring.anchor = Vector3.zero;
spring.connectedAnchor = Vector3.zero;
spring.spring = 50f;        // 弹力强度
spring.damper = 5f;        // 阻尼
spring.minDistance = 2f;   // 最小距离
spring.maxDistance = 5f;   // 最大距离
```

### 应用场景

```
铰链关节：门、杠杆、钟摆
弹簧关节：弹弓、悬挂系统
固定关节：粘性连接
相对关节：两个对象保持相对位置
```

---

## 7.8 恒力与爆炸

```csharp
public class Explosion : MonoBehaviour
{
    public float radius = 5f;
    public float force = 1000f;

    void Explode()
    {
        // -------- 获取范围内的所有碰撞体 --------
        Collider[] colliders = Physics.OverlapSphere(transform.position, radius);

        foreach (Collider col in colliders)
        {
            Rigidbody rb = col.GetComponent<Rigidbody>();
            if (rb != null && !rb.isKinematic)
            {
                // -------- 施加速度 --------
                Vector3 direction = (col.transform.position - transform.position).normalized;
                float distance = Vector3.Distance(transform.position, col.transform.position);
                float falloff = 1f - (distance / radius);  // 距离衰减

                rb.AddForce(direction * force * falloff, ForceMode.Impulse);
            }
        }
    }
}
```

---

## 7.9 Raycast（射线检测）

### 什么是 Raycast？

> **射线检测 = 从起点发出射线，检查击中了什么。** 用于 FPS 射击、视线检测、地面检测。

### 基本用法

```csharp
void Update()
{
    // -------- 向前发射射线 --------
    Ray ray = new Ray(transform.position, transform.forward);
    Debug.DrawRay(transform.position, transform.forward * 10f, Color.red);   // Scene 中显示射线

    if (Physics.Raycast(ray, out RaycastHit hit, 10f))
    {
        Debug.Log($"击中：{hit.collider.name}，距离：{hit.distance}");
        Debug.Log($"击中位置：{hit.point}");
        Debug.Log($"击中法线：{hit.normal}");
        Debug.Log($"击中对象材质：{hit.collider.material}");

        // -------- 击中特效 --------
        // Instantiate(hitEffect, hit.point, Quaternion.LookRotation(hit.normal));
    }
}
```

### 常见用法

```csharp
// -------- 从相机发射到鼠标位置 --------
Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);

// -------- 球形射线（适合检测范围内）--------
Collider[] hits = Physics.SphereCastAll(transform.position, 2f, transform.forward, 10f);

// -------- 从地面向下检测 --------
if (Physics.Raycast(transform.position + Vector3.up * 0.1f, Vector3.down, out RaycastHit groundHit, 2f))
{
    Debug.Log($"地面距离：{groundHit.distance}");
}
```

### LayerMask 过滤

```csharp
// -------- 只检测特定层 --------
LayerMask mask = LayerMask.GetMask("Enemy", "Environment");

if (Physics.Raycast(ray, out RaycastHit hit, 100f, mask))
{
    // 只检测 Enemy 和 Environment 层
}

// -------- 忽略特定层 --------
LayerMask mask = ~LayerMask.GetMask("Player");  // ~ 表示取反
```

---

## 7.10 刚体睡眠

```csharp
// -------- 性能优化：让刚体进入睡眠状态 --------
void OptimizePhysics()
{
    Rigidbody rb = GetComponent<Rigidbody>();

    // 让刚体进入睡眠（不再计算物理）
    rb.Sleep();

    // 唤醒刚体
    rb.WakeUp();

    // 检查是否在睡眠
    if (rb.IsSleeping())
    {
        Debug.Log("刚体在睡眠，节省性能");
    }
}
```

---

## 7.11 动手练习 🧪

### 练习 1：FPS 射击 ⭐
```
射线从相机发射，朝鼠标点击方向
击中物体时播放粒子特效
显示击中点的距离
```

### 练习 2：推箱子 ⭐⭐
```
3D 项目
玩家可以推动箱子
箱子受重力影响，可以掉下平台
```

### 练习 3：传送门 ⭐⭐
```
两个传送门（A 和 B）
玩家进入 A，立刻从 B 出来
用 Trigger 实现
```

### 练习 4：弹簧门 ⭐⭐
```
用 Hinge Joint 连接门板和门框
玩家靠近时门自动打开
用 Spring 实现平滑关闭
```

### 练习 5：爆炸波 ⭐⭐⭐
```
点击鼠标时产生爆炸
范围内的所有刚体被炸飞
力的大小随距离衰减
```

---

## 7.12 本章小结

```
✅ 已掌握：
├── Rigidbody 属性与移动（velocity/AddForce/MovePosition）
├── Collider 类型（Box/Sphere/Capsule/Mesh）
├── 物理材质（Physic Material）
├── Collision 碰撞响应
├── Trigger 触发器响应
├── Character Controller vs Rigidbody
├── Joint 关节（Hinge/Spring）
├── 射线检测 Raycast
├── OverlapSphere 范围检测
├── 恒力与爆炸效果
└── 刚体睡眠优化

🔜 下章预告：
第八章：UI 系统（Canvas 和 UGUI）—— 血条、背包、分数显示、按钮、动画菜单。
```

---

_📚 参考资料：《Unity 官方文档 - Physics》《Game Physics》_
