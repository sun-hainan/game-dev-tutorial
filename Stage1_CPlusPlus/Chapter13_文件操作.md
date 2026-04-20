# Chapter 13：文件操作

> 🎯 目标：学会读写存档文件、配置文件，让游戏数据永久保存。本章涵盖文本流、二进制流、文件定位、文件系统操作。

---

## 13.1 文件流概述

### 生活中的类比 📁

> **文件 = 笔记本：**
> - 把游戏数据**写入**文件 = 写笔记
> - 把游戏数据**读取**出来 = 翻看笔记
> - **文本文件** = 用铅笔写的，谁都能看
> - **二进制文件** = 用密码写的，只有程序能懂

### 三个文件流类

| 类名 | 全称 | 作用 | 常用方式 |
|-----|------|------|---------|
| `ifstream` | input file stream | 读文件（输入）| `>>` 操作符 |
| `ofstream` | output file stream | 写文件（输出）| `<<` 操作符 |
| `fstream` | file stream | 读写文件 | `<<` 和 `>>` |

### 头文件

```cpp
#include <fstream>    // 文件流
#include <filesystem> // C++17 文件系统（C++14 以下用 <dirent.h>）
using namespace std;
```

---

## 13.2 文本文件读写

### 写入文本文件

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
    // -------- 创建 ofstream，打开文件（写入模式）--------
    ofstream outFile("save.txt");

    if (!outFile) {
        cout << "文件打开失败！" << endl;
        return 1;
    }

    // -------- 用 << 写入文本 --------
    outFile << "玩家名称：孙悟空" << endl;
    outFile << "等级：85" << endl;
    outFile << "HP：5000" << endl;
    outFile << "MP：1200" << endl;
    outFile << "攻击力：250" << endl;

    outFile.close();   // 关闭文件
    cout << "存档已保存！" << endl;

    return 0;
}
```

**运行后生成 `save.txt`，内容：**
```
玩家名称：孙悟空
等级：85
HP：5000
MP：1200
攻击力：250
```

### 读取文本文件

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main()
{
    // -------- 创建 ifstream，打开文件（读取模式）--------
    ifstream inFile("save.txt");

    if (!inFile) {
        cout << "文件不存在或打开失败！" << endl;
        return 1;
    }

    // -------- 按行读取 --------
    string line;
    cout << "=== 读取存档 ===" << endl;
    while (getline(inFile, line)) {
        cout << line << endl;
    }

    inFile.close();
    return 0;
}
```

### 流式读取（用 >> 操作符）

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
    ifstream inFile("save.txt");

    string name, label;
    int level, hp, mp, attack;

    // -------- 用 >> 逐个读取（空格/换行分隔）--------
    inFile >> label >> name;       // 玩家名称：孙悟空
    inFile >> label >> level;     // 等级：85
    inFile >> label >> hp;        // HP：5000
    inFile >> label >> mp;        // MP：1200
    inFile >> label >> attack;    // 攻击力：250

    cout << "名称：" << name << endl;
    cout << "等级：" << level << endl;
    cout << "HP：" << hp << " MP：" << mp << " ATK：" << attack << endl;

    inFile.close();
    return 0;
}
```

### 自动打开/关闭（推荐写法）

```cpp
#include <iostream>
#include <fstream>
using namespace std;

// -------- 自动管理文件开关 --------
void saveGame() {
    ofstream outFile("save.txt");
    if (!outFile) return;

    outFile << "孙悟空 85 5000 1200 250" << endl;
    // ofstream 在离开作用域时自动关闭
}

void loadGame() {
    ifstream inFile("save.txt");
    if (!inFile) return;

    string name;
    int level, hp, mp, attack;
    inFile >> name >> level >> hp >> mp >> attack;

    cout << name << " Lv." << level << endl;
    // ifstream 离开作用域自动关闭
}
```

---

## 13.3 二进制文件读写

### 什么时候用二进制？

> **文本文件** = 给人看
> **二进制文件** = 给程序看（更紧凑、更快，但不能直接用记事本打开）

### 写入二进制

```cpp
#include <iostream>
#include <fstream>
using namespace std;

