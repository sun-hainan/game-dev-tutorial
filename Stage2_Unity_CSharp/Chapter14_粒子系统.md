# Chapter 14：粒子系统

> 🎯 目标：掌握 Unity Particle System，制作游戏特效。

---

## 14.1 Particle System 基础

### 添加粒子系统

```
1. GameObject → Effects → Particle System
2. Inspector 中配置各种模块
```

### 核心模块

```csharp
// -------- 代码控制 Particle System --------
ParticleSystem ps = GetComponent<ParticleSystem>();

// -------- 主模块 --------
var main = ps.main;
main.startColor = Color.red;
main.startSize = 0.1f;
main.startLifetime = 2f;
main.startSpeed = 5f;
main.simulationSpace = ParticleSystemSimulationSpace.World;  // 世界空间

// -------- Emission（发射）--------
var emission = ps.emission;
emission.rateOverTime = 10f;    // 每秒发射 10 个
emission.SetBurst(0, new ParticleSystem.Burst(0, 10));  // 一次性爆发 10 个

// -------- Shape（形状）--------
var shape = ps.shape;
shape.shapeType = ParticleSystemShapeType.Cone;   // 锥形
shape.angle = 30f;      // 锥角
shape.radius = 0.5f;   // 半径

// -------- Velocity Over Lifetime（速度）--------
var velocity = ps.velocityOverLifetime;
velocity.y = 2f;   // 向上飘

// -------- Color Over Lifetime（颜色渐变）--------
var colorOverLifetime = ps.colorOverLifetime;
Gradient gradient = new Gradient();
gradient.SetKeys(
    new GradientColorKey[] { new GradientColorKey(Color.red, 0f), new GradientColorKey(Color.yellow, 1f) },
    new GradientAlphaKey[] { new GradientAlphaKey(1f, 0f), new GradientAlphaKey(0f, 1f) }
);
colorOverLifetime.color = gradient;

// -------- Size Over Lifetime（大小变化）--------
var sizeOverLifetime = ps.sizeOverLifetime;
sizeOverLifetime.size = new ParticleSystem.MinMaxCurve(1f, new AnimationCurve(
    new Keyframe(0f, 0.5f),
    new Keyframe(1f, 0f)
));
```

---

## 14.2 常用特效制作

### 爆炸特效

```csharp
public class ExplosionEffect : MonoBehaviour
{
    public ParticleSystem explosionPS;

    public void Explode(Vector3 position)
    {
        ParticleSystem ps = Instantiate(explosionPS, position, Quaternion.identity);
        Destroy(ps.gameObject, 3f);   // 3 秒后销毁
    }
}
```

### 地面尘土

```csharp
// 在玩家移动时播放尘土粒子
public class DustTrail : MonoBehaviour
{
    public ParticleSystem dustPS;
    private ParticleSystem.EmissionModule emission;

    void Start()
    {
        emission = dustPS.emission;
        emission.rateOverTime = 0f;  // 默认关闭
    }

    void Update()
    {
        float speed = Mathf.Abs(rb.velocity.x) + Mathf.Abs(rb.velocity.z);
        emission.rateOverTime = speed > 0.1f ? 10f : 0f;
    }
}
```

---

## 14.3 章节小结

```
✅ 已掌握：
├── Particle System 组件
├── 主模块（生命周期/速度/大小）
├── Emission（发射率/爆发）
├── Shape（形状：球/锥/盒）
├── Velocity Over Lifetime
├── Color Over Lifetime
├── Size Over Lifetime
└── 代码控制粒子特效

🔜 下章预告：
第十五章：完整 2D 游戏项目 —— 从零开始，做一个完整的可运行游戏！
```

---

_📚 参考资料：《Unity 官方文档 - Particle System》_
