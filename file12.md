下面是一个改进的交互式Matplotlib应用，使用单选框选择顶点，键盘方向键控制移动，并实时显示分辨率及顶点信息：

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.widgets import RadioButtons, Button
import cv2

class InteractiveCrop:
    def __init__(self, image_path):
        # 加载图像
        self.image = cv2.cvtColor(cv2.imread(image_path), cv2.COLOR_BGR2RGB)
        self.h, self.w = self.image.shape[:2]
        
        # 创建界面
        self.fig = plt.figure(figsize=(14, 8))
        self.ax1 = plt.subplot2grid((2, 3), (0, 0), rowspan=2)  # 原图
        self.ax2 = plt.subplot2grid((2, 3), (0, 1), rowspan=2)  # 裁剪结果
        self.ax_info = plt.subplot2grid((2, 3), (0, 2))         # 信息显示
        self.ax_ctrl = plt.subplot2grid((2, 3), (1, 2))         # 控制面板
        
        # 设置子图属性
        self.ax1.set_title("原始图像 (使用方向键调整顶点)")
        self.ax2.set_title("裁剪结果")
        self.ax_info.axis('off')
        self.ax_ctrl.axis('off')
        
        # 初始化四边形顶点 [左上, 右上, 右下, 左下]
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],  # 左上
            [self.w * 0.8, self.h * 0.2],  # 右上
            [self.w * 0.8, self.h * 0.8],  # 右下
            [self.w * 0.2, self.h * 0.8]   # 左下
        ])
        
        # 当前选中的顶点索引
        self.selected_vertex = 0  # 默认选择左上角顶点
        
        # 显示原始图像
        self.ax1.imshow(self.image)
        
        # 创建四边形连线
        self.polygon = plt.Polygon(
            self.vertices, closed=True, 
            fill=False, edgecolor='lime', linewidth=2, alpha=0.8
        )
        self.ax1.add_patch(self.polygon)
        
        # 创建顶点标记
        self.vertex_markers = []
        colors = ['red', 'blue', 'green', 'purple']
        labels = ['左上', '右上', '右下', '左下']
        for i, (vertex, color, label) in enumerate(zip(self.vertices, colors, labels)):
            marker = self.ax1.scatter(
                *vertex, s=100, c=color, 
                edgecolors='white', label=label
            )
            self.vertex_markers.append(marker)
        
        # 添加图例
        self.ax1.legend(loc='upper right')
        
        # 显示裁剪结果
        self.cropped_img = self.crop_image()
        self.img_display = self.ax2.imshow(self.cropped_img)
        
        # 创建单选框
        radio_ax = plt.axes([0.75, 0.35, 0.2, 0.15])
        self.radio = RadioButtons(
            radio_ax, ('左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'),
            active=self.selected_vertex
        )
        self.radio.on_clicked(self.select_vertex)
        
        # 创建重置按钮
        btn_ax = plt.axes([0.75, 0.2, 0.2, 0.05])
        self.reset_btn = Button(btn_ax, '重置四边形 (R)')
        self.reset_btn.on_clicked(self.reset_polygon)
        
        # 连接键盘事件
        self.fig.canvas.mpl_connect('key_press_event', self.on_key_press)
        
        # 初始信息显示
        self.update_info()
        self.update_display()
        
        plt.tight_layout()
        plt.show()

    def crop_image(self):
        """根据四边形进行透视变换裁剪"""
        # 定义目标矩形尺寸
        w = int(np.linalg.norm(self.vertices[0] - self.vertices[1]))
        h = int(np.linalg.norm(self.vertices[1] - self.vertices[2]))
        
        # 目标点坐标
        dst_points = np.array([[0, 0], [w, 0], [w, h], [0, h]], dtype='float32')
        
        # 计算透视变换矩阵
        matrix = cv2.getPerspectiveTransform(
            self.vertices.astype('float32'), 
            dst_points
        )
        
        # 应用透视变换
        warped = cv2.warpPerspective(
            self.image, matrix, (w, h),
            flags=cv2.INTER_LINEAR
        )
        return warped

    def update_display(self):
        """更新所有显示元素"""
        self.polygon.set_xy(self.vertices)
        
        # 更新顶点位置
        for i, marker in enumerate(self.vertex_markers):
            marker.set_offsets(self.vertices[i])
        
        # 更新裁剪图像
        self.cropped_img = self.crop_image()
        self.img_display.set_array(self.cropped_img)
        
        # 更新信息显示
        self.update_info()
        
        # 高亮当前选中的顶点
        for i, marker in enumerate(self.vertex_markers):
            alpha = 1.0 if i == self.selected_vertex else 0.7
            marker.set_alpha(alpha)
        
        self.fig.canvas.draw_idle()

    def update_info(self):
        """更新信息显示区域"""
        # 清除原有内容
        self.ax_info.clear()
        self.ax_info.axis('off')
        
        # 获取裁剪后图像尺寸
        cropped_h, cropped_w = self.cropped_img.shape[:2]
        
        # 显示信息
        info_text = [
            f"原图分辨率: {self.w} × {self.h}",
            f"裁剪分辨率: {cropped_w} × {cropped_h}",
            "",
            "顶点坐标:",
            f"左上: ({int(self.vertices[0][0])}, {int(self.vertices[0][1])})",
            f"右上: ({int(self.vertices[1][0])}, {int(self.vertices[1][1])})",
            f"右下: ({int(self.vertices[2][0])}, {int(self.vertices[2][1])})",
            f"左下: ({int(self.vertices[3][0])}, {int(self.vertices[3][1])})",
            "",
            "操作指南:",
            "1-4: 选择顶点",
            "方向键: 移动顶点",
            "R: 重置四边形"
        ]
        
        # 添加信息文本
        for i, text in enumerate(info_text):
            self.ax_info.text(0.05, 0.95 - i * 0.07, text, 
                             fontsize=10, transform=self.ax_info.transAxes,
                             verticalalignment='top')
        
        # 添加边框
        self.ax_info.set_frame_on(True)
        self.ax_info.patch.set_alpha(0.1)
        
        # 添加标题
        self.ax_info.set_title("图像信息", pad=10)

    def select_vertex(self, label):
        """通过单选框选择顶点"""
        self.selected_vertex = ['左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'].index(label)
        self.update_display()

    def move_vertex(self, dx, dy):
        """移动当前选中的顶点"""
        x, y = self.vertices[self.selected_vertex]
        new_x = max(0, min(self.w, x + dx))
        new_y = max(0, min(self.h, y + dy))
        self.vertices[self.selected_vertex] = [new_x, new_y]
        self.update_display()

    def reset_polygon(self, event=None):
        """重置四边形位置"""
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],
            [self.w * 0.8, self.h * 0.2],
            [self.w * 0.8, self.h * 0.8],
            [self.w * 0.2, self.h * 0.8]
        ])
        self.radio.set_active(0)  # 重置为选择左上顶点
        self.selected_vertex = 0
        self.update_display()

    def on_key_press(self, event):
        """处理键盘事件"""
        step = 5  # 每次移动的步长
        
        # 数字键选择顶点
        if event.key in ['1', '2', '3', '4']:
            index = int(event.key) - 1
            self.selected_vertex = index
            self.radio.set_active(index)
        
        # 方向键移动顶点
        elif event.key == 'up':
            self.move_vertex(0, -step)
        elif event.key == 'down':
            self.move_vertex(0, step)
        elif event.key == 'left':
            self.move_vertex(-step, 0)
        elif event.key == 'right':
            self.move_vertex(step, 0)
        
        # 重置
        elif event.key.lower() == 'r':
            self.reset_polygon()

