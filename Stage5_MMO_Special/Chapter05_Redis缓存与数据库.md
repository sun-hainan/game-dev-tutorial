# Chapter 05：Redis 缓存与数据库

> 🎯 目标：掌握大型 MMORPG 数据层设计——Redis 热缓存 + MySQL 冷存储的配合方案。

---

## 5.1 为什么要分层存储？

### 生活中的类比 🏦

> **数据分层 = 银行存款：**
> - **Redis = 钱包**（随身带，随时用，但容量小）
> - **MySQL = 银行**（大容量，但取钱要排队等）
> - 钱包里有的就不去银行（缓存命中）
> - 钱包里没有的去银行取（缓存穿透）
> - 定期把钱存回银行（缓存淘汰/持久化）

---

## 5.2 Redis 适用场景

| 场景 | 数据类型 | TTL |
|-----|---------|-----|
| **在线玩家数据** | Hash（HP/MP/金币/坐标）| 与玩家在线同步 |
| **Token** | String（JWT/Session）| 24小时 |
| **排行榜** | Sorted Set | 实时更新 |
| **好友列表** | Set | 与玩家在线同步 |
| **CD/冷却** | String（剩余时间）| 到期自动删除 |
| **活动数据** | Hash | 活动结束后删除 |
| **分布式锁** | String（SET NX）| 短期 |

---

## 5.3 Redis 数据结构与操作

```go
package main

import (
    "context"
    "fmt"
    "github.com/go-redis/redis/v8"
)

type RedisClient struct {
    rdb *redis.Client
}

func NewRedisClient() *RedisClient {
    rdb := redis.NewClient(&redis.Options{
        Addr:     "localhost:6379",
        Password: "",
        DB:       0,
    })
    return &RedisClient{rdb: rdb}
}

// -------- String：Token --------
func (c *RedisClient) SetToken(playerID int64, token string) {
    key := fmt.Sprintf("token:%d", playerID)
    c.rdb.Set(context.Background(), key, token, 24*60*60*1000*1000*1000) // 24h
}

func (c *RedisClient) GetToken(playerID int64) (string, error) {
    key := fmt.Sprintf("token:%d", playerID)
    return c.rdb.Get(context.Background(), key).Result()
}

// -------- Hash：玩家数据 --------
func (c *RedisClient) SavePlayerData(p *Player) error {
    key := fmt.Sprintf("player:%d", p.PlayerID)
    return c.rdb.HMSet(context.Background(), key, map[string]interface{}{
        "name":  p.Name,
        "level": p.Level,
        "hp":    p.HP,
        "mp":    p.MP,
        "gold":  p.Gold,
        "x":     p.X,
        "y":     p.Y,
    }).Err()
}

func (c *RedisClient) LoadPlayerData(playerID int64) (*Player, error) {
    key := fmt.Sprintf("player:%d", playerID)
    data, err := c.rdb.HGetAll(context.Background(), key).Result()
    if err != nil || len(data) == 0 {
        return nil, fmt.Errorf("player not found")
    }

    return &Player{
        PlayerID: playerID,
        Name:     data["name"],
        Level:    atoi(data["level"]),
        HP:       atoi(data["hp"]),
        MP:       atoi(data["mp"]),
        Gold:     atoi(data["gold"]),
        X:        atoi(data["x"]),
        Y:        atoi(data["y"]),
    }, nil
}

// -------- Set：好友列表 --------
func (c *RedisClient) AddFriend(playerID, friendID int64) error {
    key := fmt.Sprintf("friends:%d", playerID)
    return c.rdb.SAdd(context.Background(), key, friendID).Err()
}

func (c *RedisClient) GetFriends(playerID int64) ([]int64, error) {
    key := fmt.Sprintf("friends:%d", playerID)
    members, err := c.rdb.SMembers(context.Background(), key).Result()
    if err != nil { return nil, err }

    result := make([]int64, 0, len(members))
    for _, m := range members {
        result = append(result, atoi(m))
    }
    return result, nil
}

// -------- Sorted Set：排行榜 --------
func (c *RedisClient) UpdateRank(playerID int64, score int64) error {
    return c.rdb.ZAdd(context.Background(), "rank:global",
        &redis.Z{Score: float64(score), Member: playerID}).Err()
}

func (c *RedisClient) GetTopRank(n int64) ([]int64, error) {
    return c.rdb.ZRevRange(context.Background(), "rank:global", 0, n-1).Result()
}

func (c *RedisClient) GetPlayerRank(playerID int64) (int64, error) {
    rank, err := c.rdb.ZRevRank(context.Background(), "rank:global", playerID).Result()
    return rank + 1, err // 转换为从1开始
}

// -------- String：CD 冷却 --------
func (c *RedisClient) SetCooldown(playerID int64, skillID string, durationMs int64) error {
    key := fmt.Sprintf("cd:%d:%s", playerID, skillID)
    return c.rdb.Set(context.Background(), key, "1", time.Duration(durationMs)*time.Millisecond).Err()
}

func (c *RedisClient) IsCooldownReady(playerID int64, skillID string) bool {
    key := fmt.Sprintf("cd:%d:%s", playerID, skillID)
    exists, _ := c.rdb.Exists(context.Background(), key).Result()
    return exists == 0
}
```

