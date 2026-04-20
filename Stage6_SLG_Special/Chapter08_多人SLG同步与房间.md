# Chapter 08：多人 SLG 同步与房间

> 🎯 目标：掌握多人 SLG 的房间系统和网络同步——创建房间、加入房间、回合同步、观战、Reconnect。

---

## 8.1 房间系统

```csharp
// -------- 房间 --------
[System.Serializable]
public class Room
{
    public string roomID;
    public string roomName;
    public int hostPlayerID;
    public List<int> playerIDs = new List<int>();
    public int maxPlayers = 8;
    public RoomState state = RoomState.Waiting;
    public MapConfig mapConfig;
    public TurnMode turnMode = TurnMode.RealTime;  // RealTime 或 TurnBased
}

public enum RoomState {
    Waiting,    // 等待中
    Loading,     // 加载地图
    Playing,     // 游戏中
    Finished     // 已结束
}

// -------- 房间服务器 --------
public class RoomServer
{
    private Dictionary<string, Room> rooms = new Dictionary<string, Room>();

    public Room CreateRoom(int hostID, string name, MapConfig map) {
        Room room = new Room {
            roomID = GenerateID(),
            roomName = name,
            hostPlayerID = hostID,
            mapConfig = map
        };
        rooms[room.roomID] = room;
        return room;
    }

    public bool JoinRoom(string roomID, int playerID) {
        if (!rooms.ContainsKey(roomID)) return false;
        Room room = rooms[roomID];

        if (room.playerIDs.Count >= room.maxPlayers)
            return false;

        room.playerIDs.Add(playerID);
        BroadcastToRoom(room, new PlayerJoinedMsg { playerID = playerID });
        return true;
    }
}
```

---

## 8.2 回合制多人同步

```csharp
// -------- 回合制多人同步 --------
public class TurnSyncServer
{
    public TurnState currentTurn = new TurnState {
        turnNumber = 0,
        currentPlayerIndex = 0,
        phase = TurnPhase.Idle
    };

    private Dictionary<int, PlayerTurnAction> pendingActions
        = new Dictionary<int, PlayerTurnAction>();

    // -------- 玩家提交操作 --------
    public void SubmitAction(int playerID, PlayerTurnAction action) {
        if (currentTurn.phase != TurnPhase.WaitingForActions)
            return;

        pendingActions[playerID] = action;

        // -------- 收到所有玩家操作后执行 --------
        if (pendingActions.Count >= GetLivingPlayerCount()) {
            ExecuteTurn();
        }
    }

    void ExecuteTurn() {
        currentTurn.phase = TurnPhase.Executing;

        // 按速度/优先级排序后执行
        var sorted = pendingActions.Values.OrderByDescending(a => a.priority);
        foreach (var action in sorted) {
            ExecuteAction(action);
        }

        pendingActions.Clear();
        AdvanceTurn();
    }

    void AdvanceTurn() {
        currentTurn.turnNumber++;
        currentTurn.currentPlayerIndex = 0;
        currentTurn.phase = TurnPhase.WaitingForActions;
        Broadcast(new TurnStartMsg { turn = currentTurn });
    }
}
```

---

## 8.3 观战系统

```csharp
// -------- 观战 --------
public class SpectatorManager
{
    private List<int> spectators = new List<int>();

    public void AddSpectator(int playerID) {
        if (!spectators.Contains(playerID)) {
            spectators.Add(playerID);
        }
    }

    public void BroadcastToSpectators(MsgBase msg) {
        foreach (int sid in spectators) {
            SendToPlayer(sid, msg);
        }
    }
}
```

---

## 8.4 Reconnect 处理

```csharp
// -------- 断线重连 --------
public class ReconnectHandler
{
    public void HandleReconnect(int playerID) {
        // 1. 重新加载玩家数据
        PlayerData data = LoadPlayerData(playerID);

        // 2. 重新加入房间
        Room room = GetPlayerRoom(playerID);
        if (room != null && room.state == RoomState.Playing) {
            // 3. 发送完整游戏状态
            SendFullState(playerID, room);

            // 4. 广播玩家重新上线
            Broadcast(new PlayerReconnectedMsg { playerID });
        }
    }

    void SendFullState(int playerID, Room room) {
        SendToPlayer(playerID, new FullStateMsg {
            turn = room.currentTurn,
            units = room.allUnits,
            resources = room.resources,
            buffs = room.activeBuffs
        });
    }
}
```

---

## 8.5 房间客户端

```csharp
// -------- 房间客户端 --------
public class RoomClient : MonoBehaviour
{
    public void CreateRoom(string name) {
        NetworkClient.Send(new CreateRoomReq { roomName = name });
    }

    public void JoinRoom(string roomID) {
        NetworkClient.Send(new JoinRoomReq { roomID = roomID });
    }

    void OnReceivePlayerList(SPlayerListMsg msg) {
        // 更新房间玩家列表 UI
        UIManager.Instance.UpdatePlayerList(msg.players);
    }

    void OnReceiveGameStart(SGameStartMsg msg) {
        // 加载地图，进入游戏
        SceneManager.LoadScene("GameScene");
    }

    // -------- 回合制 --------
    public void SubmitTurnAction(TurnAction action) {
        NetworkClient.Send(new TurnActionMsg { action = action });
    }

    void OnReceiveYourTurn(SYourTurnMsg msg) {
        // 显示玩家操作界面
        UIManager.Instance.ShowTurnUI(true);
    }
}
```

---

## 8.6 本章小结

```
✅ 已掌握：
├── 房间系统（创建/加入/房主）
├── 回合制多人同步（等待所有玩家操作）
├── 观战系统
├── Reconnect 断线重连
├── 房间服务器架构
└章 多人 SLG 完成！

🎉 Stage 6 SLG 专题 —— 完成！
🎉 全部两个专题完成！
```

---

_📚 参考资料：《Unity Multiplayer》《Mirror Networking》_
