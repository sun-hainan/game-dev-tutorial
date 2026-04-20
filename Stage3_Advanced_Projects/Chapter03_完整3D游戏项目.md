# Chapter 03：完整 3D 游戏项目

> 🎯 目标：从零构建一个完整的 3D 俯视角动作游戏，涵盖所有开发阶段的知识。

---

## 3.1 游戏设计文档（GDD）

### 游戏类型

> **3D 俯视角动作游戏（Top-Down Action Roguelike）**
> 类似《Hades》简化版、《暗黑破坏神》俯视角版。

### 核心玩法

| 维度 | 内容 |
|-----|------|
| **视角** | 俯视角（Top-Down），45° 倾斜 |
| **操作** | WASD 移动 + 鼠标瞄准 + 左键攻击 |
| **关卡** | 随机房间连接（Roguelike）|
| **敌人** | 多种类型，每种有独特 AI |
| **成长** | 击杀敌人获得经验，升级学新技能 |
| **目标** | 清理所有房间到达 Boss 房间 |

### 游戏功能列表

```
✅ 角色移动（3D 俯视角）
✅ 武器系统（近战 + 远程）
✅ 敌人 AI（巡逻/追击/攻击）
✅ 房间关卡生成
✅ 生命值 + 魔法值
✅ 经验值 + 升级
✅ 道具掉落
✅ 伤害数字弹出
✅ 打击感（屏幕震动 + 粒子）
✅ UI（HUD/暂停/升级菜单）
✅ 音效 + 背景音乐
✅ 存档系统
```

---

## 3.2 项目结构

```
Assets/
├── Scenes/
│   ├── MainMenu.scene
│   ├── Game.scene
│   └── Victory.scene
│
├── Scripts/
│   ├── Core/
│   │   ├── GameManager.cs
│   │   ├── GameState.cs
│   │   ├── EventManager.cs
│   │   ├── SaveManager.cs
│   │   └── AudioManager.cs
│   │
│   ├── Player/
│   │   ├── PlayerController.cs
│   │   ├── PlayerStats.cs
│   │   ├── PlayerAttack.cs
│   │   ├── PlayerHealth.cs
│   │   └── PlayerAbilities.cs
│   │
│   ├── Enemy/
│   │   ├── BaseEnemy.cs
│   │   ├── EnemyMelee.cs
│   │   ├── EnemyRanged.cs
│   │   └── EnemyBoss.cs
│   │
│   ├── Weapons/
│   │   ├── WeaponBase.cs
│   │   ├── MeleeWeapon.cs
│   │   ├── RangedWeapon.cs
│   │   └── WeaponData.cs
│   │
│   ├── Level/
│   │   ├── RoomGenerator.cs
│   │   ├── Room.cs
│   │   ├── Door.cs
│   │   └── LevelManager.cs
│   │
│   ├── UI/
│   │   ├── UIManager.cs
│   │   ├── HUD.cs
│   │   ├── HealthBar.cs
│   │   ├── LevelUpMenu.cs
│   │   ├── PauseMenu.cs
│   │   └── DamagePopup.cs
│   │
│   └── Effects/
│       ├── ScreenShake.cs
│       ├── HitStop.cs
│       └── ParticleEffects.cs
│
├── Prefabs/
│   ├── Player.prefab
│   ├── Enemies/
│   ├── Weapons/
│   └── Effects/
│
├── Animations/
│   └── Player/
│
├── Materials/
│
├── Audio/
│
└── Scenes/
```

---

## 3.3 核心系统实现

### 3.3.1 玩家控制器（俯视角 3D）