// -------- 玩家存档结构（二进制）--------
struct PlayerSave {
    char name[32];    // 固定长度，避免变长字符串问题
    int level;
    int hp;
    int mp;
    int attack;
};

int main()
{
    PlayerSave save;
    strcpy(save.name, "孙悟空");
    save.level = 85;
    save.hp = 5000;
    save.mp = 1200;
    save.attack = 250;

    // -------- 以二进制写入模式打开 --------
    ofstream outFile("save_binary.dat", ios::binary);

    if (!outFile) {
        cout << "文件打开失败！" << endl;
        return 1;
    }

    // -------- write：写入二进制（参数：地址, 字节数）--------
    outFile.write(reinterpret_cast<char*>(&save), sizeof(PlayerSave));

    outFile.close();
    cout << "二进制存档已保存！" << endl;

    return 0;
}
```

### 读取二进制

```cpp
#include <iostream>
#include <fstream>
using namespace std;

struct PlayerSave {
    char name[32];
    int level;
    int hp;
    int mp;
    int attack;
};

int main()
{
    ifstream inFile("save_binary.dat", ios::binary);

    if (!inFile) {
        cout << "文件不存在！" << endl;
        return 1;
    }

    PlayerSave save;

    // -------- read：读取二进制（参数：地址, 字节数）--------
    inFile.read(reinterpret_cast<char*>(&save), sizeof(PlayerSave));

    cout << "=== 读取二进制存档 ===" << endl;
    cout << "名称：" << save.name << endl;
    cout << "等级：" << save.level << endl;
    cout << "HP：" << save.hp << " MP：" << save.mp << " ATK：" << save.attack << endl;

    inFile.close();
    return 0;
}
```

---

## 13.4 文件定位

### 概念

> **文件指针 = 书页的页码。** 可以跳到开头、中间、末尾。

### 四个定位函数

| 函数 | 作用 |
|-----|------|
| `seekg(offset, origin)` | 设置输入（读取）指针位置 |
| `seekp(offset, origin)` | 设置输出（写入）指针位置 |
| `tellg()` | 返回当前读取指针位置 |
| `tellp()` | 返回当前写入指针位置 |

### origin 可选值

| 值 | 含义 |
|---|------|
| `ios::beg` | 从文件开头 |
| `ios::cur` | 从当前位置 |
| `ios::end` | 从文件末尾 |

### 示例：跳到指定位置读取

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
    ofstream outFile("test.dat", ios::binary);

    // -------- 写入 5 个整数 --------
    for (int i = 0; i < 5; i++) {
        outFile.write(reinterpret_cast<char*>(&i), sizeof(int));
    }
    outFile.close();

    // -------- 重新打开读取 --------
    ifstream inFile("test.dat", ios::binary);

    // -------- 跳到第 3 个整数（索引2）--------
    inFile.seekg(2 * sizeof(int), ios::beg);

    int value;
    inFile.read(reinterpret_cast<char*>(&value), sizeof(int));
    cout << "第3个整数值：" << value << endl;   // 应该是 2

    // -------- 跳到文件末尾前一个整数 --------
    inFile.seekg(-static_cast<int>(sizeof(int)), ios::end);
    inFile.read(reinterpret_cast<char*>(&value), sizeof(int));
    cout << "最后一个整数值：" << value << endl;  // 应该是 4

    inFile.close();
    return 0;
}
```

---

## 13.5 文件状态检测

### 状态标志

| 标志 | 含义 |
|-----|------|
| `eof()` | 是否到达文件末尾 |
| `fail()` | 上次 IO 操作是否失败（格式错误、文件未打开）|
| `bad()` | 是否发生严重错误（如磁盘损坏）|
| `good()` | 是否一切正常 |

