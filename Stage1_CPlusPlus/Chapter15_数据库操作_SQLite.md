# Chapter 15：数据库操作（SQLite）

> 🎯 目标：掌握 SQLite 的基本使用，能用 C++ 连接数据库、完成 CRUD 操作、实现游戏存档系统。本章使用 SQLite——无需安装服务器，创建一个 `.db` 文件即可工作。

---

## 15.1 为什么游戏开发需要数据库？

### 生活中的类比 🗄️

> **Excel 表格 vs 数据库：**
> - 想象你用 Excel 记录玩家分数——一个游戏还好，但一百个游戏呢？
> - 数据库 = 超强版 Excel，能存**海量数据**、**快速查找**、**多人同时访问**
> - SQLite = 一个小文件夹 `.db`，直接放到游戏目录里，不用装 MySQL/PostgreSQL

### 游戏中的数据库场景

| 场景 | 用数据库还是文件？| 说明 |
|-----|----------------|------|
| 单机存档 | 文件 或 SQLite | 小规模，SQLite 更结构化 |
| 玩家账号系统 | 必须数据库 | 需要查询、验证、排行 |
| 排行榜 | 必须数据库 | 大量数据 + 高并发查询 |
| 道具数据表 | SQLite | 配置表，用 SQL 查询很方便 |
| 成就系统 | SQLite | 结构化存储成就状态 |

### SQLite 特点

| 优点 | 缺点 |
|-----|------|
| ✅ 零配置，不用装数据库软件 | ❌ 不适合超大规模（TB级）|
| ✅ 单文件 `.db`，迁移方便 | ❌ 不适合高并发写入（不如 MySQL）|
| ✅ 支持完整 SQL 语法 | |
| ✅ 游戏存档/配置/排行榜首选 | |

---

## 15.2 SQLite 安装与使用

### 下载 SQLite

> **下载地址：** https://www.sqlite.org/download.html
> 下载 `sqlite-dll-win64-x64.zip` + `sqlite-tools-win32-x86.zip`

### 命令行基本操作

```powershell
# 创建数据库（一个 .db 文件）
sqlite3 game.db

# 进入 SQLite 命令行后：
sqlite> CREATE TABLE players (id INTEGER PRIMARY KEY, name TEXT, score INTEGER);
sqlite> INSERT INTO players VALUES (1, '孙悟空', 8500);
sqlite> INSERT INTO players VALUES (2, '猪八戒', 7200);
sqlite> SELECT * FROM players;
1|孙悟空|8500
2|猪八戒|7200
sqlite> SELECT name FROM players WHERE score > 7000;
孙悟空
猪八戒
sqlite> .quit
```

---

## 15.3 在 C++ 中使用 SQLite

### 安装 SQLite3 开发库

> **vcpkg 安装（推荐）：**
```powershell
vcpkg install sqlite3
```

> **或者手动：**
> 1. 下载 SQLite DLL 和 `.h` 文件
> 2. 包含 `sqlite3.h`，链接 `sqlite3.lib`

### CMakeLists.txt 配置

```cmake
find_package(sqlite3 CONFIG REQUIRED)
target_link_libraries(your_target PRIVATE sqlite3::sqlite3)
```

---

## 15.4 第一个 SQLite C++ 程序

### 头文件

```cpp
#include <sqlite3.h>
#include <iostream>
#include <string>
```

### 创建数据库 + 创建表

