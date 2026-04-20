# Chapter 11：场景管理与存档系统

> 🎯 目标：掌握 SceneManager 场景管理、DontDestroyOnLoad、PlayerPrefs 和 JSON 存档。

---

## 11.1 SceneManager 场景管理

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;

public class SceneController : MonoBehaviour
{
    // -------- 同步加载（会卡）--------
    public void LoadLevel(string sceneName)
    {
        SceneManager.LoadScene(sceneName);
        SceneManager.LoadScene("Level1", LoadSceneMode.Single);   // 完全替换
    }

    // -------- 异步加载（不卡）--------
    public IEnumerator LoadLevelAsync(string sceneName)
    {
        AsyncOperation op = SceneManager.LoadSceneAsync(sceneName);
        while (!op.isDone)
        {
            Debug.Log($"加载进度：{op.progress * 100:F0}%");
            yield return null;
        }
        Debug.Log("加载完成！");
    }

    // -------- 叠加加载（Additive）--------
    public void AddScene(string sceneName)
    {
        SceneManager.LoadScene(sceneName, LoadSceneMode.Additive);
    }

    // -------- 获取当前场景 --------
    Scene GetCurrentScene()
    {
        return SceneManager.GetActiveScene();
    }

    // -------- 判断场景是否存在 --------
    bool SceneExists(string name)
    {
        for (int i = 0; i < SceneManager.sceneCountInBuildSettings; i++)
        {
            string scenePath = SceneUtility.GetScenePathByBuildIndex(i);
            string sceneName = System.IO.Path.GetFileNameWithoutExtension(scenePath);
            if (sceneName == name) return true;
        }
        return false;
    }
}
```

---

## 11.2 DontDestroyOnLoad

```csharp
public class GameManager : MonoBehaviour
{
    public static GameManager instance;

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);   // 切换场景不销毁
        }
        else
        {
            Destroy(gameObject);   // 已有则销毁
        }
    }
}
```

---

## 11.3 PlayerPrefs 存档

```csharp
// -------- 简单数据存档（适合设置）--------

// 保存
PlayerPrefs.SetString("PlayerName", "孙悟空");
PlayerPrefs.SetInt("Level", 85);
PlayerPrefs.SetFloat("Volume", 0.8f);
PlayerPrefs.Save();   // 立即写入磁盘

// 读取
string name = PlayerPrefs.GetString("PlayerName", "默认名称");  // 默认值
int level = PlayerPrefs.GetInt("Level", 1);
float volume = PlayerPrefs.GetFloat("Volume", 1f);

// 删除
PlayerPrefs.DeleteKey("PlayerName");
PlayerPrefs.DeleteAll();

// 存档路径
string path = Application.persistentDataPath;  // 手机: /data/data/包名/files
```

---

## 11.4 JSON 存档（推荐）

```csharp
using System;
using System.IO;
using UnityEngine;

[Serializable]
public class SaveData
{
    public string playerName;
    public int level;
    public int hp;
    public int maxHP;
    public int gold;
    public float[] position = new float[3];
}

public class JsonSaveSystem
{
    public static string SavePath => Application.persistentDataPath + "/save.json";

    // -------- 保存 --------
    public static void Save(SaveData data)
    {
        string json = JsonUtility.ToJson(data, true);   // true=格式化
        File.WriteAllText(SavePath, json);
        Debug.Log($"存档已保存：{SavePath}");
    }

    // -------- 读取 --------
    public static SaveData Load()
    {
        if (File.Exists(SavePath))
        {
            string json = File.ReadAllText(SavePath);
            SaveData data = JsonUtility.FromJson<SaveData>(json);
            Debug.Log("存档已加载");
            return data;
        }
        Debug.Log("存档不存在");
        return null;
    }

    // -------- 检查存档是否存在 --------
    public static bool SaveExists()
    {
        return File.Exists(SavePath);
    }
}

// -------- 使用 --------
public class SaveManager : MonoBehaviour
{
    public SaveData currentData;

    public void SaveGame()
    {
        currentData = new SaveData
        {
            playerName = "孙悟空",
            level = 85,
            hp = 5000,
            maxHP = 5000,
            gold = 88888,
            position = new float[] {
                transform.position.x,
                transform.position.y,
                transform.position.z
            }
        };

        JsonSaveSystem.Save(currentData);
    }

    public void LoadGame()
    {
        SaveData data = JsonSaveSystem.Load();
        if (data != null)
        {
            currentData = data;
            Debug.Log($"加载：{data.playerName} Lv.{data.level}");
        }
    }
}
```

---

## 11.5 章节小结

```
✅ 已掌握：
├── SceneManager 同步/异步加载
├── DontDestroyOnLoad 跨场景持久化
├── PlayerPrefs 简单存档
├── JSON 完整存档（JsonUtility）
└── Application.persistentDataPath

🔜 下章预告：
第十二章：音频系统 —— AudioSource、AudioListener、背景音乐、音效、混音。
```

---

_📚 参考资料：《Unity 官方文档 - SceneManager》《Unity Audio 文档》_
