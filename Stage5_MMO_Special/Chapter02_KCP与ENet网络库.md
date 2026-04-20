# 第二章：KCP 与 ENet 网络库 —— 打造可靠 UDP 的双剑客

> 🎯 本章目标：理解可靠 UDP 的实现原理，掌握 KCP 和 ENet 的使用方法，能在游戏项目中实现低延迟网络通信。

---

## 1. 生活类比：快递公司的三种服务

想象你在寄快递：

| 类型 | 特点 | 对应协议 |
|------|------|---------|
| **普通快递** | 慢，但可靠，会重试 | TCP |
| **同城闪送** | 快，但不保证送达 | UDP |
| **专人直送** | 快 + 保证送达 + 按需排序 | KCP / ENet |

**TCP** 像普通快递，有追踪，迟到就重发，但绕远路；
**UDP** 像闪送，抄近路，但包裹可能丢失；
**KCP/ENet** 像专人直送，既有速度，又确保包裹必达。

---

## 2. KCP 协议原理

### 2.1 什么是 KCP？

KCP 是一个 **可靠 UDP 协议**，由社区开发者 Skywind3000 创建。它的核心思想是：**用最快的速度重传丢失的数据**。

### 2.2 核心机制

```cpp
// KCP 的关键参数 —— 就像调整赛车的换挡时机
// nodelay: 是否启用快速模式（延迟更低的重传策略）
// interval: 检查丢失包的间隔（毫秒）
// resend: 快速重传阈值（丢多少个包就触发快速重传）
// nc: 是否关闭拥塞控制（游戏通常关闭以保持稳定延迟）
ikcp_nodelay(kcp, 1, 10, 2, 1);  // 快速模式，RTO 不加倍
```

### 2.3 RTO（Retransmission TimeOut）重传机制

**生活类比**：你点外卖，等了 30 分钟还没到，你决定再等 10 分钟，如果还不来就打电话催。

KCP 的 RTO 机制类似：

```cpp
// RTO 计算 —— 就像估算外卖送达时间
// 基础延迟 + 抖动（网络波动）= 这次等待的安全时间
// 如果超时没收到确认，就重发数据
uint32_t rto = baseDelay + jitter;  // baseDelay 是网络往返时间
```

### 2.4 FastACK 快速确认机制

```go
// FastACK 的工作原理 —— 就像高铁检票口刷身份证
// 当收到重复的数据包时，说明对方没收到之前的包
// 于是立即发送 ACK，而不是等到下一个检查周期
// 这样能让丢包快速被发现和重传
```

**FastACK vs 普通 ACK**：

| 特性 | 普通 ACK | FastACK |
|------|---------|---------|
| 触发时机 | 每个包都确认 | 收到重复包才确认 |
| 延迟 | 较高 | 更低 |
| 可靠性 | 高 | 略低（但足够） |

---

## 3. ENet 特性

### 3.1 什么是 ENet？

ENet 是一个 **C 语言编写的可靠 UDP 库**，专为游戏设计，特点是小巧、跨平台、易用。

### 3.2 多通道机制

```cpp
// ENet 支持 256 个通道 —— 就像高速公路有 256 条车道
// 通道 0 通常用于可靠的消息（登录、购买等重要操作）
// 通道 1-255 用于不可靠但有序的数据（位置同步等）
typedef struct _ENetChannel {
    uint8_t channelID;           // 车道编号
    ENetPeer *peer;              // 车辆（连接）
    enet_uint32 windowSize;      // 窗口大小（一次能发多少）
} ENetChannel;
```

### 3.3 有序包 vs 无序包

```cpp
// 有序包：严格按顺序送达，延迟高但不会乱序
// 无序包：先到先得，延迟低但可能乱序

// ENet 的通道可以配置为有序或无序
// 一般实时战斗数据用无序（如移动、射击）
// 重要数据用有序（如道具使用、聊天）

// 代码示例：发送有序包
enet_peer_send(peer, 0, packet);  // 通道 0，有序，可靠

// 代码示例：发送无序包
enet_peer_send(peer, 1, packet);  // 通道 1，无序，不可靠
```

