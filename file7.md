要识别图像中多个由上下偏黑边和中间小灯珠组成的矩形区域，即使图像模糊，可以通过以下步骤实现：

### 解决方案思路：
1. **预处理**：增强对比度并减少噪点
2. **边缘检测**：识别矩形区域的上下黑边
3. **形态学操作**：连接断裂的边缘
4. **轮廓检测**：找到潜在矩形区域
5. **区域验证**：通过灯珠特征确认有效矩形

### 代码实现：
```python
import cv2
import numpy as np

def detect_rectangles(image_path):
    # 读取图像并转换为灰度图
    image = cv2.imread(image_path)
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    
    # 1. 预处理：增强对比度 + 降噪
    clahe = cv2.createCLAHE(clipLimit=3.0, tileGridSize=(8, 8))
    enhanced = clahe.apply(gray)
    blurred = cv2.GaussianBlur(enhanced, (5, 5), 0)
    
    # 2. 边缘检测（重点检测水平边缘）
    sobelx = cv2.Sobel(blurred, cv2.CV_64F, 1, 0, ksize=3)
    sobely = cv2.Sobel(blurred, cv2.CV_64F, 0, 1, ksize=3)
    edges = cv2.magnitude(sobelx, sobely)
    edges = np.uint8(edges)
    
    # 3. 二值化处理（突出暗色边缘）
    _, thresh = cv2.threshold(edges, 50, 255, cv2.THRESH_BINARY)
    
    # 4. 形态学操作：连接断裂边缘
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (30, 3))  # 水平方向连接
    closed = cv2.morphologyEx(thresh, cv2.MORPH_CLOSE, kernel)
    
    # 5. 查找轮廓
    contours, _ = cv2.findContours(closed, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    valid_rectangles = []
    
    for contour in contours:
        # 6. 筛选大尺寸轮廓
        if cv2.contourArea(contour) < 1000:
            continue
            
        # 7. 获取最小外接矩形
        rect = cv2.minAreaRect(contour)
        box = cv2.boxPoints(rect)
        box = np.int0(box)
        
        # 8. 验证灯珠特征（核心步骤）
        x, y, w, h = cv2.boundingRect(contour)
        roi = enhanced[y:y+h, x:x+w]
        
        # 检测灯珠：寻找亮区域
        _, light_mask = cv2.threshold(roi, 200, 255, cv2.THRESH_BINARY)
        
        # 计算亮区域占比（灯珠区域）
        light_ratio = cv2.countNonZero(light_mask) / (w * h)
        
        # 特征验证：灯珠占比在5%-30%之间
        if 0.05 < light_ratio < 0.3:
            valid_rectangles.append(box)
    
    # 9. 在原图上绘制结果
    result = image.copy()
    for box in valid_rectangles:
        cv2.drawContours(result, [box], 0, (0, 255, 0), 3)
    
    return result

# 使用示例
result_image = detect_rectangles("your_image.jpg")
cv2.imshow("Detected Rectangles", result_image)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 关键优化点：
1. **水平边缘强化**：
   - 使用`(30, 3)`的水平核进行形态学操作，专门连接水平方向的黑色边缘
   - Sobel算子重点检测水平边缘（上下黑边）

2. **灯珠特征验证**：
   - 在候选区域内检测高亮像素（灯珠）
   - 通过亮区占比（5%-30%）排除误检区域

3. **模糊图像处理**：
   - CLAHE增强对比度，改善模糊图像
   - 高斯模糊减少噪点干扰

4. **参数调整建议**：
   - 若漏检：降低`cv2.threshold`阈值（50→30）
   - 若误检：提高面积阈值（1000→2000）
   - 灯珠比例根据实际图像调整（0.05-0.3）

### 效果说明：
![识别效果示意图](https://example.com/path/to/demo_image.jpg)
（左：原始模糊图像，右：识别结果）

此方案能有效处理：
- 模糊图像中的矩形区域
- 断裂的边缘连接
- 通过灯珠特征排除非目标区域
- 多个相邻矩形的分离识别

> 注意：实际应用中需根据具体图像调整阈值参数，特别是灯珠亮度的阈值（200）和灯珠区域比例（0.05-0.3）可能需要微调。

---
要识别图像中由上下两条偏黑边组成的矩形区域（忽略中间的模糊灯珠），可以采用以下步骤：

### 解决方案思路：
1. **预处理**：增强黑色边缘、抑制中间灯珠区域
2. **边缘检测**：使用自适应阈值处理模糊图像
3. **形态学操作**：连接断裂的边缘并形成闭合区域
4. **轮廓检测**：查找符合矩形特征的轮廓
5. **区域过滤**：根据长宽比和面积筛选有效矩形

### 代码实现：
```python
import cv2
import numpy as np