# 使用示例
if __name__ == "__main__":
    # 替换为你的图片路径
    app = InteractiveCrop("your_image.jpg")
```

## 功能说明

1. **交互方式改进**：
   - 使用数字键1-4快速选择顶点（左上、右上、右下、左下）
   - 使用方向键移动当前选中的顶点
   - 按R键重置四边形位置

2. **信息显示区域**：
   - 显示原始图像分辨率
   - 显示裁剪后的分辨率
   - 实时显示四个顶点的坐标
   - 操作指南提示

3. **视觉反馈**：
   - 不同顶点使用不同颜色标识（红、蓝、绿、紫）
   - 当前选中的顶点会高亮显示
   - 四边形边框为绿色半透明线条

4. **控制面板**：
   - 单选框用于选择要调整的顶点
   - 重置按钮可将四边形恢复初始位置

## 操作指南

1. **选择顶点**：
   - 按数字键1选择左上顶点（红色）
   - 按数字键2选择右上顶点（蓝色）
   - 按数字键3选择右下顶点（绿色）
   - 按数字键4选择左下顶点（紫色）
   - 或使用单选框选择顶点

2. **移动顶点**：
   - ↑ 键：向上移动当前顶点
   - ↓ 键：向下移动当前顶点
   - ← 键：向左移动当前顶点
   - → 键：向右移动当前顶点

3. **重置**：
   - 按R键或点击"重置四边形"按钮恢复初始位置

4. **信息显示**：
   - 右侧信息面板实时显示原图和裁剪图的分辨率
   - 实时显示四个顶点的坐标位置

## 依赖安装

```bash
pip install matplotlib opencv-python numpy
```

这个实现提供了更精确的控制方式，通过键盘操作可以更精细地调整四边形顶点位置，同时实时显示的分辨率和坐标信息让用户能够精确控制裁剪区域。

---
下面是修改后的代码，去掉了 `tight_layout()` 并调整了布局，确保所有元素都能正确显示：

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.widgets import RadioButtons, Button
import cv2

class InteractiveCrop:
    def __init__(self, image_path):
        # 加载图像
        self.image = cv2.cvtColor(cv2.imread(image_path), cv2.COLOR_BGR2RGB)
        self.h, self.w = self.image.shape[:2]
        
        # 创建界面
        self.fig = plt.figure(figsize=(14, 8))
        
        # 设置子图位置和大小
        self.ax1 = plt.axes([0.05, 0.1, 0.4, 0.8])   # 原图
        self.ax2 = plt.axes([0.5, 0.1, 0.4, 0.8])    # 裁剪结果
        self.ax_info = plt.axes([0.75, 0.5, 0.2, 0.35])  # 信息显示
        self.ax_ctrl = plt.axes([0.75, 0.2, 0.2, 0.25])  # 控制面板
        
        # 设置子图属性
        self.ax1.set_title("原始图像 (使用方向键调整顶点)")
        self.ax2.set_title("裁剪结果")
        self.ax_info.axis('off')
        self.ax_ctrl.axis('off')
        
        # 初始化四边形顶点 [左上, 右上, 右下, 左下]
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],  # 左上
            [self.w * 0.8, self.h * 0.2],  # 右上
            [self.w * 0.8, self.h * 0.8],  # 右下
            [self.w * 0.2, self.h * 0.8]   # 左下
        ])
        
        # 当前选中的顶点索引
        self.selected_vertex = 0  # 默认选择左上角顶点
        
        # 显示原始图像
        self.ax1.imshow(self.image)
        
        # 创建四边形连线
        self.polygon = plt.Polygon(
            self.vertices, closed=True, 
            fill=False, edgecolor='lime', linewidth=2, alpha=0.8
        )
        self.ax1.add_patch(self.polygon)
        
        # 创建顶点标记
        self.vertex_markers = []
        colors = ['red', 'blue', 'green', 'purple']
        labels = ['左上', '右上', '右下', '左下']
        for i, (vertex, color, label) in enumerate(zip(self.vertices, colors, labels)):
            marker = self.ax1.scatter(
                *vertex, s=100, c=color, 
                edgecolors='white', label=label
            )
            self.vertex_markers.append(marker)
        
        # 添加图例
        self.ax1.legend(loc='upper right')
        
        # 显示裁剪结果
        self.cropped_img = self.crop_image()
        self.img_display = self.ax2.imshow(self.cropped_img)
        
        # 创建单选框
        radio_ax = plt.axes([0.76, 0.35, 0.18, 0.15])
        self.radio = RadioButtons(
            radio_ax, ('左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'),
            active=self.selected_vertex
        )
        self.radio.on_clicked(self.select_vertex)
        
        # 创建重置按钮
        btn_ax = plt.axes([0.76, 0.2, 0.18, 0.05])
        self.reset_btn = Button(btn_ax, '重置四边形 (R)')
        self.reset_btn.on_clicked(self.reset_polygon)
        
        # 连接键盘事件
        self.fig.canvas.mpl_connect('key_press_event', self.on_key_press)
        
        # 初始信息显示
        self.update_info()
        self.update_display()
        
        plt.show()

    def crop_image(self):
        """根据四边形进行透视变换裁剪"""
        # 定义目标矩形尺寸
        w = int(np.linalg.norm(self.vertices[0] - self.vertices[1]))
        h = int(np.linalg.norm(self.vertices[1] - self.vertices[2]))
        
        # 目标点坐标
        dst_points = np.array([[0, 0], [w, 0], [w, h], [0, h]], dtype='float32')
        
        # 计算透视变换矩阵
        matrix = cv2.getPerspectiveTransform(
            self.vertices.astype('float32'), 
            dst_points
        )
        
        # 应用透视变换
        warped = cv2.warpPerspective(
            self.image, matrix, (w, h),
            flags=cv2.INTER_LINEAR
        )
        return warped

    def update_display(self):
        """更新所有显示元素"""
        self.polygon.set_xy(self.vertices)
        
        # 更新顶点位置
        for i, marker in enumerate(self.vertex_markers):
            marker.set_offsets(self.vertices[i])
        
        # 更新裁剪图像
        self.cropped_img = self.crop_image()
        self.img_display.set_array(self.cropped_img)
        
        # 更新信息显示
        self.update_info()
        
        # 高亮当前选中的顶点
        for i, marker in enumerate(self.vertex_markers):
            alpha = 1.0 if i == self.selected_vertex else 0.7
            marker.set_alpha(alpha)
        
        self.fig.canvas.draw_idle()

    def update_info(self):
        """更新信息显示区域"""
        # 清除原有内容
        self.ax_info.clear()
        self.ax_info.axis('off')
        
        # 获取裁剪后图像尺寸
        cropped_h, cropped_w = self.cropped_img.shape[:2]
        
        # 显示信息
        info_text = [
            f"原图分辨率: {self.w} × {self.h}",
            f"裁剪分辨率: {cropped_w} × {cropped_h}",
            "",
            "顶点坐标:",
            f"左上: ({int(self.vertices[0][0])}, {int(self.vertices[0][1])})",
            f"右上: ({int(self.vertices[1][0])}, {int(self.vertices[1][1])})",
            f"右下: ({int(self.vertices[2][0])}, {int(self.vertices[2][1])})",
            f"左下: ({int(self.vertices[3][0])}, {int(self.vertices[3][1])})",
            "",
            "操作指南:",
            "1-4: 选择顶点",
            "方向键: 移动顶点",
            "R: 重置四边形"
        ]
        
        # 添加信息文本
        for i, text in enumerate(info_text):
            self.ax_info.text(0.05, 0.95 - i * 0.07, text, 
                             fontsize=10, transform=self.ax_info.transAxes,
                             verticalalignment='top')
        
        # 添加边框
        self.ax_info.set_frame_on(True)
        self.ax_info.patch.set_alpha(0.1)
        
        # 添加标题
        self.ax_info.set_title("图像信息", pad=10)

    def select_vertex(self, label):
        """通过单选框选择顶点"""
        self.selected_vertex = ['左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'].index(label)
        self.update_display()

    def move_vertex(self, dx, dy):
        """移动当前选中的顶点"""
        x, y = self.vertices[self.selected_vertex]
        new_x = max(0, min(self.w, x + dx))
        new_y = max(0, min(self.h, y + dy))
        self.vertices[self.selected_vertex] = [new_x, new_y]
        self.update_display()

    def reset_polygon(self, event=None):
        """重置四边形位置"""
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],
            [self.w * 0.8, self.h * 0.2],
            [self.w * 0.8, self.h * 0.8],
            [self.w * 0.2, self.h * 0.8]
        ])
        self.radio.set_active(0)  # 重置为选择左上顶点
        self.selected_vertex = 0
        self.update_display()

    def on_key_press(self, event):
        """处理键盘事件"""
        step = 5  # 每次移动的步长
        
        # 数字键选择顶点
        if event.key in ['1', '2', '3', '4']:
            index = int(event.key) - 1
            self.selected_vertex = index
            self.radio.set_active(index)
        
        # 方向键移动顶点
        elif event.key == 'up':
            self.move_vertex(0, -step)
        elif event.key == 'down':
            self.move_vertex(0, step)
        elif event.key == 'left':
            self.move_vertex(-step, 0)
        elif event.key == 'right':
            self.move_vertex(step, 0)
        
        # 重置
        elif event.key.lower() == 'r':
            self.reset_polygon()

# 使用示例
if __name__ == "__main__":
    # 替换为你的图片路径
    app = InteractiveCrop("your_image.jpg")
```

