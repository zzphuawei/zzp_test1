在 Graphviz 的 dot 语言中，实现**子图间垂直排列 + 子图内元素水平排列**需要结合 `rankdir`、`rank` 和不可见边（`style=invis`）。以下是完整解决方案：

### 核心思路：
1. **全局设置**：用 `rankdir=TB`（默认）确保整体垂直排列
2. **子图内水平排列**：用 `{rank=same; 节点列表}` 强制子图内节点同级
3. **子图间垂直排列**：用不可见边连接不同子图的锚点节点

### 示例代码：
```dot
digraph G {
    // 全局垂直排列（默认TB）
    rankdir=TB;
    node [shape=box];
    
    // ========== 子图1 ==========
    {
        rank=same; // 强制子图内水平排列
        A1 -> A2 -> A3 [style=invis]; // 水平不可见连接
    }
    
    // ========== 子图2 ==========
    {
        rank=same; // 强制子图内水平排列
        B1 -> B2 -> B3 [style=invis]; // 水平不可见连接
    }
    
    // ========== 子图间垂直连接 ==========
    edge [style=invis];
    A1 -> B1; // 垂直不可见连接（控制排列顺序）
    
    // 可选：添加子图边框
    subgraph cluster_1 {
        label="Subgraph 1";
        A1; A2; A3;
    }
    subgraph cluster_2 {
        label="Subgraph 2";
        B1; B2; B3;
    }
}
```

### 关键技巧：
1. **水平排列**：
   ```dot
   { rank=same; A1; A2; A3; } // 节点声明必须在同一代码块
   A1 -> A2 -> A3 [style=invis]; // 维持水平顺序
   ```
   
2. **垂直连接**：
   ```dot
   edge [style=invis];
   // 连接锚点节点（如子图第一个节点）
   A1 -> B1; 
   ```

3. **子图边框**（可选）：
   ```dot
   subgraph cluster_X {
       label="Label";
       节点; 列表;
   }
   ```

### 效果说明：
| **层级** | **排列方向** | **元素**        |
|----------|--------------|-----------------|
| 子图间   | 垂直         | Subgraph 1 → Subgraph 2 |
| 子图内   | 水平         | A1 → A2 → A3            |

> **注意**：`cluster_` 前缀是子图边框的必要命名约定，纯排列可不加。

### 其他排列场景：
**子图间水平 + 子图内垂直**（交换方向）：
```dot
rankdir=LR; // 全局水平
// 子图内用 {rank=same} 实现垂直排列
```


在 Graphviz 的 DOT 语言中，实现子图（subgraph）间的垂直排列可以通过以下两种核心方法实现：

### 方法 1：使用不可见边约束（推荐）
```dot
digraph G {
    // 全局设置
    rankdir=TB; // 默认就是 TB (Top to Bottom)，显式声明更清晰
    compound=true; // 允许簇间连接

    // 子图1
    subgraph cluster1 {
        label="Subgraph 1";
        a -> b;
        // 定义边界节点（用于连接）
        b [group=end1]; // 标记子图1的底部节点
    }

    // 子图2
    subgraph cluster2 {
        label="Subgraph 2";
        c -> d;
        // 定义边界节点
        c [group=start2]; // 标记子图2的顶部节点
    }

    // 强制垂直排列：用不可见边连接相邻子图
    b -> c [
        style=invis,  // 边不可见
        weight=100,   // 高权重确保强制排序
        ltail="cluster1", // 边从子图1尾部引出
        lhead="cluster2"  // 边指向子图2头部
    ];
}
```

### 方法 2：显式分层（适用于简单场景）
```dot
digraph G {
    rankdir=TB;

    // 子图1
    subgraph cluster1 {
        label="Subgraph 1";
        a -> b;
    }

    // 子图2
    subgraph cluster2 {
        label="Subgraph 2";
        c -> d;
    }

    // 强制分层：将子图1的节点与子图2的节点分配到不同层级
    { rank=same; a; b } // 子图1在同一层
    { rank=same; c; d } // 子图2在下一层
    // 添加隐式约束
    b -> c [style=invis, weight=100];
}
```

---

### 关键技巧说明：
1. **`rankdir=TB`**  
   确保全局布局方向为 **从上到下**（Top to Bottom）

2. **不可见边 + 高权重**  
   ```dot
   b -> c [style=invis, weight=100];
   ```
   - `style=invis`：隐藏连线（不显示）
   - `weight=100`：强制布局引擎优先处理此约束（确保垂直顺序）

3. **簇边界约束（更精准）**  
   ```dot
   b -> c [ltail="cluster1", lhead="cluster2"];
   ```
   - 使用 `compound=true` 后，可直接连接子图边界
   - 避免子图内部节点意外影响布局

4. **显式分层**  
   ```dot
   { rank=same; a; b }  // 水平排列子图内节点
   { rank=same; c; d }  // 下一个垂直层
   ```
   适用于子图结构简单的场景

---

### 效果：
两种方法均会产生垂直排列的子图结构：
```
+---------------+
|   Subgraph 1  |
|  a ───▶ b     |
+---------------+
       ↓ (不可见约束)
+---------------+
|   Subgraph 2  |
|  c ───▶ d     |
+---------------+
```

> **提示**：若子图内部节点较多，建议使用方法1（带`ltail/lhead`的簇边界约束），可避免子图内部节点干扰垂直排列顺序。