---

## 4. C++ 服务器接收客户端消息循环

### 4.1 使用 ENet 的服务器主循环

```cpp
#include <enet/enet.h>  // 引入 ENet 库头文件
#include <iostream>

// 主循环就像餐厅的服务员，一直等待客人（客户端）点菜（发送消息）
// 当客人点菜时，服务员记录菜单并交给厨房（游戏逻辑处理）

int main() {
    // 步骤 1：初始化 ENet 库 —— 就像打开餐厅大门
    if (enet_initialize() != 0) {
        std::cerr << "ENet 初始化失败！" << std::endl;
        return -1;
    }

    // 步骤 2：创建服务器 —— 就像餐厅准备好桌子
    ENetAddress address;
    address.host = ENET_HOST_ANY;     // 监听所有网卡的 12345 端口
    address.port = 12345;             // 端口号

    ENetHost *server = enet_host_create(
        &address,      // 监听地址
        32,            // 最多同时服务 32 个客户端
        2,            // 最多 2 个频道（通道）
        0,            // 接收带宽（0 表示不限）
        0             // 发送带宽（0 表示不限）
    );

    if (server == nullptr) {
        std::cerr << "创建服务器失败！" << std::endl;
        return -1;
    }

    std::cout << "游戏服务器启动，监听端口 12345..." << std::endl;

    // 步骤 3：主事件循环 —— 服务员一直工作
    ENetEvent event;  // 事件就像客人的各种需求
    while (true) {
        // 等待客户端事件，最多等待 1000 毫秒（1秒）
        // 如果超时，说明这段时间没有客人来
        while (enet_host_service(server, &event, 1000) > 0) {
            switch (event.type) {
                case ENET_EVENT_TYPE_CONNECT: {
                    // 有新客人进门 —— 客户端连接
                    std::cout << "玩家 " << event.peer->address.host 
                              << " 连接到服务器" << std::endl;
                    // 给玩家分配一个唯一 ID（就像餐桌号）
                    event.peer->data = (void*)malloc(sizeof(int));
                    *(int*)event.peer->data = playerID++;
                    break;
                }

                case ENET_EVENT_TYPE_RECEIVE: {
                    // 客人点了菜 —— 收到客户端消息
                    std::cout << "收到来自玩家 " << *(int*)event.peer->data 
                              << " 的消息，长度：" << event.packet->dataLength 
                              << " 字节" << std::endl;

                    // 处理消息（玩家移动、攻击、聊天等）
                    // ProcessGameMessage(event.peer, event.packet);

                    // 处理完消息后，释放数据包内存
                    enet_packet_destroy(event.packet);
                    break;
                }

                case ENET_EVENT_TYPE_DISCONNECT: {
                    // 客人结账离开 —— 客户端断开
                    std::cout << "玩家 " << *(int*)event.peer->data 
                              << " 断开连接" << std::endl;
                    free(event.peer->data);  // 释放我们分配的内存
                    event.peer->data = nullptr;
                    break;
                }
            }
        }
    }

    // 清理工作 —— 关闭餐厅
    enet_host_destroy(server);
    enet_deinitialize();
    return 0;
}
```

### 4.2 消息格式设计

```cpp
// 游戏消息格式 —— 就像餐厅小票的格式
// [消息头：4字节] [消息ID：2字节] [玩家ID：4字节] [消息体：变长]

struct GamePacket {
    uint32_t size;      // 数据包总长度
    uint16_t msgID;     // 消息类型（移动=1，攻击=2，聊天=3...）
    uint32_t playerID;  // 玩家唯一标识
    char data[8192];    // 消息内容（最大 8KB）
};
```

---

## 5. Unity C# 客户端发送接收

### 5.1 封装网络管理器

