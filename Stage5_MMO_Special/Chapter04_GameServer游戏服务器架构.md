# Chapter 04：Game Server 游戏服务器架构

> 🎯 目标：掌握大型 MMORPG 服务器分层架构——Gate/Logic/DB/Battle 服务器，理解 Go/C++ 服务器框架核心。

---

## 4.1 服务器分层架构

### 生活中的类比 🏢

> **服务器分层 = 餐厅分工：**
> - **Gate（迎宾）** = 服务员，接待顾客（玩家），检查身份
> - **Logic（后厨）** = 厨师，处理业务逻辑（战斗/经济/任务）
> - **DB（仓库）** = 仓库管理员，存取物资（玩家数据）
> - **Battle（战斗厅）** = 独立包间，实时战斗，不影响主楼

### 分层架构图

```
玩家
  ↓
Gate Server（网关）→ 消息路由/压缩/加密
  ↓
Logic Server（逻辑服）→ 业务逻辑/任务/经济
  ↓
DB Server（数据库）→ 数据持久化
  ↑
Battle Server（战斗服）→ 独立运行的战斗实例
```

---

## 4.2 服务器职责详解

### Gate Server（网关服务器）

```go
// Gate Server 职责：
// 1. 接收玩家连接
// 2. 协议解析（Protobuf / JSON）
// 3. 消息路由到 Logic Server
// 4. 心跳检测（玩家是否掉线）
// 5. 禁止业务逻辑，只做中转

package gate

type GateServer struct {
    sessions map[int64]*Session  // playerID -> session
    logicConn net.Conn           // 到 Logic Server 的连接
}

func (g *GateServer) HandleConnection(conn net.Conn) {
    // 读取登录消息（Token 验证）
    var msg LoginMsg
    decode(conn, &msg)

    // 到 Logic Server 验证 Token
    logicConn.Write(encode(msg))

    // 注册 session
    playerID := msg.PlayerID
    g.sessions[playerID] = &Session{
        playerID: playerID,
        conn: conn,
    }
}

func (g *GateServer) OnPlayerDisconnect(playerID int64) {
    // 通知 Logic Server 玩家下线
    g.logicConn.Write(encode(DisconnectMsg{PlayerID: playerID}))
    delete(g.sessions, playerID)
}
```

### Logic Server（逻辑服务器）

```go
// Logic Server 职责：
// 1. 处理所有业务逻辑
// 2. 管理玩家状态
// 3. 调用 Battle Server 处理战斗
// 4. 读写 Redis/MySQL

package logic

type Player struct {
    PlayerID int64
    Name string
    Level int
    HP, MaxHP int
    MP, MaxMP int
    Gold int64
    Items map[int64]*Item
    LastSaveTime time.Time
}

type LogicServer struct {
    players map[int64]*Player
    redis *redis.Client
    battleClient *grpc.Client
}

func (s *LogicServer) HandleMove(msg MoveMsg) {
    player := s.players[msg.PlayerID]
    if player == nil { return }

    // 验证移动是否合法（不走跃迁/穿墙）
    if !s.validateMove(player, msg.TargetX, msg.TargetY) {
        return // 非法移动，忽略
    }

    player.X = msg.TargetX
    player.Y = msg.TargetY

    // 广播给周围玩家
    s.BroadcastNearby(player)
}

func (s *LogicServer) HandleItemUse(msg ItemUseMsg) {
    player := s.players[msg.PlayerID]
    item := player.Items[msg.ItemID]
    if item == nil { return }

    // 使用道具
    switch item.Type {
    case ItemHeal:
        player.HP = min(player.HP + item.Value, player.MaxHP)
    case ItemTeleport:
        player.X, player.Y = item.TargetX, item.TargetY
    }

    // 通知客户端
    s.SendToPlayer(player.PlayerID, ItemUseResultMsg{})
}
```

### DB Server（数据库服务器）

