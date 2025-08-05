# 基于矩形形状识别多个长条区域（不依赖颜色）

以下是不依赖颜色信息，纯粹基于矩形形状识别多个长条区域的解决方案。这种方法使用边缘检测和轮廓分析来寻找图像中的长矩形结构。

```python
import cv2
import numpy as np

def detect_long_rectangles(image_path, min_length=100, min_aspect_ratio=5, min_area=1000):
    """
    识别图像中的长矩形区域
    参数:
        image_path: 图像路径
        min_length: 最小长度(像素)
        min_aspect_ratio: 最小长宽比
        min_area: 最小面积
    """
    # 读取图像
    image = cv2.imread(image_path)
    if image is None:
        print(f"错误: 无法读取图像 {image_path}")
        return None
    
    original = image.copy()
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    
    # 1. 预处理 - 减少噪声并增强边缘
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    # 2. 自适应阈值处理 - 不依赖全局光照
    thresh = cv2.adaptiveThreshold(
        blurred, 255, 
        cv2.ADAPTIVE_THRESH_GAUSSIAN_C, 
        cv2.THRESH_BINARY_INV, 11, 2
    )
    
    # 3. 形态学操作 - 连接边缘并填充小孔
    kernel = np.ones((3, 3), np.uint8)
    processed = cv2.morphologyEx(thresh, cv2.MORPH_CLOSE, kernel, iterations=2)
    processed = cv2.dilate(processed, kernel, iterations=1)
    
    # 4. 查找轮廓
    contours, _ = cv2.findContours(processed, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    # 5. 筛选长矩形轮廓
    detected_rectangles = []
    
    for contour in contours:
        # 计算轮廓面积
        area = cv2.contourArea(contour)
        if area < min_area:
            continue
            
        # 获取最小外接矩形
        rect = cv2.minAreaRect(contour)
        box = cv2.boxPoints(rect)
        box = np.int0(box)
        
        # 计算边长
        width, height = rect[1]
        long_side = max(width, height)
        short_side = min(width, height)
        
        # 筛选条件：足够长且长宽比大
        if long_side > min_length and (long_side / max(short_side, 1)) > min_aspect_ratio:
            detected_rectangles.append(box)
            
            # 绘制边界框
            cv2.drawContours(original, [box], 0, (0, 255, 0), 2)
            
            # 标记方向
            center = tuple(np.int0(rect[0]))
            angle = rect[2]
            if angle < -45:
                angle += 90
            cv2.putText(original, f"{angle:.1f}°", center, 
                        cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 0, 0), 2)
    
    # 6. 显示结果
    cv2.imshow('Processed Image', processed)
    cv2.imshow('Detected Rectangles', original)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
    
    # 返回结果图像和检测到的矩形
    return original, detected_rectangles

if __name__ == "__main__":
    # 使用示例
    result_image, rectangles = detect_long_rectangles(
        "input.jpg", 
        min_length=100,     # 最小长度
        min_aspect_ratio=5, # 最小长宽比
        min_area=500        # 最小面积
    )
    
    # 保存结果
    if result_image is not None:
        cv2.imwrite("detected_rectangles.jpg", result_image)
        print(f"检测到 {len(rectangles)} 个长矩形区域")
```

## 关键步骤说明

1. **预处理**：
   - 转换为灰度图像
   - 应用高斯模糊减少噪声

2. **自适应阈值处理**：
   - 使用`adaptiveThreshold`替代固定阈值
   - 更好地处理光照不均匀的图像

3. **形态学操作**：
   - 闭合操作连接边缘
   - 膨胀操作填充小间隙

4. **轮廓检测与分析**：
   - 查找所有外部轮廓
   - 计算每个轮廓的最小外接矩形

5. **矩形筛选**：
   - **面积过滤**：排除小面积噪点
   - **长度过滤**：确保矩形足够长
   - **长宽比过滤**：确保矩形是细长形状

## 参数调整建议

