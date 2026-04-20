# Chapter 07：反外挂与安全

> 🎯 目标：掌握大型 MMORPG 安全体系——服务器校验、数据加密、协议混淆、内存保护、第三方反作弊 SDK。

---

## 7.1 安全第一原则

> **永远不要相信客户端！** 所有客户端数据都可能被篡改。

```
客户端 ←→ 服务器

❌ 不能相信：
- 客户端发送的坐标（可能穿墙）
- 客户端计算的血量（可能锁血）
- 客户端的移动速度（可能加速）
- 客户端发送的伤害值（可能秒杀）
- 任何客户端的协议（都可能伪造）
```

---

## 7.2 服务器校验策略

### 移动校验

```go
// 移动速度校验
func (s *LogicServer) ValidateMove(player *Player, msg *MoveMsg) bool {
    // 1. 检查冷却
    if time.Since(player.LastMoveTime) < 100*time.Millisecond {
        return false // 移动太频繁
    }

    // 2. 检查距离
    dx := msg.X - player.X
    dy := msg.Y - player.Y
    distance := math.Sqrt(float64(dx*dx + dy*dy))

    // 3. 最大移动速度（每秒 10 格）
    maxSpeed := 10.0
    elapsed := time.Since(player.LastMoveTime).Seconds()
    if distance/elapsed > maxSpeed {
        return false // 疑似加速
    }

    // 4. 检查路径（不能穿墙）
    if !s.map.IsPathValid(player.X, player.Y, msg.X, msg.Y) {
        return false // 疑似穿墙
    }

    return true
}
```

### 攻击校验

```go
// 攻击频率校验
func (s *LogicServer) ValidateAttack(player *Player, msg *AttackMsg) bool {
    // 1. 检查冷却（普通攻击 1 秒 CD）
    if time.Since(player.LastAttackTime) < 1*time.Second {
        return false
    }

    // 2. 检查攻击距离
    target := s.GetPlayer(msg.TargetID)
    if target == nil { return false }

    dx := target.X - player.X
    dy := target.Y - player.Y
    distance := math.Sqrt(float64(dx*dx + dy*dy))

    // 3. 检查是否在攻击范围内
    if distance > player.AttackRange {
        return false // 疑似超距离攻击
    }

    return true
}

// 伤害计算（必须在服务器执行！）
func (s *LogicServer) CalculateDamage(attacker, defender *Player) int {
    // 服务器计算伤害，客户端只负责播放动画
    damage := attacker.Attack - defender.Defense/2
    if damage < 1 { damage = 1 }
    return damage
}
```

---

## 7.3 穿墙检测

```go
// 格子地图穿墙检测
func (m *Map) IsPathValid(fromX, fromY, toX, toY int) bool {
    // Bresenham 算法检测两点之间是否有障碍物
    dx := abs(toX - fromX)
    dy := abs(toY - fromY)
    sx := -1
    if fromX < toX { sx = 1 }
    sy := -1
    if fromY < toY { sy = 1 }
    err := dx - dy

    x, y := fromX, fromY
    for {
        if !m.IsWalkable(x, y) {
            return false // 路径上有障碍物
        }
        if x == toX && y == toY {
            break
        }
        e2 := 2 * err
        if e2 > -dy {
            err -= dy
            x += sx
        }
        if e2 < dx {
            err += dx
            y += sy
        }
    }
    return true
}
```

---

## 7.4 数据加密

### 对称加密（AES）

```go
import "crypto/aes"
import "crypto/cipher"

// AES-256 加密
func AESEncrypt(plaintext []byte, key []byte) ([]byte, error) {
    block, _ := aes.NewCipher(key)
    gcm, _ := cipher.NewGCM(block)

    // 生成随机 nonce
    nonce := make([]byte, gcm.NonceSize())
    io.ReadFull(rand.Reader, nonce)

    // 加密
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}

// 解密
func AESDecrypt(ciphertext []byte, key []byte) ([]byte, error) {
    block, _ := aes.NewCipher(key)
    gcm, _ := cipher.NewGCM(block)

    nonceSize := gcm.NonceSize()
    if len(ciphertext) < nonceSize {
        return nil, errors.New("ciphertext too short")
    }

    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    return gcm.Open(nil, nonce, ciphertext, nil)
}
```