```go
// DB Server 职责：
// 1. 玩家数据持久化
// 2. 数据加载
// 3. 定期存档

package db

type DBServer struct {
    mysql *sql.DB
    redis *redis.Client
}

func (s *DBServer) SavePlayer(p *Player) error {
    // 更新 Redis（热数据）
    key := fmt.Sprintf("player:%d", p.PlayerID)
    data, _ := json.Marshal(p)
    s.redis.Set(key, data, 24*time.Hour)

    // 异步写入 MySQL（冷数据）
    go func() {
        _, err := s.mysql.Exec(`
            UPDATE players
            SET hp=?, mp=?, gold=?, level=?
            WHERE player_id=?`,
            p.HP, p.MP, p.Gold, p.Level, p.PlayerID)
        if err != nil {
            log.Printf("保存失败: %v", err)
        }
    }()
    return nil
}

func (s *DBServer) LoadPlayer(playerID int64) (*Player, error) {
    // 先查 Redis
    key := fmt.Sprintf("player:%d", playerID)
    data, err := s.redis.Get(key).Result()
    if err == nil {
        var p Player
        json.Unmarshal([]byte(data), &p)
        return &p, nil
    }

    // Redis 没有，查 MySQL
    row := s.mysql.QueryRow(`
        SELECT player_id, name, level, hp, mp, gold, x, y
        FROM players WHERE player_id=?`, playerID)

    var p Player
    err = row.Scan(&p.PlayerID, &p.Name, &p.Level, &p.HP, &p.MP, &p.Gold, &p.X, &p.Y)
    if err != nil { return nil, err }

    // 写入 Redis
    d, _ := json.Marshal(p)
    s.redis.Set(key, d, 24*time.Hour)
    return &p, nil
}
```

---

## 4.3 Go 服务器主循环

```go
// Go 高性能游戏服务器主循环
package main

import (
    "net"
    "encoding/binary"
    "sync"
)

const (
    MSG_LOGIN = 1
    MSG_MOVE  = 2
    MSG_BATTLE = 3
)

type Message struct {
    PlayerID int64
    Type    uint16
    Data    []byte
}

type Server struct {
    listener net.Listener
    players sync.Map  // 并发安全的 map
}

func main() {
    listener, err := net.Listen("tcp", ":8888")
    if err != nil {
        panic(err)
    }
    defer listener.Close()

    server := &Server{listener: listener}
    server.Run()
}

func (s *Server) Run() {
    for {
        conn, err := s.listener.Accept()
        if err != nil { continue }

        // 为每个连接启动一个 goroutine 处理
        go s.handleConnection(conn)
    }
}

func (s *Server) handleConnection(conn net.Conn) {
    defer conn.Close()

    for {
        // -------- 读取消息头（4字节长度 + 2字节类型 + 8字节playerID）--------
        header := make([]byte, 14)
        if _, err := conn.Read(header); err != nil {
            s.onDisconnect(conn)
            return
        }

        msgLen := binary.BigEndian.Uint32(header[0:4])
        msgType := binary.BigEndian.Uint16(header[4:6])
        playerID := int64(binary.BigEndian.Int64(header[6:14]))

        // -------- 读取消息体 --------
        body := make([]byte, msgLen)
        if _, err := conn.Read(body); err != nil {
            return
        }

        // -------- 分发消息 --------
        msg := Message{PlayerID: playerID, Type: msgType, Data: body}
        s.dispatch(msg)
    }
}

func (s *Server) dispatch(msg Message) {
    switch msg.Type {
    case MSG_LOGIN:
        s.onLogin(msg)
    case MSG_MOVE:
        s.onMove(msg)
    case MSG_BATTLE:
        s.onBattle(msg)
    }
}

func (s *Server) onLogin(msg Message) {
    // 解析登录消息，创建玩家session
}

func (s *Server) onMove(msg Message) {
    // 处理移动消息
}

func (s *Server) onBattle(msg Message) {
    // 转发到 Battle Server
}
```

---

## 4.4 玩家登录流程

