# 第1章 设计模式

> 游戏开发中最常用的设计模式，让你的代码更优雅、更易维护。

---

## 1.1 单例模式（Singleton）

### 生活类比

想象一下**五星酒店的礼宾部**——无论多少客人需要服务，他们访问的都是同一个礼宾台。酒店不可能为每个客人创建一个礼宾部，那样会导致混乱和资源浪费。GameManager 就是游戏的"礼宾部"，所有系统都从它获取服务。

### 核心思想

**确保一个类只有一个实例，并提供一个全局访问点。**

- 饿汉式：程序启动时就创建实例（"饿"了，早动手）
- 懒汉式：第一次使用时才创建（"懒"，延迟加载）

### Unity C# 代码示例

```csharp
using UnityEngine;

// ============================================
// 饿汉式单例 - 程序启动就创建，天生线程安全
// 适用于：确定会被使用、资源占用小的管理器
// ============================================
public class AudioManager_Eager : MonoBehaviour
{
    // static 构造块在类首次引用时执行，线程安全
    // Unity 会自动实例化这个组件
    public static AudioManager_Eager Instance { get; private set; }

    // 饿汉式：在静态构造函数中立即初始化
    static AudioManager_Eager()
    {
        // 创建 GameObject 并添加组件（实际项目中通常预先绑定）
        var go = new GameObject("AudioManager");
        Instance = go.AddComponent<AudioManager_Eager>();
        // 标记为不随场景销毁（可选）
        // GameObject.DontDestroyOnLoad(go);
    }

    // 音频源组件引用
    public AudioSource bgmSource;
    public AudioSource sfxSource;

    // 播放背景音乐
    public void PlayBGM(AudioClip clip)
    {
        if (bgmSource != null && clip != null)
        {
            bgmSource.clip = clip;
            bgmSource.Play();
        }
    }

    // 播放音效
    public void PlaySFX(AudioClip clip)
    {
        if (sfxSource != null && clip != null)
        {
            sfxSource.PlayOneShot(clip);
        }
    }
}
```

```csharp
using UnityEngine;

// ============================================
// 懒汉式单例 - 首次访问时才创建，节省资源
// 适用于：不确定是否会被使用、占用资源大的系统
// ============================================
public class GameManager_Lazy : MonoBehaviour
{
    // volatile 保证多线程可见性
    private static volatile GameManager_Lazy _instance;

    // 锁对象，用于线程同步
    private static readonly object _lock = new object();

    // 外部访问点：属性包装
    public static GameManager_Lazy Instance
    {
        get
        {
            // 双重检查锁定：第一次检查避免每次访问都加锁
            if (_instance == null)
            {
                lock (_lock)
                {
                    // 第二次检查确保只创建一个实例
                    if (_instance == null)
                    {
                        // 在场景中查找现有实例
                        _instance = FindObjectOfType<GameManager_Lazy>();

                        // 如果场景中没有，创建一个
                        if (_instance == null)
                        {
                            var go = new GameObject("GameManager");
                            _instance = go.AddComponent<GameManager_Lazy>();
                        }
                    }
                }
            }
            return _instance;
        }
    }

    // ========== 游戏核心数据 ==========
    public int playerScore = 0;       // 当前分数
    public int playerLevel = 1;       // 当前等级
    public float gameTime = 0f;       // 游戏时间

    // 初始化标志，防止重复初始化
    private bool _isInitialized = false;

    // Awake 在所有对象的 Start 之前调用
    private void Awake()
    {
        // ========== 单例注册 ==========
        // 如果已存在实例且不是自己，说明是重复创建的，销毁自己
        if (_instance != null && _instance != this)
        {
            Destroy(gameObject);
            return;
        }

        // 注册为单例
        _instance = this;

        // 跨场景不销毁（如需）
        // DontDestroyOnLoad(gameObject);

        // ========== 初始化逻辑 ==========
        if (!_isInitialized)
        {
            InitializeGame();
            _isInitialized = true;
        }
    }

    // 游戏初始化
    private void InitializeGame()
    {
        playerScore = 0;
        playerLevel = 1;
        gameTime = 0f;
        Debug.Log("[GameManager] 游戏初始化完成");
    }

    // 更新（每帧调用）
    private void Update()
    {
        gameTime += Time.deltaTime;
    }

    // 添加分数
    public void AddScore(int points)
    {
        playerScore += points;
        Debug.Log($"[GameManager] 得分 +{points}，当前总分：{playerScore}");

        // 检查升级
        CheckLevelUp();
    }

    // 检查是否升级
    private void CheckLevelUp()
    {
        int newLevel = (playerScore / 1000) + 1;  // 每1000分升一级
        if (newLevel > playerLevel)
        {
            playerLevel = newLevel;
            Debug.Log($"[GameManager] 升级！当前等级：{playerLevel}");
            // 可以触发升级事件通知其他系统
        }
    }
}
```

```csharp
using UnityEngine;

// ============================================
// DontDestroyOnLoad 单例 - 跨场景持久化
// 适用于：主管理器、设置管理器、音频管理器等
// ============================================
public class PersistentManager : MonoBehaviour
{
    // 单例引用
    public static PersistentManager Instance { get; private set; }

    // 玩家设置数据
    public float masterVolume = 1.0f;
    public float musicVolume = 0.8f;
    public float sfxVolume = 1.0f;
    public int graphicsQuality = 2;  // 0=低 1=中 2=高

    // 场景切换时调用
    private void Awake()
    {
        // ========== 防止重复创建 ==========
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }

        // ========== 设置为持久化 ==========
        Instance = this;
        DontDestroyOnLoad(gameObject);  // 切换场景时不销毁
    }

    // 保存设置到 PlayerPrefs
    public void SaveSettings()
    {
        PlayerPrefs.SetFloat("MasterVolume", masterVolume);
        PlayerPrefs.SetFloat("MusicVolume", musicVolume);
        PlayerPrefs.SetFloat("SFXVolume", sfxVolume);
        PlayerPrefs.SetInt("GraphicsQuality", graphicsQuality);
        PlayerPrefs.Save();
        Debug.Log("[PersistentManager] 设置已保存");
    }

    // 加载设置
    public void LoadSettings()
    {
        if (PlayerPrefs.HasKey("MasterVolume"))
        {
            masterVolume = PlayerPrefs.GetFloat("MasterVolume");
            musicVolume = PlayerPrefs.GetFloat("MusicVolume");
            sfxVolume = PlayerPrefs.GetFloat("SFXVolume");
            graphicsQuality = PlayerPrefs.GetInt("GraphicsQuality");
            Debug.Log("[PersistentManager] 设置已加载");
        }
    }
}
```

```csharp
using UnityEngine;

// ============================================
// 泛型单例基类 - 一劳永逸，任何 MonoBehaviour 都能变单例
// 使用方式：public class MyManager : Singleton<MyManager>
// ============================================
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    // volatile 保证多线程可见性
    private static volatile T _instance;

    // 同步锁对象
    private static readonly object _lock = new object();

    // 标记是否正在销毁（防止 Destroy 后还被访问）
    private bool _isDestroying = false;

    // 全局访问点
    public static T Instance
    {
        get
        {
            // 如果正在销毁，返回 null
            if (_isDestroying)
                return null;

            // 快速路径：已有实例
            if (_instance != null)
                return _instance;

            // 慢速路径：需要创建
            lock (_lock)
            {
                // 再次检查（双重锁定）
                if (_instance == null)
                {
                    // 在场景中查找
                    _instance = FindObjectOfType<T>();

                    if (_instance == null)
                    {
                        // 创建新的 GameObject 并添加组件
                        var singletonObject = new GameObject();
                        _instance = singletonObject.AddComponent<T>();

                        // 设置 GameObject 名称
                        singletonObject.name = $"{typeof(T).Name} (Singleton)";

                        Debug.Log($"[Singleton] 为 {typeof(T).Name} 创建了新实例");
                    }
                }
                return _instance;
            }
        }
    }

    // 初始化（子类可重写）
    protected virtual void Awake()
    {
        // 检查是否已有实例
        if (_instance != null && _instance != this)
        {
            // 已有实例，销毁自己
            Destroy(gameObject);
            return;
        }

        // 注册实例
        _instance = this as T;
    }

    // 销毁时的处理
    protected virtual void OnDestroy()
    {
        // 只有当前实例被销毁时才清理
        if (_instance == this)
        {
            _isDestroying = true;
            _instance = null;
        }
    }
}

// ============================================
// 使用泛型基类创建单例
// ============================================
public class UIManager : Singleton<UIManager>
{
    // UIManager 特有属性
    public GameObject pauseMenuPrefab;
    private GameObject _currentPauseMenu;

    // 重写 Awake（可选）
    protected override void Awake()
    {
        base.Awake();  // 先调用基类逻辑
        Debug.Log("[UIManager] 初始化");
    }

    // 显示暂停菜单
    public void ShowPauseMenu()
    {
        if (_currentPauseMenu == null && pauseMenuPrefab != null)
        {
            _currentPauseMenu = Instantiate(pauseMenuPrefab);
        }
        Time.timeScale = 0f;  // 暂停游戏
    }

    // 隐藏暂停菜单
    public void HidePauseMenu()
    {
        if (_currentPauseMenu != null)
        {
            Destroy(_currentPauseMenu);
            _currentPauseMenu = null;
        }
        Time.timeScale = 1f;  // 恢复游戏
    }
}
```

### 游戏开发应用

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| GameManager | 懒汉式 +DontDestroyOnLoad | 游戏核心管理器，需要持久化 |
| AudioManager | 饿汉式单例 | 几乎每个游戏都需要，提前加载无妨 |
| EventManager | 泛型单例基类 | 事件系统是通用组件 |
| NetworkManager | 懒汉式 + 线程锁 | 网络管理器复杂，延迟创建 |
| 临时Manager | 普通 MonoBehaviour | 不用单例，Scene 级别的用 FindObjectOfType |

---

## 1.2 工厂模式（Factory）

### 生活类比

想象**麦当劳的点餐柜台**——你不需要知道汉堡怎么做，只需要说"我要一个巨无霸"，服务员（工厂）就会给你一个做好的汉堡。工厂模式让"使用者"不需要知道"产品"是如何被创建出来的。

### 核心思想

**将对象的创建与使用分离，由工厂类负责创建一系列相关对象。**

- 简单工厂：用一个工厂类创建所有产品（简单，但违反开闭原则）
- 工厂方法：每种产品有一个工厂（符合开闭原则，但类增多）
- 抽象工厂：创建一系列相关产品族（复杂产品线）

### Unity C# 代码示例

