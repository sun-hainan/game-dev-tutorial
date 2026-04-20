# Chapter 02：Unity 高级技术

> 🎯 目标：掌握 Unity 网络开发、热更新、性能优化、Shader 基础、Addressables 资源系统。

---

## 2.1 Unity 网络开发

### 多人游戏基础概念

| 概念 | 说明 |
|-----|------|
| **Server/Client** | 服务器-客户端模式 |
| **P2P** | 点对点，每人都是服务器 |
| **RPC** | 远程过程调用（调用其他机器上的函数）|
| **SyncVar** | 同步变量（自动同步到所有客户端）|
| **State Sync** | 状态同步 |
| **Input Sync** | 输入同步 |

### Unity 内置网络（UNet，已废弃）

> UNet 已弃用，不推荐新项目使用。

### Mirror（推荐）

> **Mirror = Unity 最流行的开源网络库**，简单易用，适合中小型游戏。

#### 安装 Mirror

```
1. Window → Package Manager
2. 点击 + → Add package from git URL
3. 输入：https://github.com/vis2k/Mirror.git
4. 或者下载 .unitypackage 导入
```

#### Mirror 基本用法

```csharp
using UnityEngine;
using Mirror;

// -------- 网络管理器（每个场景一个）--------
public class NetworkManager : MonoBehaviour
{
    // -------- 玩家预制体 --------
    [SerializeField] private GameObject playerPrefab;

    // -------- 启动服务器 --------
    public void StartServer()
    {
        NetworkServer.Listen(7777);
        NetworkServer.RegisterHandler<CreateCharacterMessage>(OnCreateCharacter);
    }

    // -------- 启动客户端 --------
    public void StartClient()
    {
        NetworkClient.RegisterHandler<CreateCharacterMessage>(OnCreateCharacter);
        NetworkClient.Connect("127.0.0.1", 7777);
    }

    // -------- 生成玩家 --------
    public void CreateCharacter(NetworkConnection conn, CreateCharacterMessage message)
    {
        GameObject player = Instantiate(playerPrefab);
        NetworkServer.AddPlayerForConnection(conn, player);
    }

    void OnCreateCharacter(NetworkConnection conn, CreateCharacterMessage msg)
    {
        CreateCharacter(conn, msg);
    }
}

// -------- 玩家网络脚本 --------
public class PlayerNetwork : NetworkBehaviour
{
    [SyncVar] private Vector3 position;   // 同步位置
    [SyncVar] private int hp;              // 同步 HP

    void Update()
    {
        if (isLocalPlayer)  // 只有本地玩家可以控制
        {
            float h = Input.GetAxis("Horizontal");
            float v = Input.GetAxis("Vertical");
            CmdMove(new Vector3(h, 0, v));
        }
        else
        {
            // 其他玩家：直接设置位置
            transform.position = position;
        }
    }

    // -------- 命令（客户端调用，服务器执行）--------
    [Command]
    void CmdMove(Vector3 direction)
    {
        transform.position += direction * 5f * Time.deltaTime;
        position = transform.position;   // 同步给其他客户端
    }
}
```

#### Transform 同步

```csharp
public class PlayerTransformSync : NetworkBehaviour
{
    [SerializeField] private float syncInterval = 0.1f;   // 同步间隔

    void Update()
    {
        if (isLocalPlayer)
        {
            // 本地玩家：发送位置到服务器
            CmdSendPosition(transform.position);
        }
    }

    // -------- 命令 --------
    [Command]
    void CmdSendPosition(Vector3 pos)
    {
        RpcUpdatePosition(pos);   // 广播给所有客户端
    }

    // -------- 客户端回调 --------
    [ClientRpc]
    void RpcUpdatePosition(Vector3 pos)
    {
        if (!isLocalPlayer)
        {
            transform.position = Vector3.Lerp(transform.position, pos, Time.deltaTime * 10f);
        }
    }
}
```

---

## 2.2 热更新

### 什么是热更新？

> **热更新 = 不重新安装 APP，在运行时更新游戏代码/资源。**

### Unity 热更新方案对比

| 方案 | 原理 | 优点 | 缺点 |
|-----|------|------|------|
| **XLua** | Lua 脚本 | 成熟稳定，生态好 | 需要学 Lua |
| **ILRuntime** | C# JIT | 纯 C# | 学习成本高 |
| **ToLua** | Lua | 性能好 | 已停止维护 |
| **Addressables** | 资源热更 | 官方推荐 | 不能热更代码 |

