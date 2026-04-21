# Chapter 14：粒子系统

> 🎯 目标：掌握 Unity Particle System 组件，能用代码控制粒子特效，完成烟花、爆炸、烟雾等常见特效制作。

---

## 14.1 生活类比：粒子系统是什么？

想象一下这些场景：

| 场景 | 粒子对应 |
|-----|---------|
| 烟花绽放 | 成百上千个小光点向外扩散 |
| 喷泉流水 | 水珠从喷口射出，受重力下落 |
| 香烟烟雾 | 细小颗粒随风飘散，逐渐消散 |
| 爆炸火光 | 高温粒子快速扩散并变暗消失 |
| 落叶飘零 | 叶子从树上飘落，轻微摆动 |

> **核心思想**：把**大量独立的小元素**放在一起，用统计规律模拟自然现象。单个粒子很简单，但成千上万个组合起来，就能骗过眼睛，产生真实感。

---

## 14.2 粒子系统的核心组件

Unity 的粒子系统（Particle System）由多个**模块（Module）**组成，每个模块控制一个方面的行为：

```
ParticleSystem
├── Main Module（主模块）—— 生命周期、速度、大小、颜色
├── Emission（发射模块）—— 发射速率、爆发
├── Shape（形状模块）—— 发射器形状：球/锥/盒/边/线
├── Velocity Over Lifetime（生命周期速度）—— 速度随时间变化
├── Color Over Lifetime（生命周期颜色）—— 颜色渐变
├── Size Over Lifetime（生命周期大小）—— 大小曲线
├── Renderer（渲染模块）—— 材质、渲染模式
└── ... 更多模块
```

---

## 14.3 粒子系统架构（ASCII 图解）

```
┌─────────────────────────────────────────────────────┐
│                  Particle System                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐      │
│  │  Shape   │───→│ Emission │───→│   Main   │      │
│  │  形状    │    │  发射器   │    │  主模块   │      │
│  └──────────┘    └──────────┘    └──────────┘      │
│       │               │               │            │
│       ↓               ↓               ↓            │
│  确定粒子从       确定发射频率       控制基础属性：   │
│  什么形状区域       和爆发数量         速度/大小/     │
│  发射出来                                颜色/生命周期 │
│                                             │        │
│                                             ↓        │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐      │
│  │ Velocity │    │  Color   │    │  Size    │      │
│  │ 速度变化 │    │ 颜色变化 │    │ 大小变化  │      │
│  └──────────┘    └──────────┘    └──────────┘      │
│       │               │               │            │
│       └───────────────┴───────────────┘            │
│                       │                            │
│                       ↓                            │
│               ┌──────────────┐                     │
│               │   Renderer   │                     │
│               │   渲染模块    │                     │
│               └──────────────┘                     │
│                       │                            │
│                       ↓                            │
│              屏幕上的可见粒子效果                    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 14.4 生命周期曲线（ASCII 图解）

```
粒子生命周期曲线
────────────────────────────→ 时间

  颜色随生命周期变化（Color Over Lifetime）：

  Color
   ↑
   │■ ■ ■ ■ ■ ■ ■□  ← 红色 → 黄色 → 透明消失
   │  红    黄    无
   │
   └────────────────→ t
   0    0.5    1.0


  大小随生命周期变化（Size Over Lifetime）：

  Size
   ↑
   │    ■■■■■■      ← 小 → 大 → 消失
   │
   └────────────────→ t
   0    0.5    1.0


  速度随生命周期变化（Velocity Over Lifetime）：

  Speed
   ↑
   │■■■■□          ← 快 → 慢（摩擦阻力）
   │
   └────────────────→ t
   0    0.5    1.0
```

---

## 14.5 代码创建粒子系统

### 从零创建粒子系统

```csharp
// ========== 创建粒子系统 ==========
using UnityEngine;

