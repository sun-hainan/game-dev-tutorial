# Chapter 07：SLG 战斗动画与表现

> 🎯 目标：掌握 SLG 战斗中的视觉表现——伤害数字、技能动画、特效、BUFF 图标、战斗摄像机。

---

## 7.1 伤害数字

```csharp
// -------- 伤害数字弹出 --------
public class DamagePopupManager : MonoBehaviour
{
    public static DamagePopupManager instance;

    void Awake() { instance = this; }

    public void ShowDamage(
        Vector3 worldPos,
        int damage,
        bool isCrit,
        DamageType type)
    {
        GameObject popup = ObjectPoolManager.instance.Spawn("DamagePopup");
        popup.transform.position = worldPos + Vector3.up * 1.5f;

        DamagePopupUI ui = popup.GetComponent<DamagePopupUI>();

        ui.Setup(damage, isCrit, type);
    }
}

public class DamagePopupUI : MonoBehaviour
{
    public TextMeshProUGUI text;
    public float floatSpeed = 2f;
    public float lifetime = 1.2f;
    private float elapsed = 0f;
    private Vector3 basePos;

    public void Setup(int damage, bool isCrit, DamageType type) {
        string prefix = isCrit ? "💥" : "";
        text.text = $"{prefix}{damage}";

        if (isCrit) text.fontSize = 48;
        else text.fontSize = 32;

        // 颜色
        switch (type) {
            case DamageType.Heal:   text.color = Color.green; break;
            case DamageType.Poison: text.color = Color.magenta; break;
            case DamageType.Magic:  text.color = Color.cyan; break;
            default:                text.color = Color.white; break;
        }

        elapsed = 0f;
        basePos = transform.position;
    }

    void Update() {
        elapsed += Time.deltaTime;

        // 上浮
        transform.position = basePos + Vector3.up * floatSpeed * elapsed;

        // 淡出
        float alpha = 1f - (elapsed / lifetime);
        text.color = new Color(text.color.r, text.color.g, text.color.b, alpha);

        if (elapsed >= lifetime) {
            ObjectPoolManager.instance.Despawn("DamagePopup", gameObject);
        }
    }
}
```

---

## 7.2 BUFF 图标

```csharp
// -------- BUFF 图标列表 --------
public class BuffIconManager : MonoBehaviour
{
    public GameObject buffIconPrefab;
    public Transform buffPanel;  // UI 挂载点

    public void AddBuffIcon(Buff buff) {
        GameObject icon = Instantiate(buffIconPrefab, buffPanel);
        buffIcon.GetComponent<BuffIconUI>().Setup(buff.iconID, buff.currentDuration, buff.buffName);
    }

    public void UpdateBuffDuration(Buff buff) {
        var icon = FindBuffIcon(buff);
        if (icon != null) {
            icon.UpdateDuration(buff.currentDuration);
        }
    }

    public void RemoveBuffIcon(Buff buff) {
        var icon = FindBuffIcon(buff);
        if (icon != null) {
            Destroy(icon.gameObject);
        }
    }
}
```

---

## 7.3 战斗摄像机

```csharp
// -------- 战斗摄像机 --------
public class BattleCamera : MonoBehaviour
{
    public Camera cam;
    public Transform target;    // 跟随目标
    public float followSpeed = 5f;

    private Vector3 shakeOffset;
    private float shakeIntensity;
    private float shakeDuration;

    void Update() {
        // 跟随
        if (target != null) {
            Vector3 targetPos = target.position + Vector3.back * 10 + Vector3.up * 5;
            transform.position = Vector3.Lerp(transform.position, targetPos, followSpeed * Time.deltaTime);
            transform.LookAt(target);
        }
    }

    // -------- 震屏 --------
    public void Shake(float intensity, float duration) {
        shakeIntensity = intensity;
        shakeDuration = duration;
        StartCoroutine(ShakeCoroutine());
    }

    IEnumerator ShakeCoroutine() {
        while (shakeDuration > 0) {
            shakeOffset = Random.insideUnitSphere * shakeIntensity;
            cam.transform.position += shakeOffset;
            yield return null;
            shakeDuration -= Time.deltaTime;
        }
        cam.transform.position -= shakeOffset;
    }

    // -------- 聚焦攻击目标 --------
    public void FocusTarget(Transform enemy) {
        StartCoroutine(FocusCoroutine(enemy));
    }

    IEnumerator FocusCoroutine(Transform enemy) {
        Transform prev = target;
        target = enemy;
        yield return new WaitForSeconds(1.5f);
        target = prev;
    }
}
```

---

## 7.4 本章小结

```
✅ 已掌握：
├── 伤害数字弹出（飘字/暴击/颜色）
├── BUFF 图标管理
├── 战斗摄像机跟随
├── 震屏效果
└章 战斗日志（TextMeshPro）
```

---

_📚 参考资料：《Unity Particle System》《DOTween》_