```csharp
using UnityEngine;
using System;
using System.Net.Sockets;
using System.Collections.Generic;

/// <summary>
/// 网络管理器 —— 就像手机里的外卖App
/// 负责和服务器（餐厅）通信，发送订单（请求）和接收外卖（响应）
/// </summary>
public class NetworkManager
{
    // 单例模式 —— 全局只有一个网络管理器
    public static NetworkManager Instance { get; private set; }

    // KCP 实例 —— 就像手机里的外卖骑手
    private IntPtr kcpHandle;

    // 远程地址 —— 餐厅地址
    private string serverIP = "127.0.0.1";
    private int serverPort = 12345;

    // Unity 主线程的回调队列 —— 就像外卖送到家门口
    private Queue<Action> mainThreadQueue = new Queue<Action>();
    private readonly object queueLock = new object();

    // 玩家的唯一标识
    public uint PlayerID { get; set; }

    /// <summary>
    /// 初始化网络 —— 打开外卖App，填写收货地址
    /// </summary>
    public void Initialize(uint playerID) {
        Instance = this;
        PlayerID = playerID;

        // 创建 KCP 实例，conv（连接号）需要和服务器一致
        // 就像订单号，要双方都对得上
        kcpHandle = KCPBindings.Create(12345);  // conv = 12345

        // 设置发送回调（KCP 会调用这个函数把数据发出去）
        KCPBindings.SetOutput(kcpHandle, OnKCPOutput);

        // 设置更新回调（每帧都要调用 KCP 的 update）
        Debug.Log("网络管理器初始化完成，KCP 连接号：" + 12345);
    }

    /// <summary>
    /// KCP 输出回调 —— 当 KCP 需要发送数据时调用
    /// 就像骑手拿着外卖出发
    /// </summary>
    private void OnKCPOutput(IntPtr data, int length) {
        // 这里需要通过 UDP socket 发送数据
        // SendToServer(data, length);
        Debug.Log($"KCP 输出数据：{length} 字节");
    }

    /// <summary>
    /// 更新网络状态 —— 每帧都要调用，就像心跳
    /// 处理 KCP 的内部逻辑，比如超时重传、确认收到等
    /// </summary>
    public void Update() {
        if (kcpHandle == IntPtr.Zero) return;

        // 更新 KCP 状态
        // currentTime 是当前时间戳（毫秒）
        KCPBindings.Update(kcpHandle, GetCurrentTimeMs());

        // 处理主线程回调
        lock (queueLock) {
            while (mainThreadQueue.Count > 0) {
                mainThreadQueue.Dequeue()?.Invoke();
            }
        }
    }

    /// <summary>
    /// 接收服务器数据 —— 就像收到外卖
    /// </summary>
    public void ReceiveData(byte[] data, int length) {
        // 把数据交给 KCP 处理
        KCPBindings.Input(kcpHandle, data, length);
    }

    /// <summary>
    /// 发送消息给服务器 —— 就像下单点外卖
    /// </summary>
    public void SendMessage(int msgID, byte[] body) {
        if (kcpHandle == IntPtr.Zero) return;

        // 构造消息包：[msgID:4字节] [body:变长]
        byte[] packet = new byte[4 + body.Length];
        BitConverter.GetBytes(msgID).CopyTo(packet, 0);  // 写入消息ID
        body.CopyTo(packet, 4);                            // 写入消息体

        // KCP 发送数据
        KCPBindings.Send(kcpHandle, packet, packet.Length);
    }

    /// <summary>
    /// 获取当前时间（毫秒）—— KCP 内部用于计算 RTO
    /// </summary>
    private uint GetCurrentTimeMs() {
        return (uint)Environment.TickCount;
    }
}
```

### 5.2 Unity 中的消息处理