public class ParticleSystemCreator : MonoBehaviour
{
    void Start()
    {
        // 第1步：创建 GameObject 并添加 ParticleSystem 组件
        GameObject obj = new GameObject("MyParticleSystem");
        ParticleSystem ps = obj.AddComponent<ParticleSystem>();

        // 第2步：配置主模块
        // startColor：粒子的初始颜色（白色 = 无色染色）
        var main = ps.main;
        main.startColor = new ParticleSystem.MinMaxGradient(
            Color.white,   // 最小颜色
            new Color(1f, 1f, 0.5f, 1f)  // 最大颜色（偏黄）
        );
        main.startSize = 0.1f;        // 初始大小 0.1 单位
        main.startLifetime = 2f;       // 生命周期 2 秒
        main.startSpeed = 5f;          // 初始速度 5 单位/秒
        main.simulationSpace = ParticleSystemSimulationSpace.World;  // 世界坐标系
        main.maxParticles = 1000;      // 最大粒子数量上限

        // 第3步：配置 Emission（发射）模块
        // rate：每秒发射数量
        var emission = ps.emission;
        emission.enabled = true;
        emission.rateOverTime = 20f;   // 每秒发射 20 个

        // Burst：一次性爆发
        // 参数1：何时爆发（时间点，秒）
        // 参数2：爆发生命周期索引
        // 参数3：爆发的粒子数量
        // 参数4：爆发的周期（重复间隔）
        // 参数5：重复次数
        emission.SetBurst(0, new ParticleSystem.Burst(0f, 10, 1, 0.5f, 3));

        // 第4步：配置 Shape（形状）模块
        // 决定粒子从什么形状的区域发射
        var shape = ps.shape;
        shape.enabled = true;
        shape.shapeType = ParticleSystemShapeType.Cone;  // 锥形
        shape.angle = 30f;       // 锥角 30 度（扩散范围）
        shape.radius = 0.5f;      // 锥底半径
        shape.rotation = new Vector3(-90f, 0f, 0f);  // 旋转朝上（默认朝下）

        // 第5步：配置 Renderer（渲染）模块
        // 需要指定材质才能显示粒子
        var renderer = ps.GetComponent<ParticleSystemRenderer>();
        renderer.material = new Material(Shader.Find("Particles/Standard Unlit")); // 内置粒子材质

        print("粒子系统已创建！");
    }
}
```

---

## 14.6 形状发射器详解

### 各种形状对比

```csharp
// ========== Shape 模块各种形状 ==========
var shape = ps.shape;

// 球形（Sphere）：粒子从球面或球内发射
shape.shapeType = ParticleSystemShapeType.Sphere;
shape.radius = 1f;           // 球半径
shape.radiusThickness = 0f;  // 0=仅表面发射，1=整个球内发射

// 半球形（Hemisphere）：上半球发射
shape.shapeType = ParticleSystemShapeType.Hemisphere;
shape.radius = 1f;

// 圆锥形（Cone）：从锥底圆面发射，向锥尖方向扩散
shape.shapeType = ParticleSystemShapeType.Cone;
shape.angle = 25f;           // 锥角（越小越集中，越大越扩散）
shape.radius = 0.5f;         // 锥底半径

// 盒形（Box）：从立方体表面或内部发射
shape.shapeType = ParticleSystemShapeType.Box;
shape.scale = new Vector3(2f, 1f, 1f);  // 盒子尺寸

// 边缘（Edge）：从线段发射
shape.shapeType = ParticleSystemShapeType.Edge;
shape.radius = 2f;  // 线段长度

// 网格（Mesh）：从 3D 模型表面发射
shape.shapeType = ParticleSystemShapeType.Mesh;
shape.mesh = someMeshFilter.mesh;
```

---

## 14.7 颜色随生命周期变化

### 完整渐变色配置

```csharp
// ========== Color Over Lifetime 模块 ==========
var colorOverLifetime = ps.colorOverLifetime;
colorOverLifetime.enabled = true;

// 创建渐变（Gradient）：控制颜色和透明度
// 渐变编辑器：
//  - 顶部色块：颜色（Color Keys）
//  - 底部滑块：透明度（Alpha Keys）

Gradient gradient = new Gradient();

// 第1个参数：颜色关键点数组
// 每个关键点 = 颜色 + 时间位置（0~1）
// 第2个参数：透明度关键点数组
// 每个关键点 = 透明度(0~1) + 时间位置
gradient.SetKeys(
    new GradientColorKey[] {
        new GradientColorKey(Color.white, 0.0f),   // 开始：白色
        new GradientColorKey(Color.yellow, 0.3f),  // 30%：黄色
        new GradientColorKey(Color.red, 0.7f),     // 70%：红色
        new GradientColorKey(new Color(0.2f, 0.2f, 0.2f), 1.0f)  // 结束：暗红
    },
    new GradientAlphaKey[] {
        new GradientAlphaKey(1.0f, 0.0f),  // 开始：完全不透明
        new GradientAlphaKey(0.8f, 0.5f),  // 中间：80% 透明
        new GradientAlphaKey(0.0f, 1.0f)   // 结束：完全透明（消失）
    }
);

