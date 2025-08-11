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