```csharp
using UnityEngine;
using System;

/// <summary>
/// 消息定义 —— 就像菜单，列出所有可以点的菜
/// </summary>
public static class MsgID {
    public const int Move = 1;          // 移动
    public const int Attack = 2;       // 攻击
    public const int UseItem = 3;      // 使用道具
    public const int Chat = 4;         // 聊天
    public const int Login = 100;      // 登录
    public const int Logout = 101;     // 登出
}

/// <summary>
/// 游戏消息处理中心 —— 就像前台服务员
/// 根据消息类型分发到不同的处理函数
/// </summary>
public class MessageHandler : MonoBehaviour
{
    void Start() {
        // 注册消息处理函数（就像登记每个部门处理什么业务）
        NetworkManager.Instance.OnMessage(MsgID.Move, OnPlayerMove);
        NetworkManager.Instance.OnMessage(MsgID.Attack, OnPlayerAttack);
    }

    /// <summary>
    /// 处理玩家移动消息
    /// </summary>
    private void OnPlayerMove(byte[] data) {
        // 解析位置数据
        float x = BitConverter.ToSingle(data, 0);
        float y = BitConverter.ToSingle(data, 4);
        float z = BitConverter.ToSingle(data, 8);

        // 更新角色位置（可能有预测回滚）
        transform.position = new Vector3(x, y, z);

        Debug.Log($"玩家移动到：({x}, {y}, {z})");
    }

    /// <summary>
    /// 处理玩家攻击消息
    /// </summary>
    private void OnPlayerAttack(byte[] data) {
        // 解析攻击信息：目标ID，技能ID，伤害值
        uint targetID = BitConverter.ToUInt32(data, 0);
        int skillID = BitConverter.ToInt32(data, 4);
        float damage = BitConverter.ToSingle(data, 8);

        Debug.Log($"玩家攻击目标 {targetID}，使用技能 {skillID}，造成 {damage} 伤害");
    }
}
```

---

## 6. 心跳保活机制

### 6.1 为什么需要心跳？

**生活类比**：你给朋友发微信说"在吗？"，如果他 30 秒内没回复，你可能会再发一次或者打电话。
网络连接也是一样，服务器需要确认客户端还"活着"。

### 6.2 心跳机制实现

```cpp
// 心跳就像健康检查 —— 医生定期检查病人是否还活着

// 服务器端：跟踪每个玩家的最后心跳时间
struct PlayerContext {
    ENetPeer* peer;              // 客户端连接
    uint32_t lastHeartbeat;     // 最后心跳时间（毫秒）
    uint32_t timeoutCount;      // 连续超时次数
};

// 服务器主循环中检查心跳
void CheckHeartbeat(ENetHost* server) {
    uint32_t now = GetCurrentTimeMs();

    for (auto& player : players) {
        // 如果 15 秒没收到心跳，发出警告
        if (now - player.lastHeartbeat > 15000) {
            player.timeoutCount++;
            std::cout << "玩家 " << player.id << " 心跳超时 "
                      << player.timeoutCount << " 次" << std::endl;

            // 超过 3 次（45秒），强制断开
            if (player.timeoutCount >= 3) {
                std::cout << "玩家 " << player.id << " 连接超时，强制断开" << std::endl;
                enet_peer_disconnect(player.peer, 0);
            }
        }
    }
}

// 客户端：定期发送心跳包
void SendHeartbeat() {
    static uint32_t lastSend = 0;
    uint32_t now = GetCurrentTimeMs();

    // 每 5 秒发送一次心跳
    if (now - lastSend >= 5000) {
        HeartbeatPacket pkt;
        pkt.type = MSG_HEARTBEAT;
        pkt.timestamp = now;
        SendToServer((byte*)&pkt, sizeof(pkt));
        lastSend = now;
    }
}
```

### 6.3 心跳参数建议

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| 心跳间隔 | 3-5 秒 | 太短浪费带宽，太长发现宕机慢 |
| 超时次数 | 3 次 | 15 秒内没回应就断开 |
| 断线重连窗口 | 30 秒 | 断线后短时间内可以重连恢复 |

---

## 7. KCP vs TCP vs UDP 对比表

| 特性 | TCP | UDP | KCP | ENet |
|------|-----|-----|-----|------|
| **可靠性** | ✅ 可靠 | ❌ 不可靠 | ✅ 可靠 | ✅ 可靠 |
| **有序性** | ✅ 有序 | ❌ 无序 | ✅ 可配置 | ✅ 可配置 |
| **延迟** | 较高 | 最低 | 低 | 低 |
| **带宽利用率** | 较低 | 高 | 高 | 高 |
| **复杂度** | 低 | 最低 | 中 | 中 |
| **适用场景** | 登录、充值 | 语音、视频 | 游戏战斗 | 游戏战斗 |
| **拥塞控制** | 有 | 无 | 可关闭 | 可配置 |
| **MTU 适配** | 自动 | 手动 | 自动 | 自动 |
| **连接数** | 高 | 高 | 高 | 中（内存） |