```csharp
using UnityEngine;

// ============================================
// 简单工厂 - 集中管理对象创建
// 适用于：产品种类较少、变化不频繁的场景
// ============================================

// 产品基类：所有武器的抽象
public abstract class Weapon
{
    public string weaponName;     // 武器名称
    public int damage;            // 伤害值
    public float attackRange;     // 攻击范围
    public float attackCooldown;  // 攻击冷却时间

    // 抽象方法：执行攻击（子类必须实现）
    public abstract void Attack(Transform attacker, Transform target);

    // 通用方法：显示信息
    public virtual void ShowInfo()
    {
        Debug.Log($"[武器] {weaponName} - 伤害:{damage} 范围:{attackRange}");
    }
}

// 具体产品：剑
public class Sword : Weapon
{
    public Sword()
    {
        weaponName = "长剑";
        damage = 25;
        attackRange = 2.5f;
        attackCooldown = 0.8f;
    }

    public override void Attack(Transform attacker, Transform target)
    {
        // 计算距离
        float distance = Vector3.Distance(attacker.position, target.position);
        if (distance <= attackRange)
        {
            Debug.Log($"[剑] {attacker.name} 挥剑攻击 {target.name}，造成 {damage} 点伤害");
            // 播放攻击动画、音效等
        }
        else
        {
            Debug.Log("[剑] 目标距离太远，攻击失败");
        }
    }
}

// 具体产品：弓
public class Bow : Weapon
{
    private GameObject arrowPrefab;  // 箭矢预制体

    public Bow()
    {
        weaponName = "长弓";
        damage = 15;
        attackRange = 20f;
        attackCooldown = 1.2f;
    }

    public override void Attack(Transform attacker, Transform target)
    {
        float distance = Vector3.Distance(attacker.position, target.position);
        if (distance <= attackRange)
        {
            Debug.Log($"[弓] {attacker.name} 射箭攻击 {target.name}，造成 {damage} 点伤害");
            // 生成箭矢对象（实际项目中会实例化箭矢预制体）
            // Instantiate(arrowPrefab, attacker.position, Quaternion.LookRotation(target.position - attacker.position));
        }
    }
}

// 具体产品：法杖
public class Staff : Weapon
{
    public Staff()
    {
        weaponName = "魔法杖";
        damage = 35;
        attackRange = 15f;
        attackCooldown = 2.0f;
    }

    public override void Attack(Transform attacker, Transform target)
    {
        float distance = Vector3.Distance(attacker.position, target.position);
        if (distance <= attackRange)
        {
            Debug.Log($"[法杖] {attacker.name} 发射魔法弹攻击 {target.name}，造成 {damage} 点伤害");
            // 生成魔法特效
        }
    }
}

// ============================================
// 简单工厂类
// 注意：违反开闭原则，新增产品需要修改工厂类
// ============================================
public static class WeaponFactory
{
    // 根据武器类型创建武器
    public static Weapon CreateWeapon(string weaponType)
    {
        switch (weaponType.ToLower())
        {
            case "sword":
                return new Sword();
            case "bow":
                return new Bow();
            case "staff":
                return new Staff();
            default:
                Debug.LogWarning($"[WeaponFactory] 未知武器类型: {weaponType}，返回默认剑");
                return new Sword();
        }
    }

    // 也可以按难度/等级生成武器
    public static Weapon CreateWeaponByLevel(int level)
    {
        if (level <= 3)
            return new Sword();
        else if (level <= 6)
            return new Bow();
        else
            return new Staff();
    }
}

// 使用示例
public class Player : MonoBehaviour
{
    private Weapon _currentWeapon;

    private void Start()
    {
        // 在游戏开始时根据玩家职业或等级获取武器
        int playerLevel = 5;
        _currentWeapon = WeaponFactory.CreateWeaponByLevel(playerLevel);
        _currentWeapon.ShowInfo();
    }

    public void AttackEnemy(Transform enemy)
    {
        if (_currentWeapon != null)
        {
            _currentWeapon.Attack(transform, enemy);
        }
    }
}
```

```csharp
using UnityEngine;

// ============================================
// 工厂方法模式 - 每个产品有独立的工厂
// 符合开闭原则，但类数量会增加
// ============================================

// 工厂接口：所有武器工厂的抽象
public interface IWeaponFactory
{
    Weapon CreateWeapon();       // 创建武器
    Weapon CreateSpecialWeapon(); // 创建特殊武器
}

// 剑类工厂
public class SwordFactory : IWeaponFactory
{
    public Weapon CreateWeapon()
    {
        return new Sword();
    }

    public Weapon CreateSpecialWeapon()
    {
        // 创建特殊剑（如火焰剑）
        return new FireSword();
    }
}

// 弓类工厂
public class BowFactory : IWeaponFactory
{
    public Weapon CreateWeapon()
    {
        return new Bow();
    }

    public Weapon CreateSpecialWeapon()
    {
        // 创建特殊弓（如冰霜弓）
        return new IceBow();
    }
}

// 特殊武器：火焰剑
public class FireSword : Sword
{
    public FireSword()
    {
        weaponName = "火焰剑";
        damage = 35;  // 比普通剑高
    }

    public override void Attack(Transform attacker, Transform target)
    {
        base.Attack(attacker, target);
        Debug.Log("[火焰剑] 附加火焰伤害！");
    }
}

// 特殊武器：冰霜弓
public class IceBow : Bow
{
    public IceBow()
    {
        weaponName = "冰霜弓";
        damage = 20;
    }

    public override void Attack(Transform attacker, Transform target)
    {
        base.Attack(attacker, target);
        Debug.Log("[冰霜弓] 附加冰冻效果！目标减速！");
    }
}

// ============================================
// 敌人工厂示例
// ============================================
public abstract class Enemy
{
    public string name;
    public int health;
    public int damage;
    public float speed;

    public abstract void Attack();
    public abstract void Move();
}

public class Slime : Enemy
{
    public Slime()
    {
        name = "史莱姆";
        health = 30;
        damage = 5;
        speed = 2f;
    }

    public override void Attack() { Debug.Log("[史莱姆] 撞击！"); }
    public override void Move() { Debug.Log("[史莱姆] 缓慢移动"); }
}

public class Skeleton : Enemy
{
    public Skeleton()
    {
        name = "骷髅兵";
        health = 60;
        damage = 12;
        speed = 3.5f;
    }

    public override void Attack() { Debug.Log("[骷髅兵] 挥剑攻击！"); }
    public override void Move() { Debug.Log("[骷髅兵] 行走"); }
}

public class Dragon : Enemy
{
    public Dragon()
    {
        name = "巨龙";
        health = 500;
        damage = 50;
        speed = 8f;
    }

    public override void Attack() { Debug.Log("[巨龙] 喷火！全屏攻击！"); }
    public override void Move() { Debug.Log("[巨龙] 飞行移动"); }
}

// 敌人工厂接口
public interface IEnemyFactory
{
    Enemy CreateEnemy();
}

// 史莱姆工厂
public class SlimeFactory : IEnemyFactory
{
    public Enemy CreateEnemy() => new Slime();
}

// 骷髅工厂
public class SkeletonFactory : IEnemyFactory
{
    public Enemy CreateEnemy() => new Skeleton();
}

// 巨龙工厂
public class DragonFactory : IEnemyFactory
{
    public Enemy CreateEnemy() => new Dragon();
}

// 敌人生成管理器：使用工厂方法
public class EnemySpawnManager : MonoBehaviour
{
    // 敌人预设表（关联 Inspector 中的预设）
    public SlimeFactory slimeFactory = new SlimeFactory();
    public SkeletonFactory skeletonFactory = new SkeletonFactory();
    public DragonFactory dragonFactory = new DragonFactory();

    // 创建普通敌人
    public Enemy CreateEnemy(string enemyType)
    {
        IEnemyFactory factory = enemyType switch
        {
            "slime" => slimeFactory,
            "skeleton" => skeletonFactory,
            "dragon" => dragonFactory,
            _ => slimeFactory
        };

        Enemy enemy = factory.CreateEnemy();
        Debug.Log($"[生成敌人] {enemy.name} HP:{enemy.health} DMG:{enemy.damage}");
        return enemy;
    }
}
```

```csharp
using UnityEngine;

// ============================================
// 抽象工厂模式 - 创建产品族
// 适用于：需要一系列相关产品的场景（如不同主题的装备套装）
// ============================================

// 产品族A：近战武器
public interface IMeleeWeapon
{
    void Attack();
}

// 产品族B：远程武器
public interface IRangedWeapon
{
    void Attack();
}

// 产品族C：护甲
public interface IArmor
{
    void Equip();
}

// ===== 骑士系列工厂（产品族1）=====
public class KnightMeleeWeapon : IMeleeWeapon
{
    public void Attack() { Debug.Log("[骑士剑] 重击！"); }
}

public class KnightRangedWeapon : IRangedWeapon
{
    public void Attack() { Debug.Log("[骑士弩] 射击！"); }
}

public class KnightArmor : IArmor
{
    public void Equip() { Debug.Log("[骑士铠甲] 穿戴完毕，防御+50"); }
}

public class KnightFactory : IEquipmentFactory
{
    public IMeleeWeapon CreateMelee() => new KnightMeleeWeapon();
    public IRangedWeapon CreateRanged() => new KnightRangedWeapon();
    public IArmor CreateArmor() => new KnightArmor();
}

// ===== 刺客系列工厂（产品族2）=====
public class AssassinMeleeWeapon : IMeleeWeapon
{
    public void Attack() { Debug.Log("[刺客匕首] 毒刃攻击！"); }
}

public class AssassinRangedWeapon : IRangedWeapon
{
    public void Attack() { Debug.Log("[飞镖] 投掷！"); }
}

public class AssassinArmor : IArmor
{
    public void Equip() { Debug.Log("[皮甲] 穿戴完毕，敏捷+30"); }
}

public class AssassinFactory : IEquipmentFactory
{
    public IMeleeWeapon CreateMelee() => new AssassinMeleeWeapon();
    public IRangedWeapon CreateRanged() => new AssassinRangedWeapon();
    public IArmor CreateArmor() => new AssassinArmor();
}

// ===== 法师系列工厂（产品族3）=====
public class MageMeleeWeapon : IMeleeWeapon
{
    public void Attack() { Debug.Log("[法杖敲击] 近战攻击！"); }
}

public class MageRangedWeapon : IRangedWeapon
{
    public void Attack() { Debug.Log("[魔法弹] 发射！"); }
}

public class MageArmor : IArmor
{
    public void Equip() { Debug.Log("[布甲] 穿戴完毕，魔力+100"); }
}

public class MageFactory : IEquipmentFactory
{
    public IMeleeWeapon CreateMelee() => new MageMeleeWeapon();
    public IRangedWeapon CreateRanged() => new MageRangedWeapon();
    public IArmor CreateArmor() => new MageArmor();
}

// 抽象工厂接口
public interface IEquipmentFactory
{
    IMeleeWeapon CreateMelee();
    IRangedWeapon CreateRanged();
    IArmor CreateArmor();
}

// 角色职业类型
public enum CharacterClass
{
    Knight,
    Assassin,
    Mage
}

// 装备系统：使用抽象工厂
public class EquipmentSystem : MonoBehaviour
{
    // 当前使用的工厂
    private IEquipmentFactory _factory;

    // 当前装备
    private IMeleeWeapon _meleeWeapon;
    private IRangedWeapon _rangedWeapon;
    private IArmor _armor;

    // 根据职业初始化工厂
    public void Initialize(CharacterClass characterClass)
    {
        _factory = characterClass switch
        {
            CharacterClass.Knight => new KnightFactory(),
            CharacterClass.Assassin => new AssassinFactory(),
            CharacterClass.Mage => new MageFactory(),
            _ => new KnightFactory()
        };

        // 创建装备
        _meleeWeapon = _factory.CreateMelee();
        _rangedWeapon = _factory.CreateRanged();
        _armor = _factory.CreateArmor();

        Debug.Log($"[装备系统] 已为 {characterClass} 初始化装备");
    }

    // 使用近战武器
    public void UseMelee() => _meleeWeapon?.Attack();

    // 使用远程武器
    public void UseRanged() => _rangedWeapon?.Attack();

    // 换装
    public void Equip() => _armor?.Equip();
}
```