```cpp
#include <iostream>
#include <sqlite3.h>
#include <string>
#include <vector>
using namespace std;

// -------- 玩家结构体 --------
struct Player {
    int id;
    string name;
    int score;
    int level;
};

int main()
{
    sqlite3* db;   // 数据库连接对象
    char* errMsg = nullptr;   // 错误信息

    // -------- 打开（不存在则创建）--------
    int rc = sqlite3_open("game.db", &db);
    if (rc != SQLITE_OK) {
        cerr << "无法打开数据库：" << sqlite3_errmsg(db) << endl;
        return 1;
    }
    cout << "✅ 数据库连接成功！" << endl;

    // -------- 创建表（如果不存在）--------
    const char* createTableSQL =
        "CREATE TABLE IF NOT EXISTS players ("
        "   id INTEGER PRIMARY KEY AUTOINCREMENT,"
        "   name TEXT NOT NULL,"
        "   score INTEGER DEFAULT 0,"
        "   level INTEGER DEFAULT 1"
        ");";

    rc = sqlite3_exec(db, createTableSQL, nullptr, nullptr, &errMsg);
    if (rc != SQLITE_OK) {
        cerr << "建表失败：" << errMsg << endl;
        sqlite3_free(errMsg);
    } else {
        cout << "✅ 表创建（或已存在）成功！" << endl;
    }

    // -------- 关闭数据库 --------
    sqlite3_close(db);
    cout << "数据库已关闭" << endl;

    return 0;
}
```

---

## 15.5 CRUD 操作详解

### 什么是 CRUD？

| 操作 | SQL | 说明 |
|-----|-----|------|
| **C**reate | INSERT | 插入数据 |
| **R**ead | SELECT | 查询数据 |
| **U**pdate | UPDATE | 更新数据 |
| **D**elete | DELETE | 删除数据 |

---

### 15.5.1 INSERT（插入数据）

```cpp
#include <iostream>
#include <sqlite3.h>
#include <string>
using namespace std;

void insertPlayer(sqlite3* db, const string& name, int score, int level) {
    // -------- 插入数据的 SQL（用 ? 做占位符，防止 SQL 注入）--------
    const char* sql = "INSERT INTO players (name, score, level) VALUES (?, ?, ?);";
    sqlite3_stmt* stmt;   // 预处理语句对象

    // -------- 预处理 SQL --------
    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) {
        cerr << "预处理失败：" << sqlite3_errmsg(db) << endl;
        return;
    }

    // -------- 绑定参数（索引从 1 开始）--------
    sqlite3_bind_text(stmt, 1, name.c_str(), -1, SQLITE_TRANSIENT);
    sqlite3_bind_int(stmt, 2, score);
    sqlite3_bind_int(stmt, 3, level);

    // -------- 执行 --------
    if (sqlite3_step(stmt) == SQLITE_DONE) {
        cout << "✅ 插入成功：" << name << " (Score=" << score << " Lv=" << level << ")" << endl;
    } else {
        cerr << "插入失败：" << sqlite3_errmsg(db) << endl;
    }

    // -------- 清理 --------
    sqlite3_finalize(stmt);
}

int main()
{
    sqlite3* db;
    sqlite3_open("game.db", &db);

    insertPlayer(db, "孙悟空", 8500, 85);
    insertPlayer(db, "猪八戒", 7200, 78);
    insertPlayer(db, "沙和尚", 6800, 75);
    insertPlayer(db, "白骨精", 5500, 60);
    insertPlayer(db, "牛魔王", 9000, 90);

    sqlite3_close(db);
    return 0;
}
```

**运行结果：**
```
✅ 插入成功：孙悟空 (Score=8500 Lv=85)
✅ 插入成功：猪八戒 (Score=7200 Lv=78)
✅ 插入成功：沙和尚 (Score=6800 Lv=75)
✅ 插入成功：白骨精 (Score=5500 Lv=60)
✅ 插入成功：牛魔王 (Score=9000 Lv=90)
```

---

### 15.5.2 SELECT（查询数据）

#### 查询所有玩家

