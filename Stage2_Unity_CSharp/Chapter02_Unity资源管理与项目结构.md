# Chapter 02：Unity 资源管理与项目结构

> 🎯 目标：理解 Unity 的资源管理机制、掌握素材导入规范、学会合理组织项目结构。

---

## 2.1 Unity 资源系统概述

### 生活中的类比 📦

> **Unity 资源 = 你的游戏零件箱：**
> - `.fbx` 模型文件 = 3D 打印件
> - `.png/.jpg` 图片 = 贴纸
> - `.mp3/.wav` 音频 = 背景音乐和音效
> - `.cs` 脚本 = 操作手册（告诉零件怎么动）
> - **所有资源都在 `Assets` 文件夹下**

### 资源工作流程

```
源资源（Source Assets）          导入后（Imported Assets）
─────────────────              ────────────────────────
PSD/AI 文件              →     自动切成 Sprite 或 Texture
FBX/OBJ 模型文件         →     自动生成 Mesh + Materials
.mp3/.wav 音频           →     音频 Clip（可设置压缩格式）
.cs 脚本                 →     实时编译，出现在 Inspector 中
```

---

## 2.2 素材导入规范

### 2D 素材（Sprite）

```csharp
// 推荐格式：PNG（支持透明）
// 建议分辨率：2^n（32, 64, 128, 256, 512, 1024, 2048）

// 导入设置：
// 1. 选中图片 → Inspector
// 2. Texture Type → Sprite (2D and UI)
// 3. Sprite Mode → Single（单图）或 Multiple（切分多图）
// 4. Pixels Per Unit → 100（重要！决定图片在 Unity 中的大小）
```

### 3D 模型素材

```csharp
// 推荐格式：FBX（跨平台通用）
// OBJ 也支持，但不支持动画

// 导入设置：
// 1. 选中 .fbx → Inspector
// 2. Model 选项卡：
//    - Scale Factor → 根据导出时的单位设置
//    - Generate Colliders → 勾选（自动生成碰撞体）
// 3. Rig 选项卡：
//    - Animation Type → 根据需求选择（None/Humanoid/Generic）
```

### 音频素材

```csharp
// 两种模式：
// - Decompress On Load：加载时解压（适合小音效，短音频）
// - Compressed In Memory：运行时解压（适合背景音乐，节省内存）

// 设置：
// 1. 选中音频文件
// 2. Load Type：
//    - BGM → Streaming（流式，边播放边加载）
//    - 音效 → Decompress On Load（短促响应快）
// 3. Compression → Vorbis（通用压缩格式）
```

---

## 2.3 推荐项目结构

### 为什么要规范项目结构？

> **规范 = 团队合作的基础 + 自己找文件的效率**

### 标准的 Unity 项目结构

```
Assets/
│
├── 📁 Scenes/                  ← 所有场景文件
│   ├── Scene1.unity
│   ├── Scene2.unity
│   └── LoadingScene.unity
│
├── 📁 Scripts/                  ← 所有 C# 脚本
│   ├── 📁 Core/                 ← 核心系统（不依赖游戏逻辑）
│   │   ├── GameManager.cs
│   │   ├── AudioManager.cs
│   │   └── SaveManager.cs
│   ├── 📁 Player/               ← 玩家相关
│   │   ├── PlayerController.cs
│   │   ├── PlayerAttack.cs
│   │   └── PlayerHealth.cs
│   ├── 📁 Enemy/                 ← 敌人相关
│   │   ├── EnemyBase.cs
│   │   ├── EnemyAI.cs
│   │   └── BossController.cs
│   └── 📁 UI/                   ← UI 相关
│       ├── UIManager.cs
│       ├── HealthBar.cs
│       └── MainMenu.cs
│
├── 📁 Prefabs/                  ← 预制体（可复用对象）
│   ├── Player.prefab
│   ├── Enemy.prefab
│   ├── Bullet.prefab
│   └── UI/
│       └── HealthBar.prefab
│
├── 📁 Materials/                ← 材质
│   ├── Ground.mat
│   └── Player.mat
│
├── 📁 Models/                   ← 3D 模型
│   ├── Character.fbx
│   └── Environment.fbx
│
├── 📁 Sprites/                  ← 2D 图片
│   ├── 📁 Characters/
│   ├── 📁 Backgrounds/
│   └── 📁 Items/
│
├── 📁 Audio/                    ← 音频
│   ├── 📁 BGM/
│   └── 📁 SFX/
│
├── 📁 Animations/               ← 动画控制器和动画片段
│   ├── PlayerAC.controller
│   ├── PlayerIdle.anim
│   └── PlayerRun.anim
│
├── 📁 ScriptableObjects/         ← ScriptableObject 数据资产
│   ├── ItemData.cs（定义数据模板）
│   └── EnemyWaveData.cs
│
├── 📁 Textures/                ← 纹理（3D 模型用）
│
├── 📁 Fonts/                    ← 字体
│
└── 📁 ThirdParty/               ← 第三方插件
    └── DOTween/
```

---

## 2.4 ScriptableObject：数据资产

### 什么是 ScriptableObject？

> **ScriptableObject = Unity 内置的"数据模板"系统。**
> 你可以创建.asset 文件作为数据资产，多个对象可以共享同一份数据，节省内存。

### 定义一个道具数据模板