### 游戏开发应用

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| 武器生成 | 简单工厂 | 武器种类有限，变化少 |
| 敌人刷新（波次） | 工厂方法 | 不同波次需要不同敌人，符合开闭 |
| 装备套装（职业系统） | 抽象工厂 | 每个职业有完整装备线 |
| 道具/掉落物 | 工厂方法 | 支持配置化生成 |
| UI 元素（弹窗/提示） | 简单工厂 | UI 元素种类固定 |

---

## 1.3 观察者模式（Observer）

### 生活类比

想象**机场的航班信息大屏**——你不需要每次都去问航空公司航班有没有延误，大屏会自动显示所有航班的最新状态。你只需要"订阅"你关心的航班，当状态变化时，大屏会主动通知你。这就是观察者模式：订阅者注册 → 主题变化 → 通知所有订阅者。

### 核心思想

**定义对象间的一对多依赖关系，当一个对象（主题）状态改变时，所有依赖它的对象（观察者）都会自动收到通知。**

### Unity C# 代码示例

```csharp
using UnityEngine;
using System;
using System.Collections.Generic;

// ============================================
// 事件参数基类
// ============================================
public class GameEventArgs : EventArgs
{
    // 可以添加通用属性，如时间戳
    public float timestamp;
    public GameEventArgs() => timestamp = Time.time;
}

// 具体事件参数：伤害事件
public class DamageEventArgs : GameEventArgs
{
    public GameObject receiver;   // 受伤者
    public GameObject attacker;  // 攻击者
    public int damageAmount;     // 伤害值
    public bool isCritical;      // 是否暴击

    public DamageEventArgs(GameObject receiver, GameObject attacker, int damage, bool crit = false)
    {
        this.receiver = receiver;
        this.attacker = attacker;
        this.damageAmount = damage;
        this.isCritical = crit;
    }
}

// 具体事件参数：分数变化事件
public class ScoreEventArgs : GameEventArgs
{
    public int previousScore;
    public int currentScore;
    public int delta;  // 变化量

    public ScoreEventArgs(int prev, int curr)
    {
        previousScore = prev;
        currentScore = curr;
        delta = curr - prev;
    }
}

// ============================================
// 事件管理器（主题/被观察者）
// 游戏中的中央事件总线
// ============================================
public class EventManager : MonoBehaviour
{
    // 单例
    public static EventManager Instance { get; private set; }

    // 事件字典：string 是事件名，Delegate 是处理函数
    // 使用 EventHandler<T> 类型安全地存储事件处理函数
    private Dictionary<string, Delegate> _eventDictionary;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
        _eventDictionary = new Dictionary<string, Delegate>();
    }

    // ========== 订阅事件 ==========
    // 无参数版本
    public void Subscribe(string eventName, Action handler)
    {
        AddDelegate(eventName, handler);
    }

    // 单参数版本
    public void Subscribe<T>(string eventName, Action<T> handler) where T : GameEventArgs
    {
        AddDelegate(eventName, handler);
    }

    // 内部方法：添加委托
    private void AddDelegate(string eventName, Delegate handler)
    {
        if (_eventDictionary.ContainsKey(eventName))
        {
            // 已有事件，合并委托（多播）
            _eventDictionary[eventName] = Delegate.Combine(_eventDictionary[eventName], handler);
        }
        else
        {
            // 新事件
            _eventDictionary[eventName] = handler;
        }
    }

    // ========== 取消订阅 ==========
    public void Unsubscribe(string eventName, Action handler)
    {
        RemoveDelegate(eventName, handler);
    }

    public void Unsubscribe<T>(string eventName, Action<T> handler) where T : GameEventArgs
    {
        RemoveDelegate(eventName, handler);
    }

    private void RemoveDelegate(string eventName, Delegate handler)
    {
        if (_eventDictionary.ContainsKey(eventName))
        {
            _eventDictionary[eventName] = Delegate.Remove(_eventDictionary[eventName], handler);

            // 如果委托列表为空，移除事件
            if (_eventDictionary[eventName] == null)
            {
                _eventDictionary.Remove(eventName);
            }
        }
    }

    // ========== 触发事件 ==========
    public void TriggerEvent(string eventName)
    {
        if (_eventDictionary.TryGetValue(eventName, out var handler))
        {
            // 转换为无参委托并调用
            (handler as Action)?.Invoke();
        }
    }

    public void TriggerEvent<T>(string eventName, T args) where T : GameEventArgs
    {
        if (_eventDictionary.TryGetValue(eventName, out var handler))
        {
            (handler as Action<T>)?.Invoke(args);
        }
    }

    // ========== 清除所有事件（场景切换时） ==========
    public void ClearAllEvents()
    {
        _eventDictionary.Clear();
    }
}
```

```csharp
using UnityEngine;
using UnityEngine.Events;  // Unity 内置的 UnityEvent

// ============================================
// UnityEvent 实现观察者模式
// UnityEvent 是 Unity 官方封装的观察者模式实现
// 支持在 Inspector 中可视化配置
// ============================================

// 定义可序列化的 UnityEvent，带参数
[System.Serializable]
public class IntEvent : UnityEvent<int> { }  // int 参数事件

[System.Serializable]
public class DamageEvent : UnityEvent<DamageEventArgs> { }  // 伤害事件

// 游戏事件 IDs（常量字符串，便于拼写检查）
public static class GameEvents
{
    // 分数相关
    public const string ON_SCORE_CHANGED = "OnScoreChanged";
    public const string ON_MULTIPLIER_CHANGED = "OnMultiplierChanged";

    // 战斗相关
    public const string ON_PLAYER_DAMAGED = "OnPlayerDamaged";
    public const string ON_ENEMY_DAMAGED = "OnEnemyDamaged";
    public const string ON_ENEMY_KILLED = "OnEnemyKilled";

    // 游戏状态
    public const string ON_GAME_PAUSED = "OnGamePaused";
    public const string ON_GAME_OVER = "OnGameOver";
    public const string ON_LEVEL_COMPLETE = "OnLevelComplete";

    // 道具/收集
    public const string ON_COIN_COLLECTED = "OnCoinCollected";
    public const string ON_POWERUP_COLLECTED = "OnPowerupCollected";
}

// ============================================
// 分数管理器
// ============================================
public class ScoreManager : MonoBehaviour
{
    // 单例
    public static ScoreManager Instance { get; private set; }

    // 当前分数
    private int _score = 0;
    public int Score
    {
        get => _score;
        private set
        {
            int oldScore = _score;
            _score = value;

            // 触发分数变化事件
            OnScoreChanged?.Invoke(_score);

            // 也可以用字符串方式触发（更灵活）
            var args = new ScoreEventArgs(oldScore, _score);
            EventManager.Instance.TriggerEvent(GameEvents.ON_SCORE_CHANGED, args);
        }
    }

    // UnityEvent 事件声明（可在 Inspector 中配置）
    public IntEvent OnScoreChanged;

    // 倍率
    private int _multiplier = 1;
    public int Multiplier
    {
        get => _multiplier;
        set
        {
            _multiplier = Mathf.Max(1, value);
            EventManager.Instance.TriggerEvent(GameEvents.ON_MULTIPLIER_CHANGED);
        }
    }

    private void Awake()
    {
        Instance = this;
    }

    // 增加分数
    public void AddScore(int amount)
    {
        Score += amount * _multiplier;
    }

    // 重置分数
    public void ResetScore()
    {
        Score = 0;
        _multiplier = 1;
    }
}

// ============================================
// UI 分数显示（观察者/订阅者）
// ============================================
public class ScoreDisplay : MonoBehaviour
{
    // UI 引用（需要从 Inspector 拖入）
    public UnityEngine.UI.Text scoreText;      // 分数文本
    public UnityEngine.UI.Image comboImage;    // 连击图片（用于显示倍率）

    private void Start()
    {
        // ========== 方式1：使用 UnityEvent 订阅（支持 Inspector 配置）==========
        if (ScoreManager.Instance != null)
        {
            ScoreManager.Instance.OnScoreChanged.AddListener(UpdateScoreDisplay);
        }

        // ========== 方式2：使用 EventManager 订阅 ==========
        EventManager.Instance.Subscribe(GameEvents.ON_SCORE_CHANGED, OnScoreChangedHandler);
        EventManager.Instance.Subscribe(GameEvents.ON_MULTIPLIER_CHANGED, OnMultiplierChanged);
    }

    private void OnDestroy()
    {
        // 取消订阅，防止内存泄漏
        if (ScoreManager.Instance != null)
        {
            ScoreManager.Instance.OnScoreChanged.RemoveListener(UpdateScoreDisplay);
        }
        EventManager.Instance.Unsubscribe(GameEvents.ON_SCORE_CHANGED, OnScoreChangedHandler);
        EventManager.Instance.Unsubscribe(GameEvents.ON_MULTIPLIER_CHANGED, OnMultiplierChanged);
    }

    // 更新分数显示（通过 UnityEvent 调用）
    private void UpdateScoreDisplay(int newScore)
    {
        if (scoreText != null)
        {
            scoreText.text = $"分数: {newScore:N0}";  // N0 表示千位分隔符
        }
    }

    // 分数变化处理（通过 EventManager 调用）
    private void OnScoreChangedHandler()
    {
        // 获取最新分数
        int score = ScoreManager.Instance.Score;
        UpdateScoreDisplay(score);

        // 可以添加动画效果
        // StartCoroutine(AnimateScoreChange());
    }

    // 倍率变化处理
    private void OnMultiplierChanged()
    {
        int multiplier = ScoreManager.Instance.Multiplier;
        Debug.Log($"[UI] 倍率变化: x{multiplier}");

        if (comboImage != null)
        {
            comboImage.gameObject.SetActive(multiplier > 1);
        }
    }
}

// ============================================
// 敌人死亡通知（发布者）
// ============================================
public class Enemy : MonoBehaviour
{
    public int maxHealth = 100;
    private int _currentHealth;

    // 事件
    public UnityEvent OnDeath;  // Unity 内置的 UnityEvent

    private void Start()
    {
        _currentHealth = maxHealth;
    }

    // 受到伤害
    public void TakeDamage(int damage, GameObject attacker)
    {
        _currentHealth -= damage;
        Debug.Log($"[敌人] 受到 {damage} 点伤害，剩余 {_currentHealth} HP");

        // 触发受伤事件
        var args = new DamageEventArgs(gameObject, attacker, damage);
        EventManager.Instance.TriggerEvent(GameEvents.ON_ENEMY_DAMAGED, args);

        if (_currentHealth <= 0)
        {
            Die(attacker);
        }
    }

    // 死亡
    private void Die(GameObject killer)
    {
        Debug.Log($"[敌人] {gameObject.name} 已死亡");

        // 触发死亡事件
        OnDeath?.Invoke();
        EventManager.Instance.TriggerEvent(GameEvents.ON_ENEMY_KILLED);

        // 通知杀敌者（便于统计分数）
        if (killer.CompareTag("Player"))
        {
            ScoreManager.Instance.AddScore(100);
        }

        // 销毁
        Destroy(gameObject);
    }
}

// ============================================
// 游戏结束检测（订阅者）
// ============================================
public class GameOverChecker : MonoBehaviour
{
    private int _enemyKillCount = 0;
    private int _requiredKills = 10;

    private void Start()
    {
        // 订阅敌人死亡事件
        EventManager.Instance.Subscribe(GameEvents.ON_ENEMY_KILLED, OnEnemyKilledHandler);
    }

    private void OnDestroy()
    {
        EventManager.Instance.Unsubscribe(GameEvents.ON_ENEMY_KILLED, OnEnemyKilledHandler);
    }

    private void OnEnemyKilledHandler()
    {
        _enemyKillCount++;
        Debug.Log($"[游戏结束检测] 已击杀 {_enemyKillCount}/{_requiredKills} 敌人");

        if (_enemyKillCount >= _requiredKills)
        {
            TriggerGameOver();
        }
    }

    private void TriggerGameOver()
    {
        Debug.Log("[游戏结束检测] 所有敌人已击杀，触发游戏胜利！");
        EventManager.Instance.TriggerEvent(GameEvents.ON_LEVEL_COMPLETE);
    }
}
```

