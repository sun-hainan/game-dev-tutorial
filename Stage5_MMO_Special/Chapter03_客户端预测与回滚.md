# 第三章：客户端预测与回滚 —— 让游戏"提前跑"的艺术

> 🎯 本章目标：理解客户端预测的原理，掌握帧号对齐和回滚机制，能够实现流畅的实时多人游戏。

---

## 1. 生活类比：外卖提前到达的秘密

你有没有过这种经历：点外卖时 App 显示"预计 30 分钟到达"，但你 20 分钟就吃到了？

**秘密在于"预测"**：App 根据你平时的习惯、骑手的速度、天气情况，提前估算你什么时候需要这份外卖。

游戏中的**客户端预测**也是同样的道理：

> **不等服务器确认，就假设操作正确并立即执行，等服务器返回真实结果后再修正。**

---

## 2. 客户端预测

### 2.1 什么是客户端预测？

**生活类比**：你过马路时，看到绿灯亮了，你不需要等交警确认"可以通行"才迈步，而是直接往前走。如果有车闯红灯，交警会拦下来，你再调整。

**游戏中的场景**：你控制角色向前移动，网络延迟 100ms。如果你等服务器确认后再移动，你会感觉到明显的卡顿。有了客户端预测，你按下移动键的瞬间，角色就开始移动了。

### 2.2 预测的执行流程

```
玩家按下"W"键（移动）
    ↓
客户端立即执行移动（预测）
    ↓
客户端发送移动请求给服务器
    ↓
客户端同时进行游戏逻辑（不等待）
    ↓
服务器校验并广播结果
    ↓
客户端收到服务器结果，对比本地预测
    ↓
如果一致：继续游戏（预测正确）
如果不一致：回滚到服务器状态（预测失败）
```

### 2.3 预测代码实现

```csharp
using UnityEngine;
using System.Collections.Generic;

/// <summary>
/// 客户端预测系统 —— 就像汽车的无钥匙进入
/// 你按下启动键，汽车"预测"你要发动，提前准备好
/// </summary>
public class ClientPrediction
{
    // 本地玩家实体
    private Entity localPlayer;

    // 预测的输入队列 —— 就像排队等待执行的指令
    private Queue<InputFrame> predictedInputs = new Queue<InputFrame>();

    // 帧号 -> 服务器状态（用于回滚）
    private Dictionary<int, EntityState> serverStates = new Dictionary<int, EntityState>();

    // 当前帧号
    private int currentFrame;

    /// <summary>
    /// 处理本地输入并立即执行预测
    /// 就像你喊"开跑"，身体立刻响应，不等大脑确认
    /// </summary>
    public void HandleLocalInput(InputFrame input) {
        // 保存输入到预测队列
        predictedInputs.Enqueue(input);

        // 立即执行预测逻辑（不等服务器确认）
        ExecutePrediction(input);

        // 记录当前帧号
        currentFrame = input.frame;

        // 发送输入给服务器（带帧号）
        NetworkManager.Instance.SendInput(input);
    }

    /// <summary>
    /// 执行预测 —— 根据输入计算本地状态
    /// </summary>
    private void ExecutePrediction(InputFrame input) {
        // 获取当前预测状态
        Vector3 pos = localPlayer.position;
        Quaternion rot = localPlayer.rotation;

        // 根据输入计算新位置
        switch (input.type) {
            case InputType.Move:
                // 移动输入：计算新位置
                Vector3 direction = GetDirection(input.key);
                float distance = input.moveSpeed * Time.deltaTime;
                pos += direction * distance;
                break;

            case InputType.Jump:
                // 跳跃输入：应用跳跃速度
                if (localPlayer.isGrounded) {
                    localPlayer.velocity.y = input.jumpForce;
                }
                break;

            case InputType.Attack:
                // 攻击输入：触发攻击动画和伤害计算
                PerformAttack(localPlayer, input.targetID);
                break;
        }

        // 更新本地实体状态（预测结果）
        localPlayer.position = pos;
        localPlayer.rotation = rot;
        localPlayer.frame = input.frame;

        Debug.Log($"[预测] 帧 {input.frame}，位置 {pos}");
    }

    /// <summary>
    /// 从服务器接收确认状态
    /// 就像外卖App告诉你"实际还有15分钟到"
    /// </summary>
    public void OnServerState(int serverFrame, EntityState serverState) {
        // 记录服务器状态（用于回滚）
        serverStates[serverFrame] = serverState;

        // 如果服务器帧号 >= 本地预测帧号，需要验证
        while (predictedInputs.Count > 0) {
            InputFrame input = predictedInputs.Peek();

            // 如果这个输入已经被服务器确认
            if (input.frame <= serverFrame) {
                predictedInputs.Dequeue();  // 从预测队列移除

                // 检查服务器状态和本地预测是否一致
                EntityState predictedState = PredictStateAtFrame(input.frame);

                if (!StatesMatch(predictedState, serverState)) {
                    // 预测失败！需要回滚
                    Debug.LogWarning($"[回滚] 帧 {input.frame} 预测失败！");
                    RollbackToFrame(input.frame, serverState);
                }
            } else {
                break;  // 还有未确认的输入，等待
            }
        }
    }

    /// <summary>
    /// 验证预测状态和服务器状态是否匹配
    /// </summary>
    private bool StatesMatch(EntityState predicted, EntityState server) {
        // 位置误差小于 0.1 米就算匹配（允许一定误差）
        float positionThreshold = 0.1f;
        float distance = Vector3.Distance(predicted.position, server.position);

        // 旋转误差小于 5 度就算匹配
        float rotationThreshold = 5f;
        float angleDiff = Quaternion.Angle(predicted.rotation, server.rotation);

        return distance < positionThreshold && angleDiff < rotationThreshold;
    }
}
```

