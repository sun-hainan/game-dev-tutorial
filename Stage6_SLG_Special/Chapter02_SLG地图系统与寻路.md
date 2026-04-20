# Chapter 02：SLG 地图系统与寻路

> 🎯 目标：掌握 SLG 格子地图系统（方形/六边形）、自定义地图编辑器、A* 寻路算法及其优化。

---

## 2.1 四边形 vs 六边形格子

### 四边形格子（Square Grid）

```csharp
// 四边形格子：Unity Grid 默认
// 方向：上下左右（4方向）或 + 对角（8方向）

public enum SquareDirection
{
    N  = new SquareDirection(0, 1),    // 北
    E  = new SquareDirection(1, 0),     // 东
    S  = new SquareDirection(0, -1),   // 南
    W  = new SquareDirection(-1, 0),   // 西
    NE = new SquareDirection(1, 1),
    NW = new SquareDirection(-1, 1),
    SE = new SquareDirection(1, -1),
    SW = new SquareDirection(-1, -1),
}

public struct SquareDirection {
    public int x, y;
    public SquareDirection(int x, int y) { this.x = x; this.y = y; }
}
```

### 六边形格子（Hex Grid）

```csharp
// 六边形格子：更适合 SLG，6 个方向，没有对角线问题
// 两种布局：Pointy-Top（尖朝上）/ Flat-Top（平朝上）

public enum HexDirection {
    E, NE, NW, W, SW, SE
}

// Hex 坐标（轴坐标系统）
public struct HexCoord {
    public int q, r;  // q + r + s = 0（s = -q-r）

    public static readonly HexCoord[] directions = {
        new HexCoord(1, 0),   // E
        new HexCoord(1, -1),  // NE
        new HexCoord(0, -1),  // NW
        new HexCoord(-1, 0),  // W
        new HexCoord(-1, 1),  // SW
        new HexCoord(0, 1)    // SE
    };

    public HexCoord Neighbor(int direction) {
        return this + directions[direction];
    }
}

// 六边形距离计算
public int Distance(HexCoord a, HexCoord b) {
    return (Mathf.Abs(a.q - b.q)
          + Mathf.Abs(a.q + a.r - b.q - b.r)
          + Mathf.Abs(a.r - b.r)) / 2;
}
```

---

## 2.2 地图编辑器

```csharp
#if UNITY_EDITOR
using UnityEngine;
using UnityEditor;

public class MapEditorWindow : EditorWindow
{
    private GridManager grid;
    private TerrainType selectedTerrain = TerrainType.Plain;

    [MenuItem("Tools/地图编辑器")]
    public static void Open() {
        GetWindow<MapEditorWindow>("SLG 地图编辑器");
    }

    void OnEnable() {
        grid = FindObjectOfType<GridManager>();
        Selection.selectionChanged = Repaint;
    }

    void OnGUI() {
        // -------- 工具栏 --------
        EditorGUILayout.BeginHorizontal();
        if (GUILayout.Button("平原地形")) selectedTerrain = TerrainType.Plain;
        if (GUILayout.Button("森林")) selectedTerrain = TerrainType.Forest;
        if (GUILayout.Button("山地")) selectedTerrain = TerrainType.Mountain;
        if (GUILayout.Button("水域")) selectedTerrain = TerrainType.Water;
        if (GUILayout.Button("道路")) selectedTerrain = TerrainType.Road;
        EditorGUILayout.EndHorizontal();

        // -------- 地形笔刷大小 --------
        int brushSize = EditorGUILayout.IntSlider("笔刷大小", 1, 5);

        // -------- 左键绘制，右键删除 --------
        if (Event.current.type == EventType.MouseDrag && Event.current.button == 0) {
            Ray ray = HandleUtility.GUIPointToWorldRay(Event.current.mousePosition);
            if (Physics.Raycast(ray, out RaycastHit hit)) {
                PaintTerrain(hit.point, brushSize);
            }
            Event.current.Use();
        }
    }

    void PaintTerrain(Vector3 worldPos, int size) {
        GridCell center = grid.GetCell(GridManager.Instance.WorldToGrid(worldPos));
        if (center == null) return;

        // 绘制笔刷范围内的格子
        for (int dx = -size+1; dx < size; dx++) {
            for (int dy = -size+1; dy < size; dy++) {
                GridCell cell = grid.GetCell(center.x + dx, center.y + dy);
                if (cell != null) {
                    Undo.RecordObject(grid, "Paint Terrain");
                    cell.terrainType = selectedTerrain;
                    cell.isWalkable = selectedTerrain != TerrainType.Mountain;
                    cell.moveCost = GetMoveCost(selectedTerrain);
                }
            }
        }
    }

    int GetMoveCost(TerrainType type) {
        switch (type) {
            case TerrainType.Road: return 1;
            case TerrainType.Plain: return 2;
            case TerrainType.Forest: return 3;
            case TerrainType.Mountain: return int.MaxValue;
            case TerrainType.Water: return int.MaxValue;
            default: return 2;
        }
    }
}
#endif
```

---

## 2.3 A* 寻路算法（完整实现）

