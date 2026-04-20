# Chapter 03：大量单位寻路 A* 优化

> 🎯 目标：掌握 SLG 中百人/千人同屏时 A* 寻路的性能优化——分块导航、路径缓存、JPS、群体行为。

---

## 3.1 分块导航网格

```csharp
// 把大地图划分为多个区域，只在当前区域和相邻区域寻路
public class HierarchicalPathfinding
{
    private GridManager grid;
    private int chunkSize = 16;  // 每块 16x16
    private Chunk[,] chunks;

    // -------- 分块 --------
    void BuildChunks()
    {
        int cx = grid.width / chunkSize;
        int cy = grid.height / chunkSize;
        chunks = new Chunk[cx, cy];

        for (int i = 0; i < cx; i++) {
            for (int j = 0; j < cy; j++) {
                chunks[i, j] = new Chunk(i, j, chunkSize);
                BuildChunkConnections(chunks[i, j]);
            }
        }
    }

    // -------- 两级寻路 --------
    public List<GridPosition> FindPath(GridPosition start, GridPosition end)
    {
        Chunk startChunk = GetChunk(start);
        Chunk endChunk = GetChunk(end);

        if (startChunk == endChunk)
        {
            // 同块内，直接 A*
            return AStar(start, end);
        }

        // -------- 跨块：先找块间路径，再细化 --------
        var macroPath = FindChunkPath(startChunk, endChunk);  // 宏观路径
        var fullPath = new List<GridPosition>();

        // 细化每段
        GridPosition current = start;
        foreach (var chunk in macroPath)
        {
            GridPosition entry = GetChunkEntry(current, chunk);
            var subPath = AStar(current, entry);
            fullPath.AddRange(subPath);
            current = GetChunkExit(entry, chunk);
        }
        fullPath.Add(end);

        return fullPath;
    }
}
```

---

## 3.2 路径缓存

```csharp
// -------- 路径缓存 --------
public class PathCache
{
    private Dictionary<(GridPosition, GridPosition), List<GridPosition>> cache
        = new Dictionary<(GridPosition, GridPosition), List<GridPosition>>();

    private int maxCacheSize = 10000;

    public List<GridPosition> GetOrCompute(
        GridPosition start, GridPosition end, Func<List<GridPosition>> compute)
    {
        var key = (start, end);

        if (cache.TryGetValue(key, out var cachedPath))
            return cachedPath;

        var path = compute();

        // 限制缓存大小（LRU 策略）
        if (cache.Count >= maxCacheSize)
        {
            // 移除最老的
            var firstKey = cache.Keys.First();
            cache.Remove(firstKey);
        }

        cache[key] = path;
        return path;
    }
}
```

---

## 3.3 Jump Point Search（JPS）

```csharp
// -------- JPS：加速 A* --------
public class JPS
{
    // JPS 核心：在对称路径上跳跃，跳过明显不需要探索的节点
    public List<GridPosition> FindPath(GridPosition start, GridPosition end)
    {
        var openSet = new PriorityQueue<JPSNode>();
        var visited = new HashSet<GridPosition>();

        openSet.Enqueue(new JPSNode(start, 0, Heuristic(start, end)));

        while (openSet.Count > 0)
        {
            JPSNode current = openSet.Dequeue();

            if (current.pos.Equals(end))
                return ReconstructPath(current);

            visited.Add(current.pos);

            // 沿主方向跳跃
            foreach (var dir in GetNaturalNeighbors(current.pos, current.dir))
            {
                Jump(current.pos, dir, end, visited, openSet);
            }
        }

        return null;
    }

    // 跳跃：跳过明显不需要探索的对称节点
    GridPosition Jump(GridPosition pos, Direction dir, GridPosition end)
    {
        GridPosition next = pos + dir;

        if (!IsWalkable(next) || visited.Contains(next))
            return null;

        // 遇到转向点才加入 openSet
        if (IsForcedNeighbor(next, dir) || next == end)
            return next;

        return Jump(next, dir, end);  // 继续跳
    }
}
```

---

## 3.4 群体寻路（Flocking）

```csharp
// -------- 群体行为：合力避障 --------
public class FlockingUnit
{
    public Vector3 position;
    public Vector3 velocity;

    // -------- 三条规则 --------
    Vector3 Cohesion(List<FlockingUnit> neighbors) {
        // 1. 聚集：向邻居中心移动
        Vector3 center = Vector3.zero;
        foreach (var n in neighbors)
            center += n.position;
        return (center / neighbors.Count) - position;
    }

    Vector3 Separation(List<FlockingUnit> neighbors) {
        // 2. 分离：避免和邻居太近
        Vector3 separation = Vector3.zero;
        foreach (var n in neighbors)
            separation -= (position - n.position).normalized;
        return separation;
    }

    Vector3 Alignment(List<FlockingUnit> neighbors) {
        // 3. 对齐：和邻居速度一致
        Vector3 avgVel = Vector3.zero;
        foreach (var n in neighbors)
            avgVel += n.velocity;
        return (avgVel / neighbors.Count) - velocity;
    }

    public void Update()
    {
        var neighbors = GetNearbyUnits();

        Vector3 force = Cohesion(neighbors) * 1.0f
                     + Separation(neighbors) * 1.5f
                     + Alignment(neighbors) * 1.0f;

        velocity += force * Time.deltaTime;
        position += velocity * Time.deltaTime;
    }
}
```

---

## 3.5 本章小结

```
✅ 已掌握：
├── 分块导航网格（Hierarchical A*）
├── 路径缓存（LRU）
├── Jump Point Search（JPS）加速
├── 群体行为（Flocking）
├── 势能场避障
└章 多单位同时寻路优化

🔜 下章预告：
第四章：回合制战斗系统 —— Buff/Debuff、伤害公式、战斗AI、战斗回放。
```

---

_📚 参考资料：《Hierarchical Path-Finding》《JPS论文》_