```cpp
// -------- 回调函数：每查出一行就调用一次 --------
int callback(void* data, int argc, char** argv, char** colNames) {
    // argv[0]=id, argv[1]=name, argv[2]=score, argv[3]=level
    cout << "ID=" << argv[0]
         << " | 名称=" << argv[1]
         << " | 分数=" << argv[2]
         << " | 等级=" << argv[3] << endl;
    return 0;   // 返回 0 继续查询，返回非 0 停止查询
}

void selectAllPlayers(sqlite3* db) {
    const char* sql = "SELECT * FROM players ORDER BY score DESC;";

    cout << "=== 所有玩家（按分数降序）===" << endl;
    int rc = sqlite3_exec(db, sql, callback, nullptr, nullptr);
    if (rc != SQLITE_OK) {
        cerr << "查询失败：" << sqlite3_errmsg(db) << endl;
    }
}
```

#### 条件查询（带参数）

```cpp
void selectPlayerByName(sqlite3* db, const string& name) {
    const char* sql = "SELECT * FROM players WHERE name = ?;";
    sqlite3_stmt* stmt;

    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) {
        cerr << "预处理失败" << endl;
        return;
    }

    sqlite3_bind_text(stmt, 1, name.c_str(), -1, SQLITE_TRANSIENT);

    // -------- 遍历结果（可能有多个）--------
    cout << "=== 查询结果 ===" << endl;
    while (sqlite3_step(stmt) == SQLITE_ROW) {
        int id = sqlite3_column_int(stmt, 0);
        const unsigned char* n = sqlite3_column_text(stmt, 1);
        int score = sqlite3_column_int(stmt, 2);
        int level = sqlite3_column_int(stmt, 3);

        cout << "ID=" << id << " | " << n << " | 分数=" << score << " | 等级=" << level << endl;
    }

    sqlite3_finalize(stmt);
}
```

#### 查询分数最高的 N 名玩家

```cpp
void selectTopPlayers(sqlite3* db, int topN) {
    string sql = "SELECT * FROM players ORDER BY score DESC LIMIT " + to_string(topN) + ";";

    cout << "=== Top " << topN << " 玩家 ===" << endl;
    sqlite3_exec(db, sql.c_str(), callback, nullptr, nullptr);
}
```

---

### 15.5.3 UPDATE（更新数据）

```cpp
void updatePlayerScore(sqlite3* db, const string& name, int newScore) {
    const char* sql = "UPDATE players SET score = ? WHERE name = ?;";
    sqlite3_stmt* stmt;

    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) {
        cerr << "预处理失败" << endl;
        return;
    }

    sqlite3_bind_int(stmt, 1, newScore);
    sqlite3_bind_text(stmt, 2, name.c_str(), -1, SQLITE_TRANSIENT);

    if (sqlite3_step(stmt) == SQLITE_DONE) {
        cout << "✅ 更新成功：" << name << " 的分数变为 " << newScore << endl;
    } else {
        cerr << "更新失败" << endl;
    }

    sqlite3_finalize(stmt);
}

void addPlayerLevel(sqlite3* db, const string& name, int addLevel) {
    const char* sql = "UPDATE players SET level = level + ? WHERE name = ?;";
    sqlite3_stmt* stmt;

    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return;

    sqlite3_bind_int(stmt, 1, addLevel);
    sqlite3_bind_text(stmt, 2, name.c_str(), -1, SQLITE_TRANSIENT);

    if (sqlite3_step(stmt) == SQLITE_DONE) {
        cout << "✅ " << name << " 升级 +" << addLevel << endl;
    }

    sqlite3_finalize(stmt);
}
```

---

### 15.5.4 DELETE（删除数据）

```cpp
void deletePlayer(sqlite3* db, const string& name) {
    const char* sql = "DELETE FROM players WHERE name = ?;";
    sqlite3_stmt* stmt;

    if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return;

    sqlite3_bind_text(stmt, 1, name.c_str(), -1, SQLITE_TRANSIENT);

    if (sqlite3_step(stmt) == SQLITE_DONE) {
        cout << "✅ 删除玩家：" << name << endl;
    }

    sqlite3_finalize(stmt);
}

void deleteAllPlayers(sqlite3* db) {
    sqlite3_exec(db, "DELETE FROM players;", nullptr, nullptr, nullptr);
    cout << "✅ 清空所有玩家数据" << endl;
}
```