colorOverLifetime.color = gradient;
```

---

## 14.8 粒子贴图和材质

### 使用自定义贴图

```csharp
// ========== 粒子材质配置 ==========
// 方案1：使用 Unity 内置粒子材质
ParticleSystemRenderer renderer = ps.GetComponent<ParticleSystemRenderer>();
renderer.material = new Material(Shader.Find("Particles/Standard Unlit"));

// 方案2：从外部加载自定义粒子贴图
public class ParticleWithTexture : MonoBehaviour
{
    public Texture2D particleTexture;  // 在 Inspector 中拖入
    public ParticleSystem ps;

    void Start()
    {
        // 创建使用自定义贴图的材质
        Material mat = new Material(Shader.Find("Particles/Standard Unlit"));
        mat.SetTexture("_MainTex", particleTexture);

        ParticleSystemRenderer renderer = ps.GetComponent<ParticleSystemRenderer>();
        renderer.material = mat;
        renderer.renderMode = ParticleSystemRenderMode.Billboard;  //  billboard 模式：始终朝向相机

        // 设置缩放：用贴图的哪个部分
        // Stretch：拉伸模式，用贴图覆盖粒子轨迹
        renderer.renderMode = ParticleSystemRenderMode.Stretch;
        renderer.velocityStretchedFacing = 0.1f;   // 速度越大拉伸越长
        renderer.velocityStretchedBillboard = 0.5f;
    }
}

// 方案3：常用的粒子 Shader 类型
// - Particles/Standard Unlit：不受光照影响，适合特效
// - Particles/Standard：受光照影响
// - Mobile/Particles/Additive：移动端加法混合（发光效果）
// - Mobile/Particles/Alpha Blended：移动端透明度混合
```

---

## 14.9 实战：爆炸特效

```csharp
// ========== 爆炸特效组件 ==========
using UnityEngine;

public class ExplosionEffect : MonoBehaviour
{
    // 爆炸粒子预制体（在 Inspector 中拖入）
    public ParticleSystem explosionPrefab;

    // ========== 触发爆炸 ==========
    public void TriggerExplosion(Vector3 position)
    {
        // 第1步：实例化粒子预制体
        ParticleSystem ps = Instantiate(
            explosionPrefab,
            position,
            Quaternion.identity  // 无旋转
        );

        // 第2步：配置主模块——快速爆发，快速消散
        var main = ps.main;
        main.startColor = new ParticleSystem.MinMaxGradient(
            Color.yellow,
            Color.red
        );
        main.startSize = 0.2f;
        main.startLifetime = 0.8f;   // 生命周期短，爆炸要快
        main.startSpeed = 8f;        // 速度快，扩散快

        // 第3步：配置爆发（不用 rateOverTime，全靠 burst）
        var emission = ps.emission;
        emission.rateOverTime = 0f;  // 关闭持续发射
        emission.SetBurst(0, new ParticleSystem.Burst(0f, 50));  // 一次性爆发 50 个

        // 第4步：配置形状——球形向外爆炸
        var shape = ps.shape;
        shape.shapeType = ParticleSystemShapeType.Sphere;
        shape.radius = 0.1f;         // 小发射源

        // 第5步：速度随生命周期衰减
        var velocity = ps.velocityOverLifetime;
        velocity.enabled = true;
        velocity.y = 2f;             // 轻微向上飘（模拟火光的升腾）

        // 第6步：大小随生命周期缩小（爆炸扩散后变小）
        var size = ps.sizeOverLifetime;
        size.enabled = true;
        size.size = new ParticleSystem.MinMaxCurve(
            1f,                      // 初始大小倍数
            new AnimationCurve(      // 大小曲线
                new Keyframe(0f, 1f),    // 开始：正常大小
                new Keyframe(0.5f, 1.5f), // 中间：略微膨胀
                new Keyframe(1f, 0f)      // 结束：完全消失
            )
        );

        // 第7步：颜色渐变——黄 → 红 → 黑（烟）
        var color = ps.colorOverLifetime;
        color.enabled = true;
        Gradient explosionGradient = new Gradient();
        explosionGradient.SetKeys(
            new GradientColorKey[] {
                new GradientColorKey(Color.yellow, 0f),
                new GradientColorKey(Color.red, 0.3f),
                new GradientColorKey(new Color(0.2f, 0.2f, 0.2f), 0.8f)
            },
            new GradientAlphaKey[] {
                new GradientAlphaKey(1f, 0f),
                new GradientAlphaKey(0.5f, 0.5f),
                new GradientAlphaKey(0f, 1f)
            }
        );
        color.color = explosionGradient;

        // 第8步：播放并自动销毁
        ps.Play();  // 开始播放
        Destroy(ps.gameObject, main.startLifetime.constantMax + 0.5f);  // 等最后一个粒子消失后再销毁
    }