1. **min_length**：设置矩形的最小长度（像素），根据图像中目标大小调整
2. **min_aspect_ratio**：长宽比阈值（长边/短边），值越大要求越细长
3. **min_area**：最小面积，过滤小噪点
4. **形态学参数**：调整核大小和迭代次数优化边缘连接

## 高级优化技巧

```python
# 在detect_long_rectangles函数中添加以下优化：

# 边缘检测替代阈值（对某些图像效果更好）
edges = cv2.Canny(blurred, 50, 150)
processed = edges.copy()

# 轮廓近似 - 减少点数，提高处理效率
peri = cv2.arcLength(contour, True)
approx = cv2.approxPolyDP(contour, 0.02 * peri, True)

# 平行线检测（验证是否为矩形）
if len(approx) == 4:
    # 计算对边平行度
    vectors = [
        approx[1][0] - approx[0][0],
        approx[2][0] - approx[1][0],
        approx[3][0] - approx[2][0],
        approx[0][0] - approx[3][0]
    ]
    # 计算相邻边夹角
    angles = []
    for i in range(4):
        v1 = vectors[i]
        v2 = vectors[(i+1) % 4]
        cos_angle = np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))
        angles.append(np.arccos(np.clip(cos_angle, -1, 1)) * 180 / np.pi)
    
    # 检查角度是否接近90度
    if all(80 < angle < 100 for angle in angles):
        # 确认是矩形
```

## 应用场景

这种方法适用于识别各种长矩形物体，无论其颜色如何：
- 建筑中的门窗结构
- 工业场景中的传送带、管道
- 文档中的表格线条
- 道路标志和标线
- 电子元件和PCB板上的长条形结构

通过调整参数，可以适应不同大小、方向和长宽比的矩形检测需求。要使用 Python 的 OpenCV 识别图像中多个偏黑色长条区域，可以按照以下步骤操作。以下代码将演示如何检测并标记这些区域：

```python
import cv2
import numpy as np

# 1. 读取图像
image = cv2.imread('input.jpg')
original = image.copy()

# 2. 转换为HSV色彩空间（更好地分离颜色）
hsv = cv2.cvtColor(image, cv2.COLOR_BGR2HSV)

# 3. 定义黑色/深色区域的HSV范围（根据实际情况调整）
lower_black = np.array([0, 0, 0])
upper_black = np.array([180, 255, 50])  # V通道上限控制暗度

# 4. 创建掩模并应用
mask = cv2.inRange(hsv, lower_black, upper_black)

# 5. 形态学操作（优化掩模）
kernel = np.ones((5, 5), np.uint8)
mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel)  # 闭合操作填补空隙
mask = cv2.morphologyEx(mask, cv2.MORPH_OPEN, kernel)   # 开操作去除噪点

# 6. 查找轮廓
contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

# 7. 筛选长条形区域
min_area = 500  # 最小面积（根据图像调整）
detected_rects = []

for cnt in contours:
    area = cv2.contourArea(cnt)
    if area < min_area:
        continue
    
    # 获取最小外接矩形
    rect = cv2.minAreaRect(cnt)
    box = cv2.boxPoints(rect)
    box = np.int0(box)
    
    # 计算长宽比（取最大值确保>1）
    width, height = rect[1]
    aspect_ratio = max(width, height) / min(width, height)
    
    # 筛选长宽比较大的区域（长条形）
    if aspect_ratio > 4:  # 调整此阈值控制长宽比要求
        detected_rects.append(box)
        cv2.drawContours(original, [box], 0, (0, 255, 0), 2)  # 绘制绿色框

# 8. 显示结果
cv2.imshow('Mask', mask)
cv2.imshow('Detected Bars', original)
cv2.waitKey(0)
cv2.destroyAllWindows()

# 9. 保存结果（可选）
cv2.imwrite('detected_bars.jpg', original)
```

