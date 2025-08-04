要实现通过变换使得图像中四个点组成的图形垂直（即成为矩形），可以使用OpenCV的透视变换（Perspective Transformation）。以下是详细步骤和代码示例：

### 核心思路：
1. **排序四个点**：将输入点按固定顺序（左上、右上、右下、左下）排序。
2. **计算目标矩形**：根据原始点集计算矩形的宽度和高度。
3. **生成变换矩阵**：使用 `cv2.getPerspectiveTransform`。
4. **应用变换**：使用 `cv2.warpPerspective` 进行图像变换。

### 代码实现：
```python
import cv2
import numpy as np

def order_points(pts):
    """按固定顺序排序四个点：左上、右上、右下、左下"""
    rect = np.zeros((4, 2), dtype="float32")
    s = pts.sum(axis=1)
    rect[0] = pts[np.argmin(s)]  # 左上：总和最小
    rect[2] = pts[np.argmax(s)]  # 右下：总和最大
    
    diff = np.diff(pts, axis=1)
    rect[1] = pts[np.argmin(diff)]  # 右上：x-y最小
    rect[3] = pts[np.argmax(diff)]  # 左下：x-y最大
    return rect

def transform_to_rectangle(image, pts):
    # 将点排序
    rect = order_points(pts)
    (tl, tr, br, bl) = rect
    
    # 计算新矩形的最大宽度和高度
    width_top = np.linalg.norm(tr - tl)
    width_bottom = np.linalg.norm(br - bl)
    max_width = max(int(width_top), int(width_bottom))
    
    height_left = np.linalg.norm(bl - tl)
    height_right = np.linalg.norm(br - tr)
    max_height = max(int(height_left), int(height_right))
    
    # 定义目标点（确保顺序与rect一致）
    dst = np.array([
        [0, 0],
        [max_width - 1, 0],
        [max_width - 1, max_height - 1],
        [0, max_height - 1]
    ], dtype="float32")
    
    # 计算透视变换矩阵并应用
    M = cv2.getPerspectiveTransform(rect, dst)
    warped = cv2.warpPerspective(image, M, (max_width, max_height))
    return warped

# 示例使用
if __name__ == "__main__":
    # 读取图像
    image = cv2.imread("input.jpg")
    
    # 假设有四个点（格式：[[x1,y1], [x2,y2], [x3,y3], [x4,y4]]）
    input_points = np.array([[50, 50], [200, 100], [220, 300], [30, 250]], dtype="float32")
    
    # 执行变换
    result = transform_to_rectangle(image, input_points)
    
    # 保存结果
    cv2.imwrite("output.jpg", result)
```

### 关键说明：
1. **点排序函数 (`order_points`)**：
   - 通过坐标和（`x+y`）确定左上（最小）和右下（最大）。
   - 通过坐标差（`x-y`）确定右上（最小）和左下（最大）。

2. **尺寸计算**：
   - **宽度**：取上边（`tr-tl`）和下边（`br-bl`）的最大长度。
   - **高度**：取左边（`bl-tl`）和右边（`br-tr`）的最大长度。

3. **目标点定义**：
   - 按 `(左上, 右上, 右下, 左下)` 顺序创建目标矩形。

4. **变换矩阵**：
   - `cv2.getPerspectiveTransform` 根据源点和目标点计算3x3变换矩阵。
   - `cv2.warpPerspective` 应用变换并输出矫正后的图像。

### 效果：
- 输入：任意四边形（如平行四边形、梯形）的图像。
- 输出：矫正后的矩形图像，四边垂直。

### 注意事项：
- 确保输入点是 **顺时针或逆时针连续** 的四边形顶点。
- 若点顺序错误，需手动调整或使用凸包（`cv2.convexHull`）处理。
- 变换后图像尺寸自动适配矩形宽高，边缘可能被裁剪。

此方法广泛用于文档矫正、车牌识别等场景，能有效将任意四边形映射为矩形。