---

## 3. 服务器校验

### 3.1 为什么需要服务器校验？

**生活类比**：你去便利店买东西，收银员会核实你付的钱对不对。如果你说"我买了 100 块的东西"，收银员会检查是否真的值 100 块。

**游戏中的场景**：客户端说"我开枪了，打中了 50 米外的敌人"。服务器需要校验：
- 这个玩家真的在那个位置吗？
- 敌人的生命值还够扣吗？
- 攻击间隔是否太短（是不是开了连发外挂）？

### 3.2 服务器校验逻辑

```cpp
// 服务器校验系统 —— 就像收银台的核对员
// 客户端说"我做了什么"，服务器验证"这是不是真的"

struct ServerValidator {
    // 校验移动速度（防止加速外挂）
    bool ValidateMove(Player* player, MoveInput& input) {
        // 计算理论最大移动距离（速度 * 时间）
        float maxSpeed = player->moveSpeed * input.dt;  // 最大速度 * 时间间隔

        // 计算实际移动距离
        Vector3 delta = input.newPosition - player->position;
        float actualDistance = delta.Length();

        // 允许 10% 的误差（网络延迟导致的微小差异）
        float threshold = maxSpeed * 1.1f;

        if (actualDistance > threshold) {
            // 移动超速，拒绝！
            printf("玩家 %d 移动校验失败: 理论 %.2f, 实际 %.2f\n",
                   player->id, maxSpeed, actualDistance);
            return false;
        }

        return true;
    }

    // 校验攻击频率（防止连发外挂）
    bool ValidateAttack(Player* player, AttackInput& input) {
        // 最小攻击间隔（毫秒）
        int minAttackInterval = 500;  // 最快每秒2次攻击

        // 上次攻击时间
        int lastAttackTime = player->lastAttackTime;
        int currentTime = GetCurrentTimeMs();

        // 如果攻击间隔太短，拒绝
        if (currentTime - lastAttackTime < minAttackInterval) {
            printf("玩家 %d 攻击频率校验失败: 间隔 %dms\n",
                   player->id, currentTime - lastAttackTime);
            return false;
        }

        return true;
    }

    // 校验攻击距离（防止透视外挂）
    bool ValidateAttackRange(Player* player, AttackInput& input) {
        // 获取目标玩家位置
        Player* target = GetPlayer(input.targetID);
        if (!target) return false;

        // 计算距离
        float distance = Distance(player->position, target->position);
        float maxRange = player->weaponRange;  // 武器最大射程

        if (distance > maxRange) {
            printf("玩家 %d 攻击距离校验失败: 射程 %.2f, 距离 %.2f\n",
                   player->id, maxRange, distance);
            return false;
        }

        return true;
    }
};
```