### 示例

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
    ifstream inFile("save.txt");

    if (!inFile) {
        cout << "文件打开失败！" << endl;
        return 1;
    }

    string line;
    while (getline(inFile, line)) {
        cout << line << endl;
    }

    // -------- 检查读取是否因 EOF 而结束 --------
    if (inFile.eof()) {
        cout << "--- 文件读取完毕 ---" << endl;
    } else if (inFile.fail()) {
        cout << "--- 读取出错 ---" << endl;
    }

    inFile.close();
    return 0;
}
```

---

## 13.6 C++17 文件系统

### 常用操作

```cpp
#include <iostream>
#include <filesystem>
#include <string>
using namespace std;
namespace fs = std::filesystem;

int main()
{
    string path = "saves";

    // -------- 创建目录 --------
    if (!fs::exists(path)) {
        fs::create_directory(path);
        cout << "目录创建成功" << endl;
    }

    // -------- 遍历目录 --------
    cout << "=== saves 目录内容 ===" << endl;
    for (const auto& entry : fs::directory_iterator(path)) {
        cout << entry.path().filename().string();
        if (entry.is_regular_file()) {
            cout << " [文件 " << entry.file_size() << " bytes]";
        } else {
            cout << " [目录]";
        }
        cout << endl;
    }

    // -------- 检查文件是否存在 --------
    if (fs::exists("save.txt")) {
        cout << "存档文件存在" << endl;
    }

    return 0;
}
```

---

## 13.7 游戏开发实战：存档系统

### 完整存档类

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
using namespace std;

// -------- 道具结构体 --------
struct Item {
    string name;
    int count;
    int value;
};

// -------- 玩家存档结构 --------
struct GameSave {
    string playerName;
    int level;
    int hp;
    int maxHP;
    int mp;
    int maxMP;
    int attack;
    int gold;
    vector<Item> items;   // 背包道具
};

// -------- 存档管理器 --------
class SaveManager {
private:
    string savePath;

public:
    SaveManager(const string& path = "savegame.dat") : savePath(path) {}

    // -------- 保存游戏 --------
    bool saveGame(const GameSave& save) {
        ofstream outFile(savePath, ios::binary);
        if (!outFile) return false;

        // -------- 写入基本信息 --------
        size_t nameLen = save.playerName.length();
        outFile.write(reinterpret_cast<char*>(&nameLen), sizeof(size_t));
        outFile.write(save.playerName.c_str(), nameLen);

        outFile.write(reinterpret_cast<char*>(&save.level), sizeof(int));
        outFile.write(reinterpret_cast<char*>(&save.hp), sizeof(int));
        outFile.write(reinterpret_cast<char*>(&save.maxHP), sizeof(int));
        outFile.write(reinterpret_cast<char*>(&save.mp), sizeof(int));
        outFile.write(reinterpret_cast<char*>(&save.maxMP), sizeof(int));
        outFile.write(reinterpret_cast<char*>(&save.attack), sizeof(int));
        outFile.write(reinterpret_cast<char*>(&save.gold), sizeof(int));

        // -------- 写入道具数量 --------
        size_t itemCount = save.items.size();
        outFile.write(reinterpret_cast<char*>(&itemCount), sizeof(size_t));

        // -------- 写入每个道具 --------
        for (const auto& item : save.items) {
            size_t inameLen = item.name.length();
            outFile.write(reinterpret_cast<char*>(&inameLen), sizeof(size_t));
            outFile.write(item.name.c_str(), inameLen);
            outFile.write(reinterpret_cast<char*>(&item.count), sizeof(int));
            outFile.write(reinterpret_cast<char*>(&item.value), sizeof(int));
        }

        outFile.close();
        cout << "✅ 游戏已保存到 " << savePath << endl;
        return true;
    }

    // -------- 读取存档 --------
    bool loadGame(GameSave& save) {
        ifstream inFile(savePath, ios::binary);
        if (!inFile) return false;

        // -------- 读取基本信息 --------
        size_t nameLen;
        inFile.read(reinterpret_cast<char*>(&nameLen), sizeof(size_t));

        string name(nameLen, '\0');
        inFile.read(&name[0], nameLen);
        save.playerName = name;

        inFile.read(reinterpret_cast<char*>(&save.level), sizeof(int));
        inFile.read(reinterpret_cast<char*>(&save.hp), sizeof(int));
        inFile.read(reinterpret_cast<char*>(&save.maxHP), sizeof(int));
        inFile.read(reinterpret_cast<char*>(&save.mp), sizeof(int));
        inFile.read(reinterpret_cast<char*>(&save.maxMP), sizeof(int));
        inFile.read(reinterpret_cast<char*>(&save.attack), sizeof(int));
        inFile.read(reinterpret_cast<char*>(&save.gold), sizeof(int));

        // -------- 读取道具 --------
        size_t itemCount;
        inFile.read(reinterpret_cast<char*>(&itemCount), sizeof(size_t));

        save.items.clear();
        for (size_t i = 0; i < itemCount; i++) {
            Item item;
            size_t inameLen;
            inFile.read(reinterpret_cast<char*>(&inameLen), sizeof(size_t));

            string iname(inameLen, '\0');
            inFile.read(&iname[0], inameLen);
            item.name = iname;

            inFile.read(reinterpret_cast<char*>(&item.count), sizeof(int));
            inFile.read(reinterpret_cast<char*>(&item.value), sizeof(int));

            save.items.push_back(item);
        }

        inFile.close();
        cout << "✅ 游戏已从 " << savePath << " 加载" << endl;
        return true;
    }

    // -------- 检查存档是否存在 --------
    bool saveExists() {
        ifstream f(savePath);
        return f.good();
    }
};

// -------- 主函数 --------
int main()
{
    SaveManager manager("saves/player1.dat");

    // -------- 首次运行：创建存档 --------
    if (!manager.saveExists()) {
        GameSave save;
        save.playerName = "孙悟空";
        save.level = 85;
        save.maxHP = save.hp = 5000;
        save.maxMP = save.mp = 1200;
        save.attack = 250;
        save.gold = 88888;

        save.items = {
            {"金创药", 10, 50},
            {"魔法药水", 5, 100},
            {"如意金箍棒", 1, 50000}
        };

        manager.saveGame(save);
    }

    // -------- 读取存档 --------
    GameSave loaded;
    if (manager.loadGame(loaded)) {
        cout << endl << "=== 存档数据 ===" << endl;
        cout << "玩家：" << loaded.playerName << " Lv." << loaded.level << endl;
        cout << "HP：" << loaded.hp << "/" << loaded.maxHP << endl;
        cout << "MP：" << loaded.mp << "/" << loaded.maxMP << endl;
        cout << "金币：" << loaded.gold << endl;

        cout << endl << "=== 背包 ===" << endl;
        for (const auto& item : loaded.items) {
            cout << "  " << item.name << " x" << item.count << " (价值：" << item.value << ")" << endl;
        }
    }

    return 0;
}
```