### 游戏开发应用

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| UI 更新 | UnityEvent | 可视化配置，简便 |
| 系统解耦 | 自定义 EventManager | 完全解耦，支持跨场景 |
| 分数/统计 | UnityEvent | 简单直接 |
| 成就系统 | EventManager | 成就条件多样化 |
| 伤害数字 | EventManager | 可能有多个监听者 |

---

## 1.4 状态模式（State）

### 生活类比

想象**电梯的控制系统**——电梯有"停止"、"上升"、"下降"三种状态。在"上升"状态下，电梯只能响应同方向（或更高楼层）的请求；在"停止"状态下，可以响应任何请求。状态模式让电梯的行为根据当前状态自动变化，而不需要大量的 if-else。

### 核心思想

**允许对象在内部状态改变时改变它的行为。看起来好像对象修改了它的类。**

### Unity C# 代码示例

```csharp
using UnityEngine;

// ============================================
// 状态接口 - 所有状态的抽象
// ============================================
public interface IPlayerState
{
    // 状态名称（调试用）
    string StateName { get; }

    // 进入状态
    void Enter(PlayerController player);

    // 退出状态
    void Exit(PlayerController player);

    // 每帧更新（逻辑更新）
    void LogicUpdate(PlayerController player);

    // 物理更新（FixedUpdate）
    void PhysicsUpdate(PlayerController player);

    // 处理输入
    void HandleInput(PlayerController player);
}

// ============================================
// 玩家控制器 - 使用状态模式
// ============================================
public class PlayerController : MonoBehaviour
{
    // 刚体引用
    private Rigidbody _rigidbody;

    // 当前状态
    private IPlayerState _currentState;

    // ========== 状态引用（所有状态实例）==========
    public PlayerIdleState IdleState { get; private set; }
    public PlayerRunState RunState { get; private set; }
    public PlayerJumpState JumpState { get; private set; }
    public PlayerAttackState AttackState { get; private set; }
    public PlayerHurtState HurtState { get; private set; }

    // ========== 玩家属性 ==========
    public float moveSpeed = 5f;
    public float jumpForce = 8f;
    public float attackRange = 2f;
    public int maxHealth = 100;
    private int _currentHealth;

    public int CurrentHealth
    {
        get => _currentHealth;
        set
        {
            _currentHealth = Mathf.Clamp(value, 0, maxHealth);
        }
    }

    // ========== 输入状态 ==========
    public float MoveInput { get; set; }  // 移动输入（-1 到 1）
    public bool JumpInput { get; set; }    // 跳跃输入
    public bool AttackInput { get; set; } // 攻击输入
    public bool IsGrounded { get; set; }  // 是否在地面上

    // ========== 组件缓存 ==========
    private void Awake()
    {
        _rigidbody = GetComponent<Rigidbody>();

        // ========== 初始化所有状态 ==========
        IdleState = new PlayerIdleState();
        RunState = new PlayerRunState();
        JumpState = new PlayerJumpState();
        AttackState = new PlayerAttackState();
        HurtState = new PlayerHurtState();

        _currentHealth = maxHealth;
    }

    private void Start()
    {
        // 初始状态为待机
        TransitionToState(IdleState);
    }

    private void Update()
    {
        // 读取输入
        MoveInput = Input.GetAxisRaw("Horizontal");  // A/D 或 左右方向键
        JumpInput = Input.GetButtonDown("Jump");     // 空格键
        AttackInput = Input.GetButtonDown("Fire1"); // 鼠标左键或 Ctrl

        // 检查是否在地面上
        IsGrounded = Physics.Raycast(transform.position, Vector3.down, 0.1f);

        // 状态逻辑更新
        _currentState?.LogicUpdate(this);

        // 状态转换检查
        _currentState?.HandleInput(this);
    }

    private void FixedUpdate()
    {
        // 物理更新
        _currentState?.PhysicsUpdate(this);
    }

    // ========== 状态转换 ==========
    public void TransitionToState(IPlayerState newState)
    {
        _currentState?.Exit(this);
        _currentState = newState;
        _currentState?.Enter(this);
        Debug.Log($"[状态转换] {_currentState.StateName}");
    }

    // ========== 提供给状态的公共方法 ==========
    public void Move(float direction)
    {
        _rigidbody.velocity = new Vector3(direction * moveSpeed, _rigidbody.velocity.y, 0);
    }

    public void Jump()
    {
        _rigidbody.velocity = new Vector3(_rigidbody.velocity.x, jumpForce, 0);
    }

    public void ApplyDamage(int damage)
    {
        CurrentHealth -= damage;
        Debug.Log($"[玩家] 受伤！剩余血量 {CurrentHealth}");

        // 受伤时切换到受击状态
        TransitionToState(HurtState);
    }

    // 播放动画（简化版）
    public void PlayAnimation(string animName)
    {
        // 实际项目中会调用 Animator
        Debug.Log($"[动画] 播放 {animName}");
    }
}

// ============================================
// 待机状态
// ============================================
public class PlayerIdleState : IPlayerState
{
    public string StateName => "待机";

    public void Enter(PlayerController player)
    {
        player.PlayAnimation("Idle");
    }

    public void Exit(PlayerController player)
    {
        // 退出时的清理
    }

    public void LogicUpdate(PlayerController player)
    {
        // 待机时只需要等待
    }

    public void PhysicsUpdate(PlayerController player)
    {
        // 停止水平移动
        player._rigidbody.velocity = new Vector3(0, player._rigidbody.velocity.y, 0);
    }

    public void HandleInput(PlayerController player)
    {
        // 有移动输入 → 奔跑
        if (Mathf.Abs(player.MoveInput) > 0.1f)
        {
            player.TransitionToState(player.RunState);
        }
        // 跳跃输入且在地面 → 跳跃
        else if (player.JumpInput && player.IsGrounded)
        {
            player.TransitionToState(player.JumpState);
        }
        // 攻击输入 → 攻击
        else if (player.AttackInput)
        {
            player.TransitionToState(player.AttackState);
        }
    }
}

// ============================================
// 奔跑状态
// ============================================
public class PlayerRunState : IPlayerState
{
    public string StateName => "奔跑";

    public void Enter(PlayerController player)
    {
        player.PlayAnimation("Run");
    }

    public void Exit(PlayerController player)
    {
        // 退出奔跑时的清理
    }

    public void LogicUpdate(PlayerController player)
    {
        // 面向移动方向
        float direction = player.MoveInput;
        if (Mathf.Abs(direction) > 0.1f)
        {
            player.transform.localScale = new Vector3(Mathf.Sign(direction), 1, 1);
        }
    }

    public void PhysicsUpdate(PlayerController player)
    {
        player.Move(player.MoveInput);
    }

    public void HandleInput(PlayerController player)
    {
        // 无移动输入 → 待机
        if (Mathf.Abs(player.MoveInput) < 0.1f)
        {
            player.TransitionToState(player.IdleState);
        }
        // 跳跃输入 → 跳跃
        else if (player.JumpInput && player.IsGrounded)
        {
            player.TransitionToState(player.JumpState);
        }
        // 攻击输入 → 攻击
        else if (player.AttackInput)
        {
            player.TransitionToState(player.AttackState);
        }
    }
}

// ============================================
// 跳跃状态
// ============================================
public class PlayerJumpState : IPlayerState
{
    public string StateName => "跳跃";

    public void Enter(PlayerController player)
    {
        player.PlayAnimation("Jump");
        player.Jump();
    }

    public void Exit(PlayerController player)
    {
        // 跳跃结束
    }

    public void LogicUpdate(PlayerController player)
    {
        // 可以添加空中动作控制
    }

    public void PhysicsUpdate(PlayerController player)
    {
        // 保持水平移动（空中可以控制方向）
        float horizontalVel = player.MoveInput * player.moveSpeed;
        player._rigidbody.velocity = new Vector3(horizontalVel, player._rigidbody.velocity.y, 0);
    }

    public void HandleInput(PlayerController player)
    {
        // 攻击输入 → 空中攻击
        if (player.AttackInput)
        {
            player.TransitionToState(player.AttackState);
        }
        // 落地检测 → 根据是否有移动输入决定是待机还是奔跑
        else if (player.IsGrounded && Mathf.Abs(player.MoveInput) < 0.1f)
        {
            player.TransitionToState(player.IdleState);
        }
        else if (player.IsGrounded && Mathf.Abs(player.MoveInput) > 0.1f)
        {
            player.TransitionToState(player.RunState);
        }
    }
}

// ============================================
// 攻击状态
// ============================================
public class PlayerAttackState : IPlayerState
{
    public string StateName => "攻击";
    private float _attackDuration = 0.3f;  // 攻击持续时间
    private float _attackTimer = 0f;

    public void Enter(PlayerController player)
    {
        _attackTimer = 0f;
        player.PlayAnimation("Attack");
        Debug.Log($"[攻击] 造成 {10} 点伤害（示例）");
    }

    public void Exit(PlayerController player)
    {
        // 清理攻击状态
    }

    public void LogicUpdate(PlayerController player)
    {
        _attackTimer += Time.deltaTime;
    }

    public void PhysicsUpdate(PlayerController player)
    {
        // 攻击时停止移动
        player._rigidbody.velocity = Vector3.zero;
    }

    public void HandleInput(PlayerController player)
    {
        // 攻击动画播放完毕
        if (_attackTimer >= _attackDuration)
        {
            // 根据是否在地面和是否有移动输入决定下一个状态
            if (!player.IsGrounded)
            {
                player.TransitionToState(player.JumpState);
            }
            else if (Mathf.Abs(player.MoveInput) > 0.1f)
            {
                player.TransitionToState(player.RunState);
            }
            else
            {
                player.TransitionToState(player.IdleState);
            }
        }
    }
}

// ============================================
// 受击状态
// ============================================
public class PlayerHurtState : IPlayerState
{
    public string StateName => "受击";
    private float _hurtDuration = 0.5f;  // 受击硬直时间
    private float _hurtTimer = 0f;

    public void Enter(PlayerController player)
    {
        _hurtTimer = 0f;
        player.PlayAnimation("Hurt");
        Debug.Log("[受击] 玩家被攻击了！");
    }

    public void Exit(PlayerController player)
    {
        // 退出受击状态
    }

    public void LogicUpdate(PlayerController player)
    {
        _hurtTimer += Time.deltaTime;
    }

    public void PhysicsUpdate(PlayerController player)
    {
        // 受击时减速/停止
        player._rigidbody.velocity *= 0.9f;
    }

    public void HandleInput(PlayerController player)
    {
        // 受击状态结束后检查死亡
        if (_hurtTimer >= _hurtDuration)
        {
            if (player.CurrentHealth <= 0)
            {
                Debug.Log("[游戏] 玩家死亡！");
                // 切换到死亡状态或触发游戏结束
            }
            else
            {
                player.TransitionToState(player.IdleState);
            }
        }
    }
}
```