---

## 5.4 缓存策略：Cache-Aside

### 读流程

```
1. 查 Redis（热数据）
   ├─ 命中 → 返回数据 ✅
   └─ 未命中
       ├─ 查 MySQL
       ├─ 写入 Redis
       └─ 返回数据
```

### 写流程

```
1. 写 MySQL
2. 删除 Redis（而不是更新，避免并发不一致）
   └─ 下次读取时自动从 MySQL 加载
```

```go
func (s *LogicServer) GetPlayer(playerID int64) (*Player, error) {
    // -------- 1. 先查 Redis --------
    player, err := s.redis.LoadPlayerData(playerID)
    if err == nil && player != nil {
        return player, nil
    }

    // -------- 2. Redis 没有，查 MySQL --------
    player, err = s.db.LoadPlayer(playerID)
    if err != nil {
        return nil, err
    }

    // -------- 3. 写入 Redis --------
    s.redis.SavePlayerData(player)

    return player, nil
}

func (s *LogicServer) SavePlayer(player *Player) error {
    // -------- 1. 写 MySQL --------
    if err := s.db.SavePlayer(player); err != nil {
        return err
    }

    // -------- 2. 更新 Redis --------
    return s.redis.SavePlayerData(player)
}
```

---

## 5.5 数据库选型

| 数据库 | 适用场景 | 特点 |
|-------|---------|------|
| **MySQL** | 玩家账户/交易记录/订单 | 关系型，事务支持 |
| **PostgreSQL** | 复杂查询/地理信息 | 功能丰富，性能好 |
| **MongoDB** | 日志/聊天记录/文档 | NoSQL，灵活 |
| **Redis** | 热数据/缓存 | 内存数据库 |
| **ClickHouse** | 数据分析/排行榜 | OLAP，高吞吐 |

---

## 5.6 数据持久化方案

### 定时存档 vs 实时存档

| 方案 | 优点 | 缺点 |
|-----|------|------|
| **实时存档** | 不丢数据 | 数据库压力大 |
| **定时存档**（每30秒）| 性能好 | 最多丢30秒数据 |
| **写前日志（WAL）** | 可恢复 | 实现复杂 |

### 实际推荐

```go
// 推荐：定时存档 + 玩家下线存档
type AutoSave struct {
    interval time.Duration // 每30秒
    tick   *time.Ticker
}

func (s *AutoSave) Start() {
    s.tick = time.NewTicker(s.interval)
    go func() {
        for range s.tick.C {
            s.saveAll()
        }
    }()
}

func (s *AutoSave) saveAll() {
    s.players.Range(func(key, value interface{}) bool {
        player := value.(*Player)
        s.db.SavePlayer(player)  // 异步
        return true
    })
}
```

---

## 5.7 分布式 ID 生成：雪花算法

```go
// 雪花算法：每秒可生成 400 万不重复 ID
// 结构：1位(固定0) + 41位(时间戳) + 10位(机器ID) + 12位(序列号)

type Snowflake struct {
    mu        sync.Mutex
    lastTime  int64
    machineID int64
    seq       int64
}

func NewSnowflake(machineID int64) *Snowflake {
    return &Snowflake{machineID: machineID}
}

func (s *Snowflake) NextID() int64 {
    s.mu.Lock()
    defer s.mu.Unlock()

    now := time.Now().UnixNano() / 1000000 // 毫秒时间戳

    if now == s.lastTime {
        s.seq = (s.seq + 1) & 4095 // 超过4096则等待下一毫秒
        if s.seq == 0 {
            for now <= s.lastTime {
                now = time.Now().UnixNano() / 1000000
            }
        }
    } else {
        s.seq = 0
    }

    s.lastTime = now

    // 组装 ID：时间戳(41位) + 机器ID(10位) + 序列号(12位)
    id := (now - 1609459200000) << 22 // 减去2021年1月1日
    id |= s.machineID << 12
    id |= s.seq
    return id
}
```

---

## 5.8 本章小结

```
✅ 已掌握：
├── Redis vs MySQL 分层存储
├── Redis 数据结构（String/Hash/Set/SortedSet）
├── Cache-Aside 缓存策略
├── Token/玩家数据/排行榜/CD 缓存设计
├── 数据库选型（MySQL/PostgreSQL/MongoDB）
├── 定时存档 + 下线存档方案
├── 雪花算法分布式 ID
└章 Redis 连接池与错误处理

🔜 下章预告：
第六章：多服架构与跨服战 —— 分服/跨服/服务器发现/全局排行榜。
```

---

_📚 参考资料：《Redis设计与实现》《Go Redis 官方文档》_