```csharp
// -------- A* 寻路 --------
public class AStarPathfinding
{
    private GridManager grid;

    public AStarPathfinding(GridManager grid) {
        this.grid = grid;
    }

    // -------- 寻路 --------
    public List<GridPosition> FindPath(GridPosition start, GridPosition end) {
        // A* = f = g + h
        // g = 从起点到当前格的实际代价
        // h = 从当前格到终点的估计代价（启发函数）

        var openSet = new PriorityQueue<AStarNode>();
        var closedSet = new HashSet<GridPosition>();
        var cameFrom = new Dictionary<GridPosition, GridPosition>();
        var gScore = new Dictionary<GridPosition, int>();

        var startNode = new AStarNode(start, 0, Heuristic(start, end));
        openSet.Enqueue(startNode);

        gScore[start] = 0;

        while (openSet.Count > 0) {
            AStarNode current = openSet.Dequeue();

            if (current.pos.Equals(end)) {
                return ReconstructPath(cameFrom, current.pos);
            }

            closedSet.Add(current.pos);

            // 遍历邻居（4 方向或 8 方向）
            foreach (var neighbor in GetNeighbors(current.pos)) {
                if (closedSet.Contains(neighbor)) continue;

                GridCell cell = grid.GetCell(neighbor.x, neighbor.y);
                if (cell == null || !cell.isWalkable) continue;

                int tentativeG = gScore.GetValueOrDefault(current.pos, int.MaxValue)
                              + cell.moveCost;

                if (tentativeG < gScore.GetValueOrDefault(neighbor, int.MaxValue)) {
                    cameFrom[neighbor] = current.pos;
                    gScore[neighbor] = tentativeG;

                    int f = tentativeG + Heuristic(neighbor, end);
                    openSet.Enqueue(new AStarNode(neighbor, tentativeG, f));
                }
            }
        }

        return null; // 没找到路径
    }

    // -------- 曼哈顿距离（启发函数）--------
    int Heuristic(GridPosition a, GridPosition b) {
        return Mathf.Abs(a.x - b.x) + Mathf.Abs(a.y - b.y);
    }

    List<GridPosition> GetNeighbors(GridPosition pos) {
        return new List<GridPosition> {
            new GridPosition(pos.x + 1, pos.y),
            new GridPosition(pos.x - 1, pos.y),
            new GridPosition(pos.x, pos.y + 1),
            new GridPosition(pos.x, pos.y - 1),
        };
    }

    List<GridPosition> ReconstructPath(
        Dictionary<GridPosition, GridPosition> cameFrom,
        GridPosition current)
    {
        var path = new List<GridPosition> { current };
        while (cameFrom.ContainsKey(current)) {
            current = cameFrom[current];
            path.Insert(0, current);
        }
        return path;
    }
}

// -------- 优先队列（最小堆）--------
public class PriorityQueue<T> {
    private List<T> heap = new List<T>();
    private IComparer<T> comparer;

    public int Count => heap.Count;

    public PriorityQueue(IComparer<T> comparer = null) {
        this.comparer = comparer ?? Comparer<T>.Default;
    }

    public void Enqueue(T item) {
        heap.Add(item);
        SiftUp(heap.Count - 1);
    }

    public T Dequeue() {
        T top = heap[0];
        int last = heap.Count - 1;
        heap[0] = heap[last];
        heap.RemoveAt(last);
        SiftDown(0);
        return top;
    }

    void SiftUp(int i) {
        while (i > 0) {
            int parent = (i - 1) / 2;
            if (comparer.Compare(heap[i], heap[parent]) >= 0) break;
            Swap(i, parent);
            i = parent;
        }
    }

    void SiftDown(int i) {
        int n = heap.Count;
        while (true) {
            int smallest = i;
            int l = 2 * i + 1;
            int r = 2 * i + 2;
            if (l < n && comparer.Compare(heap[l], heap[smallest]) < 0) smallest = l;
            if (r < n && comparer.Compare(heap[r], heap[smallest]) < 0) smallest = r;
            if (smallest == i) break;
            Swap(i, smallest);
            i = smallest;
        }
    }

    void Swap(int i, int j) {
        T tmp = heap[i]; heap[i] = heap[j]; heap[j] = tmp;
    }
}
```

---

## 2.4 Dijkstra vs A*

| 维度 | Dijkstra | A* |
|-----|---------|-----|
| 原理 | 广度优先，按实际代价 | 贪心 + 优先搜索目标方向 |
| h(n) | 0（没有启发）| 估计到目标的距离 |
| 速度 | 慢（全探索）| 快（有方向）|
| 适用 | 找最近目标 | 找特定目标 |

---

## 2.5 战争迷雾

```csharp
// -------- 战争迷雾 --------
public class FogOfWar
{
    private bool[,] explored;  // 已探索
    private bool[,] visible; // 当前可见
    private int visionRange = 3;

    public void UpdateVision(GridPosition unitPos)
    {
        // 以单位位置为中心，标记可见区域
        for (int dx = -visionRange; dx <= visionRange; dx++) {
            for (int dy = -visionRange; dy <= visionRange; dy++) {
                int dist = Mathf.Abs(dx) + Mathf.Abs(dy);
                if (dist <= visionRange) {
                    visible[unitPos.x + dx, unitPos.y + dy] = true;
                    explored[unitPos.x + dx, unitPos.y + dy] = true;
                }
            }
        }
    }

    public bool IsVisible(GridPosition pos) {
        return visible[pos.x, pos.y];
    }

    public bool IsExplored(GridPosition pos) {
        return explored[pos.x, pos.y];
    }

    // 未探索 = 全黑，探索未可见 = 半透明，可见 = 正常
    public float GetFogAlpha(GridPosition pos) {
        if (IsVisible(pos)) return 0f;        // 完全透明
        if (IsExplored(pos)) return 0.5f;    // 半透明
        return 1f;                          // 完全黑
    }
}
```

---

## 2.6 本章小结

```
✅ 已掌握：
├── 四边形 vs 六边形格子对比
├── 自定义 Unity 地图编辑器（Inspector 窗口）
├── A* 寻路完整实现（启发函数/优先队列/路径重建）
├── Dijkstra vs A* 对比
├── 战争迷雾（可见/探索/隐藏）
└章 地形属性与移动消耗
```

---

_📚 参考资料：《A* Pathfinding》《Hex Grid Math》_