```csharp
using UnityEngine;

[RequireComponent(typeof(CharacterController))]
public class PlayerController3D : MonoBehaviour
{
    [Header("移动设置")]
    public float moveSpeed = 5f;
    public float rotationSpeed = 10f;

    [Header("组件")]
    private CharacterController controller;
    private Camera mainCam;

    [Header("状态")]
    public bool canMove = true;

    private Vector3 moveDirection;
    private Vector3 lookDirection;

    void Awake()
    {
        controller = GetComponent<CharacterController>();
        mainCam = Camera.main;
    }

    void Update()
    {
        if (!canMove) return;

        // -------- 获取输入 --------
        float h = Input.GetAxisRaw("Horizontal");
        float v = Input.GetAxisRaw("Vertical");

        // -------- 移动方向（世界坐标）--------
        moveDirection = new Vector3(h, 0, v).normalized;

        // -------- 移动 --------
        Vector3 worldMove = mainCam.transform.TransformDirection(moveDirection);
        worldMove.y = 0;
        worldMove.Normalize();

        controller.Move(worldMove * moveSpeed * Time.deltaTime);

        // -------- 鼠标瞄准 --------
        Ray ray = mainCam.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out RaycastHit hit, 100f, LayerMask.GetMask("Ground")))
        {
            lookDirection = hit.point - transform.position;
            lookDirection.y = 0;
            lookDirection.Normalize();

            // -------- 平滑旋转朝向鼠标 --------
            if (lookDirection != Vector3.zero)
            {
                Quaternion targetRotation = Quaternion.LookRotation(lookDirection);
                transform.rotation = Quaternion.Slerp(
                    transform.rotation,
                    targetRotation,
                    rotationSpeed * Time.deltaTime
                );
            }
        }

        // -------- 动画 --------
        Animator animator = GetComponent<Animator>();
        if (animator != null)
        {
            animator.SetFloat("Speed", moveDirection.magnitude);
        }
    }

    // -------- 禁止移动（如剧情动画时）--------
    public void LockMovement()
    {
        canMove = false;
    }

    public void UnlockMovement()
    {
        canMove = true;
    }
}
```

### 3.3.2 武器系统

```csharp
// -------- 武器基类 --------
public abstract class WeaponBase : MonoBehaviour
{
    public string weaponName = "武器";
    public int damage = 10;
    public float attackRate = 1f;    // 每秒攻击次数
    public float range = 2f;

    protected float nextAttackTime = 0f;

    public abstract void Attack(Transform target);
    public abstract void OnEquip();
    public abstract void OnUnequip();
}

// -------- 近战武器 --------
public class MeleeWeapon : WeaponBase
{
    public float arcAngle = 90f;    // 攻击弧度
    public LayerMask enemyLayer;

    public override void Attack(Transform target)
    {
        if (Time.time < nextAttackTime) return;
        nextAttackTime = Time.time + 1f / attackRate;

        // -------- 检测弧形范围内的敌人 --------
        Vector3 origin = transform.position;
        Collider[] hits = Physics.OverlapSphere(origin, range, enemyLayer);

        foreach (var hit in hits)
        {
            Vector3 dirToEnemy = (hit.transform.position - origin).normalized;
            float angle = Vector3.Angle(transform.forward, dirToEnemy);

            if (angle <= arcAngle / 2f)
            {
                // 命中
                IDamageable dmg = hit.GetComponent<IDamageable>();
                dmg?.TakeDamage(damage);

                // 特效
                VFXManager.instance.SpawnHitVFX(hit.point);
                AudioManager.instance.PlayHitSound(hit.transform.tag);
            }
        }

        // 攻击动画
        Animator animator = GetComponentInParent<Animator>();
        animator?.SetTrigger("Attack");
    }

    public override void OnEquip() { }
    public override void OnUnequip() { }
}

// -------- 远程武器 --------
public class RangedWeapon : WeaponBase
{
    public GameObject bulletPrefab;
    public Transform firePoint;
    public float bulletSpeed = 20f;
    public int bulletsPerShot = 1;
    public float spread = 0f;   // 散布角度

    public override void Attack(Transform target)
    {
        if (Time.time < nextAttackTime) return;
        nextAttackTime = Time.time + 1f / attackRate;

        for (int i = 0; i < bulletsPerShot; i++)
        {
            Vector3 dir = (target.position - firePoint.position).normalized;
            dir = Quaternion.Euler(
                Random.Range(-spread, spread),
                Random.Range(-spread, spread),
                0
            ) * dir;

            GameObject bullet = ObjectPoolManager.instance.Spawn(
                "Bullet",
                firePoint.position,
                Quaternion.LookRotation(dir)
            );

            bullet.GetComponent<Bullet>().Initialize(damage, bulletSpeed, dir);
        }

        // 枪口火焰
        VFXManager.instance.SpawnMuzzleFlash(firePoint.position);
    }

    public override void OnEquip() { }
    public override void OnUnequip() { }
}

// -------- 子弹 --------
public class Bullet : MonoBehaviour, IPoolable
{
    private int damage;
    private float speed;
    private Vector3 direction;
    private float lifetime = 3f;

    public void Initialize(int dmg, float spd, Vector3 dir)
    {
        damage = dmg;
        speed = spd;
        direction = dir;
    }

    public void OnSpawn()
    {
        lifetime = 3f;
    }

    void Update()
    {
        transform.position += direction * speed * Time.deltaTime;

        lifetime -= Time.deltaTime;
        if (lifetime <= 0f)
        {
            ObjectPoolManager.instance.Despawn("Bullet", gameObject);
        }
    }

    void OnTriggerEnter(Collider col)
    {
        if (col.CompareTag("Enemy"))
        {
            IDamageable dmg = col.GetComponent<IDamageable>();
            dmg?.TakeDamage(damage);

            VFXManager.instance.SpawnHitVFX(transform.position);
            ObjectPoolManager.instance.Despawn("Bullet", gameObject);
        }
    }
}
```