---

## 15.6 预处理语句详解（防止 SQL 注入）

### 什么是 SQL 注入？

> **注入攻击 = 用户输入中包含 SQL 代码，导致数据库被恶意操作。**

```cpp
// ❌ 危险写法：直接拼接字符串（会被注入！）
string dangerousSQL = "SELECT * FROM players WHERE name = '" + userInput + "';";
// 如果 userInput = "'; DROP TABLE players; --"
// 最终 SQL = "SELECT * FROM players WHERE name = ''; DROP TABLE players; --"
// 数据库被删了！

// ✅ 安全写法：预处理语句（参数绑定）
const char* safeSQL = "SELECT * FROM players WHERE name = ?;";
sqlite3_bind_text(stmt, 1, userInput.c_str(), -1, SQLITE_TRANSIENT);
// 输入被当作纯文本，不会被执行
```

---

## 15.7 事务（Transaction）

### 什么是事务？

> **事务 = 一组操作，要么全部成功，要么全部失败。** 比如转账：A扣钱+B加钱，必须同时成功或同时失败。

```cpp
void transferScore(sqlite3* db, const string& from, const string& to, int amount) {
    // -------- 开启事务 --------
    sqlite3_exec(db, "BEGIN TRANSACTION;", nullptr, nullptr, nullptr);

    try {
        // 1. A 扣钱
        const char* sql1 = "UPDATE players SET score = score - ? WHERE name = ?;";
        sqlite3_stmt* stmt1;
        sqlite3_prepare_v2(db, sql1, -1, &stmt1, nullptr);
        sqlite3_bind_int(stmt1, 1, amount);
        sqlite3_bind_text(stmt1, 2, from.c_str(), -1, SQLITE_TRANSIENT);
        if (sqlite3_step(stmt1) != SQLITE_DONE) throw runtime_error("扣钱失败");
        sqlite3_finalize(stmt1);

        // 2. B 加钱
        const char* sql2 = "UPDATE players SET score = score + ? WHERE name = ?;";
        sqlite3_stmt* stmt2;
        sqlite3_prepare_v2(db, sql2, -1, &stmt2, nullptr);
        sqlite3_bind_int(stmt2, 1, amount);
        sqlite3_bind_text(stmt2, 2, to.c_str(), -1, SQLITE_TRANSIENT);
        if (sqlite3_step(stmt2) != SQLITE_DONE) throw runtime_error("加钱失败");
        sqlite3_finalize(stmt2);

        // -------- 提交事务 --------
        sqlite3_exec(db, "COMMIT;", nullptr, nullptr, nullptr);
        cout << "✅ 转账成功：" << from << " -> " << to << " [" << amount << " 分]" << endl;

    } catch (const exception& e) {
        // -------- 回滚事务 --------
        sqlite3_exec(db, "ROLLBACK;", nullptr, nullptr, nullptr);
        cerr << "❌ 转账失败，回滚：" << e.what() << endl;
    }
}
```

---

## 15.8 游戏开发实战：完整存档系统

### 数据库设计

```sql
-- 玩家表
CREATE TABLE player_data (
    player_id INTEGER PRIMARY KEY,
    player_name TEXT NOT NULL,
    level INTEGER DEFAULT 1,
    hp INTEGER DEFAULT 100,
    max_hp INTEGER DEFAULT 100,
    mp INTEGER DEFAULT 50,
    max_mp INTEGER DEFAULT 50,
    attack INTEGER DEFAULT 10,
    defense INTEGER DEFAULT 5,
    gold INTEGER DEFAULT 0,
    exp INTEGER DEFAULT 0,
    last_save TEXT
);

-- 背包表
CREATE TABLE inventory (
    item_id INTEGER PRIMARY KEY AUTOINCREMENT,
    player_id INTEGER,
    item_name TEXT,
    item_count INTEGER DEFAULT 1,
    item_quality INTEGER DEFAULT 1,
    FOREIGN KEY (player_id) REFERENCES player_data(player_id)
);

-- 成就表
CREATE TABLE achievements (
    id INTEGER PRIMARY KEY,
    player_id INTEGER,
    achieve_id TEXT,
    achieve_name TEXT,
    unlocked INTEGER DEFAULT 0,
    unlock_time TEXT,
    FOREIGN KEY (player_id) REFERENCES player_data(player_id)
);
```