```csharp
// File: Assets/Scripts/ItemData.cs
using UnityEngine;

// -------- 创建资产菜单项 --------
[CreateAssetMenu(fileName = "NewItem", menuName = "GameData/Item")]
public class ItemData : ScriptableObject
{
    public string itemName;     // 道具名称
    public Sprite icon;         // 图标
    public int itemID;          // 唯一ID
    public ItemType type;       // 道具类型

    [TextArea]                 // Inspector 里显示成多行文本框
    public string description;  // 描述

    public int maxStack;        // 最大堆叠数量
    public int sellPrice;       // 卖出价格

    public enum ItemType
    {
        Weapon,
        Armor,
        Potion,
        Material,
        Quest
    }
}
```

### 创建数据资产实例

```
1. 在 Project 窗口右键
2. Create → GameData → Item（你定义的菜单项）
3. 命名为 IronSword.asset
4. 填入属性：Name="铁剑", ID=1001, Type=Weapon...
```

### 在代码中使用

```csharp
// -------- 在 Inspector 中拖入 ----------
public class InventorySlot : MonoBehaviour
{
    public ItemData item;    // 在 Inspector 中拖入对应的 .asset 文件

    public void Display()
    {
        if (item != null)
        {
            Debug.Log("道具名：" + item.itemName);
            Debug.Log("描述：" + item.description);
        }
    }
}
```

---

## 2.5 Package Manager（包管理器）

### 内置包 vs 自定义包

| 类型 | 说明 |
|-----|------|
| 内置包 | Unity 自带（Physics, UI, Analytics 等）|
| 注册表包 | Unity 技术社区分享的包 |
| 自定义包 | `.unitypackage` 或 `.tgz` 导入 |

### 常用包

```
1. TextMeshPro（推荐必装）：
   Window → Package Manager → TextMeshPro → Install
   效果比默认 Text 好很多，支持富文本

2. DOTween（动画插件）：
   最常用的补间动画插件
   安装：下载 dotween.unitypackage，双击导入

3. ProBuilder（场景编辑器）：
   在 Unity 内直接编辑 3D 场景
   Window → Package Manager → ProBuilder → Install
```

### 导出和导入自定义包

```
导出：
1. 在 Project 窗口选中要导出的资源
2. Assets → Export Package...
3. 选择包含依赖（Include dependencies）
4. 保存为 .unitypackage

导入：
1. Assets → Import Package → Custom Package...
2. 选择 .unitypackage 文件
```

---

## 2.6 版本控制（Git）

### .gitignore（Unity 项目必须）

```gitignore
# Unity 生成的
/[Ll]ibrary/
/[Tt]emp/
/[Oo]bj/
/[Bb]uild/
/[Bb]uilds/
/[Ll]ogs/
/[Uu]ser[Ss]ettings/

# 敏感文件
/*.pem
*.key

# Visual Studio
.vs/
*.csproj
*.sln
*.user

# 只有当你的项目不依赖 .meta 文件时才加这行
# .meta
```

### Git LFS（大文件管理）

```bash
# 安装 Git LFS
git lfs install

# 跟踪大文件类型
git lfs track "*.png"
git lfs track "*.jpg"
git lfs track "*.fbx"
git lfs track "*.mp3"
git lfs track "*.wav"

# .gitattributes 会自动生成
```

### GitHub + Unity 协作规范

```
Commit 规范：
- 每完成一个功能或修复一个 bug 提交一次
- 提交信息：feat: 新增玩家攻击系统
- 提交信息：fix: 修复敌人死亡后仍可攻击的 bug

分支命名：
- main：稳定版本
- develop：开发分支
- feature/player-movement：玩家移动功能
- bugfix/enemy-collision：敌人碰撞 bug 修复
```

---

## 2.7 构建与发布

### 构建为 Windows exe

```
1. File → Build Settings
2. Platform 选择 PC, Mac & Linux Standalone
3. 点击 Switch Platform
4. 点击 Add Open Scenes（添加当前场景）
5. 点击 Build
6. 选择输出目录
7. 等待构建完成 → 生成 .exe 文件
```

### 构建为 WebGL

```
1. File → Build Settings
2. Platform 选择 WebGL
3. 点击 Switch Platform（首次需要安装 WebGL Build Support）
4. 点击 Build
5. 输出是一个文件夹，包含 index.html
6. 需要 Web 服务器才能运行（可以用 Python 简单发布）
```

---

## 2.8 动手练习 🧪

### 练习 1：整理项目结构 ⭐
```
新建一个 3D 项目，按规范创建完整的文件夹结构
创建以下 ScriptableObject 数据模板：
- EnemyData（敌人数据）
- WeaponData（武器数据）
```

### 练习 2：导入素材 ⭐
```
从网络下载免费 Sprite（PNG）
导入 Unity，设置 Texture Type 为 Sprite
拖入场景查看效果
```

### 练习 3：Git 初始化 ⭐⭐
```
在新项目的根目录初始化 Git
创建 .gitignore
创建 develop 分支
在 develop 分支提交初始结构
合并到 main
```

---

## 2.9 本章小结

```
✅ 已掌握：
├── Unity 资源系统（导入→处理→使用）
├── 2D Sprite 导入规范
├── 3D 模型导入规范
├── 音频导入与压缩设置
├── 标准项目目录结构
├── ScriptableObject 创建和使用
├── Package Manager 用法
├── Git 版本控制（.gitignore + Git LFS）
└── 构建与发布（Windows exe / WebGL）

🔜 下章预告：
第三章：C# 核心语法（对比 C++）—— 熟悉 C# 的语法、数据类型、面向对象，与 C++ 对照学习。
```

---

_📚 参考资料：《Unity 入门：从零开始学游戏开发》《C# 入门经典》_