```
1. 客户端 → Gate：发送 Token
2. Gate → Logic：验证 Token（查 Redis）
3. Logic → Gate：返回 PlayerID
4. Gate → 客户端：连接成功，分配 Logic Server 地址
5. 客户端 → Logic Server：正式连接
6. Logic → DB：加载玩家数据
7. DB → Logic：返回玩家数据
8. Logic → 客户端：推送玩家数据
9. 客户端：进入游戏世界
```

### 登录代码

```go
func (s *LogicServer) onLogin(msg Message) {
    var req LoginReq
    json.Unmarshal(msg.Data, &req)

    // 验证 Token（JWT 或 Session Token）
    playerID, err := s.validateToken(req.Token)
    if err != nil {
        s.send(msg.PlayerID, &LoginRsp{Code: 1, Msg: "认证失败"})
        return
    }

    // 加载玩家数据
    player, err := s.db.LoadPlayer(playerID)
    if err != nil {
        s.send(msg.PlayerID, &LoginRsp{Code: 2, Msg: "加载失败"})
        return
    }

    // 放入内存
    s.players.Store(playerID, player)

    // 返回成功
    s.send(msg.PlayerID, &LoginRsp{
        Code: 0,
        Player: player,
    })
}
```

---

## 4.5 分布式架构

### 微服务 vs 单体

| 维度 | 单体 | 微服务 |
|-----|------|-------|
| 扩展性 | 差 | 好 |
| 开发速度 | 快 | 慢（跨服务调用）|
| 运维复杂度 | 低 | 高 |
| 适用规模 | < 1万在线 | 10万+在线 |

### 典型 MMO 架构

```
                    ┌─────────────────────────────────────┐
                    │            负载均衡 LB              │
                    │     （Nginx / AWS ALB）             │
                    └─────────────────────────────────────┘
                                        │
            ┌───────────────┬───────────────┬───────────────┐
            ▼               ▼               ▼               ▼
       ┌────────┐     ┌────────┐     ┌────────┐     ┌────────┐
       │  Gate  │     │  Gate  │     │  Gate  │     │  Gate  │
       └────┬───┘     └────┬───┘     └────┬───┘     └────┬───┘
             │               │               │               │
             └───────────────┴───────┬───────┴───────────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 ▼                                   ▼
          ┌───────────┐                         ┌───────────┐
          │   Logic   │                         │   Logic   │   ← 无状态，可水平扩展
          └─────┬─────┘                         └─────┬─────┘
                │                                       │
       ┌────────┴────────┐                 ┌────────┴────────┐
       ▼                 ▼                 ▼                 ▼
   ┌────────┐       ┌────────┐     ┌────────┐       ┌────────┐
   │ Battle │       │   DB   │     │ Battle │       │   DB   │
   │ Server │       │ Server │     │ Server │       │ Server │
   └────────┘       └────────┘     └────────┘       └────────┘
```

---

## 4.6 玩家下线处理

```go
func (s *LogicServer) onDisconnect(playerID int64) {
    player, ok := s.players.Load(playerID)
    if !ok { return }

    p := player.(*Player)

    // -------- 1. 踢出游戏世界 --------
    s.world.RemovePlayer(playerID)

    // -------- 2. 保存数据 --------
    s.db.SavePlayer(p)

    // -------- 3. 移除在线状态 --------
    s.players.Delete(playerID)

    // -------- 4. 更新好友列表 --------
    s.notifyFriends(playerID, "Offline")

    // -------- 5. 记录下线日志 --------
    log.Printf("玩家下线: %s (ID=%d)", p.Name, playerID)
}
```

---

## 4.7 本章小结

```
✅ 已掌握：
├── Gate/Logic/DB/Battle 四层服务器架构
├── 各层职责和消息流向
├── Go 游戏服务器主循环（goroutine per connection）
├── 玩家登录完整流程（Token验证）
├── 玩家下线处理（保存/通知）
├── 分布式微服务架构
└章 服务器扩缩容策略

🔜 下章预告：
第五章：Redis 缓存与数据库 —— 大型游戏的数据层设计，热数据放 Redis，冷数据放 MySQL。
```

---

_📚 参考资料：《Go Web 编程》《大型游戏服务器架构》_
