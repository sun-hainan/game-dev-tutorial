# Chapter 15：完整 2D 游戏项目

> 🎯 目标：从零开始，做一个**完整的 2D 平台跳跃游戏**。涵盖所有 Stage 2 学到的内容。

---

## 15.1 项目概述

### 游戏类型：2D 平台跳跃

```
游戏目标：操控角色跳跃，躲避敌人，收集金币，达到终点

功能列表：
✅ 角色左右移动、跳跃
✅ 重力与物理
✅ 平台碰撞
✅ 金币收集 + 计分
✅ 敌人 AI（巡逻）
✅ 伤害系统（敌人+尖刺）
✅ 血条 UI
✅ 音效
✅ 粒子特效（收集/死亡）
✅ 游戏暂停
✅ 关卡切换
✅ 存档（PlayerPrefs）
✅ 开始界面 + Game Over 界面
✅ 敌人死亡后掉落
✅ 相机平滑跟随
```

---

## 15.2 项目结构

```
Assets/
├── Scenes/
│   ├── MainMenu.scene
│   ├── Level1.scene
│   ├── Level2.scene
│   └── GameOver.scene
├── Scripts/
│   ├── Core/
│   │   ├── GameManager.cs
│   │   ├── AudioManager.cs
│   │   └── UIManager.cs
│   ├── Player/
│   │   ├── PlayerController.cs
│   │   └── PlayerHealth.cs
│   ├── Enemy/
│   │   ├── EnemyPatrol.cs
│   │   └── EnemyChase.cs
│   ├── Level/
│   │   ├── Platform.cs
│   │   └── Coin.cs
│   └── Managers/
│       ├── GameOverManager.cs
│       └── LevelLoader.cs
├── Prefabs/
│   ├── Player.prefab
│   ├── Coin.prefab
│   ├── Enemy.prefab
│   └── ParticleEffects/
│       ├── CoinCollect.prefab
│       └── DeathEffect.prefab
├── Sprites/
│   ├── Characters/
│   ├── Platforms/
│   └── Items/
├── Audio/
│   ├── BGM/
│   └── SFX/
└── Animations/
    ├── PlayerAC.controller
    └── Coins/
```

---

## 15.3 核心脚本

### GameManager（游戏总管理）

```csharp
using UnityEngine;

public class GameManager : MonoBehaviour
{
    public static GameManager instance;

    [Header("游戏状态")]
    public bool isGamePaused = false;
    public bool isGameOver = false;

    [Header("游戏数据")]
    public int currentLevel = 1;
    public int score = 0;
    public int highScore = 0;

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void Start()
    {
        highScore = PlayerPrefs.GetInt("HighScore", 0);
    }

    public void AddScore(int points)
    {
        score += points;
        UIManager.instance.UpdateScoreUI(score);

        if (score > highScore)
        {
            highScore = score;
            PlayerPrefs.SetInt("HighScore", highScore);
        }
    }

    public void PauseGame()
    {
        isGamePaused = true;
        Time.timeScale = 0f;
        AudioManager.instance.PauseBGM();
    }

    public void ResumeGame()
    {
        isGamePaused = false;
        Time.timeScale = 1f;
        AudioManager.instance.ResumeBGM();
    }

    public void GameOver()
    {
        if (isGameOver) return;
        isGameOver = true;
        AudioManager.instance.PlayGameOver();
        UIManager.instance.ShowGameOver();
    }

    public void NextLevel()
    {
        currentLevel++;
        Time.timeScale = 1f;
        UnityEngine.SceneManagement.SceneManager.LoadScene($"Level{currentLevel}");
    }
}
```

### PlayerController（玩家控制）

```csharp
using UnityEngine;

[RequireComponent(typeof(Rigidbody2D))]
public class PlayerController : MonoBehaviour
{
    [Header("移动设置")]
    public float moveSpeed = 5f;
    public float jumpForce = 12f;

    [Header("组件引用")]
    private Rigidbody2D rb;
    private Animator animator;

    [Header("地面检测")]
    public LayerMask groundLayer;
    public Transform groundCheck;

    private bool isGrounded = false;
    private float horizontalInput;

    void Awake()
    {
        rb = GetComponent<Rigidbody2D>();
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        // -------- 输入 --------
        horizontalInput = Input.GetAxisRaw("Horizontal");

        // -------- 地面检测 --------
        isGrounded = Physics2D.Raycast(
            groundCheck.position,
            Vector2.down,
            0.1f,
            groundLayer
        );

        // -------- 跳跃 --------
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.AddForce(Vector2.up * jumpForce, ForceMode2D.Impulse);
            AudioManager.instance.PlayJump();
        }

        // -------- 动画 --------
        animator.SetFloat("Speed", Mathf.Abs(horizontalInput));
        animator.SetBool("IsGrounded", isGrounded);

        // -------- 翻转 --------
        if (horizontalInput > 0)
            transform.localScale = new Vector3(1, 1, 1);
        else if (horizontalInput < 0)
            transform.localScale = new Vector3(-1, 1, 1);
    }

    void FixedUpdate()
    {
        rb.velocity = new Vector2(horizontalInput * moveSpeed, rb.velocity.y);
    }

    void OnCollisionEnter2D(Collision2D col)
    {
        if (col.gameObject.CompareTag("MovingPlatform"))
        {
            // 跟随平台移动
            transform.SetParent(col.transform);
        }
    }

    void OnCollisionExit2D(Collision2D col)
    {
        if (col.gameObject.CompareTag("MovingPlatform"))
        {
            transform.SetParent(null);
        }
    }
}
```

### EnemyPatrol（敌人巡逻）