**运行结果：**
```
✅ 游戏已保存到 saves/player1.dat
✅ 游戏已从 saves/player1.dat 加载

=== 存档数据 ===
玩家：孙悟空 Lv.85
HP：5000/5000
MP：1200/1200
金币：88888

=== 背包 ===
  金创药 x10 (价值：50)
  魔法药水 x5 (价值：100)
  如意金箍棒 x1 (价值：50000)
```

---

## 13.8 INI 配置文件读写

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <map>
using namespace std;

// -------- 简化版 INI 读取 --------
class IniFile {
private:
    map<string, map<string, string>> data;

public:
    bool load(const string& filename) {
        ifstream inFile(filename);
        if (!inFile) return false;

        string currentSection = "";
        string line;

        while (getline(inFile, line)) {
            // 去除前后空格
            while (!line.empty() && (line[0] == ' ' || line[0] == '\t')) {
                line = line.substr(1);
            }
            while (!line.empty() && (line.back() == ' ' || line.back() == '\t')) {
                line.pop_back();
            }

            if (line.empty() || line[0] == ';' || line[0] == '#') continue;

            if (line[0] == '[') {
                // 节名
                currentSection = line.substr(1, line.find(']') - 1);
            } else {
                // 键值对
                size_t pos = line.find('=');
                if (pos != string::npos) {
                    string key = line.substr(0, pos);
                    string value = line.substr(pos + 1);
                    data[currentSection][key] = value;
                }
            }
        }

        inFile.close();
        return true;
    }