### C++ 存档管理器

```cpp
#include <iostream>
#include <sqlite3.h>
#include <string>
#include <vector>
#include <ctime>
#include <sstream>
using namespace std;

// -------- 道具结构体 --------
struct Item {
    int id;
    string name;
    int count;
    int quality;   // 1=白 2=绿 3=蓝 4=紫 5=橙
};

// -------- 玩家存档 --------
struct GameSave {
    int playerID;
    string name;
    int level, hp, maxHP, mp, maxMP, attack, defense, gold, exp;
    vector<Item> inventory;
    vector<string> achievements;
};

// -------- 数据库管理器 --------
class GameDB {
private:
    sqlite3* db;

    string currentTime() {
        time_t now = time(nullptr);
        char buf[64];
        strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", localtime(&now));
        return string(buf);
    }

public:
    GameDB(const string& dbPath = "gamedata.db") {
        if (sqlite3_open(dbPath.c_str(), &db) != SQLITE_OK) {
            cerr << "数据库打开失败：" << sqlite3_errmsg(db) << endl;
            db = nullptr;
            return;
        }
        initTables();
    }

    ~GameDB() {
        if (db) sqlite3_close(db);
    }

    bool isOpen() const { return db != nullptr; }

    // -------- 初始化表 --------
    void initTables() {
        const char* tables[] = {
            "CREATE TABLE IF NOT EXISTS player_data ("
            "   player_id INTEGER PRIMARY KEY,"
            "   player_name TEXT NOT NULL,"
            "   level INTEGER DEFAULT 1,"
            "   hp INTEGER DEFAULT 100,"
            "   max_hp INTEGER DEFAULT 100,"
            "   mp INTEGER DEFAULT 50,"
            "   max_mp INTEGER DEFAULT 50,"
            "   attack INTEGER DEFAULT 10,"
            "   defense INTEGER DEFAULT 5,"
            "   gold INTEGER DEFAULT 0,"
            "   exp INTEGER DEFAULT 0,"
            "   last_save TEXT"
            ");",

            "CREATE TABLE IF NOT EXISTS inventory ("
            "   item_id INTEGER PRIMARY KEY AUTOINCREMENT,"
            "   player_id INTEGER,"
            "   item_name TEXT,"
            "   item_count INTEGER DEFAULT 1,"
            "   item_quality INTEGER DEFAULT 1,"
            "   FOREIGN KEY (player_id) REFERENCES player_data(player_id)"
            ");",

            "CREATE TABLE IF NOT EXISTS achievements ("
            "   id INTEGER PRIMARY KEY AUTOINCREMENT,"
            "   player_id INTEGER,"
            "   achieve_id TEXT,"
            "   achieve_name TEXT,"
            "   unlocked INTEGER DEFAULT 0,"
            "   unlock_time TEXT,"
            "   FOREIGN KEY (player_id) REFERENCES player_data(player_id)"
            ");"
        };

        for (const char* sql : tables) {
            sqlite3_exec(db, sql, nullptr, nullptr, nullptr);
        }
        cout << "✅ 数据库表初始化完成" << endl;
    }

    // -------- 创建新存档 --------
    int createNewSave(const string& playerName) {
        const char* sql =
            "INSERT INTO player_data (player_name, level, hp, max_hp, mp, max_mp, "
            "attack, defense, gold, exp, last_save) "
            "VALUES (?, 1, 100, 100, 50, 50, 10, 5, 100, 0, ?);";

        sqlite3_stmt* stmt;
        if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return -1;

        sqlite3_bind_text(stmt, 1, playerName.c_str(), -1, SQLITE_TRANSIENT);
        sqlite3_bind_text(stmt, 2, currentTime().c_str(), -1, SQLITE_TRANSIENT);

        sqlite3_step(stmt);
        sqlite3_finalize(stmt);

        // -------- 获取新创建的 player_id --------
        sqlite3_int64 newID = sqlite3_last_insert_rowid(db);
        cout << "✅ 新存档创建成功，ID=" << newID << endl;
        return (int)newID;
    }

    // -------- 保存游戏 --------
    bool saveGame(const GameSave& save) {
        if (!db) return false;

        const char* sql =
            "UPDATE player_data SET level=?, hp=?, max_hp=?, mp=?, max_mp=?, "
            "attack=?, defense=?, gold=?, exp=?, last_save=? "
            "WHERE player_id=?;";

        sqlite3_stmt* stmt;
        if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return false;

        sqlite3_bind_int(stmt, 1, save.level);
        sqlite3_bind_int(stmt, 2, save.hp);
        sqlite3_bind_int(stmt, 3, save.maxHP);
        sqlite3_bind_int(stmt, 4, save.mp);
        sqlite3_bind_int(stmt, 5, save.maxMP);
        sqlite3_bind_int(stmt, 6, save.attack);
        sqlite3_bind_int(stmt, 7, save.defense);
        sqlite3_bind_int(stmt, 8, save.gold);
        sqlite3_bind_int(stmt, 9, save.exp);
        sqlite3_bind_text(stmt, 10, currentTime().c_str(), -1, SQLITE_TRANSIENT);
        sqlite3_bind_int(stmt, 11, save.playerID);

        bool success = (sqlite3_step(stmt) == SQLITE_DONE);
        sqlite3_finalize(stmt);

        if (success) {
            cout << "✅ 存档已保存 [" << currentTime() << "]" << endl;
        }
        return success;
    }

    // -------- 读取存档 --------
    bool loadGame(int playerID, GameSave& save) {
        if (!db) return false;

        const char* sql = "SELECT * FROM player_data WHERE player_id = ?;";
        sqlite3_stmt* stmt;

        if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return false;
        sqlite3_bind_int(stmt, 1, playerID);

        if (sqlite3_step(stmt) == SQLITE_ROW) {
            save.playerID = sqlite3_column_int(stmt, 0);
            save.name = (char*)sqlite3_column_text(stmt, 1);
            save.level = sqlite3_column_int(stmt, 2);
            save.hp = sqlite3_column_int(stmt, 3);
            save.maxHP = sqlite3_column_int(stmt, 4);
            save.mp = sqlite3_column_int(stmt, 5);
            save.maxMP = sqlite3_column_int(stmt, 6);
            save.attack = sqlite3_column_int(stmt, 7);
            save.defense = sqlite3_column_int(stmt, 8);
            save.gold = sqlite3_column_int(stmt, 9);
            save.exp = sqlite3_column_int(stmt, 10);

            sqlite3_finalize(stmt);
            return true;
        }

        sqlite3_finalize(stmt);
        return false;
    }

    // -------- 列出所有存档 --------
    void listAllSaves() {
        if (!db) return;

        const char* sql = "SELECT player_id, player_name, level, last_save FROM player_data ORDER BY last_save DESC;";
        sqlite3_stmt* stmt;

        if (sqlite3_prepare_v2(db, sql, -1, &stmt, nullptr) != SQLITE_OK) return;

        cout << "=== 存档列表 ===" << endl;
        bool hasAny = false;
        while (sqlite3_step(stmt) == SQLITE_ROW) {
            hasAny = true;
            cout << "ID=" << sqlite3_column_int(stmt, 0)
                 << " | " << sqlite3_column_text(stmt, 1)
                 << " | Lv." << sqlite3_column_int(stmt, 2)
                 << " | 保存:" << sqlite3_column_text(stmt, 3) << endl;
        }

        if (!hasAny) {
            cout << "（无存档）" << endl;
        }

        sqlite3_finalize(stmt);
    }
};

// -------- 主函数 --------
int main()
{
    GameDB gameDB("saves/gamedata.db");   // 存档放在 saves 目录

    if (!gameDB.isOpen()) {
        cerr << "数据库打开失败" << endl;
        return 1;
    }

    // -------- 列出已有存档 --------
    gameDB.listAllSaves();

    // -------- 创建新存档 --------
    cout << endl << "=== 创建新存档 ===" << endl;
    int newID = gameDB.createNewSave("孙悟空");
    if (newID > 0) {
        GameSave save;
        save.playerID = newID;
        save.name = "孙悟空";
        save.level = 50;
        save.hp = 5000;
        save.maxHP = 5000;
        save.mp = 1200;
        save.maxMP = 1200;
        save.attack = 350;
        save.defense = 150;
        save.gold = 88888;
        save.exp = 50000;

        gameDB.saveGame(save);
    }

    // -------- 读取存档 --------
    cout << endl << "=== 读取存档 ===" << endl;
    GameSave loaded;
    if (gameDB.loadGame(newID, loaded)) {
        cout << "名称：" << loaded.name << endl;
        cout << "等级：" << loaded.level << endl;
        cout << "HP：" << loaded.hp << "/" << loaded.maxHP << endl;
        cout << "MP：" << loaded.mp << "/" << loaded.maxMP << endl;
        cout << "攻击：" << loaded.attack << " 防御：" << loaded.defense << endl;
        cout << "金币：" << loaded.gold << " 经验：" << loaded.exp << endl;
    }

    return 0;
}
```