```csharp
using UnityEngine;

// ============================================
// Unity Animator Controller 状态机
// Unity 内置的可视化状态机
// ============================================

/*
 * 在 Unity 中使用 Animator Controller 的步骤：
 * 1. 创建 Animator Controller 资源（右键 → Create → Animator Controller）
 * 2. 在 Animator 窗口中设计状态转换
 * 3. 创建 Animation Clips（待机、跑步、跳跃等动画）
 * 4. 设置参数（float/int/bool/trigger）控制转换
 * 5. 在代码中设置参数值
 */

public class AnimatorStateExample : MonoBehaviour
{
    // Animator 组件引用
    private Animator _animator;

    // 动画参数名称（常量）
    private static readonly int SpeedParam = Animator.StringToHash("Speed");
    private static readonly int IsGroundedParam = Animator.StringToHash("IsGrounded");
    private static readonly int JumpTrigger = Animator.StringToHash("Jump");
    private static readonly int AttackTrigger = Animator.StringToHash("Attack");
    private static readonly int HurtTrigger = Animator.StringToHash("Hurt");

    private void Awake()
    {
        _animator = GetComponent<Animator>();
    }

    // 在需要的地方调用这些方法
    public void UpdateMovement(float horizontalInput, bool isGrounded)
    {
        // 设置 float 参数（Speed）
        _animator.SetFloat(SpeedParam, Mathf.Abs(horizontalInput));

        // 设置 bool 参数（IsGrounded）
        _animator.SetBool(IsGroundedParam, isGrounded);
    }

    public void TriggerJump()
    {
        // 设置 Trigger 参数（触发一次后自动清除）
        _animator.SetTrigger(JumpTrigger);
    }

    public void TriggerAttack()
    {
        _animator.SetTrigger(AttackTrigger);
    }

    public void TriggerHurt()
    {
        _animator.SetTrigger(HurtTrigger);
    }

    // 监听动画事件（通过 Animation Event 调用）
    public void OnAttackHit()
    {
        Debug.Log("[动画事件] 攻击命中！");
        // 在这里进行伤害判定、播放音效等
    }

    public void OnAnimationEnd()
    {
        Debug.Log("[动画事件] 动画播放结束");
    }
}

// ============================================
// Animator 状态机行为（State Machine Behaviour）
// 在状态进入/退出时执行逻辑
// ============================================
public class AttackStateBehaviour : StateMachineBehaviour
{
    // OnStateEnter：进入状态时调用
    override public void OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        Debug.Log($"[状态行为] 进入攻击状态: {stateInfo.shortNameHash}");
        // 可以在这里禁用玩家输入等
    }

    // OnStateUpdate：每帧调用
    override public void OnStateUpdate(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        // 可以在这里检查条件
    }

    // OnStateExit：退出状态时调用
    override public void OnStateExit(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        Debug.Log($"[状态行为] 退出攻击状态: {stateInfo.shortNameHash}");
        // 可以在这里恢复玩家输入等
    }

    // OnStateMove：更新根运动
    override public void OnStateMove(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        // 可以在这里处理根运动
    }

    // OnStateIK：处理反向动力学
    override public void OnStateIK(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        // 可以在这里处理 IK（如手的 IK 权宜）
    }
}
```

### 游戏开发应用

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| 角色动作状态机 | 代码状态机 | 灵活，可调试 |
| AI 行为状态机 | 代码状态机 | 复杂逻辑需要代码控制 |
| UI 菜单状态 | Unity Animator | 可视化，简便 |
| 游戏流程状态 | 代码状态机 | 易于扩展 |

---

## 1.5 策略模式（Strategy）

### 生活类比

想象**出行方式的选择**——你可以选择骑自行车、坐公交、打车或步行。每种方式都是"到达目的地"这个问题的不同解决策略，你可以根据天气、时间、预算等因素随时切换策略，而不需要改变出行本身。

### 核心思想

**定义一系列算法，把它们一个个封装起来，并且使它们可以相互替换。**

策略模式让算法的变化独立于使用它的客户。

### Unity C# 代码示例

```csharp
using UnityEngine;
using System;

// ============================================
// 策略接口 - 所有攻击策略的抽象
// ============================================
public interface IAttackStrategy
{
    // 策略名称
    string StrategyName { get; }

    // 执行攻击
    void Attack(Transform attacker, Transform target);

    // 获取伤害值
    int GetDamage();
}

// ============================================
// 具体策略：普通攻击
// ============================================
public class NormalAttackStrategy : IAttackStrategy
{
    public string StrategyName => "普通攻击";

    private int _damage = 10;
    private float _range = 2f;
    private float _cooldown = 0.5f;

    public void Attack(Transform attacker, Transform target)
    {
        float distance = Vector3.Distance(attacker.position, target.position);
        if (distance <= _range)
        {
            Debug.Log($"[普通攻击] {attacker.name} 对 {target.name} 造成 {_damage} 点伤害");
        }
        else
        {
            Debug.Log("[普通攻击] 目标距离太远");
        }
    }

    public int GetDamage() => _damage;
}

// ============================================
// 具体策略：暴击攻击
// ============================================
public class CriticalAttackStrategy : IAttackStrategy
{
    public string StrategyName => "暴击攻击";

    private int _baseDamage = 10;
    private float _criticalRate = 0.3f;  // 30% 暴击率
    private float _criticalMultiplier = 2f;  // 暴击伤害倍率
    private float _range = 2f;

    public void Attack(Transform attacker, Transform target)
    {
        float distance = Vector3.Distance(attacker.position, target.position);
        if (distance <= _range)
        {
            bool isCritical = UnityEngine.Random.value <= _criticalRate;
            int damage = isCritical ? (int)(_baseDamage * _criticalMultiplier) : _baseDamage;

            string attackType = isCritical ? "暴击！" : "普通";
            Debug.Log($"[{attackType}] {attacker.name} 对 {target.name} 造成 {damage} 点伤害");
        }
    }

    public int GetDamage()
    {
        return UnityEngine.Random.value <= _criticalRate
            ? (int)(_baseDamage * _criticalMultiplier)
            : _baseDamage;
    }
}

// ============================================
// 具体策略：范围攻击
// ============================================
public class AreaAttackStrategy : IAttackStrategy
{
    public string StrategyName => "范围攻击";

    private int _damage = 15;
    private float _radius = 5f;

    public void Attack(Transform attacker, Transform target)
    {
        // 检测范围内所有敌人
        Collider[] hitColliders = Physics.OverlapSphere(attacker.position, _radius);
        int hitCount = 0;

        foreach (var hit in hitColliders)
        {
            if (hit.CompareTag("Enemy"))
            {
                Debug.Log($"[范围攻击] {attacker.name} 对 {hit.name} 造成 {_damage} 点伤害");
                hitCount++;
            }
        }

        Debug.Log($"[范围攻击] 共命中 {hitCount} 个敌人");
    }

    public int GetDamage() => _damage;
}

// ============================================
// 具体策略：远程攻击
// ============================================
public class RangedAttackStrategy : IAttackStrategy
{
    public string StrategyName => "远程攻击";

    private int _damage = 20;
    private float _range = 50f;
    private GameObject _projectilePrefab;  // 子弹/箭矢预制体

    public RangedAttackStrategy(GameObject projectile)
    {
        _projectilePrefab = projectile;
    }

    public void Attack(Transform attacker, Transform target)
    {
        float distance = Vector3.Distance(attacker.position, target.position);
        if (distance <= _range)
        {
            Debug.Log($"[远程攻击] {attacker.name} 向 {target.name} 发射子弹");

            // 实例化子弹
            if (_projectilePrefab != null)
            {
                Vector3 direction = (target.position - attacker.position).normalized;
                // var projectile = UnityEngine.Object.Instantiate(
                //     _projectilePrefab,
                //     attacker.position + direction,
                //     Quaternion.LookRotation(direction)
                // );
            }
        }
        else
        {
            Debug.Log("[远程攻击] 目标超出射程");
        }
    }

    public int GetDamage() => _damage;
}

// ============================================
// 敌人 AI - 使用策略模式
// ============================================
public class EnemyAI : MonoBehaviour
{
    // 当前攻击策略
    private IAttackStrategy _currentStrategy;

    // 可用策略列表
    public NormalAttackStrategy NormalAttack = new NormalAttackStrategy();
    public CriticalAttackStrategy CriticalAttack = new CriticalAttackStrategy();
    public AreaAttackStrategy AreaAttack = new AreaAttackStrategy();

    // 目标引用
    public Transform target;

    private void Start()
    {
        // 默认使用普通攻击
        SetStrategy(NormalAttack);
    }

    // 设置攻击策略
    public void SetStrategy(IAttackStrategy strategy)
    {
        _currentStrategy = strategy;
        Debug.Log($"[敌人AI] 切换策略: {strategy.StrategyName}");
    }

    private void Update()
    {
        // 检测目标
        if (target == null) return;

        // 如果玩家在范围内，执行攻击
        float distance = Vector3.Distance(transform.position, target.position);

        // 简单 AI：根据距离选择策略
        if (distance <= 3f)
        {
            // 近距离：普通攻击
            SetStrategy(NormalAttack);
        }
        else if (distance <= 8f)
        {
            // 中距离：暴击
            SetStrategy(CriticalAttack);
        }
        else
        {
            // 远距离：范围攻击
            SetStrategy(AreaAttack);
        }

        // 执行攻击
        if (distance <= 10f)
        {
            _currentStrategy.Attack(transform, target);
        }
    }
}

// ============================================
// 玩家武器系统 - 策略模式应用
// ============================================
public class WeaponSystem : MonoBehaviour
{
    // 当前武器策略
    private IAttackStrategy _weaponStrategy;

    // 武器类型枚举
    public enum WeaponType
    {
        Sword,
        Axe,
        Hammer
    }

    // 设置武器
    public void SetWeapon(WeaponType type)
    {
        _weaponStrategy = type switch
        {
            WeaponType.Sword => new SwordStrategy(),
            WeaponType.Axe => new AxeStrategy(),
            WeaponType.Hammer => new HammerStrategy(),
            _ => new SwordStrategy()
        };
        Debug.Log($"[武器系统] 切换武器: {type}");
    }

    // 攻击
    public void Attack(Transform target)
    {
        _weaponStrategy?.Attack(transform, target);
    }
}

// 剑策略
public class SwordStrategy : IAttackStrategy
{
    public string StrategyName => "剑";

    public void Attack(Transform attacker, Transform target)
    {
        Debug.Log($"[剑] 快速挥砍！");
    }

    public int GetDamage() => 15;
}

// 斧策略
public class AxeStrategy : IAttackStrategy
{
    public string StrategyName => "斧";

    public void Attack(Transform attacker, Transform target)
    {
        Debug.Log($"[斧] 重劈！");
    }

    public int GetDamage() => 25;
}

// 锤策略
public class HammerStrategy : IAttackStrategy
{
    public string StrategyName => "锤";

    public void Attack(Transform attacker, Transform target)
    {
        Debug.Log($"[锤] 猛烈砸击！有击退效果！");
    }

    public int GetDamage() => 35;
}
```

