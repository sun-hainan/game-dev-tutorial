# Chapter 05：即时战术 RTS 系统

> 🎯 目标：掌握 RTS 多人单位控制——框选/编队/阵型/群体移动/AOE。

---

## 5.1 框选单位

```csharp
// -------- RTS 框选 --------
public class SelectionManager
{
    private List<Unit> selectedUnits = new List<Unit>();
    private Vector3 dragStart;

    void Update() {
        // -------- 按住左键开始框选 --------
        if (Input.GetMouseButtonDown(0) && !IsOverUI()) {
            dragStart = MouseWorldPosition();
        }

        // -------- 拖动 --------
        if (Input.GetMouseButton(0)) {
            DrawSelectionBox(dragStart, MouseWorldPosition());
        }

        // -------- 松开：选中单位 --------
        if (Input.GetMouseButtonUp(0)) {
            Rect box = GetSelectionBox(dragStart, MouseWorldPosition());

            selectedUnits.Clear();
            foreach (var unit in allUnits) {
                if (box.Contains(WorldToScreen(unit.transform.position))) {
                    if (Input.GetKey(KeyCode.LeftShift)) {
                        // Shift+框选：追加
                        if (!selectedUnits.Contains(unit))
                            selectedUnits.Add(unit);
                    } else {
                        selectedUnits.Add(unit);
                    }
                }
            }

            // 高亮选中单位
            HighlightUnits(selectedUnits);
        }
    }

    // -------- Ctrl+点击追加/取消 --------
    if (Input.GetKey(KeyCode.LeftControl) && Input.GetMouseButtonDown(0)) {
        Unit clicked = GetUnitAtMouse();
        if (clicked != null) {
            if (selectedUnits.Contains(clicked))
                selectedUnits.Remove(clicked);
            else
                selectedUnits.Add(clicked);
        }
    }
}
```

---

## 5.2 编队

```csharp
// -------- RTS 编队 --------
public class Unit
{
    public int unitID;
    public Vector3 screenPos;  // 在屏幕上的位置（用于阵型）
}

public class SelectionManager
{
    private Dictionary<int, List<Unit>> controlGroups
        = new Dictionary<int, List<Unit>>();

    // -------- 编队（Ctrl+1~9）--------
    void MakeControlGroup(int groupID) {
        if (selectedUnits.Count == 0) return;
        controlGroups[groupID] = new List<Unit>(selectedUnits);
        Debug.Log($"编队 {groupID}：{selectedUnits.Count} 个单位");
    }

    // -------- 选中编队（按 1~9）--------
    void SelectControlGroup(int groupID) {
        if (!controlGroups.ContainsKey(groupID)) return;

        selectedUnits.Clear();
        selectedUnits.AddRange(controlGroups[groupID]);
        HighlightUnits(selectedUnits);
    }
}
```

---

## 5.3 阵型系统

```csharp
// -------- 阵型 --------
public enum Formation {
    Line,      // 横队
    Box,       // 方阵
    Triangle,  // 三角
    VShape,    // V 字
}

public class FormationHelper
{
    public static List<Vector3> GetFormation(
        List<Unit> units,
        Vector3 targetCenter,
        Formation formation)
    {
        var positions = new List<Vector3>();
        int count = units.Count;

        switch (formation) {
            case Formation.Line:
                // 横队：水平排列
                float spacing = 1.5f;
                float startX = -spacing * (count - 1) / 2f;
                for (int i = 0; i < count; i++) {
                    Vector3 pos = targetCenter + new Vector3(startX + i * spacing, 0, 0);
                    positions.Add(pos);
                }
                break;

            case Formation.Box:
                // 方阵：尽量成正方形
                int perRow = Mathf.CeilToInt(Mathf.Sqrt(count));
                float bx = -(perRow - 1) / 2f;
                float bz = -(perRow - 1) / 2f;
                for (int i = 0; i < count; i++) {
                    int row = i / perRow;
                    int col = i % perRow;
                    positions.Add(targetCenter + new Vector3(bx + col * 1.5f, 0, bz + row * 1.5f));
                }
                break;

            case Formation.VShape:
                // V 字
                float vs = 1.5f;
                for (int i = 0; i < count; i++) {
                    float side = (i % 2 == 0) ? 1f : -1f;
                    int idx = i / 2;
                    float x = -idx * vs * side;
                    float z = -idx * vs;
                    positions.Add(targetCenter + new Vector3(x, 0, z));
                }
                break;
        }

        return positions;
    }
}
```

---

## 5.4 阵型移动

```csharp
// -------- 阵型移动 --------
public class FormationMove
{
    public void MoveFormation(
        List<Unit> units,
        Vector3 targetCenter,
        Formation formation)
    {
        var positions = FormationHelper.GetFormation(units, targetCenter, formation);

        // -------- 先 A* 寻路 --------
        List<List<GridPosition>> allPaths = new List<List<GridPosition>>();
        int maxPathLength = 0;

        foreach (var unit in units) {
            var path = AStar.FindPath(unit.gridPos, WorldToGrid(targetCenter));
            if (path != null) {
                allPaths.Add(path);
                maxPathLength = Mathf.Max(maxPathLength, path.Count);
            }
        }

        // -------- 对齐步数（让所有单位同时到达）--------
        for (int i = 0; i < units.Count; i++) {
            var path = allPaths[i];
            if (path.Count < maxPathLength) {
                // 在起点等待
                while (path.Count < maxPathLength)
                    path.Insert(0, path[0]);
            }
        }

        // -------- 启动协程 --------
        StartCoroutine(MoveAlongPaths(units, allPaths));
    }

    IEnumerator MoveAlongPaths(
        List<Unit> units,
        List<List<GridPosition>> allPaths)
    {
        int steps = allPaths[0].Count;

        for (int step = 0; step < steps; step++) {
            for (int i = 0; i < units.Count; i++) {
                var unit = units[i];
                var path = allPaths[i];
                unit.MoveTo(path[step]);
            }
            yield return new WaitForSeconds(0.1f);  // 每步间隔
        }
    }
}
```

---

## 5.5 本章小结

```
✅ 已掌握：
├── RTS 框选（鼠标拖动矩形）
├── Ctrl+点击追加/取消选择
├── 编队（Ctrl+1~9 / 按键选中）
├── 阵型系统（横队/方阵/V字）
├── 阵型移动（保持相对位置）
└章 AOE 攻击与碰撞检测
```

---

_📚 参考资料：《Command & Conquer 源码分析》《StarCraft II RTS》_