### XLua 基础用法

```csharp
using XLua;

// -------- C# 端 --------
public class XLuaManager
{
    private LuaEnv luaEnv;

    void Start()
    {
        luaEnv = new LuaEnv();
        luaEnv.DoString("print('Hello from Lua!')");

        // -------- 加载 Lua 文件 --------
        luaEnv.DoString(require("Main"));
    }

    // -------- 调用 Lua 函数 --------
    public void CallLuaFunction()
    {
        LuaFunction func = luaEnv.Global.GetInPath<LuaFunction>("CS.GameMain.OnUpdate");
        func.Call();
    }

    void OnDestroy()
    {
        luaEnv.Dispose();
    }
}
```

### Lua 脚本示例

```lua
-- Main.lua
local M = {}

function M.OnUpdate()
    print("Lua Update called!")
end

function M.OnAttack(targetName, damage)
    print("Attack " .. targetName .. " for " .. damage .. " damage!")
end

return M
```

### Addressables 资源热更（不需要代码热更）

```csharp
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;

// -------- 异步加载资源 --------
public IEnumerator LoadAssetAsync<T>(string address, System.Action<T> onLoaded) where T : Object
{
    AsyncOperationHandle<T> handle = Addressables.LoadAssetAsync<T>(address);
    yield return handle;

    if (handle.Status == AsyncOperationStatus.Succeeded)
    {
        onLoaded?.Invoke(handle.Result);
    }
    else
    {
        Debug.LogError($"Failed to load {address}: {handle.OperationException}");
    }

    Addressables.Release(handle);
}

// -------- 使用 --------
void Start()
{
    StartCoroutine(LoadAssetAsync<Sprite>("EnemyIcons/Goblin", (sprite) => {
        enemyIcon.sprite = sprite;
    }));
}
```

---

## 2.3 性能优化

### Profiler 使用

```
1. Window → Analysis → Profiler
2. 播放游戏，查看各项指标：
   - CPU：哪部分代码最耗时
   - Rendering：Draw Call 数量
   - Memory：内存分配
   - Audio：音频占用
```

### Draw Call 批处理

```csharp
// -------- 静态批处理（不需要移动的物体）--------
// 在 Inspector 中：Static → Contiguous Static
// Unity 会自动合并相同材质的静态物体

// -------- 动态批处理（移动的物体）--------
// 条件：
// - 顶点数 < 900
// - 使用相同材质
// - 不受光照影响（Realtime Light）
// Unity 自动合并

// -------- GPU Instancing（大量相同物体）--------
MaterialPropertyBlock props = new MaterialPropertyBlock();
props.SetFloat("_Position", Time.time);

Graphics.DrawMeshInstanced(
    mesh, 0,
    material,
    matrices,    // 矩阵数组
    count,
    props
);
```

### 对象池优化

```csharp
// -------- 对象池核心代码（已在 Stage 2 讲过）--------
// 关键点：
// - 预创建足够数量（根据同屏最大数量）
// - 取/还 O(1)
// - 不要频繁 Instantiate/Destroy
```

### 代码优化

```csharp
// -------- 避免每帧分配内存 --------
void BadExample()
{
    // ❌ 每次 new 一个 Vector3
    transform.position = new Vector3(x, y, z);

    // ❌ 字符串拼接（每帧分配）
    Debug.Log("Player " + playerName + " at " + position);
}

void GoodExample()
{
    // ✅ 复用缓存的 Vector3
    cachedPosition.Set(x, y, z);
    transform.position = cachedPosition;

    // ✅ 字符串插值（编译时优化）
    Debug.Log($"Player {playerName} at {position}");
}

// -------- 避免 LINQ 在 Update 中使用 --------
void Update()
{
    // ❌ LINQ 在热路径上会有 GC
    var enemies = GameObject.FindGameObjectsWithTag("Enemy")
        .Where(e => e.GetComponent<Health>().hp > 0)
        .ToList();

    // ✅ 手动循环，避免 GC
    Enemy[] allEnemies = GetEnemyArray();
    for (int i = 0; i < count; i++)
    {
        if (allEnemies[i].hp > 0) { ... }
    }
}

// -------- Cache 查询结果 --------
private Collider[] cachedColliders = new Collider[100];

void FixedUpdate()
{
    // ✅ 复用数组，不每次 new
    int count = Physics.OverlapSphereNonAlloc(
        transform.position, radius, cachedColliders);
}
```

### LOD（Level of Detail）