## 布局调整说明

1. **去掉了 `tight_layout()`**：
   - 使用绝对坐标定位所有元素
   - 手动设置每个子图的位置和大小

2. **重新设计的布局**：
   ```python
   # 图像区域
   self.ax1 = plt.axes([0.05, 0.1, 0.4, 0.8])   # 原图 (左侧)
   self.ax2 = plt.axes([0.5, 0.1, 0.4, 0.8])    # 裁剪结果 (中间)
   
   # 控制面板区域
   self.ax_info = plt.axes([0.75, 0.5, 0.2, 0.35])  # 信息显示 (右上)
   self.ax_ctrl = plt.axes([0.75, 0.2, 0.2, 0.25])  # 控制面板 (右下)
   
   # 单选框和按钮
   radio_ax = plt.axes([0.76, 0.35, 0.18, 0.15])   # 单选框位置
   btn_ax = plt.axes([0.76, 0.2, 0.18, 0.05])      # 按钮位置
   ```

3. **布局特点**：
   - 左侧40%宽度：原始图像
   - 中间40%宽度：裁剪结果
   - 右侧20%宽度：控制面板和信息区域
   - 控制面板和信息区域垂直排列，占据右侧空间

4. **视觉优化**：
   - 增加了信息面板的边框和透明度
   - 调整了文本大小和间距
   - 优化了单选框和按钮的位置