### 3.3 服务器拒绝非法操作

```cpp
// 服务器处理移动请求
void HandleMoveRequest(Player* player, MoveInput& input) {
    // 校验移动是否合法
    if (!validator.ValidateMove(player, input)) {
        // 校验失败，发送正确的服务器状态给客户端
        SendCorrectState(player);
        return;  // 不执行移动
    }

    // 校验通过，执行移动
    player->position = input.newPosition;
    player->rotation = input.newRotation;

    // 广播给所有客户端（包括发送者，让他修正预测）
    BroadcastPlayerMove(player);
}

// 服务器处理攻击请求
void HandleAttackRequest(Player* player, AttackInput& input) {
    // 多个校验
    if (!validator.ValidateAttack(player, input)) return;       // 攻击频率
    if (!validator.ValidateAttackRange(player, input)) return;   // 攻击距离
    if (!validator.ValidateLineOfSight(player, input)) return;  // 视线遮挡

    // 校验通过，执行攻击
    Player* target = GetPlayer(input.targetID);
    if (target) {
        target->hp -= player->attackDamage;
        BroadcastAttackResult(player->id, target->id, player->attackDamage);
    }
}
```

---

## 4. 客户端回滚

### 4.1 什么是回滚？

**生活类比**：你在 Excel 里编辑表格，复制粘贴了一些数据，但发现贴错位置了，你按 `Ctrl+Z` 撤销，回到之前的状态。

**游戏中的场景**：客户端预测角色走到了 (10, 0, 10)，但服务器说"不对，你实际应该在 (9.5, 0, 10)"。客户端需要**撤销预测**并**应用服务器状态**——这就是回滚。

### 4.2 回滚机制的实现

```csharp
/// <summary>
/// 回滚系统 —— 就像游戏存档点
/// 当服务器说"你们的状态不对"时，我们回到上一个正确的存档点
/// </summary>
public class RollbackManager
{
    // 实体状态历史（帧号 -> 状态）
    private Dictionary<int, EntityStateSnapshot> stateHistory = new Dictionary<int, EntityStateSnapshot>();

    // 最大回滚帧数（防止回滚太远）
    private const int MaxRollbackFrames = 30;  // 最多回滚 0.5 秒（30帧）

    /// <summary>
    /// 保存状态快照（每帧都要保存，用于回滚）
    /// 就像游戏自动存档，每秒存好几次
    /// </summary>
    public void SaveSnapshot(int frame, Entity entity) {
        // 如果历史记录太长，删除最旧的
        if (stateHistory.Count > MaxRollbackFrames) {
            int oldestFrame = GetOldestFrame();
            stateHistory.Remove(oldestFrame);
        }

        stateHistory[frame] = new EntityStateSnapshot {
            frame = frame,
            position = entity.position,
            rotation = entity.rotation,
            velocity = entity.velocity,
            hp = entity.hp,
            animationState = entity.animState
        };
    }

    /// <summary>
    /// 执行回滚 —— 回到服务器确认的正确状态
    /// </summary>
    public void Rollback(int frame, EntityState serverState) {
        Debug.Log($"执行回滚：回到帧 {frame}");

        // 找到最近的服务器确认状态
        if (!stateHistory.ContainsKey(frame)) {
            Debug.LogError($"找不到帧 {frame} 的快照，无法回滚！");
            return;
        }

        // 获取快照
        EntityStateSnapshot snapshot = stateHistory[frame];

        // 应用到实体（撤销本地预测，恢复到服务器认可的状态）
        Entity entity = GetLocalPlayer();
        entity.position = snapshot.position;
        entity.rotation = snapshot.rotation;
        entity.velocity = snapshot.velocity;
        entity.hp = snapshot.hp;
        entity.animState = snapshot.animationState;

        // 清空所有比这个帧更新的预测输入
        ClearPredictedInputsAfter(frame);

        // 重置实体帧号
        entity.frame = frame;

        Debug.Log($"回滚完成：位置 {entity.position}");

        // 重新应用所有预测输入（从回滚点开始重新预测）
        ReapplyPredictions(frame);
    }

    /// <summary>
    /// 重新应用预测输入（从回滚点开始）
    /// </summary>
    private void ReapplyPredictions(int fromFrame) {
        // 获取所有比 fromFrame 新的输入
        List<InputFrame> futureInputs = GetPredictedInputsAfter(fromFrame);

        // 重新执行这些输入
        foreach (InputFrame input in futureInputs) {
            // 重新计算位置（基于服务器确认的状态）
            ExecuteInputPrediction(input);

            // 保存新的快照
            SaveSnapshot(input.frame, GetLocalPlayer());
        }
    }
}
```

