# Chapter 13：动画系统（Animator）

> 🎯 目标：掌握 Unity Animator 状态机、动画参数、角色动画控制。

---

## 13.1 Animator 组件

```
组成：
- Animator：挂在 GameObject 上
- Animator Controller：状态机资产（.controller 文件）
- Animation Clips：动画片段（.anim 文件）
- Avatar：骨骼映射（3D 动画用）
```

### Animator Controller 创建

```
1. Project → Create → Animator Controller
2. 双击打开Animator窗口
3. 右键空白 → Create State → New State
4. 把 Animation Clip 拖到 State 上
5. 拖入 GameObject 的 Animator 组件的 Controller 字段
```

---

## 13.2 动画参数

| 类型 | 说明 |
|-----|------|
| **Float** | 速度、方向等连续值 |
| **Int** | 状态 ID |
| **Bool** | 是否在播放、是否跳跃等开关 |
| **Trigger** | 一次性触发（自动归零）|

### 代码控制参数

```csharp
Animator animator;

void Start()
{
    animator = GetComponent<Animator>();
}

// -------- 设置 Bool --------
animator.SetBool("IsRunning", true);
animator.SetBool("IsJumping", false);

// -------- 设置 Float --------
animator.SetFloat("Speed", 5f);

// -------- 设置 Int --------
animator.SetInteger("State", 1);

// -------- 设置 Trigger（一次性触发）--------
animator.SetTrigger("Attack");
animator.SetTrigger("Jump");
```

### 动画过渡条件

```
在 Animator 窗口：
1. 选择两个 State 之间的 Transition（箭头）
2. Conditions 添加条件：
   - 例如：IsJumping == true
   - 例如：Speed > 0.1
3. Has Exit Time：是否等待动画播放完再切换
```

---

## 13.3 完整角色动画控制器

### 2D 混合树

```csharp
// -------- 2D 混合树控制 --------
void Update()
{
    float h = Input.GetAxisRaw("Horizontal");
    float v = Input.GetAxisRaw("Vertical");

    animator.SetFloat("Horizontal", h);
    animator.SetFloat("Vertical", v);

    // -------- 速度混合 --------
    float speed = Mathf.Abs(h) + Mathf.Abs(v);
    animator.SetFloat("Speed", speed);

    // -------- 方向 --------
    if (h > 0) transform.localScale = new Vector3(1, 1, 1);
    else if (h < 0) transform.localScale = new Vector3(-1, 1, 1);
}
```

### 跳跃动画

```csharp
bool isGrounded;

void Update()
{
    if (Input.GetButtonDown("Jump") && isGrounded)
    {
        animator.SetTrigger("Jump");
    }

    animator.SetBool("IsGrounded", isGrounded);
}

void OnCollisionEnter(Collision col)
{
    if (col.gameObject.CompareTag("Ground"))
    {
        isGrounded = true;
    }
}
```

---

## 13.4 攻击动画事件

```csharp
// -------- 在动画中调用代码 --------
public class PlayerAttack : MonoBehaviour
{
    public void OnAttackHit()
    {
        // -------- 动画事件：在 Attack 动画的第 0.3 秒处调用 --------
        Debug.Log("攻击命中！");
        // 实例化特效
        // 检测范围内敌人
    }
}
```

### 在 Animation Clip 中添加事件

```
1. 在 Project 中选中 Animation Clip
2. 在 Inspector 中点击 Edit
3. 在动画时间线上右键 → Add Animation Event
4. 把时间线拖到要触发的地方
5. Function 填写 "OnAttackHit"
```

---

## 13.5 章节小结

```
✅ 已掌握：
├── Animator 组件和 Controller
├── Animation State Machine（状态机）
├── 动画参数（Float/Bool/Int/Trigger）
├── 代码控制动画参数
├── 2D 混合树
├── 动画过渡条件
├── 动画事件（Animation Event）
└── 角色动画控制器实战

🔜 下章预告：
第十四章：粒子系统 —— 特效制作，火花、爆炸、烟雾、流水效果。
```

---

_📚 参考资料：《Unity 官方文档 - Animator》_
