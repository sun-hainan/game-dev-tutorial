# Chapter 09：协程与异步编程

> 🎯 目标：深入掌握协程，理解异步加载资源、计时器、进度条等高级用法。

---

## 9.1 协程深入

### 协程的本质

> **协程不是线程。** 它在主线程上运行，但可以在任意点暂停和恢复。

```csharp
IEnumerator MyCoroutine()
{
    Debug.Log("步骤1");
    yield return null;              // 暂停，等待下一帧
    Debug.Log("步骤2");
    yield return new WaitForSeconds(2f);  // 暂停 2 秒
    Debug.Log("步骤3");
    yield return null;
    Debug.Log("完成");
}
```

### 常用 Wait 类型

```csharp
IEnumerator WaitTypes()
{
    yield return null;                      // 等待一帧
    yield return new WaitForFixedUpdate();  // 等待物理更新
    yield return new WaitForSeconds(2f);    // 等待 2 秒
    yield return new WaitForSecondsRealtime(2f);  // 真实时间（不受 Time.timeScale 影响）
    yield return new WaitUntil(() => hp <= 0);    // 等待条件为真
    yield return new WaitWhile(() => hp > 0);      // 等待条件为假

    // -------- WWW（网络请求）--------
    WWW www = new WWW("http://example.com/data");
    yield return www;
    Debug.Log("下载完成：" + www.text);

    // -------- AsyncOperation（异步加载）--------
    AsyncOperation op = SceneManager.LoadSceneAsync("Scene2");
    yield return op;
    Debug.Log("场景加载完成");
}
```

---

## 9.2 计时器系统

```csharp
public class TimerSystem : MonoBehaviour
{
    // -------- 简单计时器 --------
    public IEnumerator DelayCall(float delay, System.Action callback)
    {
        yield return new WaitForSeconds(delay);
        callback?.Invoke();
    }

    void Start()
    {
        StartCoroutine(DelayCall(3f, () => {
            Debug.Log("3 秒后执行！");
        }));
    }

    // -------- 重复执行 --------
    public IEnumerator RepeatEvery(float interval, System.Action action, int repeatCount = -1)
    {
        int count = 0;
        while (repeatCount < 0 || count < repeatCount)
        {
            yield return new WaitForSeconds(interval);
            action?.Invoke();
            count++;
        }
    }

    // -------- 倒计时 --------
    public IEnumerator Countdown(float seconds, System.Action<float> onTick, System.Action onComplete)
    {
        while (seconds > 0)
        {
            yield return new WaitForSeconds(1f);
            seconds--;
            onTick?.Invoke(seconds);
        }
        onComplete?.Invoke();
    }
}
```

---

## 9.3 异步场景加载

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneLoader : MonoBehaviour
{
    public CanvasGroup loadingCanvas;
    public UnityEngine.UI.Slider progressSlider;

    // -------- 显示加载界面 --------
    public void LoadScene(string sceneName)
    {
        StartCoroutine(LoadSceneAsync(sceneName));
    }

    IEnumerator LoadSceneAsync(string sceneName)
    {
        // 显示加载界面
        if (loadingCanvas != null)
        {
            loadingCanvas.blocksRaycasts = true;
            loadingCanvas.DOFade(1f, 0.3f);
        }

        AsyncOperation op = SceneManager.LoadSceneAsync(sceneName);
        op.allowSceneActivation = false;   // 加载完不自动跳转

        while (!op.isDone)
        {
            float progress = Mathf.Clamp01(op.progress / 0.9f);  // 0~1
            if (progressSlider != null)
                progressSlider.value = progress;

            Debug.Log($"加载进度：{progress * 100:F0}%");

            if (op.progress >= 0.9f)
            {
                // 加载完成，显示"点击继续"
                Debug.Log("加载完成，点击任意键继续");

                if (Input.anyKeyDown)
                {
                    op.allowSceneActivation = true;
                }
            }

            yield return null;
        }

        Debug.Log("场景已切换");
    }
}
```

---

## 9.4 异步资源加载

```csharp
using UnityEngine;

