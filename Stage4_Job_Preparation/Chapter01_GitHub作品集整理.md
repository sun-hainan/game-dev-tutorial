# Chapter 01：GitHub 作品集整理

> 🎯 目标：把 GitHub 打造成面试官眼前一亮的技术名片，让招聘者第一眼就看出你的实力。

---

## 1.1 GitHub 主页优化

### 为什么 GitHub 重要？

> **GitHub = 程序员的简历。** 面试官看简历只能想象，看 GitHub 能看到真实代码水平。

### README.md 个人介绍

```markdown
# 👋 你好，我是 [名字]

**🎮 游戏开发者** | **C++ / Unity C#** | **热爱技术**

## 📝 关于我
- 🏆 参与过 [X] 个游戏项目
- 🔧 擅长：游戏逻辑、引擎定制、性能优化
- 📚 正在学习：[新技术方向]

## 📊 技术栈

| 分类 | 技术 |
|-----|------|
| 引擎 | Unity / Unreal / Cocos |
| 语言 | C++ / C# / Lua / Python |
| 工具 | Git / Perforce / JIRA |
| 领域 | 2D/3D 游戏 / 物理 / AI / 网络 |

## 🌟 项目

| 项目 | 描述 | 技术 |
|-----|------|------|
| [项目名] | 2D 平台游戏 | Unity / C# |
| [项目名] | 3D 动作游戏 | Unity / C# / Shader |
| [项目名] | 算法学习仓库 | C++ / STL |

## 📫 联系我
- 📧 邮箱：xxx@example.com
- 💬 微信：[联系方式]
- 🐦 博客：[CSDN/知乎]

---

*如果你觉得我的项目有帮助，点个 ⭐ 吧！*
```

### Profile 模板推荐

> 直接访问 https://github.com/anuraghazra/github-readme-stats
> 生成动态统计卡片：
```markdown
![Anurag's GitHub stats](https://github-readme-stats.vercel.app/api?username=sun-hainan&theme=radical)

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=sun-hainan&layout=compact)
```

### Pin 重要项目

```
1. 质量 > 数量，Pin 最能体现水平的 3-5 个项目
2. 项目要求：
   - 有完整的 README
   - 有可运行的代码
   - 有截图/演示
3. 不要 Pin 作业、课程实验等低价值项目
```

---

## 1.2 README 规范

### 项目 README 标准模板

```markdown
# 项目名称

> 一句话描述项目是做什么的

![演示截图](screenshots/demo.gif)

## 🎮 游戏类型 / 📚 项目类型

[简单描述]

## ✨ 特色功能

- ✅ 功能1
- ✅ 功能2
- ✅ 功能3

## 🛠️ 技术栈

| 类别 | 技术 |
|-----|------|
| 引擎 | Unity 2022.3 LTS |
| 语言 | C# / C++ |
| 工具 | Git / JIRA |

## 🚀 运行方法

```bash
# 克隆
git clone https://github.com/username/project.git

# 打开 Unity 项目
cd project
open Project/Project.unity   # macOS
start Project/Project.unity  # Windows

# 或用命令行
Unity.exe -projectPath /path/to/project
```

## 📁 项目结构

```
Assets/
├── Scripts/          # 核心代码
├── Prefabs/          # 预制体
├── Scenes/           # 场景
└── README.md
```

## 🎯 TODO

- [ ] 功能1
- [x] 功能2
- [ ] 功能3

## 📄 许可证

MIT License - 详见 [LICENSE](LICENSE)
```

### 徽章（Badge）使用

```markdown
[![Unity](https://img.shields.io/badge/Unity-2022.3-blue)](https://unity.com)
[![C#](https://img.shields.io/badge/C%23-10.0-purple)](https://docs.microsoft.com/dotnet/csharp/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/username/project?style=social)](https://github.com/username/project/stargazers)

<!-- 动态徽章 -->
[![Build Status](https://img.shields.io/github/actions/workflow/status/username/project/ci.yml)](https://github.com/username/project/actions)
```

### 截图/GIF 展示

```
截图技巧：
1. 使用 LICEcap 或 ScreenToGif 录制 GIF
2. 分辨率不要太大（最大 800px 宽）
3. 展示核心玩法，不要展示 loading 界面
4. 用 GIMP 或 Photoshop 标注关键区域

存放位置：
project/
├── screenshots/     # 截图
│   ├── gameplay.gif
│   └── menu.png
└── README.md
```

---

## 1.3 游戏项目展示要点

### 功能列表

```markdown
## 🎮 游戏功能

### 核心玩法
- [x] WASD 移动 + 空格跳跃
- [x] 鼠标瞄准 + 左键射击
- [x] 敌人 AI（巡逻 + 追击）
- [x] 血量系统 + 死亡判定

### 道具系统
- [x] 金币收集 + 计分
- [x] 血瓶回复
- [x] 武器切换

### UI 系统
- [x] HUD（血条/分数/时间）
- [x] 暂停菜单
- [x] 游戏结束画面
```

### 技术栈描述

```markdown
## 🛠️ 技术实现

### 架构设计
- 采用 **组件化架构**，每个功能独立成组件
- 使用 **单例模式** 管理 GameManager/AudioManager
- 使用 **对象池** 优化子弹和敌人实例化

### 关键系统
- **物理系统**：Rigidbody 2D + Capsule Collider 2D
- **碰撞检测**：Trigger + Collision 事件系统
- **动画系统**：Animator Controller + Animation Events
- **对象池**：预创建 20 发子弹，循环复用，GC 降低 80%
```

