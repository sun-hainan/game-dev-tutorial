# Chapter 08：UI 系统（Canvas 和 UGUI）

> 🎯 目标：掌握 Unity UGUI 系统——Canvas、血条、分数、按钮、菜单界面。

---

## 8.1 Unity UI 概述

### UGUI 核心组件

| 组件 | 作用 |
|-----|------|
| **Canvas** | UI 容器，所有 UI 元素必须在 Canvas 下 |
| **Rect Transform** | UI 元素的定位（比 Transform 更强大）|
| **Image** | 显示图片 |
| **Text** | 显示文本 |
| **Button** | 按钮 |
| **Slider** | 滑条（血条/音量）|
| **Toggle** | 开关（复选框）|

---

## 8.2 Canvas 画布

### Canvas 渲染模式

| 模式 | 说明 | 适用场景 |
|-----|------|---------|
| **Screen Space - Overlay** | 覆盖在屏幕最上层 | 最常用，2D UI |
| **Screen Space - Camera** | 由相机渲染 | 需要透视效果时 |
| **World Space** | 在 3D 场景中 | 3D HUD、VR |

### Canvas 设置

```csharp
// -------- 在代码中动态创建 Canvas --------
Canvas canvas = new GameObject("Canvas").AddComponent<Canvas>();
canvas.renderMode = RenderMode.ScreenSpaceOverlay;

CanvasScaler scaler = canvas.gameObject.AddComponent<CanvasScaler>();
scaler.uiScaleMode = CanvasScaler.ScaleMode.ScaleWithScreenSize;
scaler.referenceResolution = new Vector2(1920, 1080);   // 目标分辨率
scaler.matchWidthOrHeight = 0.5f;   // 0=宽度适配，1=高度适配

canvas.gameObject.AddComponent<GraphicRaycaster>();   // UI 响应点击
```

---

## 8.3 Rect Transform

### 与 Transform 的区别

| Transform | Rect Transform |
|----------|---------------|
| Position | Anchors（锚点）+ Pivot（轴心）+ Offset |
| 适用于任意大小 | 专为 UI 元素设计 |
| 无 | 支持相对父对象的定位 |

### 锚点（Anchor）

```
锚点 = 相对父对象的固定点
- 锚点设置在父对象的哪个位置，子对象就相对哪个位置布局
- 常见锚点预设：
  - 中间：Center
  - 拉伸：Stretch（子对象填充父对象）
  - 四个角：按需固定边角
```

### Pivot（轴心）

```
轴心 = 元素旋转和缩放的中心点
- 轴心在中心，旋转以中心为圆心
- 轴心在顶部，旋转以上边为轴
```

---

## 8.4 Image（图片）

### 基本使用

```csharp
public Image healthBarFill;   // 拖入 Inspector

void UpdateHealth(float currentHP, float maxHP)
{
    // -------- 填充血条（常用）--------
    healthBarFill.fillAmount = currentHP / maxHP;  // 0~1 之间
}
```

### Image Type

```csharp
Image image = GetComponent<Image>();

// -------- Simple：简单显示整张图 --------
image.sprite = mySprite;

// -------- Sliced：九宫格拉伸（按钮背景）--------
image.type = Image.Type.Sliced;

// -------- Tiled：平铺 --------
image.type = Image.Type.Tiled;

// -------- Filled：填充（血条/进度条）--------
image.type = Image.Type.Filled;
image.fillMethod = Image.FillMethod.Horizontal;   // 水平填充
image.fillAmount = 0.5f;   // 50%
```

---

## 8.5 Text（文本）

```csharp
public Text scoreText;       // 拖入 Inspector
public TextMeshProUGUI tmp;  // 如果用 TextMeshPro

void UpdateScore(int score)
{
    // -------- 普通 Text --------
    scoreText.text = "分数：" + score;

    // -------- 富文本（HTML 风格）--------
    scoreText.text = "<color=red>危险！</color> HP: <b>100</b>";

    // -------- TextMeshPro（推荐）--------
    tmp.text = $"分数：{score:N0}";   // N0 = 千分位分隔
}
```

### Font 设置

```
字体资源：
1. 下载 .ttf 或 .otf 字体文件
2. 拖入 Assets/Fonts
3. 选中 Text → Font 字段选择字体
```

---

## 8.6 Button（按钮）

### Inspector 配置

```
1. 添加 Button 组件
2. Transition：状态切换效果
   - Color Tint：颜色变化（Normal/Highlighted/Pressed/Disabled）
   - Sprite Swap：图片切换
   - Animation：动画切换
3. On Click()：绑定事件
```