## 使用说明

1. **图像位置**：
   - 左侧：原始图像，显示绿色四边形和彩色顶点
   - 中间：实时更新的裁剪结果

2. **控制区域**：
   - 右上：信息面板，显示分辨率、顶点坐标和操作指南
   - 右下：单选框（选择顶点）和重置按钮

3. **键盘操作**：
   - `1`：选择左上顶点（红色）
   - `2`：选择右上顶点（蓝色）
   - `3`：选择右下顶点（绿色）
   - `4`：选择左下顶点（紫色）
   - 方向键：移动当前选中的顶点
   - `R`：重置四边形到初始位置

这个布局更加稳定，不会因为 `tight_layout()` 的自动调整而导致元素重叠或显示不全。所有组件都有明确的位置和大小，确保在各种屏幕尺寸上都能正常显示。

## 依赖安装

```bash
pip install matplotlib opencv-python numpy
```


---
下面是将信息文本和控件移到底部以避免遮挡裁剪图的优化版本：

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.widgets import RadioButtons, Button
import cv2

class InteractiveCrop:
    def __init__(self, image_path):
        # 加载图像
        self.image = cv2.cvtColor(cv2.imread(image_path), cv2.COLOR_BGR2RGB)
        self.h, self.w = self.image.shape[:2]
        
        # 创建界面
        self.fig = plt.figure(figsize=(14, 10))
        
        # 设置子图位置和大小 - 图像区域在上部
        self.ax1 = plt.axes([0.05, 0.35, 0.4, 0.6])   # 原图 (左侧)
        self.ax2 = plt.axes([0.5, 0.35, 0.4, 0.6])    # 裁剪结果 (右侧)
        
        # 控制面板和信息区域在底部
        self.ax_info = plt.axes([0.05, 0.05, 0.55, 0.25])  # 信息显示 (左下)
        self.ax_ctrl = plt.axes([0.65, 0.05, 0.3, 0.25])   # 控制面板 (右下)
        
        # 设置子图属性
        self.ax1.set_title("原始图像 (使用方向键调整顶点)")
        self.ax2.set_title("裁剪结果")
        self.ax_info.axis('off')
        self.ax_ctrl.axis('off')
        
        # 初始化四边形顶点 [左上, 右上, 右下, 左下]
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],  # 左上
            [self.w * 0.8, self.h * 0.2],  # 右上
            [self.w * 0.8, self.h * 0.8],  # 右下
            [self.w * 0.2, self.h * 0.8]   # 左下
        ])
        
        # 当前选中的顶点索引
        self.selected_vertex = 0  # 默认选择左上角顶点
        
        # 显示原始图像
        self.ax1.imshow(self.image)
        
        # 创建四边形连线
        self.polygon = plt.Polygon(
            self.vertices, closed=True, 
            fill=False, edgecolor='lime', linewidth=2, alpha=0.8
        )
        self.ax1.add_patch(self.polygon)
        
        # 创建顶点标记
        self.vertex_markers = []
        colors = ['red', 'blue', 'green', 'purple']
        labels = ['左上', '右上', '右下', '左下']
        for i, (vertex, color, label) in enumerate(zip(self.vertices, colors, labels)):
            marker = self.ax1.scatter(
                *vertex, s=100, c=color, 
                edgecolors='white', label=label
            )
            self.vertex_markers.append(marker)
        
        # 添加图例
        self.ax1.legend(loc='upper right')
        
        # 显示裁剪结果
        self.cropped_img = self.crop_image()
        self.img_display = self.ax2.imshow(self.cropped_img)
        
        # 创建单选框 - 放在控制面板区域
        radio_ax = plt.axes([0.7, 0.15, 0.25, 0.25])
        self.radio = RadioButtons(
            radio_ax, ('左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'),
            active=self.selected_vertex
        )
        self.radio.on_clicked(self.select_vertex)
        
        # 创建重置按钮 - 放在控制面板区域
        btn_ax = plt.axes([0.7, 0.05, 0.25, 0.07])
        self.reset_btn = Button(btn_ax, '重置四边形 (R)')
        self.reset_btn.on_clicked(self.reset_polygon)
        
        # 连接键盘事件
        self.fig.canvas.mpl_connect('key_press_event', self.on_key_press)
        
        # 初始信息显示
        self.update_info()
        self.update_display()
        
        # 添加标题
        plt.suptitle("交互式图像裁剪工具", fontsize=16)
        plt.show()

    def crop_image(self):
        """根据四边形进行透视变换裁剪"""
        # 定义目标矩形尺寸
        w = int(np.linalg.norm(self.vertices[0] - self.vertices[1]))
        h = int(np.linalg.norm(self.vertices[1] - self.vertices[2]))
        
        # 目标点坐标
        dst_points = np.array([[0, 0], [w, 0], [w, h], [0, h]], dtype='float32')
        
        # 计算透视变换矩阵
        matrix = cv2.getPerspectiveTransform(
            self.vertices.astype('float32'), 
            dst_points
        )
        
        # 应用透视变换
        warped = cv2.warpPerspective(
            self.image, matrix, (w, h),
            flags=cv2.INTER_LINEAR
        )
        return warped

    def update_display(self):
        """更新所有显示元素"""
        self.polygon.set_xy(self.vertices)
        
        # 更新顶点位置
        for i, marker in enumerate(self.vertex_markers):
            marker.set_offsets(self.vertices[i])
        
        # 更新裁剪图像
        self.cropped_img = self.crop_image()
        self.img_display.set_array(self.cropped_img)
        
        # 更新信息显示
        self.update_info()
        
        # 高亮当前选中的顶点
        for i, marker in enumerate(self.vertex_markers):
            alpha = 1.0 if i == self.selected_vertex else 0.7
            marker.set_alpha(alpha)
        
        self.fig.canvas.draw_idle()

    def update_info(self):
        """更新信息显示区域"""
        # 清除原有内容
        self.ax_info.clear()
        self.ax_info.axis('off')
        
        # 获取裁剪后图像尺寸
        cropped_h, cropped_w = self.cropped_img.shape[:2]
        
        # 显示信息 - 使用两列布局
        info_text = [
            f"原图分辨率: {self.w} × {self.h}",
            f"裁剪分辨率: {cropped_w} × {cropped_h}",
            "",
            "顶点坐标:",
            f"左上: ({int(self.vertices[0][0])}, {int(self.vertices[0][1])})",
            f"右上: ({int(self.vertices[1][0])}, {int(self.vertices[1][1])})",
            f"右下: ({int(self.vertices[2][0])}, {int(self.vertices[2][1])})",
            f"左下: ({int(self.vertices[3][0])}, {int(self.vertices[3][1])})"
        ]
        
        # 操作指南 - 单独一列
        guide_text = [
            "操作指南:",
            "1-4: 选择顶点",
            "方向键: 移动顶点",
            "R: 重置四边形",
            "",
            "顶点颜色:",
            "红色: 左上",
            "蓝色: 右上",
            "绿色: 右下",
            "紫色: 左下"
        ]
        
        # 添加信息文本 (左侧)
        for i, text in enumerate(info_text):
            self.ax_info.text(0.05, 0.95 - i * 0.1, text, 
                             fontsize=10, transform=self.ax_info.transAxes,
                             verticalalignment='top')
        
        # 添加操作指南 (右侧)
        for i, text in enumerate(guide_text):
            self.ax_info.text(0.55, 0.95 - i * 0.1, text, 
                             fontsize=10, transform=self.ax_info.transAxes,
                             verticalalignment='top')
        
        # 添加边框
        self.ax_info.set_frame_on(True)
        self.ax_info.patch.set_alpha(0.1)
        
        # 添加标题
        self.ax_info.set_title("图像信息与控制指南", pad=10)

    def select_vertex(self, label):
        """通过单选框选择顶点"""
        self.selected_vertex = ['左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'].index(label)
        self.update_display()

    def move_vertex(self, dx, dy):
        """移动当前选中的顶点"""
        x, y = self.vertices[self.selected_vertex]
        new_x = max(0, min(self.w, x + dx))
        new_y = max(0, min(self.h, y + dy))
        self.vertices[self.selected_vertex] = [new_x, new_y]
        self.update_display()

    def reset_polygon(self, event=None):
        """重置四边形位置"""
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],
            [self.w * 0.8, self.h * 0.2],
            [self.w * 0.8, self.h * 0.8],
            [self.w * 0.2, self.h * 0.8]
        ])
        self.radio.set_active(0)  # 重置为选择左上顶点
        self.selected_vertex = 0
        self.update_display()

    def on_key_press(self, event):
        """处理键盘事件"""
        step = 5  # 每次移动的步长
        
        # 数字键选择顶点
        if event.key in ['1', '2', '3', '4']:
            index = int(event.key) - 1
            self.selected_vertex = index
            self.radio.set_active(index)
        
        # 方向键移动顶点
        elif event.key == 'up':
            self.move_vertex(0, -step)
        elif event.key == 'down':
            self.move_vertex(0, step)
        elif event.key == 'left':
            self.move_vertex(-step, 0)
        elif event.key == 'right':
            self.move_vertex(step, 0)
        
        # 重置
        elif event.key.lower() == 'r':
            self.reset_polygon()

