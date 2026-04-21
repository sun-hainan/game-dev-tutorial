# Stage 5：MMO 服务器专题

> 目标：掌握网络游戏同步核心技（帧同步/状态同步），理解 GameServer 架构，成为游戏服务器开发者。

## 📖 章节列表

| # | 章节 | 核心内容 |
|---|------|---------|
| 01 | 帧同步状态同步 | 帧同步 vs 状态同步原理，适合的游戏类型 |
| 02 | KCP 与 ENet 底层网络 | KCP 可靠UDP，ENet 封装，网络协议选择 |
| 03 | 客户端预测与帧同步回滚 | 客户端预测，延迟补偿，帧间回滚 |
| 04 | GameServer 游戏服务器架构 | 多进程架构，Gate/Logic/DB 分层 |
| 05 | Redis 缓存与数据库 | Redis 高速缓存，MySQL 数据库设计 |
| 06 | 登录鉴权安全 | Token/JWT，登录流程，安全防护 |
| 07 | 房间匹配与战斗 | 房间管理，匹配算法，战斗服务 |
| 08 | Docker 部署游戏服务器 | Docker/K8s，容器化部署，运维基础 |

## 🎯 学习目标

- 理解帧同步与状态同步的技术细节
- 掌握 KCP/ENet 等可靠 UDP 传输
- 能够设计并实现 GameServer 多进程架构
- 掌握 Docker 部署与运维基础

## 📂 文件结构

```
Stage5_MMO_Special/
├── README.md
├── Chapter01_帧同步状态同步.md
├── Chapter02_KCP与ENet底层网络.md
├── Chapter03_客户端预测与帧同步回滚.md
├── Chapter04_GameServer游戏服务器架构.md
├── Chapter05_Redis缓存与数据库.md
├── Chapter06_登录鉴权安全.md
├── Chapter07_房间匹配与战斗.md
└── Chapter08_Docker部署游戏服务器.md
```

## 🔗 前置要求

- Stage 3 设计模式基础
- 网络基础知识（TCP/UDP）
- 推荐先完成 Stage 2 Unity 基础