### 关键参数说明：
1. **HSV范围调整**：
   - `upper_black = [180, 255, 50]`：V值（亮度）上限控制暗度，值越小要求越暗
   - 使用颜色选择器工具精确获取目标颜色范围

2. **形态学操作**：
   - `kernel = (5,5)`：根据噪点/间隙大小调整核尺寸
   - 图像噪点多时增加开运算次数

3. **长条筛选条件**：
   - `min_area`：过滤小面积噪点
   - `aspect_ratio > 4`：长宽比阈值，值越大要求越细长

### 实际应用建议：
1. **预处理优化**：
   ```python
   # 示例：增加高斯模糊减少噪点
   blurred = cv2.GaussianBlur(hsv, (5, 5), 0)
   mask = cv2.inRange(blurred, lower_black, upper_black)
   ```

2. **处理光照不均**：
   ```python
   # 使用自适应阈值替代固定阈值
   gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
   thresh = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                                 cv2.THRESH_BINARY_INV, 11, 2)
   ```

3. **精确角度检测**（需要时）：
   ```python
   # 在绘制矩形前获取角度
   angle = rect[2]
   if angle < -45:
       angle += 90
   print(f"检测到长条，角度: {angle:.1f}°")
   ```

### 调试技巧：
1. 实时调整参数：
   ```python
   # 创建调试窗口
   cv2.namedWindow('Adjustments')
   cv2.createTrackbar('V Max', 'Adjustments', 50, 255, lambda x: None)
   
   while True:
       v_max = cv2.getTrackbarPos('V Max', 'Adjustments')
       upper_black = np.array([180, 255, v_max])
       # ... 更新处理流程 ...
       if cv2.waitKey(1) & 0xFF == ord('q'):
           break
   ```

2. 多步骤可视化：
   ```python
   cv2.imshow('1. Original', image)
   cv2.imshow('2. Mask', mask)
   cv2.imshow('3. Processed', processed)
   ```

此方案适用于检测深色条状物体，如：机械零件、电子元件、道路标线等。实际应用中需根据具体图像特点调整参数阈值。

---
# 基于矩形形状识别多个长条区域（不依赖颜色）

以下是不依赖颜色信息，纯粹基于矩形形状识别多个长条区域的解决方案。这种方法使用边缘检测和轮廓分析来寻找图像中的长矩形结构。

```python
import cv2
import numpy as np

def detect_long_rectangles(image_path, min_length=100, min_aspect_ratio=5, min_area=1000):
    """
    识别图像中的长矩形区域
    参数:
        image_path: 图像路径
        min_length: 最小长度(像素)
        min_aspect_ratio: 最小长宽比
        min_area: 最小面积
    """
    # 读取图像
    image = cv2.imread(image_path)
    if image is None:
        print(f"错误: 无法读取图像 {image_path}")
        return None
    
    original = image.copy()
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    
    # 1. 预处理 - 减少噪声并增强边缘
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    # 2. 自适应阈值处理 - 不依赖全局光照
    thresh = cv2.adaptiveThreshold(
        blurred, 255, 
        cv2.ADAPTIVE_THRESH_GAUSSIAN_C, 
        cv2.THRESH_BINARY_INV, 11, 2
    )
    
    # 3. 形态学操作 - 连接边缘并填充小孔
    kernel = np.ones((3, 3), np.uint8)
    processed = cv2.morphologyEx(thresh, cv2.MORPH_CLOSE, kernel, iterations=2)
    processed = cv2.dilate(processed, kernel, iterations=1)
    
    # 4. 查找轮廓
    contours, _ = cv2.findContours(processed, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    # 5. 筛选长矩形轮廓
    detected_rectangles = []
    
    for contour in contours:
        # 计算轮廓面积
        area = cv2.contourArea(contour)
        if area < min_area:
            continue
            
        # 获取最小外接矩形
        rect = cv2.minAreaRect(contour)
        box = cv2.boxPoints(rect)
        box = np.int0(box)
        
        # 计算边长
        width, height = rect[1]
        long_side = max(width, height)
        short_side = min(width, height)
        
        # 筛选条件：足够长且长宽比大
        if long_side > min_length and (long_side / max(short_side, 1)) > min_aspect_ratio:
            detected_rectangles.append(box)
            
            # 绘制边界框
            cv2.drawContours(original, [box], 0, (0, 255, 0), 2)
            
            # 标记方向
            center = tuple(np.int0(rect[0]))
            angle = rect[2]
            if angle < -45:
                angle += 90
            cv2.putText(original, f"{angle:.1f}°", center, 
                        cv2.FONT_HERSHEY_SIMPLEX, 0.5, (255, 0, 0), 2)
    
    # 6. 显示结果
    cv2.imshow('Processed Image', processed)
    cv2.imshow('Detected Rectangles', original)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
    
    # 返回结果图像和检测到的矩形
    return original, detected_rectangles

if __name__ == "__main__":
    # 使用示例
    result_image, rectangles = detect_long_rectangles(
        "input.jpg", 
        min_length=100,     # 最小长度
        min_aspect_ratio=5, # 最小长宽比
        min_area=500        # 最小面积
    )
    
    # 保存结果
    if result_image is not None:
        cv2.imwrite("detected_rectangles.jpg", result_image)
        print(f"检测到 {len(rectangles)} 个长矩形区域")
```