### 游戏开发应用

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| 敌人 AI 行为 | 策略模式 | 行为多样化，容易切换 |
| 武器系统 | 策略模式 | 武器种类多，行为各异 |
| 敌人出生/死亡 | 工厂模式 | 创建逻辑更合适 |
| 伤害计算 | 策略模式 | 不同类型伤害计算规则不同 |

---

## 1.6 命令模式（Command）

### 生活类比

想象**餐厅的点餐系统**——顾客不需要直接去厨房，而是把订单交给服务员，服务员再把订单传给厨师。厨师不需要知道是谁点的菜，只需要按订单做菜。命令模式把"请求"（点菜）封装成对象，可以存储、排队、执行撤销。

### 核心思想

**将请求封装成对象，从而可以用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤销的操作。**

### Unity C# 代码示例

```csharp
using UnityEngine;
using System;
using System.Collections.Generic;

// ============================================
// 命令接口 - 所有命令的抽象
// ============================================
public interface ICommand
{
    // 执行命令
    void Execute();

    // 撤销命令
    void Undo();
}

// ============================================
// 移动命令
// ============================================
public class MoveCommand : ICommand
{
    private Transform _target;
    private Vector3 _direction;
    private float _distance;

    public MoveCommand(Transform target, Vector3 direction, float distance)
    {
        _target = target;
        _direction = direction.normalized;
        _distance = distance;
    }

    public void Execute()
    {
        _target.position += _direction * _distance;
        Debug.Log($"[移动命令] {_target.name} 移动 {_distance} 单位");
    }

    public void Undo()
    {
        _target.position -= _direction * _distance;
        Debug.Log($"[撤销] 取消移动");
    }
}

// ============================================
// 旋转命令
// ============================================
public class RotateCommand : ICommand
{
    private Transform _target;
    private Vector3 _rotation;

    public RotateCommand(Transform target, Vector3 rotation)
    {
        _target = target;
        _rotation = rotation;
    }

    public void Execute()
    {
        _target.rotation = Quaternion.Euler(_target.eulerAngles + _rotation);
        Debug.Log($"[旋转命令] {_target.name} 旋转 {_rotation}");
    }

    public void Undo()
    {
        _target.rotation = Quaternion.Euler(_target.eulerAngles - _rotation);
        Debug.Log($"[撤销] 取消旋转");
    }
}

// ============================================
// 攻击命令
// ============================================
public class AttackCommand : ICommand
{
    private PlayerController _player;
    private int _damage;

    // 记录攻击前的状态（用于可能的撤销）
    private Vector3 _playerPosition;

    public AttackCommand(PlayerController player, int damage)
    {
        _player = player;
        _damage = damage;
    }

    public void Execute()
    {
        _playerPosition = _player.transform.position;
        // 执行攻击逻辑
        Debug.Log($"[攻击命令] {_player.name} 发动攻击，造成 {_damage} 伤害");
        // _player.PlayAttackAnimation();
        // _player.DealDamage(_damage);
    }

    public void Undo()
    {
        Debug.Log($"[撤销] 取消攻击");
        // 如果需要，可以恢复位置等状态
    }
}

// ============================================
// 命令管理器 - 存储历史记录，支持撤销/重做
// ============================================
public class CommandManager : MonoBehaviour
{
    // 单例
    public static CommandManager Instance { get; private set; }

    // 命令历史栈（用于撤销）
    private Stack<ICommand> _commandHistory = new Stack<ICommand>();

    // 重做栈
    private Stack<ICommand> _redoHistory = new Stack<ICommand>();

    // 最大历史记录数
    private const int MaxHistorySize = 100;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;
    }

    // 执行命令
    public void ExecuteCommand(ICommand command)
    {
        command.Execute();
        _commandHistory.Push(command);

        // 清空重做历史（新命令后不能重做）
        _redoHistory.Clear();

        // 限制历史大小
        if (_commandHistory.Count > MaxHistorySize)
        {
            var tempStack = new Stack<ICommand>();
            for (int i = 0; i < MaxHistorySize; i++)
            {
                tempStack.Push(_commandHistory.Pop());
            }
            _commandHistory = tempStack;
        }
    }

    // 撤销
    public void Undo()
    {
        if (_commandHistory.Count == 0)
        {
            Debug.Log("[命令管理] 没有可撤销的命令");
            return;
        }

        ICommand command = _commandHistory.Pop();
        command.Undo();
        _redoHistory.Push(command);

        Debug.Log($"[命令管理] 撤销成功，剩余 {_commandHistory.Count} 条历史");
    }

    // 重做
    public void Redo()
    {
        if (_redoHistory.Count == 0)
        {
            Debug.Log("[命令管理] 没有可重做的命令");
            return;
        }

        ICommand command = _redoHistory.Pop();
        command.Execute();
        _commandHistory.Push(command);

        Debug.Log($"[命令管理] 重做成功");
    }

    // 清空历史
    public void ClearHistory()
    {
        _commandHistory.Clear();
        _redoHistory.Clear();
        Debug.Log("[命令管理] 历史已清空");
    }
}

// ============================================
// 输入处理 - 将输入转换为命令
// ============================================
public class InputHandler : MonoBehaviour
{
    public PlayerController player;

    private void Update()
    {
        // 移动命令
        float horizontal = Input.GetAxisRaw("Horizontal");
        float vertical = Input.GetAxisRaw("Vertical");

        if (horizontal != 0 || vertical != 0)
        {
            Vector3 direction = new Vector3(horizontal, 0, vertical).normalized;
            MoveCommand moveCmd = new MoveCommand(player.transform, direction, 1f);
            CommandManager.Instance.ExecuteCommand(moveCmd);
        }

        // 旋转命令
        if (Input.GetKeyDown(KeyCode.Q))
        {
            RotateCommand rotateCmd = new RotateCommand(player.transform, new Vector3(0, -45, 0));
            CommandManager.Instance.ExecuteCommand(rotateCmd);
        }
        if (Input.GetKeyDown(KeyCode.E))
        {
            RotateCommand rotateCmd = new RotateCommand(player.transform, new Vector3(0, 45, 0));
            CommandManager.Instance.ExecuteCommand(rotateCmd);
        }

        // 攻击命令
        if (Input.GetButtonDown("Fire1"))
        {
            AttackCommand attackCmd = new AttackCommand(player, 20);
            CommandManager.Instance.ExecuteCommand(attackCmd);
        }

        // 撤销（Ctrl+Z）
        if (Input.GetKeyDown(KeyCode.Z) && Input.GetKey(KeyCode.LeftControl))
        {
            CommandManager.Instance.Undo();
        }

        // 重做（Ctrl+Y）
        if (Input.GetKeyDown(KeyCode.Y) && Input.GetKey(KeyCode.LeftControl))
        {
            CommandManager.Instance.Redo();
        }
    }
}

// ============================================
// 宏命令 - 将多个命令组合成一个
// ============================================
public class MacroCommand : ICommand
{
    private ICommand[] _commands;

    public MacroCommand(ICommand[] commands)
    {
        _commands = commands;
    }

    public void Execute()
    {
        foreach (var command in _commands)
        {
            command.Execute();
        }
    }

    public void Undo()
    {
        // 逆序撤销
        for (int i = _commands.Length - 1; i >= 0; i--)
        {
            _commands[i].Undo();
        }
    }
}

// ============================================
// 技能系统 - 命令模式应用
// ============================================
public class SkillCommand : ICommand
{
    private PlayerController _player;
    private string _skillName;
    private float _cooldown;
    private Action _skillAction;

    public SkillCommand(PlayerController player, string skillName, float cooldown, Action skillAction)
    {
        _player = player;
        _skillName = skillName;
        _cooldown = cooldown;
        _skillAction = skillAction;
    }

    public void Execute()
    {
        Debug.Log($"[技能命令] 使用技能: {_skillName}");
        _skillAction?.Invoke();
        // 可以添加冷却时间管理
    }

    public void Undo()
    {
        Debug.Log($"[撤销] 技能 {_skillName} 已撤销（返还冷却等）");
    }
}
```

### 游戏开发应用

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| 操作历史/撤销 | 命令模式 | 天然支持撤销/重做 |
| 输入处理 | 命令模式 | 解耦输入与执行 |
| 录像/回放 | 命令模式 | 存储命令序列即可 |
| 宏命令 | 命令模式 | 组合多个操作 |
| 技能系统 | 命令模式 | 支持冷却、撤销 |

---

## 1.7 对象池模式（Pool）

> 详见 Stage 2 章节，这里深化讨论。

### 生活类比

想象**餐厅的餐具管理**——餐厅不会每来一个客人就买一套新餐具，而是准备一定数量的餐具，用完清洗后再用。对象池就是游戏的"餐具管理"，预先创建对象，用完回收复用，避免频繁的创建/销毁带来的性能开销。

### 核心思想

**预先分配一定数量的对象，需要时获取，用完后回收复用，而非销毁。**

### Unity C# 代码示例

