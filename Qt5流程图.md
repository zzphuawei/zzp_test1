# Python Qt5 流程图绘制

下面是一个使用Python和Qt5绘制两种流程图的示例代码：

```python
import sys
from PyQt5.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, 
                             QHBoxLayout, QLabel, QPushButton, QFrame)
from PyQt5.QtCore import Qt, QRectF
from PyQt5.QtGui import QPainter, QPen, QBrush, QFont, QColor

class FlowChartWidget(QWidget):
    def __init__(self, chart_type="sync"):
        super().__init__()
        self.chart_type = chart_type
        self.setMinimumSize(600, 400)
        
    def paintEvent(self, event):
        painter = QPainter(self)
        painter.setRenderHint(QPainter.Antialiasing)
        
        if self.chart_type == "sync":
            self.draw_sync_flowchart(painter)
        else:
            self.draw_sequential_flowchart(painter)
    
    def draw_sync_flowchart(self, painter):
        # 设置画笔和字体
        pen = QPen(Qt.black, 2)
        painter.setPen(pen)
        painter.setFont(QFont("Arial", 10))
        
        width = self.width()
        height = self.height()
        
        # 开始节点
        start_rect = QRectF(width/2 - 50, 50, 100, 40)
        painter.drawRect(start_rect)
        painter.drawText(start_rect, Qt.AlignCenter, "开始")
        
        # 分支线
        painter.drawLine(width/2, 90, width/2, 130)
        painter.drawLine(width/2, 130, width/4, 130)
        painter.drawLine(width/2, 130, 3*width/4, 130)
        
        # X轴步骤1
        x1_rect = QRectF(width/4 - 50, 130, 100, 40)
        painter.drawRect(x1_rect)
        painter.drawText(x1_rect, Qt.AlignCenter, "X轴步骤1")
        
        # X轴步骤2
        x2_rect = QRectF(width/4 - 50, 180, 100, 40)
        painter.drawRect(x2_rect)
        painter.drawText(x2_rect, Qt.AlignCenter, "X轴步骤2")
        
        # Y轴步骤1
        y1_rect = QRectF(3*width/4 - 50, 130, 100, 40)
        painter.drawRect(y1_rect)
        painter.drawText(y1_rect, Qt.AlignCenter, "Y轴步骤1")
        
        # Y轴步骤2
        y2_rect = QRectF(3*width/4 - 50, 180, 100, 40)
        painter.drawRect(y2_rect)
        painter.drawText(y2_rect, Qt.AlignCenter, "Y轴步骤2")
        
        # 合并线
        painter.drawLine(width/4, 220, width/4, 260)
        painter.drawLine(3*width/4, 220, 3*width/4, 260)
        painter.drawLine(width/4, 260, width/2, 260)
        painter.drawLine(3*width/4, 260, width/2, 260)
        
        # 结束节点
        end_rect = QRectF(width/2 - 50, 260, 100, 40)
        painter.drawRect(end_rect)
        painter.drawText(end_rect, Qt.AlignCenter, "结束")
    
    def draw_sequential_flowchart(self, painter):
        # 设置画笔和字体
        pen = QPen(Qt.black, 2)
        painter.setPen(pen)
        painter.setFont(QFont("Arial", 10))
        
        width = self.width()
        height = self.height()
        
        # 开始节点
        start_rect = QRectF(50, height/2 - 20, 80, 40)
        painter.drawRect(start_rect)
        painter.drawText(start_rect, Qt.AlignCenter, "开始")
        
        # X轴步骤1
        x1_rect = QRectF(150, height/2 - 20, 80, 40)
        painter.drawRect(x1_rect)
        painter.drawText(x1_rect, Qt.AlignCenter, "X轴步骤1")
        
        # X轴步骤2
        x2_rect = QRectF(250, height/2 - 20, 80, 40)
        painter.drawRect(x2_rect)
        painter.drawText(x2_rect, Qt.AlignCenter, "X轴步骤2")
        
        # Y轴步骤1
        y1_rect = QRectF(350, height/2 - 20, 80, 40)
        painter.drawRect(y1_rect)
        painter.drawText(y1_rect, Qt.AlignCenter, "Y轴步骤1")
        
        # Y轴步骤2
        y2_rect = QRectF(450, height/2 - 20, 80, 40)
        painter.drawRect(y2_rect)
        painter.drawText(y2_rect, Qt.AlignCenter, "Y轴步骤2")
        
        # 结束节点
        end_rect = QRectF(550, height/2 - 20, 80, 40)
        painter.drawRect(end_rect)
        painter.drawText(end_rect, Qt.AlignCenter, "结束")
        
        # 连接线
        painter.drawLine(130, height/2, 150, height/2)
        painter.drawLine(230, height/2, 250, height/2)
        painter.drawLine(330, height/2, 350, height/2)
        painter.drawLine(430, height/2, 450, height/2)
        painter.drawLine(530, height/2, 550, height/2)

class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("Qt5 流程图示例")
        self.setGeometry(100, 100, 1200, 600)
        
        # 创建中央窗口部件
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        
        # 创建主布局
        main_layout = QVBoxLayout(central_widget)
        
        # 添加标题
        title_label = QLabel("两种流程图示例")
        title_label.setAlignment(Qt.AlignCenter)
        title_label.setStyleSheet("font-size: 18px; font-weight: bold; margin: 10px;")
        main_layout.addWidget(title_label)
        
        # 创建两个流程图的水平布局
        charts_layout = QHBoxLayout()
        main_layout.addLayout(charts_layout)
        
        # 创建同步流程图
        sync_frame = QFrame()
        sync_frame.setFrameStyle(QFrame.Box)
        sync_layout = QVBoxLayout(sync_frame)
        
        sync_label = QLabel("同步流程图")
        sync_label.setAlignment(Qt.AlignCenter)
        sync_label.setStyleSheet("font-weight: bold; margin: 5px;")
        sync_layout.addWidget(sync_label)
        
        self.sync_chart = FlowChartWidget("sync")
        sync_layout.addWidget(self.sync_chart)
        
        # 创建顺序流程图
        seq_frame = QFrame()
        seq_frame.setFrameStyle(QFrame.Box)
        seq_layout = QVBoxLayout(seq_frame)
        
        seq_label = QLabel("顺序流程图")
        seq_label.setAlignment(Qt.AlignCenter)
        seq_label.setStyleSheet("font-weight: bold; margin: 5px;")
        seq_layout.addWidget(seq_label)
        
        self.seq_chart = FlowChartWidget("sequential")
        seq_layout.addWidget(self.seq_chart)
        
        # 将两个流程图添加到水平布局
        charts_layout.addWidget(sync_frame)
        charts_layout.addWidget(seq_frame)
        
        # 添加说明
        explanation_label = QLabel(
            "第一种: 同步流程 - X轴和Y轴的步骤同时执行\n"
            "第二种: 顺序流程 - 所有步骤按顺序依次执行"
        )
        explanation_label.setAlignment(Qt.AlignCenter)
        explanation_label.setStyleSheet("margin: 10px;")
        main_layout.addWidget(explanation_label)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec_())
```

## 代码说明

这个程序创建了一个Qt5应用程序，展示了两种不同类型的流程图：

1. **同步流程图**：
   - 开始后，X轴和Y轴的步骤同时执行
   - X轴步骤1和步骤2与Y轴步骤1和步骤2并行执行
   - 所有步骤完成后，流程结束

2. **顺序流程图**：
   - 所有步骤按顺序依次执行
   - 开始 → X轴步骤1 → X轴步骤2 → Y轴步骤1 → Y轴步骤2 → 结束

## 运行要求

要运行此代码，您需要安装以下Python包：
```
pip install PyQt5
```

## 功能特点

- 使用QPainter自定义绘制流程图
- 清晰的流程图节点和连接线
- 响应式布局，适应窗口大小变化
- 两种流程图并排显示，便于比较

您可以根据需要修改流程图的步骤内容、布局样式或添加更多交互功能。