```
设置：
1. LOD Group 组件添加到对象
2. 设置不同距离的模型（LOD0=近/LOD1=中/LOD2=远）
3. 相机越远，使用面数越少的模型

Unity 默认：
- LOD 0: 100%
- LOD 1: 50%
- LOD 2: 25%
- LOD 3: 12.5%
```

---

## 2.4 Shader 基础

### Shader 类型

| 类型 | 适用场景 |
|-----|---------|
| **Surface Shader** | Unity 封装，最常用 |
| **Vertex-Fragment Shader** | 底层控制，性能最好 |
| **Compute Shader** | GPU 并行计算 |

### 简单 Surface Shader（溶解效果）

```csharp
Shader "Custom/Dissolve"
{
    Properties
    {
        _Color ("Color", Color) = (1,1,1,1)
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _DissolveThreshold ("Dissolve", Range(0,1)) = 0
        _EdgeColor ("Edge Color", Color) = (1,0,0,1)
        _EdgeWidth ("Edge Width", Range(0,0.1)) = 0.02
    }

    SubShader
    {
        Tags { "RenderType"="Opaque" }
        LOD 200

        CGPROGRAM
        #pragma surface surf Standard fullforwardshadows

        sampler2D _MainTex;
        fixed4 _Color;
        float _DissolveThreshold;
        fixed4 _EdgeColor;
        float _EdgeWidth;

        struct Input
        {
            float2 uv_MainTex;
            float3 worldPos;
        };

        void surf (Input IN, inout SurfaceOutputStandard o)
        {
            // -------- 溶解噪声 --------
            float noise = tex2D(_MainTex, IN.uv_MainTex).r;

            if (noise < _DissolveThreshold)
            {
                discard;
            }

            // -------- 边缘发光 --------
            float edge = _DissolveThreshold + _EdgeWidth;
            if (noise < edge && noise >= _DissolveThreshold)
            {
                o.Emission = _EdgeColor.rgb;
            }

            fixed4 c = tex2D(_MainTex, IN.uv_MainTex) * _Color;
            o.Albedo = c.rgb;
        }
        ENDCG
    }
    FallBack "Diffuse"
}
```

### 简单 Vertex-Fragment Shader（颜色渐变）

```csharp
Shader "Custom/VertexColor"
{
    SubShader
    {
        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            struct appdata
            {
                float4 vertex : POSITION;
                float4 color : COLOR;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float4 color : COLOR;
            };

            v2f vert(appdata v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.color = v.color;   // 传递顶点颜色
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                return i.color;
            }
            ENDCG
        }
    }
}
```

### Shader Graph 简介

```
创建方法：
1. Project → Create → Shader → Unlit Graph（或 PBR Graph）
2. 双击打开 Shader Graph 编辑器
3. 拖入节点（Node）：
   - Texture Sample（纹理）
   - Color（颜色）
   - Multiply（乘法）
   - Lerp（插值）
   - UV（贴图坐标）
4. 连接节点到 Base Color
5. 点击 Save Asset

适用场景：
- 无需写代码的视觉效果
- 快速迭代
- 设计师友好
```

### URP 自定义 Shader

```csharp
// URP 简单无光着色器
Shader "Custom/URPUnlit"
{
    Properties
    {
        _BaseColor ("Base Color", Color) = (1,1,1,1)
        _BaseMap ("Base Map", 2D) = "white" {}
    }

    SubShader
    {
        Tags { "RenderPipeline" = "UniversalPipeline" }

        Pass
        {
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

            struct Attributes
            {
                float4 position : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct Varyings
            {
                float4 position : SV_POSITION;
                float2 uv : TEXCOORD0;
            };

            TEXTURE2D(_BaseMap);
            SAMPLER(sampler_BaseMap);
            CBUFFER_START(UnityPerMaterial)
                float4 _BaseColor;
            CBUFFER_END

            Varyings vert(Attributes input)
            {
                Varyings output;
                output.position = TransformObjectToHClip(input.position.xyz);
                output.uv = input.uv;
                return output;
            }

            half4 frag(Varyings input) : SV_Target
            {
                half4 baseMap = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, input.uv);
                return baseMap * _BaseColor;
            }
            ENDHLSL
        }
    }
}
```

---

## 2.5 Addressables 资源系统

### Resource vs Addressables

| 维度 | Resources | Addressables |
|-----|----------|--------------|
| 加载 | 同步/异步 | 异步 |
| 内存 | 全加载到内存 | 按需加载 |
| 更新 | 重新打包 | 可热更新 |
| 打包 | Resources 目录 | Addressables Group |