---

## 5. 帧号对齐

### 5.1 为什么需要帧号？

**生活类比**：你和朋友微信聊天，你说"我明天 10 点到"，朋友问"10 点哪儿？"。没有具体时间，你们就对不上。

游戏中的**帧号**就是让所有客户端和服务器**在同一帧上**对齐的工具。

### 5.2 帧号同步机制

```csharp
/// <summary>
/// 帧号同步系统 —— 就像音乐节拍器
/// 让所有人都跟着同一个节拍跳舞
/// </summary>
public class FrameSync
{
    // 当前帧号（游戏逻辑帧）
    private int localFrame = 0;

    // 服务器当前帧（可能领先本地）
    private int serverFrame = 0;

    // 帧时间（固定 33ms，约 30 FPS）
    private const int FrameTimeMs = 33;

    // 玩家输入缓冲（等待帧号对齐）
    private Dictionary<int, List<InputFrame>> inputBuffer = new Dictionary<int, List<InputFrame>>();

    /// <summary>
    /// 主循环：游戏逻辑每帧调用一次
    /// </summary>
    public void Update() {
        // 获取服务器当前帧
        serverFrame = NetworkManager.Instance.GetServerFrame();

        // 如果本地帧落后服务器太多，追赶
        while (localFrame < serverFrame - MaxBufferedFrames) {
            // 跳过本地帧（快进到服务器帧）
            localFrame++;
        }

        // 检查是否可以执行当前帧
        while (localFrame <= serverFrame) {
            // 获取当前帧的输入
            List<InputFrame> inputs = GetInputsForFrame(localFrame);

            // 执行输入（如果没有输入，就空闲等待）
            ExecuteFrame(localFrame, inputs);

            localFrame++;
        }
    }

    /// <summary>
    /// 收到服务器帧更新
    /// </summary>
    public void OnServerFrameUpdate(int frame, EntityState[] states) {
        serverFrame = frame;

        // 更新所有实体的服务器状态（用于回滚校验）
        foreach (EntityState state in states) {
            // 更新实体的服务器状态
            UpdateServerState(state);
        }
    }

    /// <summary>
    /// 发送本地输入
    /// </summary>
    public void SendInput(InputType type, byte[] data) {
        InputFrame input = new InputFrame {
            frame = localFrame + NetworkDelayFrames,  // 预测未来的帧
            playerID = localPlayerID,
            type = type,
            data = data,
            timestamp = GetCurrentTimeMs()
        };

        // 放入输入缓冲
        AddToBuffer(input);

        // 发送输入给服务器
        NetworkManager.Instance.SendInput(input);
    }
}
```

