# Chapter 01：帧同步与状态同步

> 🎯 目标：深入理解 MMORPG 网络同步的核心——帧同步与状态同步的原理、优缺点、适用场景，是大型多人游戏的技术基础。

---

## 1.1 多人游戏同步的核心问题

### 生活中的类比 🚗

> **多人游戏 = 多辆车在高速公路上行驶：**
> - 每辆车（客户端）看到的路况略有不同（延迟）
> - 需要**所有车保持一致的速度和方向**才能不撞车
> - **帧同步** = 所有车按同一个"时刻表"行驶
> - **状态同步** = 每辆车实时广播自己的速度和位置

### 核心问题

| 问题 | 原因 |
|-----|------|
| 延迟 | 玩家 A 在北京，玩家 B 在广州，延迟 30-50ms |
| 丢包 | 网络波动，消息丢失 |
| 不一致 | 延迟导致每个客户端看到的画面不同 |
| 作弊 | 客户端不可信，可以修改本地数据 |

---

## 1.2 帧同步（Lockstep）

### 什么是帧同步？

> **帧同步 = 所有客户端严格按相同的"帧"执行操作。**
> 就像乐谱：所有乐手按同一个节拍演奏，每个人的乐器虽然不同，但演奏的内容完全一致。

### 原理

```
帧同步流程（以 30fps 为例，每帧约 33ms）：

T=0ms: 服务器广播 → "第 1 帧开始"
T=33ms: 所有客户端发送操作（移动/攻击）
T=66ms: 服务器收集所有操作，按固定顺序执行
T=99ms: 服务器广播 → "第 1 帧执行结果"
T=132ms: 所有客户端收到结果，渲染相同画面
        ↓
        第 2 帧开始...
```

### 帧同步代码示例

```csharp
// 帧同步 Tick 管理器
public class LockstepTick : MonoBehaviour
{
    public const int TICK_RATE = 30;  // 每秒 30 个 Tick
    public const int TICK_INTERVAL = 1000 / TICK_RATE;  // 33ms

    private int currentTick = 0;
    private int serverTimestamp = 0;
    private List<PlayerInput> pendingInputs = new List<PlayerInput>();

    void Update()
    {
        // -------- 每帧检查是否到 Tick --------
        if (Environment.TickCount - serverTimestamp >= TICK_INTERVAL)
        {
            serverTimestamp = Environment.TickCount;
            ProcessTick();
        }
    }

    void ProcessTick()
    {
        // -------- 收集本帧玩家输入 --------
        PlayerInput input = GatherLocalInput();

        // -------- 发送到服务器 --------
        if (isLocalPlayer)
        {
            NetworkClient.SendToServer(new TickInputMessage
            {
                tick = currentTick,
                input = input
            });
        }

        // -------- 等待服务器广播（实际需要等待服务器指令）--------
    }

    PlayerInput GatherLocalInput()
    {
        return new PlayerInput
        {
            tick = currentTick,
            moveDir = new Vector3(Input.GetAxis("Horizontal"), 0, Input.GetAxis("Vertical")),
            attack = Input.GetButton("Fire1"),
            timestamp = Environment.TickCount
        };
    }
}

// -------- 服务器端（收到所有客户端输入后执行 --------
public class LockstepServer
{
    private int currentTick = 0;
    private Dictionary<int, PlayerInput> tickInputs = new Dictionary<int, PlayerInput>();
    private int requiredInputs = 2;  // 需要的玩家数量

    public void OnReceiveInput(TickInputMessage msg)
    {
        tickInputs[msg.playerID] = msg.input;

        // -------- 收到所有玩家输入后才执行 --------
        if (tickInputs.Count >= requiredInputs)
        {
            ExecuteTick(currentTick);
            BroadcastResult(currentTick);
            currentTick++;
            tickInputs.Clear();
        }
    }

    void ExecuteTick(int tick)
    {
        // -------- 确定性执行（所有客户端结果必须一致）--------
        foreach (var input in tickInputs.Values)
        {
            ExecuteInput(input);
        }
    }

    void ExecuteInput(PlayerInput input)
    {
        // -------- 关键：必须用完全确定的逻辑 --------
        // 不允许：Random.Range(seed) / Time.time / System.Environment.TickCount
        // 必须：所有随机用固定的 seed
        Unit unit = GetUnit(input.playerID);
        Vector3 dir = input.moveDir.normalized;
        unit.position += dir * unit.speed * FIXED_DELTA_TIME;
    }
}
```