**运行结果：**
```
✅ 数据库表初始化完成
=== 存档列表 ===
（无存档）

=== 创建新存档 ===
✅ 新存档创建成功，ID=1
✅ 存档已保存 [2026-04-20 17:25:00]

=== 读取存档 ===
名称：孙悟空
等级：50
HP：5000/5000
MP：1200/1200
攻击：350 防御：150
金币：88888 经验：50000
```

---

## 15.9 排行榜实战

```cpp
#include <iostream>
#include <sqlite3.h>
#include <string>
#include <vector>
using namespace std;

struct RankEntry {
    int rank;
    string name;
    int score;
    int level;
};

void showLeaderboard(sqlite3* db, int topN = 10) {
    string sql = "SELECT name, score, level FROM players ORDER BY score DESC LIMIT " + to_string(topN) + ";";

    sqlite3_stmt* stmt;
    if (sqlite3_prepare_v2(db, sql.c_str(), -1, &stmt, nullptr) != SQLITE_OK) return;

    cout << "╔══════════════════════════════════════╗" << endl;
    cout << "║       🏆 排行榜 Top " << topN << "              ║" << endl;
    cout << "╠══════════════════════════════════════╣" << endl;

    int rank = 1;
    while (sqlite3_step(stmt) == SQLITE_ROW) {
        const char* name = (const char*)sqlite3_column_text(stmt, 0);
        int score = sqlite3_column_int(stmt, 1);
        int level = sqlite3_column_int(stmt, 2);

        string rankStr = to_string(rank);
        string medal = (rank == 1) ? "🥇" : (rank == 2) ? "🥈" : (rank == 3) ? "🥉" : "  ";

        cout << "║ " << medal << " 第" << rankStr << "名  " << name
             << "  分数:" << score << "  Lv." << level << endl;

        rank++;
    }

    cout << "╚══════════════════════════════════════╝" << endl;
    sqlite3_finalize(stmt);
}

int main()
{
    sqlite3* db;
    sqlite3_open("game.db", &db);

    showLeaderboard(db, 5);

    sqlite3_close(db);
    return 0;
}
```