### RSA 密钥交换

```
1. 服务器生成 RSA 密钥对（公钥/私钥）
2. 客户端内置服务器公钥
3. 客户端用公钥加密会话密钥，发送给服务器
4. 服务器用私钥解密得到会话密钥
5. 后续用会话密钥做 AES 对称加密
```

---

## 7.5 协议混淆

```go
// 简单 XOR 混淆（实际用更复杂算法）
func XORMask(data []byte, key []byte) []byte {
    result := make([]byte, len(data))
    for i := range data {
        result[i] = data[i] ^ key[i%len(key)]
    }
    return result
}

// 每次连接用不同的 key
sessionKey := generateRandomKey(32)
encryptedKey := rsaEncrypt(sessionKey, serverPublicKey)
sendToServer(encryptedKey)
```

---

## 7.6 内存保护（Unity IL2CPP）

```
保护等级：
1. IL2CPP（AOT 编译，IL 中间代码不可逆）
2. CodeStripping（删除未使用代码）
3. 加密 Resources 资源
4. Obfuscator（代码混淆）
5. Anti-Cheat Expert（第三方反作弊 SDK）
```

### Unity IL2CPP 配置

```
Player Settings：
1. Scripting Backend → IL2CPP
2. Managed Stripping Level → High
3. C++ Compiler Configuration → Release
4. Enable Managed Code Stripping → Yes
```

---

## 7.7 第三方反作弊 SDK

| SDK | 公司 | 功能 |
|-----|------|-----|
| **Tencent TP** | 腾讯 | 反外挂/反加速 |
| **易盾** | 网易 | 安全/反作弊 |
| **FairGuard** | 独立 | Unity 保护 |
| **Anti-Cheat Toolkit** | Unity Asset Store | 本地保护 |

### FairGuard 基本接入

```csharp
// 在 Unity 启动时初始化
void Awake()
{
    // 初始化 FairGuard
    FairGuardSDK.Initialize();

    // 开启保护
    FairGuardSDK.EnableAntiCheat();

    // 注册异常回调
    FairGuardSDK.OnCheatDetected += OnCheatDetected;
}

void OnCheatDetected(CheatType type) {
    Debug.Log($"检测到作弊: {type}");
    // 踢出玩家 / 封号
}
```

---

## 7.8 防篡改检测

```csharp
using UnityEngine;

public class AntiCheat : MonoBehaviour
{
    // -------- CRC 校验 --------
    public bool CheckIntegrity()
    {
        // 检查关键 DLL 的 CRC
        string dllPath = Application.dataPath + "/Managed/Assembly-CSharp.dll";
        uint crc = CalculateCRC(dllPath);
        return crc == EXPECTED_CRC;
    }

    // -------- 调试器检测 --------
    public bool IsDebuggerAttached()
    {
        return System.Diagnostics.Debugger.IsAttached;
    }

    // -------- 加速器检测 --------
    public bool IsSpeedHack()
    {
        // 记录时间差
        float delta1 = Time.time;
        System.Threading.Thread.Sleep(100);
        float delta2 = Time.time;

        // 真实时间应该过 100ms，加速器会跳过
        return (delta2 - delta1) < 0.05f;
    }

    void Update()
    {
        if (!CheckIntegrity() || IsDebuggerAttached() || IsSpeedHack())
        {
            // 作弊检测到！
            KickPlayer();
        }
    }
}
```

---

## 7.9 本章小结

```
✅ 已掌握：
├── 安全第一原则（永远不相信客户端）
├── 服务器校验（移动/攻击/穿墙）
├── 伤害必须在服务器计算
├── AES/RSA 加密方案
├── 协议混淆
├── IL2CPP 内存保护
├── 第三方反作弊 SDK（FairGuard/TP）
├── 防篡改检测（CRC/调试器/加速器）
└章 封号与举报系统

🔜 下章预告：
第八章：Docker 部署游戏服务器 —— 用 Docker 和 K8s 部署高可用游戏服务器集群。
```

---

_📚 参考资料：《游戏安全白皮书》《FairGuard 文档》《AES 加密原理》_