### 3.3.3 敌人 AI

```csharp
// -------- 敌人接口 --------
public interface IDamageable
{
    void TakeDamage(int damage);
    int CurrentHP { get; }
    bool IsDead { get; }
}

// -------- 基类敌人 --------
public abstract class BaseEnemy : MonoBehaviour, IDamageable
{
    [Header("属性")]
    public int maxHP = 100;
    public int attack = 10;
    public float moveSpeed = 3f;
    public float attackRange = 2f;
    public float detectionRange = 10f;

    protected int currentHP;
    protected Transform player;
    protected bool isDead = false;
    protected Animator animator;

    public int CurrentHP => currentHP;
    public bool IsDead => isDead;

    protected virtual void Awake()
    {
        currentHP = maxHP;
        player = GameObject.FindGameObjectWithTag("Player").transform;
        animator = GetComponent<Animator>();
    }

    public virtual void TakeDamage(int damage)
    {
        if (isDead) return;

        currentHP -= damage;
        currentHP = Mathf.Max(0, currentHP);

        // 受伤反馈
        StartCoroutine(HitFeedback());

        // 伤害数字
        DamagePopup.Show(transform.position + Vector3.up, damage);

        if (currentHP <= 0)
        {
            Die();
        }
    }

    protected virtual void Die()
    {
        isDead = true;
        animator.SetTrigger("Die");

        // 掉落道具
        DropManager.Instance.Drop(transform.position);

        // 经验值
        PlayerStats.instance.AddXP(10);

        Destroy(gameObject, 1.5f);
    }

    protected virtual IEnumerator HitFeedback()
    {
        // 变红闪烁
        SpriteRenderer sr = GetComponent<SpriteRenderer>();
        if (sr != null)
        {
            sr.color = Color.red;
            yield return new WaitForSeconds(0.1f);
            sr.color = Color.white;
        }

        // 屏幕震动
        ScreenShake.instance.Shake(0.1f, 0.2f);
    }

    protected bool IsPlayerInRange(float range)
    {
        if (player == null) return false;
        return Vector3.Distance(transform.position, player.position) <= range;
    }
}

// -------- 近战敌人：巡逻 + 追击 --------
public class EnemyMelee : BaseEnemy
{
    private enum State { Patrol, Chase, Attack, Return }
    private State currentState = State.Patrol;

    public Transform[] patrolPoints;
    private int currentPatrolIndex = 0;
    private Vector3 origin;
    private float chaseTimer = 0f;

    protected override void Awake()
    {
        base.Awake();
        origin = transform.position;
    }

    protected override void Update()
    {
        base.Update();
        if (isDead) return;

        switch (currentState)
        {
            case State.Patrol:
                Patrol();
                if (IsPlayerInRange(detectionRange))
                    currentState = State.Chase;
                break;

            case State.Chase:
                Chase();
                if (!IsPlayerInRange(detectionRange))
                {
                    chaseTimer += Time.deltaTime;
                    if (chaseTimer > 2f)
                        currentState = State.Return;
                }
                else
                {
                    chaseTimer = 0f;
                }

                if (IsPlayerInRange(attackRange))
                    currentState = State.Attack;
                break;

            case State.Attack:
                AttackPlayer();
                if (!IsPlayerInRange(attackRange))
                    currentState = State.Chase;
                break;

            case State.Return:
                ReturnToOrigin();
                if (IsPlayerInRange(detectionRange))
                    currentState = State.Chase;
                break;
        }
    }

    void Patrol()
    {
        Transform target = patrolPoints[currentPatrolIndex];
        MoveTo(target.position);

        if (Vector3.Distance(transform.position, target.position) < 0.5f)
        {
            currentPatrolIndex = (currentPatrolIndex + 1) % patrolPoints.Length;
        }
    }

    void Chase()
    {
        MoveTo(player.position);
        animator.SetFloat("Speed", 1f);
    }

    void ReturnToOrigin()
    {
        if (Vector3.Distance(transform.position, origin) < 0.5f)
        {
            currentState = State.Patrol;
        }
        else
        {
            MoveTo(origin);
        }
        animator.SetFloat("Speed", 0.5f);
    }

    void AttackPlayer()
    {
        animator.SetTrigger("Attack");
    }

    void MoveTo(Vector3 target)
    {
        Vector3 dir = (target - transform.position).normalized;
        dir.y = 0;
        transform.position += dir * moveSpeed * Time.deltaTime;
        transform.forward = dir;
    }
}
```