# 使用示例
if __name__ == "__main__":
    # 替换为你的图片路径
    app = InteractiveCrop("your_image.jpg")
```

## 布局优化说明

### 1. 重新设计的布局结构：
```python
# 图像区域在上部（占据60%高度）
self.ax1 = plt.axes([0.05, 0.35, 0.4, 0.6])  # 原图 (左侧)
self.ax2 = plt.axes([0.5, 0.35, 0.4, 0.6])   # 裁剪结果 (右侧)

# 控制面板和信息区域在底部（占据25%高度）
self.ax_info = plt.axes([0.05, 0.05, 0.55, 0.25])  # 信息显示 (左下)
self.ax_ctrl = plt.axes([0.65, 0.05, 0.3, 0.25])   # 控制面板 (右下)
```

### 2. 控件位置调整：
```python
# 单选框位置 - 在控制面板区域内
radio_ax = plt.axes([0.7, 0.15, 0.25, 0.25])

# 重置按钮位置 - 在控制面板区域内
btn_ax = plt.axes([0.7, 0.05, 0.25, 0.07])
```

### 3. 信息显示优化：
- 使用两列布局显示信息：
  - 左侧：分辨率信息和顶点坐标
  - 右侧：操作指南和顶点颜色说明
- 增加标题："图像信息与控制指南"
- 优化文本间距和字体大小

### 4. 其他改进：
- 添加主标题："交互式图像裁剪工具"
- 增加控件之间的间距
- 优化信息面板的透明度效果
- 调整了整体窗口大小（高度增加）

## 布局特点

1. **垂直分区**：
   - 上部60%：图像显示区域
   - 下部25%：信息和控制区域
   - 预留15%空间作为边距和标题

2. **水平分区**：
   - 左侧45%：原始图像
   - 中间45%：裁剪结果
   - 右侧10%：留白（避免元素拥挤）

3. **信息区域**：
   - 左侧55%宽度：图像分辨率和顶点坐标
   - 右侧45%宽度：操作指南和颜色说明

4. **控制区域**：
   - 包含单选框和重置按钮
   - 垂直排列，避免重叠

## 使用说明

1. **图像区域**：
   - 左侧：原始图像，显示绿色四边形和彩色顶点
   - 右侧：实时更新的裁剪结果

2. **底部信息区域**：
   - 左侧：显示原图和裁剪图的分辨率
   - 左侧：显示四个顶点的坐标位置
   - 右侧：操作指南和顶点颜色说明

3. **底部控制区域**：
   - 单选框：选择要调整的顶点
   - 重置按钮：将四边形恢复初始位置

4. **键盘操作**：
   - `1-4`：选择顶点（左上、右上、右下、左下）
   - `方向键`：移动当前选中的顶点
   - `R`：重置四边形位置

这个布局确保所有控件和信息都位于窗口底部，不会遮挡图像区域，同时提供了清晰的操作指南和信息显示。

## 依赖安装

```bash
pip install matplotlib opencv-python numpy
```

这个优化版本解决了控件遮挡图像的问题，同时保持了所有功能的完整性，提供了更好的用户体验。

---
下面是在原有基础上新增功能的完整代码，支持直接输入数字调整裁剪四边形和键盘移动步长：

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.widgets import RadioButtons, Button, TextBox
import cv2

class InteractiveCrop:
    def __init__(self, image_path):
        # 加载图像
        self.image = cv2.cvtColor(cv2.imread(image_path), cv2.COLOR_BGR2RGB)
        self.h, self.w = self.image.shape[:2]
        
        # 创建界面
        self.fig = plt.figure(figsize=(14, 10))
        
        # 设置子图位置和大小 - 图像区域在上部
        self.ax1 = plt.axes([0.05, 0.35, 0.4, 0.6])   # 原图 (左侧)
        self.ax2 = plt.axes([0.5, 0.35, 0.4, 0.6])    # 裁剪结果 (右侧)
        
        # 控制面板和信息区域在底部
        self.ax_info = plt.axes([0.05, 0.05, 0.55, 0.25])  # 信息显示 (左下)
        self.ax_ctrl = plt.axes([0.65, 0.05, 0.3, 0.25])   # 控制面板 (右下)
        
        # 设置子图属性
        self.ax1.set_title("原始图像 (使用方向键调整顶点)")
        self.ax2.set_title("裁剪结果")
        self.ax_info.axis('off')
        self.ax_ctrl.axis('off')
        
        # 初始化四边形顶点 [左上, 右上, 右下, 左下]
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],  # 左上
            [self.w * 0.8, self.h * 0.2],  # 右上
            [self.w * 0.8, self.h * 0.8],  # 右下
            [self.w * 0.2, self.h * 0.8]   # 左下
        ])
        
        # 当前选中的顶点索引
        self.selected_vertex = 0  # 默认选择左上角顶点
        self.step_size = 5  # 默认键盘移动步长
        
        # 显示原始图像
        self.ax1.imshow(self.image)
        
        # 创建四边形连线
        self.polygon = plt.Polygon(
            self.vertices, closed=True, 
            fill=False, edgecolor='lime', linewidth=2, alpha=0.8
        )
        self.ax1.add_patch(self.polygon)
        
        # 创建顶点标记
        self.vertex_markers = []
        colors = ['red', 'blue', 'green', 'purple']
        labels = ['左上', '右上', '右下', '左下']
        for i, (vertex, color, label) in enumerate(zip(self.vertices, colors, labels)):
            marker = self.ax1.scatter(
                *vertex, s=100, c=color, 
                edgecolors='white', label=label
            )
            self.vertex_markers.append(marker)
        
        # 添加图例
        self.ax1.legend(loc='upper right')
        
        # 显示裁剪结果
        self.cropped_img = self.crop_image()
        self.img_display = self.ax2.imshow(self.cropped_img)
        
        # 创建单选框 - 放在控制面板区域
        radio_ax = plt.axes([0.7, 0.15, 0.25, 0.25])
        self.radio = RadioButtons(
            radio_ax, ('左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'),
            active=self.selected_vertex
        )
        self.radio.on_clicked(self.select_vertex)
        
        # 创建重置按钮 - 放在控制面板区域
        btn_ax = plt.axes([0.7, 0.05, 0.25, 0.05])
        self.reset_btn = Button(btn_ax, '重置四边形 (R)')
        self.reset_btn.on_clicked(self.reset_polygon)
        
        # 创建顶点坐标输入框
        self.vertex_inputs = []
        input_positions = [
            (0.05, 0.85, "左上 X:", "左上 Y:"),
            (0.05, 0.65, "右上 X:", "右上 Y:"),
            (0.05, 0.45, "右下 X:", "右下 Y:"),
            (0.05, 0.25, "左下 X:", "左下 Y:")
        ]
        
        for i, (x, y, label_x, label_y) in enumerate(input_positions):
            # 添加标签
            plt.figtext(x, y + 0.02, label_x, fontsize=9)
            plt.figtext(x + 0.15, y + 0.02, label_y, fontsize=9)
            
            # 创建X坐标输入框
            ax_x = plt.axes([x + 0.05, y, 0.08, 0.04])
            text_box_x = TextBox(ax_x, '', initial=str(int(self.vertices[i][0])))
            text_box_x.on_submit(lambda text, idx=i: self.update_vertex_coord(idx, 0, text))
            self.vertex_inputs.append(text_box_x)
            
            # 创建Y坐标输入框
            ax_y = plt.axes([x + 0.20, y, 0.08, 0.04])
            text_box_y = TextBox(ax_y, '', initial=str(int(self.vertices[i][1])))
            text_box_y.on_submit(lambda text, idx=i: self.update_vertex_coord(idx, 1, text))
            self.vertex_inputs.append(text_box_y)
        
        # 创建步长设置输入框
        plt.figtext(0.05, 0.05, "键盘移动步长:", fontsize=9)
        step_ax = plt.axes([0.18, 0.05, 0.08, 0.04])
        self.step_box = TextBox(step_ax, '', initial=str(self.step_size))
        self.step_box.on_submit(self.update_step_size)
        
        # 连接键盘事件
        self.fig.canvas.mpl_connect('key_press_event', self.on_key_press)
        
        # 初始信息显示
        self.update_info()
        self.update_display()
        
        # 添加标题
        plt.suptitle("交互式图像裁剪工具", fontsize=16)
        plt.show()

    def crop_image(self):
        """根据四边形进行透视变换裁剪"""
        # 定义目标矩形尺寸
        w = int(np.linalg.norm(self.vertices[0] - self.vertices[1]))
        h = int(np.linalg.norm(self.vertices[1] - self.vertices[2]))
        
        # 目标点坐标
        dst_points = np.array([[0, 0], [w, 0], [w, h], [0, h]], dtype='float32')
        
        # 计算透视变换矩阵
        matrix = cv2.getPerspectiveTransform(
            self.vertices.astype('float32'), 
            dst_points
        )
        
        # 应用透视变换
        warped = cv2.warpPerspective(
            self.image, matrix, (w, h),
            flags=cv2.INTER_LINEAR
        )
        return warped

    def update_display(self):
        """更新所有显示元素"""
        self.polygon.set_xy(self.vertices)
        
        # 更新顶点位置
        for i, marker in enumerate(self.vertex_markers):
            marker.set_offsets(self.vertices[i])
        
        # 更新裁剪图像
        self.cropped_img = self.crop_image()
        self.img_display.set_array(self.cropped_img)
        
        # 更新信息显示
        self.update_info()
        
        # 更新输入框值
        for i in range(4):
            self.vertex_inputs[i*2].set_val(str(int(self.vertices[i][0])))
            self.vertex_inputs[i*2+1].set_val(str(int(self.vertices[i][1])))
        
        # 高亮当前选中的顶点
        for i, marker in enumerate(self.vertex_markers):
            alpha = 1.0 if i == self.selected_vertex else 0.7
            marker.set_alpha(alpha)
        
        self.fig.canvas.draw_idle()

    def update_info(self):
        """更新信息显示区域"""
        # 清除原有内容
        self.ax_info.clear()
        self.ax_info.axis('off')
        
        # 获取裁剪后图像尺寸
        cropped_h, cropped_w = self.cropped_img.shape[:2]
        
        # 显示信息 - 使用两列布局
        info_text = [
            f"原图分辨率: {self.w} × {self.h}",
            f"裁剪分辨率: {cropped_w} × {cropped_h}",
            f"键盘步长: {self.step_size} 像素",
            "",
            "顶点坐标:",
            f"左上: ({int(self.vertices[0][0])}, {int(self.vertices[0][1])})",
            f"右上: ({int(self.vertices[1][0])}, {int(self.vertices[1][1])})",
            f"右下: ({int(self.vertices[2][0])}, {int(self.vertices[2][1])})",
            f"左下: ({int(self.vertices[3][0])}, {int(self.vertices[3][1])})"
        ]
        
        # 操作指南 - 单独一列
        guide_text = [
            "操作指南:",
            "1-4: 选择顶点",
            "方向键: 移动顶点",
            "R: 重置四边形",
            "",
            "直接输入:",
            "· 顶点坐标可输入数值",
            "· 步长可输入新值",
            "",
            "顶点颜色:",
            "红色: 左上",
            "蓝色: 右上",
            "绿色: 右下",
            "紫色: 左下"
        ]
        
        # 添加信息文本 (左侧)
        for i, text in enumerate(info_text):
            self.ax_info.text(0.05, 0.95 - i * 0.08, text, 
                             fontsize=10, transform=self.ax_info.transAxes,
                             verticalalignment='top')
        
        # 添加操作指南 (右侧)
        for i, text in enumerate(guide_text):
            self.ax_info.text(0.55, 0.95 - i * 0.08, text, 
                             fontsize=10, transform=self.ax_info.transAxes,
                             verticalalignment='top')
        
        # 添加边框
        self.ax_info.set_frame_on(True)
        self.ax_info.patch.set_alpha(0.1)
        
        # 添加标题
        self.ax_info.set_title("图像信息与控制指南", pad=10)

    def select_vertex(self, label):
        """通过单选框选择顶点"""
        self.selected_vertex = ['左上 (1)', '右上 (2)', '右下 (3)', '左下 (4)'].index(label)
        self.update_display()

    def move_vertex(self, dx, dy):
        """移动当前选中的顶点"""
        x, y = self.vertices[self.selected_vertex]
        new_x = max(0, min(self.w, x + dx))
        new_y = max(0, min(self.h, y + dy))
        self.vertices[self.selected_vertex] = [new_x, new_y]
        self.update_display()

    def reset_polygon(self, event=None):
        """重置四边形位置"""
        self.vertices = np.array([
            [self.w * 0.2, self.h * 0.2],
            [self.w * 0.8, self.h * 0.2],
            [self.w * 0.8, self.h * 0.8],
            [self.w * 0.2, self.h * 0.8]
        ])
        self.radio.set_active(0)  # 重置为选择左上顶点
        self.selected_vertex = 0
        self.update_display()
    
    def update_vertex_coord(self, vertex_idx, coord_idx, text):
        """更新顶点坐标"""
        try:
            # 转换输入为整数
            value = int(text)
            
            # 确保坐标在图像范围内
            if coord_idx == 0:  # X坐标
                value = max(0, min(self.w, value))
            else:  # Y坐标
                value = max(0, min(self.h, value))
            
            # 更新顶点坐标
            self.vertices[vertex_idx][coord_idx] = value
            self.update_display()
        except ValueError:
            # 输入无效，恢复原值
            self.vertex_inputs[vertex_idx*2 + coord_idx].set_val(
                str(int(self.vertices[vertex_idx][coord_idx]))
    
    def update_step_size(self, text):
        """更新键盘移动步长"""
        try:
            # 转换输入为整数
            new_step = int(text)
            
            # 确保步长在合理范围内
            if 1 <= new_step <= 100:
                self.step_size = new_step
                self.update_info()
            else:
                # 步长超出范围，恢复原值
                self.step_box.set_val(str(self.step_size))
        except ValueError:
            # 输入无效，恢复原值
            self.step_box.set_val(str(self.step_size))

    def on_key_press(self, event):
        """处理键盘事件"""
        # 数字键选择顶点
        if event.key in ['1', '2', '3', '4']:
            index = int(event.key) - 1
            self.selected_vertex = index
            self.radio.set_active(index)
        
        # 方向键移动顶点
        elif event.key == 'up':
            self.move_vertex(0, -self.step_size)
        elif event.key == 'down':
            self.move_vertex(0, self.step_size)
        elif event.key == 'left':
            self.move_vertex(-self.step_size, 0)
        elif event.key == 'right':
            self.move_vertex(self.step_size, 0)
        
        # 重置
        elif event.key.lower() == 'r':
            self.reset_polygon()

# 使用示例
if __name__ == "__main__":
    # 替换为你的图片路径
    app = InteractiveCrop("your_image.jpg")
```