```csharp
using UnityEngine;
using System;
using System.Collections.Generic;

// ============================================
// 通用对象池
// ============================================
public class ObjectPool
{
    // 对象预制体
    private GameObject _prefab;

    // 可用对象队列
    private Queue<GameObject> _availableQueue = new Queue<GameObject>();

    // 父物体（用于整理 Hierarchy）
    private Transform _parent;

    // 初始预热数量
    private int _preloadCount;

    // 已创建对象总数（调试用）
    public int TotalCreated { get; private set; }

    // 当前可用数量
    public int AvailableCount => _availableQueue.Count;

    // 构造函数
    public ObjectPool(GameObject prefab, int preloadCount = 0, Transform parent = null)
    {
        _prefab = prefab;
        _parent = parent;
        _preloadCount = preloadCount;

        // 预热
        Preload();
    }

    // 预创建对象
    private void Preload()
    {
        for (int i = 0; i < _preloadCount; i++)
        {
            CreateNewObject();
        }
    }

    // 创建新对象
    private GameObject CreateNewObject()
    {
        GameObject obj = UnityEngine.Object.Instantiate(_prefab, _parent);
        obj.SetActive(false);  // 创建后默认禁用
        obj.AddComponent<PooledObject>().Pool = this;  // 添加池引用
        _availableQueue.Enqueue(obj);
        TotalCreated++;
        return obj;
    }

    // 获取对象
    public GameObject Get(Vector3 position, Quaternion rotation)
    {
        GameObject obj;

        if (_availableQueue.Count > 0)
        {
            // 从池中取
            obj = _availableQueue.Dequeue();
        }
        else
        {
            // 池空了，创建新的
            obj = CreateNewObject();
        }

        // 设置Transform
        obj.transform.position = position;
        obj.transform.rotation = rotation;
        obj.SetActive(true);

        return obj;
    }

    // 获取对象（使用默认位置和旋转）
    public GameObject Get()
    {
        return Get(Vector3.zero, Quaternion.identity);
    }

    // 归还对象到池
    public void Return(GameObject obj)
    {
        obj.SetActive(false);
        _availableQueue.Enqueue(obj);
    }
}

// ============================================
// 池中对象组件 - 自动归还到池
// ============================================
public class PooledObject : MonoBehaviour
{
    public ObjectPool Pool { get; set; }

    // 当对象被销毁时自动归还到池
    private void OnDestroy()
    {
        if (Pool != null && gameObject.activeInHierarchy)
        {
            Pool.Return(gameObject);
        }
    }
}

// ============================================
// 对象池管理器 - 集中管理所有池
// ============================================
public class PoolManager : MonoBehaviour
{
    // 单例
    public static PoolManager Instance { get; private set; }

    // 所有池的字典
    private Dictionary<string, ObjectPool> _pools = new Dictionary<string, ObjectPool>();

    // 预制体字典（Inspector 中配置）
    [Serializable]
    public class PoolConfig
    {
        public string poolName;      // 池名称（唯一标识）
        public GameObject prefab;    // 预制体
        public int preloadCount = 10; // 预加载数量
    }

    public PoolConfig[] poolConfigs;

    private void Awake()
    {
        if (Instance != null && Instance != this)
        {
            Destroy(gameObject);
            return;
        }
        Instance = this;

        // 初始化所有池
        InitializePools();
    }

    // 初始化所有配置的池
    private void InitializePools()
    {
        foreach (var config in poolConfigs)
        {
            if (config.prefab == null) continue;

            CreatePool(config.poolName, config.prefab, config.preloadCount);
        }
    }

    // 创建新池
    public void CreatePool(string name, GameObject prefab, int preloadCount = 0)
    {
        if (_pools.ContainsKey(name))
        {
            Debug.LogWarning($"[PoolManager] 池 '{name}' 已存在");
            return;
        }

        var pool = new ObjectPool(prefab, preloadCount, transform);
        _pools[name] = pool;
        Debug.Log($"[PoolManager] 创建池 '{name}'，预加载 {preloadCount} 个");
    }

    // 获取对象
    public GameObject Get(string poolName, Vector3 position, Quaternion rotation)
    {
        if (!_pools.TryGetValue(poolName, out var pool))
        {
            Debug.LogError($"[PoolManager] 池 '{poolName}' 不存在！");
            return null;
        }

        return pool.Get(position, rotation);
    }

    public GameObject Get(string poolName)
    {
        return Get(poolName, Vector3.zero, Quaternion.identity);
    }

    // 归还对象
    public void Return(GameObject obj)
    {
        // 通过组件获取池
        var pooledObj = obj.GetComponent<PooledObject>();
        if (pooledObj != null && pooledObj.Pool != null)
        {
            pooledObj.Pool.Return(obj);
        }
        else
        {
            // 没有 Pool 组件，直接销毁
            Destroy(obj);
        }
    }

    // 清空指定池
    public void ClearPool(string poolName)
    {
        if (_pools.TryGetValue(poolName, out var pool))
        {
            // 清空操作（需要修改 ObjectPool 实现）
            Debug.Log($"[PoolManager] 清空池 '{poolName}'");
        }
    }

    // 获取池信息
    public void PrintPoolStatus()
    {
        Debug.Log("========== 对象池状态 ==========");
        foreach (var kvp in _pools)
        {
            Debug.Log($"池 '{kvp.Key}': 可用 {kvp.Value.AvailableCount}, 总创建 {kvp.Value.TotalCreated}");
        }
        Debug.Log("================================");
    }
}

// ============================================
// 子弹系统示例 - 使用对象池
// ============================================
public class BulletSystem : MonoBehaviour
{
    // 子弹预制体
    public GameObject bulletPrefab;

    // 发射点
    public Transform firePoint;

    // 发射间隔
    public float fireRate = 0.1f;
    private float _nextFireTime = 0f;

    private void Start()
    {
        // 创建子弹池
        if (bulletPrefab != null)
        {
            PoolManager.Instance.CreatePool("Bullet", bulletPrefab, 20);
        }
    }

    private void Update()
    {
        // 按住鼠标左键连续发射
        if (Input.GetButton("Fire1") && Time.time >= _nextFireTime)
        {
            Fire();
            _nextFireTime = Time.time + fireRate;
        }
    }

    // 发射子弹
    private void Fire()
    {
        if (firePoint == null) return;

        // 从池中获取子弹
        var bullet = PoolManager.Instance.Get("Bullet", firePoint.position, firePoint.rotation);

        if (bullet != null)
        {
            // 设置子弹方向
            // bullet.GetComponent<Rigidbody>().velocity = firePoint.forward * 50f;

            // 子弹生命周期管理
            StartCoroutine(DisableBulletAfterTime(bullet, 2f));
        }
    }

    // 延迟归还子弹
    private System.Collections.IEnumerator DisableBulletAfterTime(GameObject bullet, float delay)
    {
        yield return new WaitForSeconds(delay);

        if (bullet != null && bullet.activeSelf)
        {
            PoolManager.Instance.Return(bullet);
        }
    }
}

// ============================================
// 敌人生成系统 - 使用对象池
// ============================================
public class EnemySpawner : MonoBehaviour
{
    public GameObject[] enemyPrefabs;  // 敌人预制体数组
    public int initialPoolSize = 10;

    private void Start()
    {
        // 为每种敌人创建池
        for (int i = 0; i < enemyPrefabs.Length; i++)
        {
            string poolName = $"Enemy_{i}";
            PoolManager.Instance.CreatePool(poolName, enemyPrefabs[i], initialPoolSize);
        }
    }

    // 生成敌人
    public void SpawnEnemy(int enemyType, Vector3 position)
    {
        string poolName = $"Enemy_{enemyType}";
        var enemy = PoolManager.Instance.Get(poolName, position, Quaternion.identity);

        if (enemy != null)
        {
            // 初始化敌人
            enemy.GetComponent<Enemy>()?.Initialize();
        }
    }
}
```

### 游戏开发应用

| 场景 | 推荐对象 | 预加载数量建议 |
|------|---------|--------------|
| 子弹/箭矢 | 50-100 | 高频发射，大量预加载 |
| 敌人 | 20-50 | 根据波次大小调整 |
| 粒子特效 | 30-50 | 特效短暂但频繁 |
| UI 元素 | 10-20 | 弹窗、提示等 |
| 掉落物品 | 20-30 | 金币、道具 |

---

## 1.8 外观模式（Facade）

### 生活类比

想象**酒店前台**——你不需要记住酒店有多少层、有几个餐厅、健身房几点开门、游泳池怎么去。你只需要到前台，他们帮你处理一切。外观模式就是游戏的"前台"，提供一个统一接口，隐藏所有子系统的复杂性。

### 核心思想

**为子系统中的一组接口提供一个统一的接口。外观定义了一个高层接口，使子系统更易于使用。**

### Unity C# 代码示例