### 3.3.4 关卡生成器

```csharp
// -------- 房间 --------
[System.Serializable]
public class Room
{
    public string id;
    public Vector2Int gridPos;    // 在网格中的位置
    public GameObject roomObject;   // 房间预制体实例
    public bool isCleared = false;
    public bool isBossRoom = false;
    public List<Door> doors = new List<Door>();
}

// -------- 关卡生成器 --------
public class RoomGenerator : MonoBehaviour
{
    public int roomCount = 10;
    public float roomSize = 20f;

    public GameObject[] roomPrefabs;     // 普通房间预制体
    public GameObject bossRoomPrefab;
    public GameObject startRoomPrefab;

    private List<Room> generatedRooms = new List<Room>();
    private Dictionary<Vector2Int, Room> roomGrid = new Dictionary<Vector2Int, Room>();

    public void Generate()
    {
        ClearLevel();

        // -------- 生成房间位置 --------
        List<Vector2Int> positions = GenerateRoomPositions();

        // -------- 实例化房间 --------
        for (int i = 0; i < positions.Count; i++)
        {
            Vector2Int pos = positions[i];
            bool isStart = i == 0;
            bool isBoss = i == positions.Count - 1;

            GameObject prefab = isStart ? startRoomPrefab :
                               isBoss ? bossRoomPrefab :
                               roomPrefabs[Random.Range(0, roomPrefabs.Length)];

            Vector3 worldPos = new Vector3(pos.x * roomSize, 0, pos.y * roomSize);
            GameObject roomObj = Instantiate(prefab, worldPos, Quaternion.identity);

            Room room = new Room
            {
                id = $"Room_{i}",
                gridPos = pos,
                roomObject = roomObj,
                isBossRoom = isBoss
            };

            generatedRooms.Add(room);
            roomGrid[pos] = room;
        }

        // -------- 连接相邻房间 --------
        ConnectRooms();
    }

    List<Vector2Int> GenerateRoomPositions()
    {
        List<Vector2Int> positions = new List<Vector2Int>();
        positions.Add(Vector2Int.zero);   // 起点在中心

        Vector2Int[] directions = {
            Vector2Int.up, Vector2Int.down,
            Vector2Int.left, Vector2Int.right
        };

        while (positions.Count < roomCount)
        {
            Vector2Int basePos = positions[Random.Range(0, positions.Count)];
            Vector2Int newPos = basePos + directions[Random.Range(0, 4)];

            if (!positions.Contains(newPos))
            {
                positions.Add(newPos);
            }
        }

        return positions;
    }

    void ConnectRooms()
    {
        foreach (var room in generatedRooms)
        {
            Vector2Int[] dirs = { Vector2Int.up, Vector2Int.down,
                                  Vector2Int.left, Vector2Int.right };

            foreach (var dir in dirs)
            {
                Vector2Int neighborPos = room.gridPos + dir;
                if (roomGrid.TryGetValue(neighborPos, out Room neighbor))
                {
                    // 在两个房间之间开门
                    OpenDoorBetween(room, neighbor);
                }
            }
        }
    }

    void OpenDoorBetween(Room roomA, Room roomB)
    {
        Vector3 doorPos = (roomA.roomObject.transform.position +
                          roomB.roomObject.transform.position) / 2f;

        // 实例化门（设为开启状态）
        GameObject door = ObjectPoolManager.instance.Spawn("Door", doorPos, Quaternion.identity);
        // Door 脚本处理碰撞逻辑
    }

    void ClearLevel()
    {
        foreach (var room in generatedRooms)
        {
            if (room.roomObject != null)
                Destroy(room.roomObject);
        }
        generatedRooms.Clear();
        roomGrid.Clear();
    }

    public Room GetStartRoom()
    {
        return generatedRooms.Count > 0 ? generatedRooms[0] : null;
    }
}
```