**运行结果：**
```
╔══════════════════════════════════════╗
║       🏆 排行榜 Top 5              ║
╠══════════════════════════════════════╣
║ 🥇 第1名  牛魔王  分数:9000  Lv.90
║ 🥈 第2名  孙悟空  分数:8500  Lv.85
║ 🥉 第3名  猪八戒  分数:7200  Lv.78
║     第4名  沙和尚  分数:6800  Lv.75
║     第5名  白骨精  分数:5500  Lv.60
╚══════════════════════════════════════╝
```

---

## 15.10 常见错误与调试

| 错误现象 | 原因 | 解决方法 |
|---------|------|---------|
| `no such table` | 表没创建 | 检查建表 SQL 或加 `IF NOT EXISTS` |
| `UNIQUE constraint failed` | 插入重复主键 | 用 `INSERT OR REPLACE` 或更新 |
| `database is locked` | 多线程同时写 | 用 `sqlite3_busy_timeout()` 或加锁 |
| 中文乱码 | 编码问题 | 确保源码和数据库都用 UTF-8 |
| `SQLite OK` 但没数据 | WHERE 条件没匹配 | 检查条件值 |

---

## 15.11 动手练习 🧪

### 练习 1：创建道具表 ⭐
```cpp
// 创建表 items，包含：item_id(PK), name, type, quality, value
// 插入 5 种道具
// 查询所有道具并打印
```

