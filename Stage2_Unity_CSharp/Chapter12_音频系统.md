# Chapter 12：音频系统

> 🎯 目标：掌握 Unity 音频组件——AudioSource、AudioListener、AudioMixer，实现背景音乐和音效。

---

## 12.1 音频基础组件

| 组件 | 作用 |
|-----|------|
| **AudioSource** | 播放音频（喇叭）|
| **AudioListener** | 接收声音（耳朵，通常挂在相机上）|
| **AudioClip** | 音频文件 |
| **AudioMixer** | 混音器（控制音量/混响）|

---

## 12.2 AudioSource 基本用法

```csharp
// -------- 添加 AudioSource 组件 --------
AudioSource audioSource = gameObject.AddComponent<AudioSource>();

// -------- 属性 --------
audioSource.clip = bgmClip;            // 播放的音频文件
audioSource.playOnAwake = false;       // 启动时是否播放
audioSource.loop = true;               // 是否循环
audioSource.volume = 0.8f;              // 音量 0~1
audioSource.pitch = 1f;                // 音调 1=正常，2=快/高，0.5=慢/低
audioSource.spatialBlend = 0f;         // 0=2D音效，1=3D音效

// -------- 方法 --------
audioSource.Play();           // 播放
audioSource.Pause();          // 暂停
audioSource.Stop();           // 停止
audioSource.PlayOneShot(clip, volume);  // 播放音效（可叠加）

// -------- 3D 音效设置 --------
audioSource.spatialBlend = 1f;
audioSource.minDistance = 1f;    // 开始衰减的距离
audioSource.maxDistance = 10f;  // 衰减结束的距离
audioSource.rolloffMode = AudioRolloffMode.Linear;  // 线性衰减
```

---

## 12.3 音效管理器

```csharp
using UnityEngine;

public class AudioManager : MonoBehaviour
{
    public static AudioManager instance;

    public AudioSource bgmSource;
    public AudioSource sfxSource;

    public AudioClip jumpSound;
    public AudioClip attackSound;
    public AudioClip coinSound;
    public AudioClip bgmClip;

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

    // -------- BGM --------
    public void PlayBGM(AudioClip clip)
    {
        if (bgmSource.clip == clip) return;
        bgmSource.clip = clip;
        bgmSource.loop = true;
        bgmSource.volume = 0.6f;
        bgmSource.Play();
    }

    // -------- 音效（一次性）--------
    public void PlaySFX(AudioClip clip, float volume = 1f)
    {
        sfxSource.PlayOneShot(clip, volume);
    }

    public void PlayJump() { PlaySFX(jumpSound); }
    public void PlayAttack() { PlaySFX(attackSound); }
    public void PlayCoin() { PlaySFX(coinSound); }

    // -------- 音量控制 --------
    public void SetBGMVolume(float volume)
    {
        bgmSource.volume = Mathf.Clamp01(volume);
    }

    public void SetSFXVolume(float volume)
    {
        sfxSource.volume = Mathf.Clamp01(volume);
    }

    // -------- 淡入淡出 --------
    public System.Collections.IEnumerator FadeBGM(float targetVolume, float duration)
    {
        float startVolume = bgmSource.volume;
        float elapsed = 0f;

        while (elapsed < duration)
        {
            elapsed += Time.deltaTime;
            bgmSource.volume = Mathf.Lerp(startVolume, targetVolume, elapsed / duration);
            yield return null;
        }

        bgmSource.volume = targetVolume;
    }
}
```

---

## 12.4 章节小结

```
✅ 已掌握：
├── AudioSource 和 AudioListener
├── 2D/3D 音效
├── Play / PlayOneShot / Stop / Pause
├── 音效管理器（单例）
└── 淡入淡出

🔜 下章预告：
第十三章：动画系统（Animator）—— 状态机、动画参数、角色动画控制。
```

---

_📚 参考资料：《Unity 官方文档 - Audio》_