def detect_rectangles(image_path):
    # 读取图像并转为灰度图
    img = cv2.imread(image_path)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    
    # 1. 预处理：抑制中间灯珠区域
    # 使用高斯模糊减少噪声
    blurred = cv2.GaussianBlur(gray, (5, 5), 0)
    
    # 2. 增强黑色边缘：通过反相和自适应阈值处理
    # 反转图像使黑边变白（便于阈值处理）
    inverted = cv2.bitwise_not(blurred)
    
    # 自适应阈值处理（更适合模糊图像）
    thresh = cv2.adaptiveThreshold(
        inverted, 255,
        cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
        cv2.THRESH_BINARY,
        11, 2
    )
    
    # 3. 形态学操作：连接断裂的边缘
    # 创建水平方向的结构元素（重点连接水平边缘）
    horizontal_kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (30, 1))
    morph = cv2.morphologyEx(thresh, cv2.MORPH_CLOSE, horizontal_kernel, iterations=2)
    
    # 4. 检测轮廓
    contours, _ = cv2.findContours(morph, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    
    detected_rectangles = []
    for cnt in contours:
        # 5. 筛选有效矩形区域
        # 计算轮廓面积
        area = cv2.contourArea(cnt)
        # 获取最小外接矩形
        rect = cv2.minAreaRect(cnt)
        box = cv2.boxPoints(rect)
        box = np.int0(box)
        
        # 计算矩形长宽
        width = np.linalg.norm(box[0] - box[1])
        height = np.linalg.norm(box[1] - box[2])
        aspect_ratio = max(width, height) / min(width, height)
        
        # 过滤条件（根据实际图像调整参数）
        min_area = 500  # 最小区域面积
        max_aspect_ratio = 4  # 最大长宽比
        
        if area > min_area and aspect_ratio < max_aspect_ratio:
            detected_rectangles.append(box)
    
    # 在原图上绘制结果
    result = img.copy()
    for box in detected_rectangles:
        cv2.drawContours(result, [box], 0, (0, 255, 0), 2)
    
    return result

# 使用示例
result_image = detect_rectangles("your_image.jpg")
cv2.imshow("Detected Rectangles", result_image)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 关键参数调整说明：
1. **自适应阈值参数**（`blockSize=11, C=2`）：
   - 增大 `blockSize` 可处理更模糊的图像
   - 调整 `C` 值控制边缘检测灵敏度

2. **形态学核大小**（`(30, 1)`）：
   - 第一个值（30）：控制水平连接强度（根据实际灯珠间距调整）
   - 第二个值（1）：保持垂直方向原始边缘

3. **过滤条件**：
   - `min_area`：根据图像中矩形实际大小调整
   - `max_aspect_ratio`：排除细长非矩形区域

### 处理效果说明：
1. **抑制灯珠**：通过水平方向的形态学操作，使灯珠之间的间隙被连接，同时保持上下边缘的完整性
2. **模糊处理**：自适应阈值对模糊图像更鲁棒
3. **矩形识别**：通过长宽比过滤可排除不规则区域

### 注意事项：
1. 如果图像中矩形大小差异较大，建议使用多尺度处理
2. 光照不均匀时，可先进行直方图均衡化
3. 实际应用中可能需要调整参数，建议用不同图像测试

此方案专注于检测由上下黑边组成的矩形区域，通过形态学处理和轮廓过滤有效避开了中间模糊灯珠的干扰。