### 代码绑定事件

```csharp
public class UIManager : MonoBehaviour
{
    public Button attackButton;
    public Button jumpButton;

    void Start()
    {
        // -------- 方式1：Inspector 拖入 --------
        // 在 Inspector 的 OnClick() 点击事件列表中拖入对象和方法

        // -------- 方式2：代码订阅 --------
        attackButton.onClick.AddListener(OnAttackClicked);
        jumpButton.onClick.AddListener(OnJumpClicked);
    }

    void OnAttackClicked()
    {
        Debug.Log("攻击按钮被点击！");
    }

    void OnJumpClicked()
    {
        Debug.Log("跳跃按钮被点击！");
    }

    void OnDestroy()
    {
        // -------- 取消订阅（避免内存泄漏）--------
        attackButton.onClick.RemoveListener(OnAttackClicked);
        jumpButton.onClick.RemoveListener(OnJumpClicked);
    }
}
```

### 动态创建按钮

```csharp
public Button buttonPrefab;

void CreateButton()
{
    Button btn = Instantiate(buttonPrefab, transform);
    btn.GetComponentInChildren<Text>().text = "新按钮";
    btn.onClick.AddListener(() => Debug.Log("点击了！"));
}
```

---

## 8.7 血条（Health Bar）实战

### 血条结构

```
Canvas
└── HealthBarPanel（锚点：顶部中央）
    └── BackgroundImage（深色背景）
        └── FillImage（绿色填充，Filled 类型）
```

### 完整血条脚本

```csharp
using UnityEngine;
using UnityEngine.UI;

public class HealthBar : MonoBehaviour
{
    [Header("组件引用")]
    public Image healthFillImage;    // 填充 Image
    public Text healthText;          // 数值文本
    public Image damageFlashImage;   // 受伤闪红

    [Header("数值")]
    public float maxHealth = 100f;
    private float currentHealth;

    [Header("设置")]
    public float flashSpeed = 5f;

    void Start()
    {
        currentHealth = maxHealth;
        UpdateHealthBar();
    }

    public void TakeDamage(float damage)
    {
        currentHealth -= damage;
        currentHealth = Mathf.Clamp(currentHealth, 0f, maxHealth);
        UpdateHealthBar();

        // -------- 受伤闪红 --------
        if (damageFlashImage != null)
        {
            StartCoroutine(FlashRed());
        }

        if (currentHealth <= 0f)
        {
            OnDeath();
        }
    }

    public void Heal(float amount)
    {
        currentHealth += amount;
        currentHealth = Mathf.Clamp(currentHealth, 0f, maxHealth);
        UpdateHealthBar();
    }

    void UpdateHealthBar()
    {
        float fill = currentHealth / maxHealth;
        healthFillImage.fillAmount = fill;

        // -------- 颜色变化：绿→黄→红 --------
        if (fill > 0.6f)
        {
            healthFillImage.color = Color.green;
        }
        else if (fill > 0.3f)
        {
            healthFillImage.color = Color.yellow;
        }
        else
        {
            healthFillImage.color = Color.red;
        }

        // -------- 显示数值 --------
        if (healthText != null)
        {
            healthText.text = $"{Mathf.CeilToInt(currentHealth)} / {Mathf.CeilToInt(maxHealth)}";
        }
    }

    System.Collections.IEnumerator FlashRed()
    {
        damageFlashImage.enabled = true;
        damageFlashImage.color = new Color(1f, 0f, 0f, 0.5f);
        yield return new WaitForSeconds(0.1f);

        float elapsed = 0f;
        while (elapsed < 0.3f)
        {
            elapsed += Time.deltaTime;
            float alpha = Mathf.Lerp(0.5f, 0f, elapsed / 0.3f);
            damageFlashImage.color = new Color(1f, 0f, 0f, alpha);
            yield return null;
        }

        damageFlashImage.enabled = false;
    }

    void OnDeath()
    {
        Debug.Log("玩家死亡！");
        // 显示 Game Over 界面...
    }
}
```

---