---

## 6. 延迟补偿

### 6.1 什么是延迟补偿？

**生活类比**：你看足球直播，画面有 3 秒延迟。你看到球员射门，但球可能已经进了。这是因为画面是 3 秒前的。

**游戏中的场景**：你看到敌人正好露头，你开枪了。但实际上敌人已经跑回去了——因为你看到的是 100ms 前的画面。这就是**延迟射击**问题。

**延迟补偿**就是让服务器根据延迟调整"射击判定时间"，确保公平。

### 6.2 延迟补偿的实现

```cpp
// 延迟补偿系统 —— 就像裁判看 VAR 回放
// 回放"事发当时"的情况，而不是"看到事发"的时候

struct DelayCompensation {
    int localDelay;      // 本客户端延迟（毫秒）
    int serverFrameTime; // 每帧时间（毫秒）

    // 服务器处理射击请求
    void HandleShoot(int shooterID, int targetID, int shootFrame) {
        // shootFrame 是客户端发送的帧号（已经包含延迟）
        // 服务器需要根据延迟，反推"射击时"目标在哪里

        // 计算延迟导致的帧差
        int delayFrames = localDelay / serverFrameTime;

        // 反推实际射击帧（目标当时的状态）
        int actualFrame = shootFrame - delayFrames;

        // 获取目标在那帧的位置
        EntityState targetState = GetEntityStateAtFrame(targetID, actualFrame);

        // 使用历史位置进行判定
        Vector3 targetPosition = targetState.position;

        // ... 进行射击判定（是否命中等）
    }
};

// 客户端发送射击请求时，包含客户端当前的帧号
void ClientSendShoot(int targetID) {
    ShootRequest req;
    req.shootFrame = localFrame;       // 当前本地帧
    req.targetID = targetID;
    req.timestamp = GetCurrentTimeMs();
    req.position = localPlayer.position;

    SendToServer(MSG_SHOOT, req);
}
```

---

## 7. 输入延迟容忍

### 7.1 什么是输入延迟容忍？

**生活类比**：你玩节奏游戏，按键时机和音乐差 50ms 之内，系统都算你"准"。这个 50ms 就是容忍窗口。

**游戏中的场景**：网络波动导致某个输入延迟了 200ms 才到服务器。如果直接丢弃，玩家会感觉"我的操作消失了"。如果等它到来并执行，玩家会感觉"怎么角色反应慢了"。

**解决方案**：给输入一个容忍窗口，在窗口内的延迟都视为正常。

### 7.2 输入容忍实现

```csharp
/// <summary>
/// 输入延迟容忍系统 —— 就像快递的"24小时内送达都算准时"
/// 只要输入在一定时间内到达，就视为有效
/// </summary>
public class InputTolerance
{
    // 容忍窗口：300ms 之内的延迟都接受
    private const int ToleranceWindowMs = 300;

    // 帧时间（毫秒）
    private const int FrameTimeMs = 33;

    // 输入缓冲
    private Queue<InputFrame> inputQueue = new Queue<InputFrame>();

    /// <summary>
    /// 服务器接收客户端输入
    /// </summary>
    public void ServerReceiveInput(InputFrame input) {
        int now = GetCurrentTimeMs();
        int delay = now - input.timestamp;  // 客户端发送时间 -> 服务器收到时间

        // 如果延迟在容忍窗口内，接受
        if (delay <= ToleranceWindowMs) {
            // 计算输入应该属于的帧（回溯）
            int frameDelta = delay / FrameTimeMs;
            input.frame = input.frame + frameDelta;  // 调整帧号

            Debug.Log($"[服务器] 接收输入，延迟 {delay}ms，调整到帧 {input.frame}");
            inputQueue.Enqueue(input);
        } else {
            // 延迟太长，丢弃
            Debug.LogWarning($"[服务器] 输入延迟 {delay}ms，超出容忍窗口，丢弃");
        }
    }

    /// <summary>
    /// 客户端重发丢失的输入
    /// </summary>
    public void ClientResendLostInputs() {
        int now = GetCurrentTimeMs();

        foreach (InputFrame input in pendingInputs) {
            int elapsed = now - input.sendTime;

            // 如果输入发送超过 500ms 还没收到确认，重新发送
            if (elapsed > 500 && !input.confirmed) {
                Debug.Log($"重发丢失的输入：帧 {input.frame}");
                NetworkManager.Instance.SendInput(input);
                input.sendTime = now;  // 更新发送时间
            }
        }
    }
}
```