### 场景选择建议

```
登录/充值  → TCP（可靠，复杂度低）
聊天消息  → TCP 或 KCP 有序
位置同步  → KCP/ENet 无序
语音通话  → UDP（实时性更重要）
```

---

## 8. 代码示例：Go 实现简单 KCP 服务器

### 8.1 Go KCP 服务器

```go
package main

import (
	"fmt"
	"net"
	"time"

	"github.com/skywind3000/kcp-go/v2"
)

// Player 结构体 —— 就像游戏厅里的玩家
type Player struct {
	ID       uint32
	Conn     net.Conn
	LastBeat time.Time
}

// 服务器上下文
type GameServer struct {
	Players   map[uint32]*Player  // 所有在线玩家
	Broadcast chan []byte          // 广播消息队列
}

// 初始化服务器
func NewGameServer() *GameServer {
	return &GameServer{
		Players:   make(map[uint32]*Player),
		Broadcast: make(chan []byte, 1024),
	}
}

// 处理客户端连接
func (s *GameServer) HandleConn(conn net.Conn) {
	// 分配玩家ID
	playerID := uint32(len(s.Players) + 1)

	// 创建玩家记录
	player := &Player{
		ID:       playerID,
		Conn:     conn,
		LastBeat: time.Now(),
	}
	s.Players[playerID] = player

	fmt.Printf("玩家 %d 加入游戏，当前在线：%d 人\n", playerID, len(s.Players))

	// 持续读取客户端数据
	buf := make([]byte, 4096)
	for {
		// 设置读取超时，避免永久阻塞
		conn.SetReadDeadline(time.Now().Add(30 * time.Second))

		// 读取 KCP 数据
		n, err := conn.Read(buf)
		if err != nil {
			fmt.Printf("玩家 %d 连接断开: %v\n", playerID, err)
			delete(s.Players, playerID)
			return
		}

		// 更新心跳时间
		player.LastBeat = time.Now()

		// 处理消息
		s.ProcessMessage(player, buf[:n])
	}
}

// 处理游戏消息
func (s *GameServer) ProcessMessage(player *Player, data []byte) {
	// 消息格式：[msgID:4字节][body:变长]
	if len(data) < 4 {
		return
	}

	msgID := uint32(data[0]) | uint32(data[1])<<8 | uint32(data[2])<<16 | uint32(data[3])<<24

	switch msgID {
	case 1: // 移动消息
		if len(data) >= 16 {
			x := float32(data[4]) | float32(data[5])<<8 | float32(data[6])<<16 | float32(data[7])<<24
			y := float32(data[8]) | float32(data[9])<<8 | float32(data[10])<<16 | float32(data[11])<<24
			z := float32(data[12]) | float32(data[13])<<8 | float32(data[14])<<16 | float32(data[15])<<24
			fmt.Printf("玩家 %d 移动到 (%.2f, %.2f, %.2f)\n", player.ID, x, y, z)
		}
	case 2: // 攻击消息
		fmt.Printf("玩家 %d 发起攻击\n", player.ID)
	default:
		fmt.Printf("玩家 %d 发送未知消息: %d\n", player.ID, msgID)
	}
}

// 检查心跳超时（goroutine 运行）
func (s *GameServer) CheckHeartbeat() {
	ticker := time.NewTicker(5 * time.Second)  // 每5秒检查一次
	defer ticker.Stop()

	for range ticker.C {
		now := time.Now()
		for id, player := range s.Players {
			// 超过30秒没心跳，踢出游戏
			if now.Sub(player.LastBeat) > 30*time.Second {
				fmt.Printf("玩家 %d 心跳超时，强制断开\n", id)
				player.Conn.Close()
				delete(s.Players, id)
			}
		}
	}
}

func main() {
	// 创建 KCP 监听，端口 12345
	addr, err := net.ResolveTCPAddr("tcp", "0.0.0.0:12345")
	if err != nil {
		panic(err)
	}

	// 启动 KCP 服务器
	listener, err := kcp.Listen(addr)
	if err != nil {
		panic(err)
	}

	fmt.Println("KCP 游戏服务器启动，监听端口 12345...")

	server := NewGameServer()
	go server.CheckHeartbeat()  // 启动心跳检查

	// 主循环：接受所有客户端连接
	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Printf("接受连接失败: %v\n", err)
			continue
		}

		// 为每个客户端创建一个 goroutine 处理
		go server.HandleConn(conn)
	}
}
```