### Addressables 使用流程

```
1. Window → Asset Management → Addressables → Groups
2. 创建 Addressables Group
3. 把资源拖入 Group
4. 给资源设置 Address（标签名）
5. 代码中通过 Address 加载
```

### 完整示例

```csharp
using UnityEngine;
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;
using UnityEngine.ResourceManagement.ResourceLocations;

public class AddressablesManager : MonoBehaviour
{
    // -------- 加载单个资源 --------
    public void LoadEnemyPrefab(string address, System.Action<GameObject> onComplete)
    {
        AsyncOperationHandle<GameObject> handle =
            Addressables.LoadAssetAsync<GameObject>(address);

        handle.Completed += (op) =>
        {
            if (op.Status == AsyncOperationStatus.Succeeded)
            {
                onComplete?.Invoke(op.Result);
            }
            else
            {
                Debug.LogError($"Load failed: {op.OperationException}");
            }
        };
    }

    // -------- 加载多个资源 --------
    public void LoadMultiple<T>(string[] addresses, System.Action<T[]> onComplete) where T : Object
    {
        AsyncOperationHandle<T[]> handle =
            Addressables.LoadAssetsAsync<T>(addresses, null, Addressables.MergeMode.Union);

        handle.Completed += (op) =>
        {
            if (op.Status == AsyncOperationStatus.Succeeded)
            {
                onComplete?.Invoke(op.Result);
            }
        };
    }

    // -------- 释放资源 --------
    public void Release<T>(T obj) where T : Object
    {
        Addressables.Release(obj);
    }

    // -------- 异步加载带进度 --------
    public IEnumerator LoadWithProgress<T>(string address, System.Action<T> onComplete,
        System.Action<float> onProgress)
    {
        AsyncOperationHandle<T> handle = Addressables.LoadAssetAsync<T>(address);

        while (!handle.IsDone)
        {
            onProgress?.Invoke(handle.PercentComplete);
            yield return null;
        }

        if (handle.Status == AsyncOperationStatus.Succeeded)
        {
            onComplete?.Invoke(handle.Result);
        }
    }
}
```

---

## 2.6 常见性能陷阱

| 陷阱 | 后果 | 解决方法 |
|-----|------|---------|
| 每帧 `GameObject.Find()` | O(n) 每帧调用，卡顿 | Cache 结果或用 Tag 查找一次 |
| 每帧 `Instantiate/Destroy` | GC 峰值，帧率抖动 | 用对象池 |
| 每帧 `string.Format` | 内存分配 | 预分配或字符串插值 |
| Realtime GI + 大量面数 | GPU 过载 | 烘焙光照，降低 Realtime 质量 |
| 不设置 Texture Read/Write | Copy Memory | 需要 Readable 时勾选 |
| 使用 `GetComponent` 每帧 | CPU 开销 | 在 Start/Awake 中 Cache |

---

## 2.7 动手练习 🧪

### 练习 1：Mirror 玩家移动 ⭐⭐
```
安装 Mirror
实现玩家可以在局域网内同步移动
用 [SyncVar] 同步位置
```

### 练习 2：XLua 热更新 ⭐⭐
```
安装 XLua
创建 Lua 脚本定义技能
C# 调用 Lua 函数
```

### 练习 3：性能优化实战 ⭐⭐⭐
```
在 Unity Profiler 中找到卡顿原因
使用对象池优化子弹
减少 Draw Call
```

### 练习 4：编写溶解 Shader ⭐⭐⭐
```
参考本章 Surface Shader 代码
实现从边缘向中心溶解的效果
边缘有发光颜色
```

### 练习 5：Addressables 资源管理 ⭐⭐
```
创建 Addressables Group
实现异步加载怪物预制体
实现卸载资源
```

---

## 2.8 本章小结

```
✅ 已掌握：
├── Mirror 网络开发（Server/Client/RPC/SyncVar）
├── 热更新概念（XLua / Addressables）
├── Profiler 性能分析
├── Draw Call 批处理
├── 代码优化（避免 GC）
├── LOD 细节层级
├── Shader 基础（Surface/Vertex-Fragment）
├── HLSL 语法入门
├── Shader Graph 可视化
├── URP 自定义 Shader
├── Addressables 异步加载
└── 常见性能陷阱

🎉 Stage 3 进阶内容完成！
```

---

_📚 参考资料：《Unity 官方文档 - Netcode》《Mirror GitHub》《HLSL 编程指南》_