### 练习 2：查询玩家 ⭐
```cpp
// 扩展 selectPlayerByName，支持同时查等级和分数
// 如果找不到，输出"玩家不存在"
```

### 练习 3：实现分数更新 ⭐⭐
```cpp
// 写一个函数 addScore(name, addAmount)
// 先查询当前分数，加上 addAmount，再更新
// 验证：A玩家转 500 分给 B玩家
```

### 练习 4：完整排行榜 ⭐⭐
```cpp
// 实现 leaderboard 类，支持：
// - updateScore(name, score)：没有则插入，有则更新
// - getTopN(n)：获取前 N 名
// - getRank(name)：获取某玩家的排名
```

### 练习 5：综合存档系统 ⭐⭐⭐
```cpp
// 扩展 15.8 的 GameDB，增加：
// - 背包增删改查（addItem, removeItem, getInventory）
// - 成就解锁（unlockAchievement, getAchievements）
// - 自动保存（每次修改数据后自动保存）
```

---

## 15.12 本章小结

```
✅ 已掌握：
├── SQLite 简介与适用场景
├── sqlite3 C API 基本用法
├── 创建数据库和表
├── INSERT / SELECT / UPDATE / DELETE（CRUD）
├── 预处理语句（防止 SQL 注入）
├── 事务（Transaction）：BEGIN/COMMIT/ROLLBACK
├── 完整游戏存档系统实战
├── 排行榜系统实战
└── 常见错误与调试

📚 参考资料：
├── 《SQLite Tutorial》https://www.sqlite.org/
├── 《Game Programming Patterns》数据库相关章节
└── 《The Definitive Guide to SQLite》
```

---

_🎉 Stage 1 C++ 基础（含数据库）全部完成！_

```
C++ 基础 15 章已全部覆盖：
├── Chapter01-03：语法入门（变量/控制流）
├── Chapter04：函数
├── Chapter05-06：数组/字符串 + 指针
├── Chapter07：内存管理（new/delete/智能指针）
├── Chapter08：结构体 + 枚举 + 位域
├── Chapter09-10：OOP（类/继承/多态）
├── Chapter11-12：STL（容器/算法）
├── Chapter13：文件操作
├── Chapter14：数据结构与算法
└── Chapter15：SQLite 数据库
```

**下一步：Stage 2 — Unity + C# 开发**