## 新增功能说明

### 1. 直接输入数字调整裁剪四边形

在底部信息区域增加了四组输入框，每组对应一个顶点的X和Y坐标：

```python
# 创建顶点坐标输入框
self.vertex_inputs = []
input_positions = [
    (0.05, 0.85, "左上 X:", "左上 Y:"),
    (0.05, 0.65, "右上 X:", "右上 Y:"),
    (0.05, 0.45, "右下 X:", "右下 Y:"),
    (0.05, 0.25, "左下 X:", "左下 Y:")
]

for i, (x, y, label_x, label_y) in enumerate(input_positions):
    # 添加标签
    plt.figtext(x, y + 0.02, label_x, fontsize=9)
    plt.figtext(x + 0.15, y + 0.02, label_y, fontsize=9)
    
    # 创建X坐标输入框
    ax_x = plt.axes([x + 0.05, y, 0.08, 0.04])
    text_box_x = TextBox(ax_x, '', initial=str(int(self.vertices[i][0])))
    text_box_x.on_submit(lambda text, idx=i: self.update_vertex_coord(idx, 0, text))
    self.vertex_inputs.append(text_box_x)
    
    # 创建Y坐标输入框
    ax_y = plt.axes([x + 0.20, y, 0.08, 0.04])
    text_box_y = TextBox(ax_y, '', initial=str(int(self.vertices[i][1])))
    text_box_y.on_submit(lambda text, idx=i: self.update_vertex_coord(idx, 1, text))
    self.vertex_inputs.append(text_box_y)
```