## 8.8 分数系统

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class ScoreManager : MonoBehaviour
{
    public static ScoreManager instance;   // 单例

    [Header("UI 引用")]
    public TextMeshProUGUI scoreText;
    public TextMeshProUGUI comboText;

    [Header("数值")]
    private int score = 0;
    private int combo = 0;
    private int highScore = 0;

    void Awake()
    {
        if (instance == null)
            instance = this;
        else
            Destroy(gameObject);
    }

    void Start()
    {
        highScore = PlayerPrefs.GetInt("HighScore", 0);   // 读取最高分
        UpdateUI();
    }

    public void AddScore(int points)
    {
        combo++;
        int comboMultiplier = Mathf.Min(combo / 5 + 1, 10);   // 最高 10 倍
        int finalPoints = points * comboMultiplier;

        score += finalPoints;

        if (score > highScore)
        {
            highScore = score;
            PlayerPrefs.SetInt("HighScore", highScore);   // 保存最高分
        }

        UpdateUI();
    }

    public void ResetCombo()
    {
        combo = 0;
        UpdateUI();
    }

    void UpdateUI()
    {
        scoreText.text = $"{score:N0}";   // 千分位格式

        if (combo > 1)
        {
            comboText.text = $"x{combo} COMBO!";
            comboText.gameObject.SetActive(true);
        }
        else
        {
            comboText.gameObject.SetActive(false);
        }
    }
}
```

---

## 8.9 暂停菜单

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class PauseMenu : MonoBehaviour
{
    public GameObject pauseMenuUI;   // 拖入 PauseMenu Panel
    private bool isPaused = false;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            if (isPaused)
                Resume();
            else
                Pause();
        }
    }

    public void Resume()
    {
        pauseMenuUI.SetActive(false);
        Time.timeScale = 1f;   // 恢复游戏时间
        isPaused = false;
    }

    public void Pause()
    {
        pauseMenuUI.SetActive(true);
        Time.timeScale = 0f;   // 暂停游戏时间（所有物理和 Update 都暂停）
        isPaused = true;
    }

    public void Restart()
    {
        Time.timeScale = 1f;   // 恢复时间
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex);
    }

    public void MainMenu()
    {
        Time.timeScale = 1f;
        SceneManager.LoadScene("MainMenuScene");   // 加载主菜单场景
    }

    public void QuitGame()
    {
        Application.Quit();
        #if UNITY_EDITOR
        UnityEditor.EditorApplication.isPlaying = false;
        #endif
    }
}
```

---

## 8.10 动画菜单（DOTween）

```csharp
using UnityEngine;
using DG.Tweening;   // 需要安装 DOTween

public class AnimatedMenu : MonoBehaviour
{
    public RectTransform menuPanel;
    public CanvasGroup canvasGroup;

    void Start()
    {
        // -------- 初始状态：隐藏 --------
        canvasGroup.alpha = 0f;
        menuPanel.localScale = Vector3.zero;

        // -------- 淡入动画 --------
        void ShowMenu()
        {
            canvasGroup.DOFade(1f, 0.3f);
            menuPanel.DOScale(1f, 0.3f).SetEase(Ease.OutBack);
        }

        // -------- 淡出动画 --------
        void HideMenu()
        {
            canvasGroup.DOFade(0f, 0.3f);
            menuPanel.DOScale(0f, 0.3f).SetEase(Ease.InBack);
        }
    }
}
```

---

## 8.11 动手练习 🧪

### 练习 1：血条系统 ⭐
```
创建血条 UI（背景+填充）
点击鼠标扣血，右键回血
血量变化时血条动画过渡
```

### 练习 2：暂停菜单 ⭐⭐
```
按 ESC 暂停游戏
显示暂停菜单（继续/重新开始/退出）
继续后游戏恢复
```

### 练习 3：计分板 ⭐⭐
```
敌人死亡 +100 分
击败 5 个敌人后 +500 额外奖励
分数达到 1000 时触发胜利
```

### 练习 4：背包 UI ⭐⭐
```
Grid Layout Group 实现道具格子
点击道具显示详情
道具可拖拽交换位置
```

### 练习 5：动态货币显示 ⭐⭐⭐
```
击杀敌人掉落金币
金币飞向 UI 的货币图标
数字跳动动画
```

---

## 8.12 本章小结

```
✅ 已掌握：
├── Canvas 渲染模式（Overlay/Camera/World Space）
├── Rect Transform（Anchors/Pivot/Offset）
├── Image（Simple/Sliced/Tiled/Filled）
├── Text 和 TextMeshPro
├── Button（Inspector + 代码绑定）
├── 完整血条系统（填充+颜色+闪红）
├── 分数与 Combo 系统
├── 暂停菜单（Time.timeScale）
├── DOTween 动画菜单
└── 常用 UI 设计模式

🔜 下章预告：
第九章：协程与异步编程 —— 深入协程、Task异步、异步资源加载。
```

---

_📚 参考资料：《Unity UI 开发实战》《DOTween 文档》_