---

## 1.3 状态同步（State Synchronization）

### 什么是状态同步？

> **状态同步 = 每个客户端把状态变化告诉服务器，服务器再广播给所有客户端。**
> 就像微信群：每个人报告自己的位置（状态），大家看到的是最新位置。

### 原理

```
状态同步流程：

玩家A操作 → 客户端立即执行（预测）
             → 发送操作到服务器
服务器收到 → 校验合法性
           → 更新服务器状态
           → 广播给所有客户端
其他客户端 → 收到状态更新 → 平滑插值到新位置
```

### 状态同步代码示例

```csharp
// -------- 状态同步的 Entity --------
public class NetworkEntity : MonoBehaviour
{
    [SyncVar] private Vector3 position;  // 同步位置
    [SyncVar] private Quaternion rotation;  // 同步旋转
    [SyncVar] private int hp;  // 同步 HP
    [SyncVar] private int state;  // 同步状态（0=待机/1=移动/2=攻击）

    [SerializeField] private float syncSpeed = 10f;  // 插值速度
    private Vector3 velocity;  // 速度（用于平滑）

    [Header("预测设置")]
    public bool usePrediction = true;
    private Vector3 targetPosition;
    private Quaternion targetRotation;

    void Update()
    {
        if (isLocalPlayer)
        {
            // -------- 本地玩家：客户端预测 --------
            HandleLocalInput();
        }
        else
        {
            // -------- 远程玩家：插值同步 --------
            SyncRemote();
        }
    }

    void HandleLocalInput()
    {
        // 本地立即执行（预测）
        float h = Input.GetAxisRaw("Horizontal");
        float v = Input.GetAxisRaw("Vertical");
        Vector3 move = new Vector3(h, 0, v) * moveSpeed * Time.deltaTime;

        transform.position += move;

        // -------- 发送状态到服务器（不等待响应）--------
        CmdUpdateState(transform.position, transform.rotation, hp, state);
    }

    [Command]  // 客户端调用，服务器执行
    void CmdUpdateState(Vector3 pos, Quaternion rot, int hp, int state)
    {
        // -------- 服务器校验和更新 --------
        if (ValidateMove(pos))
        {
            this.position = pos;
            this.rotation = rot;
            this.hp = hp;
            this.state = state;
        }
    }

    void SyncRemote()
    {
        // -------- 平滑插值到目标位置 --------
        targetPosition = position;
        targetRotation = rotation;

        transform.position = Vector3.Lerp(transform.position, targetPosition, syncSpeed * Time.deltaTime);
        transform.rotation = Quaternion.Lerp(transform.rotation, targetRotation, syncSpeed * Time.deltaTime);
    }
}
```

---

## 1.4 帧同步 vs 状态同步 对比

| 维度 | 帧同步（Lockstep）| 状态同步（State Sync）|
|-----|-----------------|---------------------|
| **原理** | 所有客户端按相同帧执行相同逻辑 | 客户端发状态，服务器广播 |
| **一致性** | ✅ 极高（ Deterministic ）| ⚠️ 最终一致（有延迟）|
| **延迟感知** | 需要等待所有输入，高 | 立即响应，低 |
| **断线重连** | ❌ 困难（需要重放所有帧）| ✅ 容易（只需要拉取当前状态）|
| **反外挂** | ❌ 困难（逻辑在客户端）| ✅ 容易（服务器校验）|
| **带宽** | 低（只传输入）| 高（传完整状态）|
| **适用游戏** | RTS（如《星际争霸》）| MMORPG（如《魔兽世界》）|
| **技术难度** | 高（需要确定性引擎）| 中（需要插值平滑）|
| **随机数** | 必须固定 seed | 可以在服务器生成 |