**功能特点：**
- 每个顶点有独立的X和Y坐标输入框
- 输入框显示当前坐标值
- 输入新值后按Enter键立即更新
- 输入验证确保值在有效范围内
- 当通过键盘或鼠标调整顶点时，输入框值自动更新

### 2. 调整键盘移动步长

在底部信息区域增加了步长设置输入框：

```python
# 创建步长设置输入框
plt.figtext(0.05, 0.05, "键盘移动步长:", fontsize=9)
step_ax = plt.axes([0.18, 0.05, 0.08, 0.04])
self.step_box = TextBox(step_ax, '', initial=str(self.step_size))
self.step_box.on_submit(self.update_step_size)
```

**功能特点：**
- 默认步长为5像素
- 可输入1-100范围内的整数
- 输入新值后按Enter键立即生效
- 无效输入自动恢复为原值
- 当前步长值在信息面板中实时显示

### 3. 更新顶点坐标函数

```python
def update_vertex_coord(self, vertex_idx, coord_idx, text):
    """更新顶点坐标"""
    try:
        # 转换输入为整数
        value = int(text)
        
        # 确保坐标在图像范围内
        if coord_idx == 0:  # X坐标
            value = max(0, min(self.w, value))
        else:  # Y坐标
            value = max(0, min(self.h, value))
        
        # 更新顶点坐标
        self.vertices[vertex_idx][coord_idx] = value
        self.update_display()
    except ValueError:
        # 输入无效，恢复原值
        self.vertex_inputs[vertex_idx*2 + coord_idx].set_val(
            str(int(self.vertices[vertex_idx][coord_idx]))
```