```csharp
using UnityEngine;

public class EnemyPatrol : MonoBehaviour
{
    public float speed = 2f;
    public Transform[] waypoints;   // 巡逻点
    private int currentWaypoint = 0;
    private Animator animator;

    void Awake()
    {
        animator = GetComponent<Animator>();
    }

    void Update()
    {
        Transform target = waypoints[currentWaypoint];
        Vector3 dir = (target.position - transform.position).normalized;

        transform.position += dir * speed * Time.deltaTime;
        transform.localScale = new Vector3(dir.x > 0 ? 1 : -1, 1, 1);

        // 到达目标点
        if (Vector3.Distance(transform.position, target.position) < 0.1f)
        {
            currentWaypoint = (currentWaypoint + 1) % waypoints.Length;
        }

        animator.SetFloat("Speed", speed);
    }

    void OnCollisionEnter2D(Collision2D col)
    {
        if (col.gameObject.CompareTag("Player"))
        {
            // 玩家碰到敌人
            PlayerHealth ph = col.gameObject.GetComponent<PlayerHealth>();
            if (ph != null)
            {
                Vector3 knockbackDir = (col.transform.position - transform.position).normalized;
                ph.TakeDamage(1, knockbackDir);
            }
        }
    }
}
```

### PlayerHealth（玩家生命）

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class PlayerHealth : MonoBehaviour
{
    public int maxHP = 3;
    private int currentHP;
    private bool isInvincible = false;
    private SpriteRenderer sr;

    [Header("无敌时间")]
    public float invincibilityDuration = 1.5f;

    void Awake()
    {
        sr = GetComponent<SpriteRenderer>();
    }

    void Start()
    {
        currentHP = maxHP;
        UIManager.instance.UpdateHealthUI(currentHP, maxHP);
    }

    public void TakeDamage(int damage, Vector3 knockbackDir)
    {
        if (isInvincible) return;

        currentHP -= damage;
        AudioManager.instance.PlayHurt();

        // 击退
        GetComponent<Rigidbody2D>().velocity = knockbackDir * 5f;

        // 无敌帧
        StartCoroutine(InvincibilityCoroutine());

        UIManager.instance.UpdateHealthUI(currentHP, maxHP);

        if (currentHP <= 0)
        {
            Die();
        }
    }

    System.Collections.IEnumerator InvincibilityCoroutine()
    {
        isInvincible = true;
        float elapsed = 0f;
        while (elapsed < invincibilityDuration)
        {
            sr.enabled = !sr.enabled;   // 闪烁
            yield return new WaitForSeconds(0.1f);
            elapsed += 0.1f;
        }
        sr.enabled = true;
        isInvincible = false;
    }

    void Die()
    {
        AudioManager.instance.PlayDeath();
        GameManager.instance.GameOver();
        Destroy(gameObject);
    }

    public void Heal(int amount)
    {
        currentHP = Mathf.Min(currentHP + amount, maxHP);
        UIManager.instance.UpdateHealthUI(currentHP, maxHP);
    }
}
```

### Coin（收集金币）

```csharp
using UnityEngine;

public class Coin : MonoBehaviour
{
    public int value = 10;
    public ParticleSystem collectEffect;

    void OnTriggerEnter2D(Collider2D col)
    {
        if (col.CompareTag("Player"))
        {
            // 计分
            GameManager.instance.AddScore(value);
            AudioManager.instance.PlayCoin();

            // 特效
            if (collectEffect != null)
            {
                ParticleSystem ps = Instantiate(collectEffect, transform.position, Quaternion.identity);
                Destroy(ps.gameObject, 1f);
            }

            Destroy(gameObject);
        }
    }
}
```

---

## 15.4 关卡设计

### Level1 布局

```
地面：[====G====]          [=====G=====]
         P                    C  C  C
        / \                  / \     / \
   G        C         G           G
   |    C   |   C     |     C     |   C
========================================
G=敌人(巡逻)  C=金币  P=玩家起点
```

---

## 15.5 游戏流程

```
┌─────────────────┐
│   MainMenu      │ ← 开始界面（按空格开始）
└────────┬────────┘
         ↓
┌─────────────────┐
│    Level 1      │ ← 平台跳跃+金币+敌人
└────────┬────────┘
         ↓ (达到终点或按E跳过)
┌─────────────────┐
│    Level 2      │ ← 更难，更多敌人
└────────┬────────┘
         ↓ (通关)
┌─────────────────┐
│  VictoryScreen  │ ← 胜利画面
└─────────────────┘

GameOver → 任意关卡死亡 → GameOverScreen → MainMenu
```

---

## 15.6 章节小结

```
✅ 完成内容：
├── 游戏总管理器（GameManager 单例）
├── 玩家控制器（移动/跳跃/物理）
├── 敌人 AI（巡逻）
├── 生命系统（受伤/无敌帧/死亡）
├── 金币收集（分数/特效/音效）
├── UI 系统（血条/分数/暂停/开始/结束界面）
├── 音频管理器（BGM/SFX）
├── 关卡切换
├── PlayerPrefs 存档
└── 完整可运行游戏

🎉 Stage 2 Unity + C# 开发 —— 全部完成！
🎉 恭喜你完成了两个阶段的内容！
```

---

## 📚 接下来该做什么？

```
✅ Stage 1：C++ 基础（15 章）
✅ Stage 2：Unity + C# 开发（15 章）

📖 Stage 3（可选）：
- Unity 进阶：网络多人、Shader、热更新
- 毕业项目：完整游戏开发
- 作品集整理

🚀 下一步：求职准备 / 个人项目 / 开源贡献
```

---

_📚 参考资料：《Unity 官方文档》《Unity 2D Game Development》《Game Programming Patterns》_
