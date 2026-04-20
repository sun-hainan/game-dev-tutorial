# Chapter 06：多服架构与跨服战

> 🎯 目标：掌握大型 MMO 的分服/跨服架构、服务器发现、跨服战实现、分布式 ID 与全局排行榜。

---

## 6.1 分服 vs 跨服

### 分服模式（传统 MMO）

> **每个服务器独立，玩家只能看到同服玩家。**
> 类似《传奇》区服系统。

```
服务器 A（玩家1-100）    服务器 B（玩家1-100）
┌─────────────┐          ┌─────────────┐
│ 玩家甲      │          │ 玩家乙      │
│ 服务器内可见 │          │ 服务器内可见 │
└─────────────┘          └─────────────┘
```

**优点**：简单，故障隔离
**缺点**：玩家分散，新人难组队

### 跨服模式（现代 MMO）

> **多个服务器玩家进入同一个"跨服战场"。**
> 类似《王者荣耀》匹配不同区服的玩家。

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│ 服务器A │  │ 服务器B │  │ 服务器C │
└────┬────┘  └────┬────┘  └────┬────┘
     │              │              │
     └──────────────┼──────────────┘
                    ▼
           ┌─────────────┐
           │  跨服战场   │
           │ Battle Server │
           │ 玩家甲+玩家乙 │
           └─────────────┘
```

---

## 6.2 服务器发现

### Consul 服务注册与发现

```go
package discovery

import (
    "github.com/hashicorp/consul/api"
)

// 服务注册
func RegisterService(name, addr string, port int) error {
    cfg := api.DefaultConfig()
    cfg.Address = "consul:8500"

    client, _ := api.NewClient(cfg)
    registration := &api.AgentServiceRegistration{
        ID:   name + "-" + addr,
        Name: name,
        Port: port,
        Check: &api.AgentServiceCheck{
            HTTP: "http://" + addr + ":" + itoa(port) + "/health",
            Interval: "10s",
        },
    }
    return client.Agent().ServiceRegister(registration)
}

// 服务发现
func DiscoverService(name string) (string, error) {
    cfg := api.DefaultConfig()
    cfg.Address = "consul:8500"

    client, _ := api.NewClient(cfg)
    entries, _, err := client.Health().Service(name, "", true, nil)
    if err != nil || len(entries) == 0 {
        return "", fmt.Errorf("no service found: %s", name)
    }

    return entries[0].Service.Address + ":" + itoa(entries[0].Service.Port), nil
}
```

---

## 6.3 跨服战架构

### 跨服战流程

```
1. 玩家发起跨服战申请
2. Logic Server 记录申请
3. 跨服匹配服务匹配到足够玩家
4. 分配 Battle Server
5. 玩家从 Logic Server 踢出（短暂离开）
6. 玩家连接到 Battle Server
7. Battle Server 开始战斗
8. 战斗结束，结算奖励
9. 玩家断开 Battle Server
10. 玩家重新连接 Logic Server
```

### 跨服 Battle Server

```go
package battle

type BattleServer struct {
    battleID     string
    players      map[int64]*BattlePlayer
    battleState  BattleState
    turnManager  *TurnManager
}

type BattlePlayer struct {
    PlayerID     int64
    Name         string
    HP, MaxHP   int
    Skills      []int
    Position    int  // 战场位置
}

func (s *BattleServer) Start() {
    s.battleState = BattleStateRunning
    go s.runTurnLoop()
}

func (s *BattleServer) runTurnLoop() {
    for s.battleState == BattleStateRunning {
        // 回合开始，等待玩家操作
        s.waitForPlayerActions()
        // 执行战斗
        s.executeTurn()
        // 广播结果
        s.broadcastTurnResult()
        // 检查胜负
        if s.checkGameOver() {
            s.endGame()
        }
    }
}