### 4. 更新步长函数

```python
def update_step_size(self, text):
    """更新键盘移动步长"""
    try:
        # 转换输入为整数
        new_step = int(text)
        
        # 确保步长在合理范围内
        if 1 <= new_step <= 100:
            self.step_size = new_step
            self.update_info()
        else:
            # 步长超出范围，恢复原值
            self.step_box.set_val(str(self.step_size))
    except ValueError:
        # 输入无效，恢复原值
        self.step_box.set_val(str(self.step_size))
```

## 布局优化

1. **输入框布局：**
   - 四组输入框垂直排列在左侧信息区域
   - 每组包含X和Y坐标输入框
   - 每个输入框都有清晰的标签

2. **步长设置：**
   - 位于底部左侧信息区域
   - 与顶点输入框分离，避免混淆

3. **信息面板更新：**
   - 增加了当前步长显示
   - 更新了操作指南，包含直接输入功能的说明

4. **整体布局：**
   - 保持了图像区域在上部，控制区域在下部的结构
   - 增加了输入框区域的垂直间距
   - 优化了文本大小和间距

## 使用指南

1. **直接输入顶点坐标：**
   - 在对应顶点的X和Y输入框中输入新值
   - 按Enter键确认输入
   - 系统会自动验证输入的有效性

2. **调整步长：**
   - 在"键盘移动步长"输入框中输入新值（1-100）
   - 按Enter键确认输入
   - 新的步长会立即生效

3. **键盘操作：**
   - `1-4`：选择顶点
   - `方向键`：按当前步长移动选中的顶点
   - `R`：重置四边形位置

4. **顶点颜色：**
   - 红色：左上顶点
   - 蓝色：右上顶点
   - 绿色：右下顶点
   - 紫色：左下顶点

这个增强版本提供了更精确的控制方式，用户可以直接输入坐标值进行精细调整，同时可以根据需要设置键盘移动的步长，大大提高了工具的灵活性和实用性。