    void Update()
    {
        // 按空格键测试爆炸
        if (Input.GetKeyDown(KeyCode.Space))
        {
            TriggerExplosion(transform.position);
        }
    }
}
```

---

## 14.10 实战：地面尘土

```csharp
// ========== 地面尘土粒子 ==========
using UnityEngine;

[RequireComponent(typeof(Rigidbody))]
public class DustTrail : MonoBehaviour
{
    public ParticleSystem dustPS;  // 尘土粒子预制体
    private ParticleSystem.EmissionModule emission;
    private Rigidbody rb;

    void Start()
    {
        // 获取刚体组件
        rb = GetComponent<Rigidbody>();

        // 获取 Emission 模块的引用（性能优化：避免每帧访问）
        emission = dustPS.emission;
        emission.rateOverTime = 0f;  // 默认关闭发射
    }

    void Update()
    {
        // 第1步：计算移动速度（XZ 平面）
        float horizontalSpeed = Mathf.Abs(rb.velocity.x) + Mathf.Abs(rb.velocity.z);
        float verticalSpeed = Mathf.Abs(rb.velocity.y);

        // 第2步：根据速度控制尘土强度
        // speed > 0.1 才算在移动，避免静止时误触发
        if (horizontalSpeed > 0.1f)
        {
            // 速度越大尘土越多（插值平滑过渡）
            float dustIntensity = Mathf.Lerp(0f, 30f, horizontalSpeed / 10f);
            emission.rateOverTime = dustIntensity;
        }
        else
        {
            // 静止时不发射，但需要让已有的粒子消散
            emission.rateOverTime = 0f;
        }

        // 第3步：如果角色跳跃，添加跳跃尘土爆发
        if (verticalSpeed > 3f && !IsGrounded())
        {
            var burst = dustPS.emission;
            burst.SetBurst(0, new ParticleSystem.Burst(0f, 15));
            dustPS.Play();  // 触发 burst
        }
    }

    // 简单的地面检测
    bool IsGrounded()
    {
        return Physics.Raycast(transform.position, Vector3.down, 1.1f);
    }
}
```

---

## 14.11 常见错误

| 错误 | 原因 | 解决方法 |
|-----|------|---------|
| 粒子不显示 | 没设置材质或材质没 shader | 添加 ParticleSystem 材质 |
| 粒子一闪就消失 | `startLifetime` 太短 | 增大 startLifetime |
| 粒子堆积在一个点 | Shape 的 radius 太小 | 增大 shape.radius |
| 粒子数量极少 | Emission rate 太低 | 增大 emission.rateOverTime |
| 粒子颜色没变化 | 没开启 Color Over Lifetime | `colorOverLifetime.enabled = true` |
| 粒子穿过地面 | 没加 Collision 模块 | 添加 Collision 模块并设置 Plane |
| 性能卡顿 | 粒子数量过多（>10000） | 减少 maxParticles，降低发射率 |
| Burst 不触发 | `rateOverTime > 0` 且 burst 索引不对 | 设置 `rateOverTime = 0` 并正确设置 Burst 的 time 参数 |

---

## 14.12 章节小结

```
✅ 已掌握：
├── 粒子系统概念：大量小元素的统计效果
├── ParticleSystem 核心模块架构
├── 主模块（startColor/size/lifetime/speed）
├── Emission（发射率和 Burst 爆发）
├── Shape（形状：球/锥/盒/半球/边缘）
├── Velocity Over Lifetime（速度曲线）
├── Color Over Lifetime（渐变颜色）
├── Size Over Lifetime（大小曲线）
├── Renderer 模块（材质、贴图、渲染模式）
├── 代码创建和配置粒子系统
├── 爆炸特效实战
├── 尘土拖尾特效实战
└── 常见错误排查

🔜 下章预告：
第十五章：完整 2D 游戏项目 —— 从零开始，做一个完整的可运行游戏！
```

---

_📚 参考资料：《Unity 官方文档 - Particle System》_