---

## 8. 代码示例：C# 预测与回滚完整实现

```csharp
using UnityEngine;
using System;
using System.Collections.Generic;

/// <summary>
/// 输入类型枚举
/// </summary>
public enum InputType : byte
{
    Move = 1,
    Jump = 2,
    Attack = 3,
    UseSkill = 4
}

/// <summary>
/// 输入帧结构 —— 记录一次完整的玩家输入
/// </summary>
[System.Serializable]
public struct InputFrame
{
    public int frame;           // 帧号（用于同步）
    public uint playerID;       // 玩家ID
    public InputType type;      // 输入类型
    public Vector3 direction;   // 移动方向/攻击方向
    public int timestamp;       // 时间戳（毫秒）
}

/// <summary>
/// 实体状态快照 —— 用于回滚和校验
/// </summary>
[System.Serializable]
public struct EntityState
{
    public uint entityID;
    public int frame;
    public Vector3 position;
    public Quaternion rotation;
    public Vector3 velocity;
    public float hp;
    public byte animState;
}

/// <summary>
/// 预测与回滚管理器 —— 游戏客户端核心组件
/// </summary>
public class PredictionManager : MonoBehaviour
{
    // 单例
    public static PredictionManager Instance { get; private set; }

    // 本地玩家
    public uint localPlayerID;

    // 所有实体状态（本地预测的）
    private Dictionary<uint, EntityState> localEntities = new Dictionary<uint, EntityState>();

    // 服务器确认的状态（用于校验）
    private Dictionary<int, Dictionary<uint, EntityState>> serverFrameStates = new Dictionary<int, Dictionary<uint, EntityState>>();

    // 待确认的输入队列
    private Queue<InputFrame> unconfirmedInputs = new Queue<InputFrame>();

    // 每帧时间（毫秒）
    private const int FrameTimeMs = 33;

    // 最大回滚帧数
    private const int MaxRollbackFrames = 30;

    void Awake()
    {
        Instance = this;
    }

    /// <summary>
    /// 处理本地输入（产生预测）
    /// </summary>
    public void ProcessLocalInput(InputType type, Vector3 direction)
    {
        // 创建输入帧
        InputFrame input = new InputFrame
        {
            frame = GameFrame.Current,
            playerID = localPlayerID,
            type = type,
            direction = direction,
            timestamp = Environment.TickCount
        };

        // 保存到待确认队列
        unconfirmedInputs.Enqueue(input);

        // 立即执行本地预测
        ApplyInputToLocalState(input);

        // 发送输入给服务器
        SendInputToServer(input);

        Debug.Log($"[本地] 处理输入，帧 {input.frame}");
    }

    /// <summary>
    /// 应用输入到本地状态（预测执行）
    /// </summary>
    private void ApplyInputToLocalState(InputFrame input)
    {
        if (!localEntities.ContainsKey(localPlayerID))
        {
            localEntities[localPlayerID] = new EntityState { entityID = localPlayerID };
        }

        EntityState state = localEntities[localPlayerID];

        switch (input.type)
        {
            case InputType.Move:
                // 根据方向计算新位置（简单示例：直接移动）
                state.position += input.direction * 5f * (FrameTimeMs / 1000f);
                break;

            case InputType.Jump:
                state.velocity.y = 10f;  // 跳跃速度
                break;

            case InputType.Attack:
                // 触发攻击（不改变位置）
                Debug.Log($"[本地预测] 玩家攻击，方向 {input.direction}");
                break;
        }

        state.frame = input.frame;
        localEntities[localPlayerID] = state;

        // 保存快照
        SaveSnapshot(state.frame);
    }

    /// <summary>
    /// 发送输入到服务器
    /// </summary>
    private void SendInputToServer(InputFrame input)
    {
        // 序列化为字节数组
        byte[] data = SerializeInput(input);

        // 通过网络管理器发送
        NetworkManager.Instance.SendPacket(MSG_INPUT, data);
    }

    /// <summary>
    /// 服务器确认状态到达（可能触发回滚）
    /// </summary>
    public void OnServerStateBatch(int serverFrame, EntityState[] states)
    {
        // 保存服务器状态
        serverFrameStates[serverFrame] = new Dictionary<uint, EntityState>();
        foreach (EntityState state in states)
        {
            serverFrameStates[serverFrame][state.entityID] = state;
        }

        // 校验本地预测和服务器状态
        CheckPredictionErrors(serverFrame);

        // 清理旧状态（节省内存）
        CleanupOldStates(serverFrame);
    }

    /// <summary>
    /// 校验预测错误，可能触发回滚
    /// </summary>
    private void CheckPredictionErrors(int serverFrame)
    {
        // 检查每个待确认的输入
        Queue<InputFrame> remainingInputs = new Queue<InputFrame>();
        bool needRollback = false;
        int firstErrorFrame = 0;
        EntityState correctState = new EntityState();

        while (unconfirmedInputs.Count > 0)
        {
            InputFrame input = unconfirmedInputs.Dequeue();

            // 如果输入帧 <= 服务器帧，说明已经有结果了
            if (input.frame <= serverFrame)
            {
                // 获取服务器确认的状态
                if (serverFrameStates.ContainsKey(input.frame) &&
                    serverFrameStates[input.frame].ContainsKey(localPlayerID))
                {
                    EntityState serverState = serverFrameStates[input.frame][localPlayerID];

                    // 对比本地预测和服务器状态
                    float positionError = Vector3.Distance(
                        localEntities[localPlayerID].position,
                        serverState.position);

                    // 如果误差超过阈值，需要回滚
                    if (positionError > 0.1f)
                    {
                        needRollback = true;
                        firstErrorFrame = input.frame;
                        correctState = serverState;
                        break;
                    }
                }
            }
            else
            {
                // 还有未确认的输入
                remainingInputs.Enqueue(input);
            }
        }

        // 保存剩余输入
        unconfirmedInputs = remainingInputs;

        // 如果需要回滚
        if (needRollback)
        {
            Debug.LogWarning($"[回滚] 帧 {firstErrorFrame} 预测错误，开始回滚");
            PerformRollback(firstErrorFrame, correctState);
        }
    }

    /// <summary>
    /// 执行回滚
    /// </summary>
    private void PerformRollback(int frame, EntityState correctState)
    {
        // 应用服务器确认的正确状态
        localEntities[localPlayerID] = correctState;

        // 更新游戏对象位置（视觉上也要体现）
        UpdateEntityVisual(localPlayerID, correctState);

        // 清空所有比回滚帧更新的输入（它们都基于错误状态）
        ClearInputsAfter(frame);

        // 重置帧号（游戏逻辑帧回到回滚点）
        GameFrame.Current = frame;

        Debug.Log($"[回滚完成] 位置重置为 {correctState.position}");

        // 重新应用所有还在队列中的输入
        ReapplyPendingInputs();
    }

    /// <summary>
    /// 重新应用待确认的输入（回滚后的重新预测）
    /// </summary>
    private void ReapplyPendingInputs()
    {
        // 所有还在队列中的输入都是"还没被服务器确认"的
        // 需要重新应用它们（基于新的正确状态）
        foreach (InputFrame input in unconfirmedInputs)
        {
            ApplyInputToLocalState(input);
        }
    }

    /// <summary>
    /// 更新实体视觉位置（让玩家看到回滚效果）
    /// </summary>
    private void UpdateEntityVisual(uint entityID, EntityState state)
    {
        // 查找对应的游戏对象并更新位置
        // GameObject entity = FindEntityGameObject(entityID);
        // if (entity != null)
        // {
        //     entity.transform.position = state.position;
        //     entity.transform.rotation = state.rotation;
        // }
    }

    /// <summary>
    /// 保存状态快照
    /// </summary>
    private void SaveSnapshot(int frame)
    {
        if (!localEntities.ContainsKey(localPlayerID)) return;

        EntityState state = localEntities[localPlayerID];

        // 只保存最近 N 帧的快照
        if (serverFrameStates.Count > MaxRollbackFrames)
        {
            // 删除最老的帧
            int oldestFrame = GetOldestFrame(serverFrameStates.Keys);
            serverFrameStates.Remove(oldestFrame);
        }
    }

    /// <summary>
    /// 清理旧状态
    /// </summary>
    private void CleanupOldStates(int currentFrame)
    {
        int threshold = currentFrame - MaxRollbackFrames;
        List<int> toRemove = new List<int>();

        foreach (int frame in serverFrameStates.Keys)
        {
            if (frame < threshold)
                toRemove.Add(frame);
        }

        foreach (int frame in toRemove)
        {
            serverFrameStates.Remove(frame);
        }
    }

    /// <summary>
    /// 序列化输入
    /// </summary>
    private byte[] SerializeInput(InputFrame input)
    {
        // 简单序列化：[frame:4][type:1][x:4][y:4][z:4][timestamp:4]
        byte[] data = new byte[17];
        Array.Copy(BitConverter.GetBytes(input.frame), 0, data, 0, 4);
        data[4] = (byte)input.type;
        Array.Copy(BitConverter.GetBytes(input.direction.x), 0, data, 5, 4);
        Array.Copy(BitConverter.GetBytes(input.direction.y), 0, data, 9, 4);
        Array.Copy(BitConverter.GetBytes(input.direction.z), 0, data, 13, 4);
        return data;
    }

    private int GetOldestFrame(IEnumerable<int> frames)
    {
        int oldest = int.MaxValue;
        foreach (int f in frames) if (f < oldest) oldest = f;
        return oldest;
    }

    private void ClearInputsAfter(int frame)
    {
        // 清空基于错误状态的输入
        // 实际上只是让它们重新应用（重新预测）
    }
}
```