    string get(const string& section, const string& key, const string& def = "") {
        if (data.count(section) && data[section].count(key)) {
            return data[section][key];
        }
        return def;
    }
};

int main()
{
    // -------- 创建配置文件 --------
    ofstream out("config.ini");
    out << "; 游戏配置文件\n"
        << "[Game]\n"
        << "title=我的RPG游戏\n"
        << "version=1.0.0\n"
        << "\n"
        << "[Player]\n"
        << "name=孙悟空\n"
        << "starting_level=1\n"
        << "\n"
        << "[Audio]\n"
        << "bgm_volume=80\n"
        << "sfx_volume=100\n"
        << "mute=false\n";
    out.close();

    // -------- 读取配置 --------
    IniFile config;
    if (config.load("config.ini")) {
        cout << "游戏：" << config.get("Game", "title") << " v" << config.get("Game", "version") << endl;
        cout << "玩家：" << config.get("Player", "name") << endl;
        cout << "起始等级：" << config.get("Player", "starting_level") << endl;
        cout << "BGM 音量：" << config.get("Audio", "bgm_volume") << endl;
    }

    return 0;
}
```

---

## 13.9 常见错误与调试

| 错误现象 | 原因 | 解决方法 |
|---------|------|---------|
| 文件读取不到 | 路径错误 | 用绝对路径或检查工作目录 |
| 写入后文件为空 | 没 close 就退出了 | 确保 `close()` 或用 RAII |
| 二进制读出乱码 | 读写结构体大小不一致 | 确保 `sizeof(Struct)` 一致 |
| `ios::binary` 忘记加 | 文本模式下 Windows 会改换行符 | 二进制读写一定要加 `ios::binary` |
| 文件打开失败 | 文件被占用或权限不足 | 检查是否有其他程序占用 |

---

## 13.10 动手练习 🧪

### 练习 1：保存分数 ⭐
```cpp
// 用文本方式保存分数到 "score.txt"
// 格式：玩家名 分数 最高分
// 然后读取并显示
```

### 练习 2：二进制读写玩家结构体 ⭐
```cpp
// struct Player { char name[20]; int score; };
// 写入3个玩家，然后读取第2个玩家
```

### 练习 3：文件定位 ⭐⭐
```cpp
// 在一个文件里写入 10 个 int
// 用 seekg 跳到第 5 个，读取并修改值
```

### 练习 4：存档管理器类 ⭐⭐
```cpp
// 设计一个 SaveManager 类，支持：
// - save(GameSave&) 和 load(GameSave&)
// - 检查存档是否存在
// - 支持多个存档槽位（save1.dat, save2.dat...）
```

### 练习 5：综合：游戏配置 + 存档 ⭐⭐⭐
```cpp
// 用 INI 读取游戏配置（音量、难度）
// 用二进制保存玩家进度（名字、等级、道具）
// 实现"新建游戏"和"读取进度"功能
```

---

## 13.11 本章小结

```
✅ 已掌握：
├── ifstream / ofstream / fstream 文件流
├── 文本文件读写（<< 和 >>）
├── 二进制文件读写（read/write）
├── 文件定位（seekg/seekp/tellg/tellp）
├── 文件状态检测（eof/fail/bad/good）
├── C++17 filesystem 基本操作
├── 完整存档系统实战
├── INI 配置文件读写
└── 常见错误与调试

🔜 下章预告：
第十四章：数据结构与算法 —— 冒泡/快速/归并排序、二分查找、二叉树、游戏开发中的实际应用。
   学完这章，你对 C++ 的掌握已经可以应对 Unity/C# 开发中的所有基础了！
```

---

_📚 参考资料：《C++ Primer》《Effective C++》《算法导论》_