---

## 1.5 帧同步的确定性要求

### 关键原则：所有客户端必须得出完全相同的结果

```csharp
// ❌ 禁止在帧同步中使用
float random = Random.Range(0f, 1f);  // 每次结果不同！
float time = Time.time;  // 浮点数精度问题
int hash = GetHashCode();  // 不确定

// ✅ 正确做法：使用确定性随机
private static uint rngSeed = 12345;  // 固定种子

uint DeterministicRandom()
{
    rngSeed = rngSeed * 1103515245 + 12345;
    return (rngSeed / 65536) % 32768;
}

// ✅ 正确做法：固定时间步长
public const float FIXED_DELTA_TIME = 1f / 30f;  // 30fps 固定
```

### Unity 帧同步框架推荐

| 框架 | 说明 |
|-----|------|
| **Lockstep-Lite** | 轻量帧同步框架 |
| **Ravenfield Lockstep** | 开源参考 |
| **DarkRift** | 帧同步 + 网络插件 |
| **Mirror** | 主要做状态同步，也可做帧同步 |
| **Forge** | Unity 帧同步网络 |

---

## 1.6 延迟补偿

### 客户端延迟处理

```csharp
public class LagCompensation
{
    // -------- 客户端延迟补偿 --------
    // 玩家看到的敌人位置 = 敌人实际位置 + 网络延迟
    // 射击判定应该在"敌人实际在的位置"而不是"看到的延迟位置"

    // -------- 预测 + 回滚 --------
    // 1. 客户端立即显示本地操作（预测）
    // 2. 服务器校验，如果不匹配则回滚

    // -------- 插值延迟 --------
    // 不显示当前时刻的位置
    // 显示 100ms 前的位置（让数据有缓冲时间）
    public const float INTERP_DELAY = 0.1f;  // 100ms

    void InterpolateRemoteEntity(NetworkEntity entity)
    {
        // 使用过去的状态插值，而不是当前状态
        // 保证流畅性
    }
}
```

---

## 1.7 混合同步策略

### 实际大型游戏的选择

> **不是非此即彼，而是混合使用。**

```csharp
// 混合同步策略
public class HybridSyncSystem
{
    // -------- 使用状态同步的场景 --------
    // - 移动（实时性要求高）
    // - 生命值变化
    // - 技能释放

    // -------- 使用帧同步的场景 --------
    // - 战斗伤害结算（确定性要求高）
    // - AI 行为逻辑
    // - 物理碰撞

    void HandleMove()
    {
        // 移动用状态同步（立即响应）
        CmdMove(transform.position);
    }

    void HandleCombat()
    {
        // 战斗用帧同步（确定性）
        // 发送操作到帧同步队列
        FrameSyncManager.Instance.QueueInput(new CombatInput { ... });
    }
}
```

---

## 1.8 本章小结

```
✅ 已掌握：
├── 多人游戏同步的核心问题（延迟/丢包/一致性/作弊）
├── 帧同步（Lockstep）原理和流程
├── 状态同步（State Sync）原理和流程
├── 帧同步 vs 状态同步完整对比
├── 帧同步的确定性要求（禁止随机/Time.time）
├── 延迟补偿（预测/插值/回滚）
├── 混合同步策略
└章 主流帧同步框架推荐

🔜 下章预告：
第二章：高性能网络库 KCP / ENet —— 如何用可靠 UDP 实现低延迟游戏网络。
```

---

_📚 参考资料：《游戏编程精粹》《Real-Time Rendering》《Lockstep Unity》_