// 玩家转移
func (s *BattleServer) TransferPlayerToBattle(playerID int64) error {
    // 1. 通知 Logic Server 踢出玩家
    logicConn := s.getLogicConn(playerID)
    logicConn.Send(&KickPlayerMsg{Reason: "进入跨服战"})

    // 2. 建立与 Battle Server 的连接
    battleConn := connectToBattle(s.battleID)

    // 3. 发送战场初始数据
    battleConn.Send(&EnterBattleMsg{
        PlayerID:   playerID,
        BattleID:   s.battleID,
        BattleAddr: s.addr,
    })

    return nil
}
```

---

## 6.4 玩家迁移

```go
// 玩家在服务器之间迁移数据
type PlayerTransfer struct {
    FromServer, ToServer string
    PlayerID int64
}

func (m *MigrationManager) MigratePlayer(req *PlayerTransfer) error {
    // 1. 在原服冻结玩家（禁止操作）
    fromLogic := getLogicServer(req.FromServer)
    fromLogic.FreezePlayer(req.PlayerID)

    // 2. 导出玩家数据
    playerData := fromLogic.ExportPlayerData(req.PlayerID)

    // 3. 在新服创建玩家
    toLogic := getLogicServer(req.ToServer)
    toLogic.ImportPlayerData(req.PlayerID, playerData)

    // 4. 删除原服玩家（可选，保留记录用于审计）
    fromLogic.ArchivePlayer(req.PlayerID)

    return nil
}
```

---

## 6.5 分布式 ID 补充：雪花算法

```go
// 分布式 ID：每台机器有独立 machineID
type Snowflake struct {
    mu        sync.Mutex
    lastTime  int64
    machineID int64  // 机器 ID（0-1023）
    seq       int64
    epoch     int64 = 1609459200000  // 2021-01-01
}

func (s *Snowflake) NextID() int64 {
    s.mu.Lock()
    defer s.mu.Unlock()

    now := time.Now().UnixNano()/1000000

    if now == s.lastTime {
        s.seq = (s.seq + 1) & 4095
        if s.seq == 0 {
            for now <= s.lastTime {
                now = time.Now().UnixNano()/1000000
            }
        }
    } else {
        s.seq = 0
    }
    s.lastTime = now

    id := (now - s.epoch) << 22
    id |= s.machineID << 12
    id |= s.seq
    return id
}
```

---

## 6.6 全局排行榜

```go
// 使用 Redis Sorted Set 实现全局排行榜
func (s *RedisClient) UpdateGlobalRank(playerID int64, score int64) error {
    // ZADD：添加或更新分数
    return s.rdb.ZAdd(context.Background(), "rank:global", &redis.Z{
        Score:  float64(score),
        Member: playerID,
    }).Err()
}

// 获取 Top N
func (s *RedisClient) GetTopPlayers(n int64) ([]*RankEntry, error) {
    // ZREVRANGE：从大到小取
    results, err := s.rdb.ZRevRangeWithScores(context.Background(),
        "rank:global", 0, n-1).Result()
    if err != nil { return nil, err }

    entries := make([]*RankEntry, 0, len(results))
    for i, z := range results {
        entries = append(entries, &RankEntry{
            Rank:    int64(i + 1),
            PlayerID: z.Member.(int64),
            Score:   int64(z.Score),
        })
    }
    return entries, nil
}

// 获取玩家排名
func (s *RedisClient) GetPlayerRank(playerID int64) (int64, error) {
    // ZREVRANK：从大到小排第几（0-based）
    rank, err := s.rdb.ZRevRank(context.Background(), "rank:global", playerID).Result()
    return rank + 1, err // 转换为 1-based
}
```

---

## 6.7 本章小结

```
✅ 已掌握：
├── 分服 vs 跨服架构区别
├── Consul 服务发现
├── 跨服战完整流程
├── 跨服 Battle Server 设计
├── 玩家服务器间迁移
├── 雪花算法分布式 ID
└章 Redis Sorted Set 全局排行榜

🔜 下章预告：
第七章：反外挂与安全 —— 服务器校验、内存保护、数据加密、反作弊 SDK。
```

---

_📚 参考资料：《大型 MMO 服务器架构》《Consul 官方文档》_