---

## 本章小结

本章我们学习了客户端预测与回滚的核心机制：

| 知识点 | 关键收获 |
|--------|----------|
| **客户端预测** | 不等服务器确认，立即执行本地操作，提升响应感 |
| **服务器校验** | 服务器才是可信的，校验移动速度、攻击频率、攻击范围 |
| **客户端回滚** | 当预测错误时，撤销本地操作，恢复到服务器确认的正确状态 |
| **帧号对齐** | 用帧号作为所有客户端和服务器的同步基准 |
| **延迟补偿** | 服务器根据延迟反推"射击时"的真实状态 |
| **输入延迟容忍** | 在一定时间窗口内的延迟都视为正常，避免丢包感 |

---

> 💡 **实战建议**：
> 1. **先做本地预测，再做回滚校验** —— 不要一开始就做复杂的回滚，先让本地操作流畅
> 2. **容忍小误差** —— 位置误差 < 0.1m、角度误差 < 5° 都视为"正确"，避免频繁回滚
> 3. **限制回滚范围** —— 最多回滚 0.5-1 秒的帧，超出范围的直接断开重连
> 4. **注意用户体验** —— 回滚时如果视觉跳动太大，可以做平滑插值

下一章我们将学习 **游戏服务器架构**，了解如何设计一个能承载万人同时在线的服务器系统。