### 运行说明

```markdown
## 🚀 如何运行

### 环境要求
- Unity 2022.3 LTS 或更高版本
- Windows 10+ / macOS 10.15+

### 运行步骤
1. `git clone` 本仓库
2. 用 Unity 打开 `Project/` 目录
3. 打开 `Scenes/GameScene.unity`
4. 点击 Play 运行

### 测试
```bash
# 单元测试（如果有）
cd Tests
dotnet test
```
```

---

## 1.4 代码质量

### .gitignore 标准模板（Unity）

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
*.pem
*.key

# Visual Studio
.vs/
*.csproj
*.sln
*.user
*.suo
*.userosscache
*.sln.docstates

# macOS
.DS_Store

# JetBrains Rider
.idea/

# Git LFS（如果使用）
*.png
*.jpg
*.psd
*.mp3
*.wav
*.fbx
*.unitypackage

# 第三方库（不提交编译产物）
External/

# 日志
*.log

# 上传的文件
*.apk
*.exe
*.app
*.ipa
```

### 代码规范

```csharp
// ✅ 命名规范示例
public class PlayerController : MonoBehaviour
{
    // -------- 私有字段用 _ 或 m_ 前缀 --------
    private int _health;
    private float _moveSpeed;

    // -------- 公有属性用 PascalCase --------
    public int Health => _health;

    // -------- 方法名用 PascalCase --------
    public void TakeDamage(int damage) { }

    // -------- 常量用 PascalCase --------
    public const int MAX_HEALTH = 100;

    // -------- 注释：描述"为什么"，不只是"是什么" --------
    // 用 0.02 的原因是 Unity 默认 FixedUpdate 间隔
    yield return new WaitForSeconds(0.02f);
}
```

### 分支管理

```bash
# 分支命名
main          # 稳定版本
develop       # 开发分支
feature/xxx   # 功能分支
bugfix/xxx    # 修复分支

# 常用命令
git checkout -b feature/player-movement    # 创建功能分支
git commit -m "feat: 实现玩家移动系统"
git push -u origin feature/player-movement
# PR → code review → merge to develop → merge to main
```

---

## 1.5 Green Square（贡献墙）

### 为什么重要？

> **贡献墙 = 证明你长期坚持写代码。** 面试官看到连续一年的 green square，比任何简历描述都有说服力。

### 建立习惯

```bash
# 每天至少 commit 一次（可以是文档、练习题、README）
git add .
git commit -m "docs: 更新 README 说明"
git push

# 建议时间：
# - 早上：规划今天的工作
# - 晚上：记录今天学了什么
```

### 自动化学时记录

```bash
# 在 Git hooks 中自动记录学习时间
# .git/hooks/post-commit
#!/bin/bash
echo "$(date '+%Y-%m-%d %H:%M') - Learning session" >> learning_log.md
```

---

## 1.6 简历链接 GitHub

### 简历中这样写

```markdown
## 项目经历

### 🎮 2D 平台跳跃游戏（Unity）
- **GitHub**: https://github.com/username/2d-platformer
- **技术栈**: Unity 2022.3 / C# / UGUI
- **核心功能**:
  - 实现了完整的玩家控制器（移动/跳跃/碰撞）
  - 使用对象池管理子弹，GC 降低 80%
  - 自定义关卡编辑器，支持可视化配置
- **亮点**: 帧率稳定 60FPS，优化后内存占用减少 40%
```

### 面试官会看的重点

```
✅ 看什么：
1. 提交历史是否持续（有 green square）
2. README 是否详细（说明不只是为了交作业）
3. 代码是否规范（命名/注释/架构）
4. 项目是否有 README + 截图
5. 是否解决了实际问题

❌ 不看什么：
1. 有多少 star（游戏项目 star 少很正常）
2. 代码行数（不是越多越好）
3. README 写了但代码跑不通
```

---

## 1.7 动手练习 🧪

### 练习 1：整理 GitHub 主页 ⭐
```
创建个人 GitHub README
包含：简介/技术栈/项目/联系方式
使用 GitHub Readme Stats 生成统计卡片
```

### 练习 2：完善项目 README ⭐⭐
```
选择一个项目，完善 README
包含：截图/功能列表/技术栈/运行方法
```

### 练习 3：创建算法仓库分支 ⭐⭐
```
整理 C++/C# 算法学习代码
按分类（排序/查找/DP/图）组织
每个算法有注释和复杂度说明
```

### 练习 4：连续 30 天 commit ⭐⭐⭐
```
建立每天 commit 的习惯
记录学习日志
30 天后检验效果
```

---

## 1.8 本章小结

```
✅ 已掌握：
├── GitHub Profile README 编写
├── 项目 README 标准模板
├── 徽章（Badge）使用
├── 代码规范和 .gitignore
├── Git 分支管理策略
├── Green Square 贡献墙建设
└── 简历中 GitHub 的展示方式

🔜 下章预告：
第二章：简历撰写与算法面试 —— 如何写一份打动 HR 的简历，以及游戏公司算法面试高频考点。
```

---

_📚 参考资料：《GitHub 官方文档》《简历写这个，HR看了贼开心》_