### 8.2 Go KCP 客户端

```go
package main

import (
	"fmt"
	"net"
	"time"

	"github.com/skywind3000/kcp-go/v2"
)

// KCP 客户端
type KCPClient struct {
	conn     net.Conn
	playerID uint32
}

func NewKCPClient(serverAddr string) (*KCPClient, error) {
	// 连接 KCP 服务器
	conn, err := kcp.Dial(serverAddr)
	if err != nil {
		return nil, fmt.Errorf("连接服务器失败: %v", err)
	}

	client := &KCPClient{
		conn: conn,
	}
	return client, nil
}

// 发送消息
func (c *KCPClient) SendMessage(msgID uint32, data []byte) error {
	// 构造数据包：[msgID:4字节][data:变长]
	packet := make([]byte, 4+len(data))
	packet[0] = byte(msgID & 0xFF)
	packet[1] = byte((msgID >> 8) & 0xFF)
	packet[2] = byte((msgID >> 16) & 0xFF)
	packet[3] = byte((msgID >> 24) & 0xFF)
	copy(packet[4:], data)

	_, err := c.conn.Write(packet)
	return err
}

// 发送移动消息
func (c *KCPClient) SendMove(x, y, z float32) error {
	// x, y, z 各占4字节
	data := make([]byte, 12)
	putFloat32(data, 0, x)
	putFloat32(data, 4, y)
	putFloat32(data, 8, z)
	return c.SendMessage(1, data)
}

// 辅助函数：写入 float32
func putFloat32(b []byte, offset int, v float32) {
	bits := float32bits(v)
	b[offset] = byte(bits & 0xFF)
	b[offset+1] = byte((bits >> 8) & 0xFF)
	b[offset+2] = byte((bits >> 16) & 0xFF)
	b[offset+3] = byte((bits >> 24) & 0xFF)
}

// 模拟 Unity 客户端
func main() {
	client, err := NewKCPClient("127.0.0.1:12345")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("连接服务器成功")

	// 模拟玩家移动
	for i := 0; i < 10; i++ {
		x := float32(i) * 0.5
		y := float32(0)
		z := float32(i) * 0.3

		err := client.SendMove(x, y, z)
		if err != nil {
			fmt.Printf("发送失败: %v\n", err)
			break
		}
		fmt.Printf("发送移动: (%.2f, %.2f, %.2f)\n", x, y, z)
		time.Sleep(200 * time.Millisecond)  // 每200ms发送一次
	}

	client.conn.Close()
}
```

---

## 本章小结

本章我们学习了游戏网络通信的核心知识：

| 知识点 | 关键收获 |
|--------|----------|
| **KCP 协议** | 可靠 UDP，通过 RTO 重传和 FastACK 实现低延迟数据传递 |
| **ENet 库** | 多通道设计，有序/无序包，适合实时游戏 |
| **服务器消息循环** | 事件驱动模型，处理连接、消息、断开三种事件 |
| **Unity C# 客户端** | KCP 绑定，消息封装，主线程回调分发 |
| **心跳保活** | 定期检查连接状态，超时断开，保证服务稳定性 |
| **协议选择** | TCP 适合登录充值，KCP/ENet 适合实时战斗 |

下一章我们将学习 **客户端预测与回滚**，实现更流畅的游戏体验。

---

> 💡 **实战建议**：先从 TCP 开始开发游戏原型，等功能稳定后再切换到 KCP/ENet，这样可以降低早期调试难度。等你熟练掌握 UDP 的可靠传输后，可以进一步优化游戏体验。