---

## 3.4 UI 系统

### 伤害数字弹出

```csharp
public class DamagePopup : MonoBehaviour
{
    public TextMesh text;
    public float floatSpeed = 2f;
    public float lifetime = 1f;

    private float timer = 0f;

    public static void Show(Vector3 position, int damage, bool isCrit = false)
    {
        GameObject obj = ObjectPoolManager.instance.Spawn("DamagePopup", position, Quaternion.identity);
        DamagePopup popup = obj.GetComponent<DamagePopup>();
        popup.Setup(damage, isCrit);
    }

    public void Setup(int damage, bool isCrit)
    {
        text.text = isCrit ? $"{damage}!" : damage.ToString();
        text.color = isCrit ? Color.yellow : Color.white;
        text.fontSize = isCrit ? 48 : 32;

        // 初始偏移
        Vector3 offset = new Vector3(Random.Range(-0.5f, 0.5f), 1f, 0);
        transform.position += offset;

        timer = 0f;
    }

    void Update()
    {
        timer += Time.deltaTime;

        // 上浮
        transform.position += Vector3.up * floatSpeed * Time.deltaTime;

        // 淡出
        float alpha = 1f - (timer / lifetime);
        text.color = new Color(text.color.r, text.color.g, text.color.b, alpha);

        if (timer >= lifetime)
        {
            ObjectPoolManager.instance.Despawn("DamagePopup", gameObject);
        }
    }
}
```

### HUD

```csharp
public class HUD : MonoBehaviour
{
    public Slider healthSlider;
    public Slider manaSlider;
    public Slider xpSlider;
    public Text levelText;
    public Text healthText;
    public Text scoreText;

    void Update()
    {
        PlayerStats stats = PlayerStats.instance;

        // HP
        healthSlider.maxValue = stats.maxHP;
        healthSlider.value = stats.currentHP;
        healthText.text = $"{stats.currentHP} / {stats.maxHP}";

        // MP
        manaSlider.maxValue = stats.maxMP;
        manaSlider.value = stats.currentMP;

        // XP
        xpSlider.maxValue = stats.xpToNextLevel;
        xpSlider.value = stats.currentXP;

        // 等级
        levelText.text = $"Lv.{stats.level}";

        // 分数
        scoreText.text = $"分数：{GameManager.instance.score}";
    }
}
```

---

## 3.5 打击感

### 屏幕震动

```csharp
public class ScreenShake : MonoBehaviour
{
    public static ScreenShake instance;

    private float shakeIntensity = 0f;
    private float shakeDuration = 0f;

    void Awake()
    {
        instance = this;
    }

    public void Shake(float intensity, float duration)
    {
        shakeIntensity = intensity;
        shakeDuration = duration;
    }

    void Update()
    {
        if (shakeDuration > 0)
        {
            Vector3 shakeOffset = Random.insideUnitSphere * shakeIntensity;
            Camera.main.transform.position += shakeOffset;

            shakeDuration -= Time.deltaTime;
            if (shakeDuration <= 0)
            {
                shakeDuration = 0;
                // 恢复相机位置（需要记录原始位置）
            }
        }
    }
}
```

### 命中停顿（Hit Stop）

```csharp
public class HitStop : MonoBehaviour
{
    public static HitStop instance;

    private float stopDuration = 0f;

    void Awake()
    {
        instance = this;
    }

    public void Stop(float duration)
    {
        stopDuration = duration;
    }

    void Update()
    {
        if (stopDuration > 0)
        {
            stopDuration -= Time.deltaTime;
            if (stopDuration <= 0)
            {
                Time.timeScale = 1f;
            }
            else
            {
                Time.timeScale = 0f;
            }
        }
    }
}

// 使用：命中敌人时调用
// HitStop.instance.Stop(0.05f);
```

---

## 3.6 关卡编辑器

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

[CustomEditor(typeof(LevelManager))]
public class LevelEditorWindow : EditorWindow
{
    private LevelManager levelManager;
    private Vector2 scrollPos;

    [MenuItem("Tools/关卡编辑器")]
    public static void Open()
    {
        GetWindow<LevelEditorWindow>("关卡编辑器");
    }

    void OnEnable()
    {
        levelManager = FindObjectOfType<LevelManager>();
    }