```csharp
using UnityEngine;
using System;

// ============================================
// 子系统：音频系统
// ============================================
public class AudioSystem
{
    public void PlayBGM(string clipName)
    {
        Debug.Log($"[音频] 播放背景音乐: {clipName}");
    }

    public void StopBGM()
    {
        Debug.Log("[音频] 停止背景音乐");
    }

    public void PlaySFX(string clipName)
    {
        Debug.Log($"[音频] 播放音效: {clipName}");
    }

    public void SetMasterVolume(float volume)
    {
        Debug.Log($"[音频] 主音量: {volume}");
    }
}

// ============================================
// 子系统：战斗系统
// ============================================
public class CombatSystem
{
    public void StartCombat()
    {
        Debug.Log("[战斗] 开始战斗");
    }

    public void EndCombat()
    {
        Debug.Log("[战斗] 结束战斗");
    }

    public void PlayerTakeDamage(int damage)
    {
        Debug.Log($"[战斗] 玩家受到 {damage} 点伤害");
    }

    public void EnemyTakeDamage(int damage)
    {
        Debug.Log($"[战斗] 敌人受到 {damage} 点伤害");
    }

    public void TriggerSlowMotion(float duration)
    {
        Debug.Log($"[战斗] 触发慢动作，持续 {duration} 秒");
    }
}

// ============================================
// 子系统：UI 系统
// ============================================
public class UISystem
{
    public void ShowHUD()
    {
        Debug.Log("[UI] 显示 HUD");
    }

    public void HideHUD()
    {
        Debug.Log("[UI] 隐藏 HUD");
    }

    public void UpdateHealthBar(int current, int max)
    {
        Debug.Log($"[UI] 更新血条: {current}/{max}");
    }

    public void UpdateScore(int score)
    {
        Debug.Log($"[UI] 更新分数: {score}");
    }

    public void ShowGameOver()
    {
        Debug.Log("[UI] 显示游戏结束画面");
    }

    public void ShowVictory()
    {
        Debug.Log("[UI] 显示胜利画面");
    }
}

// ============================================
// 子系统：存档系统
// ============================================
public class SaveSystem
{
    public void SaveGame()
    {
        Debug.Log("[存档] 保存游戏");
    }

    public void LoadGame()
    {
        Debug.Log("[存档] 加载游戏");
    }

    public void DeleteSave()
    {
        Debug.Log("[存档] 删除存档");
    }
}

// ============================================
// 子系统：粒子系统
// ============================================
public class VFXSystem
{
    public void PlayHitEffect(Vector3 position)
    {
        Debug.Log($"[特效] 在 {position} 播放命中特效");
    }

    public void PlayDeathEffect(Vector3 position)
    {
        Debug.Log($"[特效] 在 {position} 播放死亡特效");
    }

    public void PlayScreenShake(float intensity)
    {
        Debug.Log($"[特效] 屏幕震动，强度: {intensity}");
    }
}

// ============================================
// 外观类：游戏Facade
// 统一管理所有子系统，提供简洁接口
// ============================================
public class GameFacade
{
    // 单例
    private static GameFacade _instance;
    public static GameFacade Instance => _instance ??= new GameFacade();

    // 所有子系统
    private AudioSystem _audioSystem;
    private CombatSystem _combatSystem;
    private UISystem _uiSystem;
    private SaveSystem _saveSystem;
    private VFXSystem _vfxSystem;

    // 初始化所有子系统
    public void Initialize()
    {
        _audioSystem = new AudioSystem();
        _combatSystem = new CombatSystem();
        _uiSystem = new UISystem();
        _saveSystem = new SaveSystem();
        _vfxSystem = new VFXSystem();

        Debug.Log("[GameFacade] 初始化完成");
    }

    // ========== 游戏流程 ==========

    // 开始新游戏
    public void StartNewGame()
    {
        Debug.Log("========== 开始新游戏 ==========");
        _audioSystem.StopBGM();
        _audioSystem.SetMasterVolume(1f);
        _uiSystem.HideHUD();
        // 其他初始化...

        Debug.Log("[GameFacade] 新游戏已开始");
    }

    // 开始游戏（游戏中）
    public void StartGameplay()
    {
        Debug.Log("========== 进入游戏 ==========");
        _audioSystem.PlayBGM("gameplay_bgm");
        _uiSystem.ShowHUD();
        _combatSystem.StartCombat();
    }

    // 暂停游戏
    public void PauseGame()
    {
        Debug.Log("[GameFacade] 暂停游戏");
        _audioSystem.SetMasterVolume(0.3f);  // 背景音乐降低
        // Time.timeScale = 0f;  // 实际项目中会暂停
    }

    // 恢复游戏
    public void ResumeGame()
    {
        Debug.Log("[GameFacade] 恢复游戏");
        _audioSystem.SetMasterVolume(1f);
        // Time.timeScale = 1f;
    }

    // 游戏结束
    public void GameOver()
    {
        Debug.Log("========== 游戏结束 ==========");
        _audioSystem.StopBGM();
        _audioSystem.PlaySFX("game_over");
        _uiSystem.ShowGameOver();
        _combatSystem.EndCombat();
        _saveSystem.SaveGame();
    }

    // 胜利
    public void Victory()
    {
        Debug.Log("========== 胜利 ==========");
        _audioSystem.StopBGM();
        _audioSystem.PlayBGM("victory_bgm");
        _audioSystem.PlaySFX("victory_fanfare");
        _uiSystem.ShowVictory();
        _combatSystem.EndCombat();
        _saveSystem.SaveGame();
    }

    // ========== 战斗交互 ==========

    // 玩家攻击命中
    public void OnPlayerHit(Transform enemy, int damage)
    {
        _combatSystem.EnemyTakeDamage(damage);
        _vfxSystem.PlayHitEffect(enemy.position);
        _vfxSystem.PlayScreenShake(0.3f);
        _audioSystem.PlaySFX("hit_sfx");
    }

    // 玩家受伤
    public void OnPlayerDamaged(int damage)
    {
        _combatSystem.PlayerTakeDamage(damage);
        _vfxSystem.PlayScreenShake(0.5f);
        _audioSystem.PlaySFX("player_hurt");
    }

    // 敌人死亡
    public void OnEnemyKilled(Transform enemy)
    {
        _vfxSystem.PlayDeathEffect(enemy.position);
        _audioSystem.PlaySFX("enemy_death");
    }

    // ========== UI 更新 ==========

    // 更新生命值
    public void UpdateHealth(int current, int max)
    {
        _uiSystem.UpdateHealthBar(current, max);
    }

    // 更新分数
    public void UpdateScore(int score)
    {
        _uiSystem.UpdateScore(score);
    }

    // ========== 存档 ==========

    public void Save()
    {
        _saveSystem.SaveGame();
    }

    public void Load()
    {
        _saveSystem.LoadGame();
    }
}

// ============================================
// 使用外观模式的游戏管理器
// ============================================
public class GameManager : MonoBehaviour
{
    private void Awake()
    {
        // 初始化外观
        GameFacade.Instance.Initialize();
    }

    private void Start()
    {
        GameFacade.Instance.StartNewGame();
        GameFacade.Instance.StartGameplay();
    }

    private void Update()
    {
        // ESC 暂停
        if (Input.GetKeyDown(KeyCode.Escape))
        {
            GameFacade.Instance.PauseGame();
        }

        // 模拟玩家受伤
        if (Input.GetKeyDown(KeyCode.F))
        {
            GameFacade.Instance.OnPlayerDamaged(20);
            GameFacade.Instance.UpdateHealth(80, 100);
        }

        // 模拟击败敌人
        if (Input.GetKeyDown(KeyCode.K))
        {
            GameFacade.Instance.OnPlayerHit(transform, 50);
            GameFacade.Instance.UpdateScore(1000);
        }
    }
}

// ============================================
// 客户端代码示例 - 使用 Facade 前后的对比
// ============================================
public class ClientCodeExample : MonoBehaviour
{
    // 不使用 Facade：直接调用所有子系统
    private void OldWay()
    {
        // 需要记住所有子系统
        var audio = FindObjectOfType<AudioSystem>();
        var combat = FindObjectOfType<CombatSystem>();
        var ui = FindObjectOfType<UISystem>();
        var vfx = FindObjectOfType<VFXSystem>();

        audio.PlaySFX("hit");
        combat.EnemyTakeDamage(50);
        vfx.PlayHitEffect(transform.position);
        vfx.PlayScreenShake(0.3f);
        ui.UpdateHealthBar(80, 100);
        ui.UpdateScore(1000);
    }

    // 使用 Facade：一行代码搞定
    private void NewWay()
    {
        GameFacade.Instance.OnPlayerHit(transform, 50);
        GameFacade.Instance.UpdateHealth(80, 100);
        GameFacade.Instance.UpdateScore(1000);
    }
}
```

### 游戏开发应用

| 场景 | 适用性 | 原因 |
|------|--------|------|
| 游戏总管理器 | ✅ 必须使用 | 统一管理所有子系统 |
| 网络/匹配 | ✅ 推荐使用 | 网络逻辑复杂 |
| UI 管理器 | ✅ 推荐使用 | UI 子系统多 |
| 存档系统 | ⚠️ 可选 | 存档逻辑简单可不使用 |
| 小型游戏 | ⚠️ 可能过度设计 | 功能少，不需要Facade |

---

## 练习题

### ⭐ 第一题：单例模式（难度：⭐）
**题目**：创建一个 `SettingsManager` 单例类，存储游戏设置（音量、画质、语言），支持从 PlayerPrefs 加载和保存。

**要求**：
1. 使用泛型单例基类
2. 添加 `Load()` 和 `Save()` 方法
3. 提供 `GetVolume()`、`SetVolume()` 等属性

---

### ⭐⭐ 第二题：工厂模式（难度：⭐⭐）
**题目**：为道具系统实现工厂模式。

**要求**：
1. 创建 `Item` 基类，包含名称、图标、价格
2. 创建 `HealthPotion`、`ManaPotion`、`Shield` 具体道具类
3. 实现 `ItemFactory` 工厂，根据 ID 生成道具
4. 创建一个"开箱"功能，随机生成道具

---

### ⭐⭐ 第三题：观察者模式（难度：⭐⭐）
**题目**：实现一个成就系统，当玩家达成特定条件时解锁成就并通知 UI。

**要求**：
1. 定义成就相关的事件参数（`AchievementEventArgs`）
2. 事件类型：击杀10个敌人、完成第一关、收集100金币
3. 实现 `AchievementManager` 发布者
4. 实现 `AchievementUI` 订阅者，显示成就弹窗

---

### ⭐⭐⭐ 第四题：状态模式（难度：⭐⭐⭐）
**题目**：为敌人实现完整的状态机。

**要求**：
1. 状态：巡逻、追击、攻击、逃跑、死亡
2. 实现 `IEnemyState` 接口
3. 敌人根据距离玩家的距离自动切换状态
4. 在 Unity 中使用 Animator Controller 配合代码状态机

---

### ⭐⭐⭐⭐ 第五题：综合设计模式应用（难度：⭐⭐⭐⭐）
**题目**：设计一个武器升级系统，综合运用多种设计模式。

**要求**：
1. **工厂模式**：创建不同品质的武器（普通、稀有、传说）
2. **策略模式**：实现不同的攻击方式（近战、远程、AOE）
3. **观察者模式**：武器升级时通知 UI 显示动画
4. **命令模式**：武器切换操作可以撤销

---

## 答案提示

<details>
<summary>点击展开答案提示（请先自己思考）</summary>

**第一题提示**：
- 继承 `Singleton<SettingsManager>`
- 使用 `PlayerPrefs.GetFloat("Volume")` 等方法
- 记得处理 Key 不存在的情况

**第二题提示**：
- 简单工厂即可满足需求
- 随机生成可以用 `Random.Range()`
- 道具可以是 ScriptableObject 或普通类

**第三题提示**：
- 使用 EventManager 订阅/发布
- 成就条件在对应系统（战斗/关卡/道具）中检查
- AchievementUI 需要在 OnDestroy 中取消订阅

**第四题提示**：
- 距离判断：< 5 近距离、5-15 中距离、> 15 远距离
- 逃跑状态：血量低于 20%
- 使用 `animator.SetBool()` 配合 Animator Controller

**第五题提示**：
- Facade 可以统一管理所有系统
- 命令模式记录切换历史
- 策略模式可以运行时切换
</details>

---

## 总结

| 模式 | 核心目的 | Unity 中的典型应用 |
|------|---------|------------------|
| **单例** | 全局唯一访问点 | GameManager、AudioManager |
| **工厂** | 对象创建封装 | 武器/敌人/道具生成 |
| **观察者** | 事件通知解耦 | UI更新、成就系统 |
| **状态** | 行为随状态变化 | 角色动作、AI行为 |
| **策略** | 算法可替换 | 攻击策略、AI决策 |
| **命令** | 操作封装、可撤销 | 输入处理、回放系统 |
| **对象池** | 性能优化 | 子弹、特效、敌人 |
| **外观** | 简化复杂系统 | GameFacade |

**设计模式不是银弹**：根据实际需求选择合适的模式，避免过度设计。
