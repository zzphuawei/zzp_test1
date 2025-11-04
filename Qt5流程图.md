以下是一个只包含流程图的简化版本，移除了所有其他控制面板和参数输入部分：

```python
import sys
from PyQt5.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QGraphicsView, QGraphicsScene, QGraphicsItem)
from PyQt5.QtCore import Qt, QRectF, QPointF
from PyQt5.QtGui import QPen, QBrush, QColor, QFont

class FlowChartItem(QGraphicsItem):
    """流程图元素类"""
    def __init__(self, text, x, y, width=120, height=40, item_type="normal"):
        super().__init__()
        self.text = text
        self.x = x
        self.y = y
        self.width = width
        self.height = height
        self.item_type = item_type  # normal, decision, start, end
        self.is_active = False
        
    def boundingRect(self):
        return QRectF(self.x, self.y, self.width, self.height)
    
    def paint(self, painter, option, widget):
        # 设置画笔和画刷
        if self.is_active:
            painter.setPen(QPen(QColor(0, 120, 215), 2))
            painter.setBrush(QBrush(QColor(230, 240, 255)))
        else:
            painter.setPen(QPen(Qt.black, 1))
            painter.setBrush(QBrush(QColor(240, 240, 240)))
        
        # 根据类型绘制不同形状
        if self.item_type == "decision":
            # 菱形表示判断/循环
            points = [
                QPointF(self.x + self.width/2, self.y),
                QPointF(self.x + self.width, self.y + self.height/2),
                QPointF(self.x + self.width/2, self.y + self.height),
                QPointF(self.x, self.y + self.height/2)
            ]
            painter.drawPolygon(points)
        elif self.item_type == "start":
            # 椭圆表示开始/结束
            painter.drawEllipse(self.x, self.y, self.width, self.height)
        else:
            # 圆角矩形表示普通步骤
            painter.drawRoundedRect(self.x, self.y, self.width, self.height, 10, 10)
        
        # 绘制文字
        painter.setPen(QPen(Qt.black))
        painter.setFont(QFont("Arial", 8))
        painter.drawText(QRectF(self.x, self.y, self.width, self.height), Qt.AlignCenter, self.text)

class FlowChartWindow(QMainWindow):
    """只包含流程图的窗口"""
    
    def __init__(self):
        super().__init__()
        self.init_ui()
        self.setup_flowchart()
        
    def init_ui(self):
        """初始化UI界面"""
        self.setWindowTitle("循环操作流程图")
        self.setGeometry(100, 100, 800, 900)
        
        # 创建中央部件
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        
        # 主布局
        main_layout = QVBoxLayout()
        central_widget.setLayout(main_layout)
        
        # 流程图
        self.flowchart_view = QGraphicsView()
        self.flowchart_scene = QGraphicsScene()
        self.flowchart_view.setScene(self.flowchart_scene)
        main_layout.addWidget(self.flowchart_view)
        
    def setup_flowchart(self):
        """设置流程图"""
        # 创建流程图元素
        self.flow_items = []
        
        # 开始节点
        start_item = FlowChartItem("开始", 350, 50, item_type="start")
        self.flowchart_scene.addItem(start_item)
        self.flow_items.append(("start", start_item))
        
        # 设置循环次数
        cycle_item = FlowChartItem("设置循环次数\nN=5", 350, 150)
        self.flowchart_scene.addItem(cycle_item)
        self.flow_items.append(("set_cycle", cycle_item))
        
        # 步骤1
        step1_item = FlowChartItem("步骤1\n方向:正向\n行程:100mm\n速度:50%", 350, 250)
        self.flowchart_scene.addItem(step1_item)
        self.flow_items.append(("step1", step1_item))
        
        # 等待1
        wait1_item = FlowChartItem("等待1\n时长:1秒", 350, 350)
        self.flowchart_scene.addItem(wait1_item)
        self.flow_items.append(("wait1", wait1_item))
        
        # 步骤2
        step2_item = FlowChartItem("步骤2\n方向:反向\n行程:100mm\n速度:50%", 350, 450)
        self.flowchart_scene.addItem(step2_item)
        self.flow_items.append(("step2", step2_item))
        
        # 等待2
        wait2_item = FlowChartItem("等待2\n时长:1秒", 350, 550)
        self.flowchart_scene.addItem(wait2_item)
        self.flow_items.append(("wait2", wait2_item))
        
        # 循环判断
        decision_item = FlowChartItem("循环判断\n当前:i/总数:N", 350, 650, item_type="decision")
        self.flowchart_scene.addItem(decision_item)
        self.flow_items.append(("decision", decision_item))
        
        # 结束节点
        end_item = FlowChartItem("结束", 350, 750, item_type="start")
        self.flowchart_scene.addItem(end_item)
        self.flow_items.append(("end", end_item))
        
        # 绘制连接线
        self.draw_connections()
        
    def draw_connections(self):
        """绘制连接线"""
        pen = QPen(Qt.black, 2)
        
        # 开始 -> 设置循环次数
        self.flowchart_scene.addLine(400, 90, 400, 150, pen)
        
        # 设置循环次数 -> 步骤1
        self.flowchart_scene.addLine(400, 190, 400, 250, pen)
        
        # 步骤1 -> 等待1
        self.flowchart_scene.addLine(400, 290, 400, 350, pen)
        
        # 等待1 -> 步骤2
        self.flowchart_scene.addLine(400, 390, 400, 450, pen)
        
        # 步骤2 -> 等待2
        self.flowchart_scene.addLine(400, 490, 400, 550, pen)
        
        # 等待2 -> 循环判断
        self.flowchart_scene.addLine(400, 590, 400, 650, pen)
        
        # 循环判断 -> 步骤1 (循环线 - 未完成分支)
        self.flowchart_scene.addLine(350, 670, 350, 700, pen)
        self.flowchart_scene.addLine(350, 700, 200, 700, pen)
        self.flowchart_scene.addLine(200, 700, 200, 250, pen)
        self.flowchart_scene.addLine(200, 250, 350, 250, pen)
        
        # 添加循环箭头
        self.flowchart_scene.addLine(350, 250, 355, 245, pen)
        self.flowchart_scene.addLine(350, 250, 355, 255, pen)
        
        # 循环判断 -> 结束 (完成分支)
        self.flowchart_scene.addLine(410, 670, 410, 720, pen)
        self.flowchart_scene.addLine(410, 720, 400, 720, pen)
        self.flowchart_scene.addLine(400, 720, 400, 750, pen)
        
        # 添加条件标注
        from PyQt5.QtWidgets import QGraphicsTextItem
        condition_no = QGraphicsTextItem("i < N")
        condition_no.setPos(250, 680)
        self.flowchart_scene.addItem(condition_no)
        
        condition_yes = QGraphicsTextItem("i = N")
        condition_yes.setPos(420, 680)
        self.flowchart_scene.addItem(condition_yes)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = FlowChartWindow()
    window.show()
    sys.exit(app.exec_())
```

这个简化版本只包含：

1. **流程图显示区域** - 占据整个窗口
2. **完整的流程图形元素**：
   - 开始/结束节点（椭圆形）
   - 普通步骤节点（圆角矩形）
   - 判断节点（菱形）
3. **清晰的连接线** - 带有箭头指示流向
4. **循环标注** - 显示循环条件

流程图清晰地展示了：
- 开始 → 设置循环次数 → 步骤1 → 等待1 → 步骤2 → 等待2 → 循环判断
- 循环判断有两个分支：继续循环（回到步骤1）或结束循环

窗口标题为"循环操作流程图"，专注于展示操作流程的结构。