    void OnGUI()
    {
        if (levelManager == null)
        {
            EditorGUILayout.HelpBox("场景中没有 LevelManager", MessageType.Warning);
            return;
        }

        scrollPos = EditorGUILayout.BeginScrollView(scrollPos);

        EditorGUILayout.LabelField("关卡配置", EditorStyles.boldLabel);

        levelManager.roomCount = EditorGUILayout.IntSlider("房间数量",
            levelManager.roomCount, 5, 30);

        levelManager.roomSize = EditorGUILayout.FloatField("房间大小",
            levelManager.roomSize);

        if (GUILayout.Button("生成关卡"))
        {
            levelManager.Generate();
        }

        if (GUILayout.Button("保存关卡"))
        {
            SaveLevel();
        }

        if (GUILayout.Button("加载关卡"))
        {
            LoadLevel();
        }

        EditorGUILayout.Space();
        EditorGUILayout.LabelField("房间列表", EditorStyles.boldLabel);

        foreach (var room in levelManager.rooms)
        {
            EditorGUILayout.BeginHorizontal();
            EditorGUILayout.LabelField(room.id);
            EditorGUILayout.LabelField(room.isBossRoom ? "[Boss]" : "[普通]");
            if (GUILayout.Button("选中", GUILayout.Width(60)))
            {
                Selection.activeGameObject = room.gameObject;
                SceneView.lastActiveSceneView.FrameSelected();
            }
            EditorGUILayout.EndHorizontal();
        }

        EditorGUILayout.EndScrollView();
    }

    void SaveLevel()
    {
        string path = EditorUtility.SaveFilePanel("保存关卡", "Assets", "Level1", "json");
        if (!string.IsNullOrEmpty(path))
        {
            string json = JsonUtility.ToJson(levelManager.GetLevelData(), true);
            System.IO.File.WriteAllText(path, json);
            Debug.Log($"关卡已保存到 {path}");
        }
    }

    void LoadLevel()
    {
        string path = EditorUtility.OpenFilePanel("加载关卡", "Assets", "json");
        if (!string.IsNullOrEmpty(path))
        {
            string json = System.IO.File.ReadAllText(path);
            LevelData data = JsonUtility.FromJson<LevelData>(json);
            levelManager.LoadLevelData(data);
            Debug.Log($"关卡已从 {path} 加载");
        }
    }
}
#endif
```

---

## 3.7 构建与发布

### PC 构建

```csharp
// File → Build Settings
// Platform: PC, Mac & Linux Standalone
// Target Platform: Windows
// Architecture: x86_64
// 点击 Build
```

### WebGL 构建

```
1. 安装 WebGL Build Support 模块
2. Edit → Project Settings → Player → WebGL Settings
   - Compression Format: Gzip
   - Decompression Fallback: 勾选
3. Build Settings → WebGL → Build
4. 需要 Web 服务器运行（本地 file:// 不行）
```

### 命令行构建（自动化）

```bash
# Windows
"C:\Program Files\Unity\2022.3\Editor\Unity.exe" ^
  -quit ^
  -batchmode ^
  -projectPath "D:\MyUnityProject" ^
  -buildOutput "D:\Build\Game.exe" ^
  -buildTarget standalone ^
  -executeMethod BuildScript.BuildWindows
```

---

## 3.8 项目总结

```
✅ 已实现系统：
├── 3D 俯视角角色控制器
├── 近战 + 远程武器系统
├── 对象池管理
├── 敌人 AI（巡逻/追击/攻击）
├── 随机房间关卡生成
├── 伤害数字弹出
├── 屏幕震动 + 命中停顿
├── HUD（血条/魔法条/XP条/分数）
├── 自定义关卡编辑器
├── 存档系统
└── 打击感综合

💡 可扩展方向：
├── BOSS 战机制
├── 更多道具类型
├── 多人联机
├── 成就系统
├── Steam 集成
└── 手机适配
```

---

## 3.9 本章小结

```
🎉 Stage 3 进阶项目 —— 完成！

✅ 已掌握：
├── 设计模式（单例/工厂/观察者/状态/策略/命令/对象池/外观）
├── Unity 高级技术（Mirror网络/XLua热更新/Shader/Addressables）
├── 性能优化（Profiler/DrawCall/GC/代码优化/LOD）
└── 完整 3D 游戏项目（从 GDD 到构建发布）

🎉 全部三个阶段全部完成！
```

---

_📚 参考资料：《Game Programming Patterns》《Unity 官方文档》《HLSL 编程指南》_