## 关键步骤说明

1. **预处理**：
   - 转换为灰度图像
   - 应用高斯模糊减少噪声

2. **自适应阈值处理**：
   - 使用`adaptiveThreshold`替代固定阈值
   - 更好地处理光照不均匀的图像

3. **形态学操作**：
   - 闭合操作连接边缘
   - 膨胀操作填充小间隙

4. **轮廓检测与分析**：
   - 查找所有外部轮廓
   - 计算每个轮廓的最小外接矩形

5. **矩形筛选**：
   - **面积过滤**：排除小面积噪点
   - **长度过滤**：确保矩形足够长
   - **长宽比过滤**：确保矩形是细长形状

## 参数调整建议

1. **min_length**：设置矩形的最小长度（像素），根据图像中目标大小调整
2. **min_aspect_ratio**：长宽比阈值（长边/短边），值越大要求越细长
3. **min_area**：最小面积，过滤小噪点
4. **形态学参数**：调整核大小和迭代次数优化边缘连接

## 高级优化技巧

```python
# 在detect_long_rectangles函数中添加以下优化：

# 边缘检测替代阈值（对某些图像效果更好）
edges = cv2.Canny(blurred, 50, 150)
processed = edges.copy()

# 轮廓近似 - 减少点数，提高处理效率
peri = cv2.arcLength(contour, True)
approx = cv2.approxPolyDP(contour, 0.02 * peri, True)

# 平行线检测（验证是否为矩形）
if len(approx) == 4:
    # 计算对边平行度
    vectors = [
        approx[1][0] - approx[0][0],
        approx[2][0] - approx[1][0],
        approx[3][0] - approx[2][0],
        approx[0][0] - approx[3][0]
    ]
    # 计算相邻边夹角
    angles = []
    for i in range(4):
        v1 = vectors[i]
        v2 = vectors[(i+1) % 4]
        cos_angle = np.dot(v1, v2) / (np.linalg.norm(v1) * np.linalg.norm(v2))
        angles.append(np.arccos(np.clip(cos_angle, -1, 1)) * 180 / np.pi)
    
    # 检查角度是否接近90度
    if all(80 < angle < 100 for angle in angles):
        # 确认是矩形
```

## 应用场景

这种方法适用于识别各种长矩形物体，无论其颜色如何：
- 建筑中的门窗结构
- 工业场景中的传送带、管道
- 文档中的表格线条
- 道路标志和标线
- 电子元件和PCB板上的长条形结构

通过调整参数，可以适应不同大小、方向和长宽比的矩形检测需求。