public class AsyncResourceLoad : MonoBehaviour
{
    // -------- 异步加载 Prefab --------
    public IEnumerator LoadPrefabAsync(string path, System.Action<GameObject> onLoaded)
    {
        ResourceRequest request = Resources.LoadAsync<GameObject>(path);
        yield return request;

        if (request.asset != null)
        {
            GameObject prefab = request.asset as GameObject;
            onLoaded?.Invoke(prefab);
        }
    }

    // -------- 异步加载 Sprite --------
    public IEnumerator LoadSpriteAsync(string path, System.Action<Sprite> onLoaded)
    {
        ResourceRequest request = Resources.LoadAsync<Sprite>(path);
        yield return request;

        Sprite sprite = request.asset as Sprite;
        onLoaded?.Invoke(sprite);
    }

    // -------- 实际使用 --------
    void Start()
    {
        StartCoroutine(LoadPrefabAsync("Prefabs/Enemy", (prefab) => {
            Instantiate(prefab, Vector3.zero, Quaternion.identity);
            Debug.Log("敌人已生成！");
        }));
    }
}
```

---

## 9.5 技能 CD 系统

```csharp
public class SkillSystem : MonoBehaviour
{
    // -------- 技能定义 --------
    [System.Serializable]
    public class Skill
    {
        public string skillName;
        public float cooldown;        // CD 时间（秒）
        public float currentCD;       // 当前 CD
        public UnityEngine.UI.Image cdImage;  // CD 遮罩图

        public bool IsReady => currentCD <= 0f;

        public void Tick(float delta)
        {
            if (currentCD > 0f)
            {
                currentCD -= delta;
                if (currentCD < 0f) currentCD = 0f;

                if (cdImage != null)
                {
                    cdImage.fillAmount = 1f - (currentCD / cooldown);
                }
            }
        }
    }

    public Skill[] skills;

    void Update()
    {
        foreach (var skill in skills)
        {
            skill.Tick(Time.deltaTime);
        }

        // 按 1/2/3 释放技能
        if (Input.GetKeyDown(KeyCode.Alpha1)) UseSkill(0);
        if (Input.GetKeyDown(KeyCode.Alpha2)) UseSkill(1);
        if (Input.GetKeyDown(KeyCode.Alpha3)) UseSkill(2);
    }

    public void UseSkill(int index)
    {
        if (index < 0 || index >= skills.Length) return;

        Skill skill = skills[index];
        if (!skill.IsReady)
        {
            Debug.Log($"{skill.skillName} 还在冷却，剩余 {skill.currentCD:F1}s");
            return;
        }

        // 执行技能逻辑
        Debug.Log($"释放 {skill.skillName}！");

        // 开始 CD
        skill.currentCD = skill.cooldown;
    }
}
```

---

## 9.6 移动端震动

```csharp
#if UNITY_IOS || UNITY_ANDROID
using UnityEngine;
#endif

public class HapticFeedback : MonoBehaviour
{
#if UNITY_IOS || UNITY_ANDROID
    private static bool isInitialized = false;
#endif

    public static void Light()
    {
#if UNITY_IOS
        iOSHapticFeedback.SetFeedback(HapticFeedbackType.Light);
#elif UNITY_ANDROID
        Handheld.Vibrate();
#endif
    }

    public static void Medium()
    {
#if UNITY_IOS
        iOSHapticFeedback.SetFeedback(HapticFeedbackType.Medium);
#elif UNITY_ANDROID
        Handheld.Vibrate();
#endif
    }

    public static void Heavy()
    {
#if UNITY_IOS
        iOSHapticFeedback.SetFeedback(HapticFeedbackType.Heavy);
#elif UNITY_ANDROID
        Handheld.Vibrate();
#endif
    }
}
```

---

## 9.7 章节小结

```
✅ 已掌握：
├── 协程 vs 线程的本质区别
├── WaitForSeconds / WaitForFixedUpdate / WaitUntil / WaitWhile
├── WWW 异步下载
├── AsyncOperation 场景异步加载
├── Resources.LoadAsync 异步加载
├── 技能 CD 系统
└── Haptic 震动反馈

🔜 下章预告：
第十章：预制体与对象池 —— 性能优化核心，减少 Instantiate/Destroy 开销。
```

---

_📚 参考资料：《Unity 协程文档》《C# async/await》_
