下面是一个基于PyQt5的应用程序设计方案，它可以根据输入参数循环执行操作，并在UI上展示循环流程图。

## 程序设计与实现

```python
import sys
import time
from PyQt5.QtWidgets import (QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout, 
                             QGroupBox, QLabel, QLineEdit, QPushButton, QSpinBox, 
                             QTextEdit, QFrame, QGraphicsView, QGraphicsScene, QGraphicsItem)
from PyQt5.QtCore import Qt, QTimer, pyqtSignal, QRectF, QPointF
from PyQt5.QtGui import QFont, QPen, QBrush, QColor

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
        painter.drawText(QRectF(self.x, self.y, self.width, self.height), Qt.AlignCenter, self.text)

class OperationController:
    """操作控制器"""
    def __init__(self):
        self.is_running = False
        self.current_cycle = 0
        self.total_cycles = 0
        self.current_step = 0
        
    def execute_operation(self, direction, distance, speed, wait_time):
        """执行单个操作"""
        # 这里模拟实际操作的执行
        print(f"执行操作: 方向={direction}, 行程={distance}, 速度={speed}, 等待={wait_time}秒")
        # 模拟执行时间
        time.sleep(0.1)
        
    def run_cycle(self, params, update_callback):
        """运行一个完整循环"""
        self.is_running = True
        
        for cycle in range(self.total_cycles):
            if not self.is_running:
                break
                
            self.current_cycle = cycle + 1
            print(f"开始第 {self.current_cycle}/{self.total_cycles} 个循环")
            
            # 步骤1
            self.current_step = 1
            update_callback(self.current_cycle, self.current_step)
            self.execute_operation(params['direction1'], params['distance1'], params['speed1'], 0)
            
            # 等待1
            self.current_step = 2
            update_callback(self.current_cycle, self.current_step)
            time.sleep(params['wait1'])
            
            # 步骤2
            self.current_step = 3
            update_callback(self.current_cycle, self.current_step)
            self.execute_operation(params['direction2'], params['distance2'], params['speed2'], 0)
            
            # 等待2
            self.current_step = 4
            update_callback(self.current_cycle, self.current_step)
            time.sleep(params['wait2'])
            
            # 循环判断
            self.current_step = 5
            update_callback(self.current_cycle, self.current_step)
        
        self.is_running = False
        self.current_step = 0
        update_callback(self.current_cycle, self.current_step)

class MainWindow(QMainWindow):
    """主窗口"""
    update_flowchart = pyqtSignal(int, int)  # 循环编号, 步骤编号
    
    def __init__(self):
        super().__init__()
        self.controller = OperationController()
        self.init_ui()
        self.setup_flowchart()
        
        # 连接信号
        self.update_flowchart.connect(self.highlight_flow_step)
        
    def init_ui(self):
        """初始化UI界面"""
        self.setWindowTitle("循环操作控制系统")
        self.setGeometry(100, 100, 1200, 700)
        
        # 创建中央部件
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        
        # 主布局
        main_layout = QHBoxLayout()
        central_widget.setLayout(main_layout)
        
        # 左侧：流程图
        left_layout = QVBoxLayout()
        flowchart_group = QGroupBox("循环流程图")
        self.flowchart_view = QGraphicsView()
        self.flowchart_scene = QGraphicsScene()
        self.flowchart_view.setScene(self.flowchart_scene)
        left_layout.addWidget(self.flowchart_view)
        flowchart_group.setLayout(left_layout)
        
        # 右侧：控制面板
        right_layout = QVBoxLayout()
        
        # 循环参数组
        cycle_group = QGroupBox("循环参数")
        cycle_layout = QVBoxLayout()
        
        # 循环次数
        cycle_count_layout = QHBoxLayout()
        cycle_count_layout.addWidget(QLabel("循环次数:"))
        self.cycle_count_input = QSpinBox()
        self.cycle_count_input.setRange(1, 1000)
        self.cycle_count_input.setValue(5)
        cycle_count_layout.addWidget(self.cycle_count_input)
        cycle_count_layout.addStretch()
        cycle_layout.addLayout(cycle_count_layout)
        
        cycle_group.setLayout(cycle_layout)
        right_layout.addWidget(cycle_group)
        
        # 步骤1参数组
        step1_group = QGroupBox("步骤1参数")
        step1_layout = QVBoxLayout()
        
        # 方向1
        dir1_layout = QHBoxLayout()
        dir1_layout.addWidget(QLabel("方向:"))
        self.direction1_input = QLineEdit("正向")
        dir1_layout.addWidget(self.direction1_input)
        step1_layout.addLayout(dir1_layout)
        
        # 行程1
        dist1_layout = QHBoxLayout()
        dist1_layout.addWidget(QLabel("行程:"))
        self.distance1_input = QLineEdit("100")
        dist1_layout.addWidget(self.distance1_input)
        dist1_layout.addWidget(QLabel("mm"))
        step1_layout.addLayout(dist1_layout)
        
        # 速度1
        speed1_layout = QHBoxLayout()
        speed1_layout.addWidget(QLabel("速度:"))
        self.speed1_input = QLineEdit("50")
        speed1_layout.addWidget(self.speed1_input)
        speed1_layout.addWidget(QLabel("%"))
        step1_layout.addLayout(speed1_layout)
        
        step1_group.setLayout(step1_layout)
        right_layout.addWidget(step1_group)
        
        # 等待1参数
        wait1_group = QGroupBox("等待1参数")
        wait1_layout = QHBoxLayout()
        wait1_layout.addWidget(QLabel("等待时长:"))
        self.wait1_input = QLineEdit("1")
        wait1_layout.addWidget(self.wait1_input)
        wait1_layout.addWidget(QLabel("秒"))
        wait1_group.setLayout(wait1_layout)
        right_layout.addWidget(wait1_group)
        
        # 步骤2参数组
        step2_group = QGroupBox("步骤2参数")
        step2_layout = QVBoxLayout()
        
        # 方向2
        dir2_layout = QHBoxLayout()
        dir2_layout.addWidget(QLabel("方向:"))
        self.direction2_input = QLineEdit("反向")
        dir2_layout.addWidget(self.direction2_input)
        step2_layout.addLayout(dir2_layout)
        
        # 行程2
        dist2_layout = QHBoxLayout()
        dist2_layout.addWidget(QLabel("行程:"))
        self.distance2_input = QLineEdit("100")
        dist2_layout.addWidget(self.distance2_input)
        dist2_layout.addWidget(QLabel("mm"))
        step2_layout.addLayout(dist2_layout)
        
        # 速度2
        speed2_layout = QHBoxLayout()
        speed2_layout.addWidget(QLabel("速度:"))
        self.speed2_input = QLineEdit("50")
        speed2_layout.addWidget(self.speed2_input)
        speed2_layout.addWidget(QLabel("%"))
        step2_layout.addLayout(speed2_layout)
        
        step2_group.setLayout(step2_layout)
        right_layout.addWidget(step2_group)
        
        # 等待2参数
        wait2_group = QGroupBox("等待2参数")
        wait2_layout = QHBoxLayout()
        wait2_layout.addWidget(QLabel("等待时长:"))
        self.wait2_input = QLineEdit("1")
        wait2_layout.addWidget(self.wait2_input)
        wait2_layout.addWidget(QLabel("秒"))
        wait2_group.setLayout(wait2_layout)
        right_layout.addWidget(wait2_group)
        
        # 按钮组
        button_layout = QHBoxLayout()
        self.start_btn = QPushButton("开始执行")
        self.stop_btn = QPushButton("停止执行")
        self.stop_btn.setEnabled(False)
        button_layout.addWidget(self.start_btn)
        button_layout.addWidget(self.stop_btn)
        right_layout.addLayout(button_layout)
        
        # 日志输出
        log_group = QGroupBox("执行日志")
        log_layout = QVBoxLayout()
        self.log_output = QTextEdit()
        self.log_output.setReadOnly(True)
        log_layout.addWidget(self.log_output)
        log_group.setLayout(log_layout)
        right_layout.addWidget(log_group)
        
        # 连接按钮信号
        self.start_btn.clicked.connect(self.start_execution)
        self.stop_btn.clicked.connect(self.stop_execution)
        
        # 添加到主布局
        main_layout.addWidget(flowchart_group, 2)
        main_layout.addLayout(right_layout, 1)
        
    def setup_flowchart(self):
        """设置流程图"""
        # 创建流程图元素
        self.flow_items = []
        
        # 开始节点
        start_item = FlowChartItem("开始", 50, 50, item_type="start")
        self.flowchart_scene.addItem(start_item)
        self.flow_items.append(("start", start_item))
        
        # 设置循环次数
        cycle_item = FlowChartItem("设置循环次数", 50, 150)
        self.flowchart_scene.addItem(cycle_item)
        self.flow_items.append(("set_cycle", cycle_item))
        
        # 步骤1
        step1_item = FlowChartItem("步骤1\n方向:{dir}\n行程:{dist}\n速度:{speed}".format(
            dir=self.direction1_input.text(),
            dist=self.distance1_input.text(),
            speed=self.speed1_input.text()
        ), 50, 250)
        self.flowchart_scene.addItem(step1_item)
        self.flow_items.append(("step1", step1_item))
        
        # 等待1
        wait1_item = FlowChartItem("等待1\n时长:{time}秒".format(
            time=self.wait1_input.text()
        ), 50, 350)
        self.flowchart_scene.addItem(wait1_item)
        self.flow_items.append(("wait1", wait1_item))
        
        # 步骤2
        step2_item = FlowChartItem("步骤2\n方向:{dir}\n行程:{dist}\n速度:{speed}".format(
            dir=self.direction2_input.text(),
            dist=self.distance2_input.text(),
            speed=self.speed2_input.text()
        ), 50, 450)
        self.flowchart_scene.addItem(step2_item)
        self.flow_items.append(("step2", step2_item))
        
        # 等待2
        wait2_item = FlowChartItem("等待2\n时长:{time}秒".format(
            time=self.wait2_input.text()
        ), 50, 550)
        self.flowchart_scene.addItem(wait2_item)
        self.flow_items.append(("wait2", wait2_item))
        
        # 循环判断
        decision_item = FlowChartItem("循环判断\n当前:0/总数:0", 50, 650, item_type="decision")
        self.flowchart_scene.addItem(decision_item)
        self.flow_items.append(("decision", decision_item))
        
        # 结束节点
        end_item = FlowChartItem("结束", 250, 650, item_type="start")
        self.flowchart_scene.addItem(end_item)
        self.flow_items.append(("end", end_item))
        
        # 绘制连接线（简化版）
        self.draw_connections()
        
    def draw_connections(self):
        """绘制连接线"""
        pen = QPen(Qt.black, 2)
        
        # 开始 -> 设置循环次数
        self.flowchart_scene.addLine(100, 90, 100, 150, pen)
        
        # 设置循环次数 -> 步骤1
        self.flowchart_scene.addLine(100, 190, 100, 250, pen)
        
        # 步骤1 -> 等待1
        self.flowchart_scene.addLine(100, 290, 100, 350, pen)
        
        # 等待1 -> 步骤2
        self.flowchart_scene.addLine(100, 390, 100, 450, pen)
        
        # 步骤2 -> 等待2
        self.flowchart_scene.addLine(100, 490, 100, 550, pen)
        
        # 等待2 -> 循环判断
        self.flowchart_scene.addLine(100, 590, 100, 650, pen)
        
        # 循环判断 -> 步骤1 (循环线)
        self.flowchart_scene.addLine(50, 670, 50, 700, pen)
        self.flowchart_scene.addLine(50, 700, -50, 700, pen)
        self.flowchart_scene.addLine(-50, 700, -50, 250, pen)
        self.flowchart_scene.addLine(-50, 250, 50, 250, pen)
        
        # 循环判断 -> 结束
        self.flowchart_scene.addLine(110, 670, 250, 670, pen)
        
    def highlight_flow_step(self, cycle, step):
        """高亮显示当前流程步骤"""
        # 重置所有项目状态
        for name, item in self.flow_items:
            item.is_active = False
            item.update()
        
        # 根据步骤高亮对应项目
        if cycle == 0 and step == 0:
            # 未开始状态
            self.flow_items[0][1].is_active = True
        elif step == 1:
            self.flow_items[2][1].is_active = True  # 步骤1
        elif step == 2:
            self.flow_items[3][1].is_active = True  # 等待1
        elif step == 3:
            self.flow_items[4][1].is_active = True  # 步骤2
        elif step == 4:
            self.flow_items[5][1].is_active = True  # 等待2
        elif step == 5:
            self.flow_items[6][1].is_active = True  # 循环判断
            
            # 更新循环判断文本
            decision_item = self.flow_items[6][1]
            decision_item.text = f"循环判断\n当前:{cycle}/总数:{self.controller.total_cycles}"
            decision_item.update()
        
        # 更新场景
        self.flowchart_scene.update()
        
    def start_execution(self):
        """开始执行循环操作"""
        # 获取参数
        params = {
            'direction1': self.direction1_input.text(),
            'distance1': self.distance1_input.text(),
            'speed1': self.speed1_input.text(),
            'wait1': float(self.wait1_input.text()),
            'direction2': self.direction2_input.text(),
            'distance2': self.distance2_input.text(),
            'speed2': self.speed2_input.text(),
            'wait2': float(self.wait2_input.text())
        }
        
        # 设置控制器参数
        self.controller.total_cycles = self.cycle_count_input.value()
        
        # 更新按钮状态
        self.start_btn.setEnabled(False)
        self.stop_btn.setEnabled(True)
        
        # 在日志中记录开始
        self.log_output.append(f"开始执行循环操作，总共 {self.controller.total_cycles} 个循环")
        
        # 使用QTimer来避免阻塞UI线程
        self.execution_timer = QTimer()
        self.execution_timer.timeout.connect(lambda: self.run_cycle_in_thread(params))
        self.execution_timer.start(100)
        
    def run_cycle_in_thread(self, params):
        """在线程中运行循环（简化版，实际应用中应使用QThread）"""
        self.execution_timer.stop()
        
        # 在实际应用中，这里应该使用QThread来避免阻塞UI
        # 这里使用简单实现
        import threading
        thread = threading.Thread(target=self.controller.run_cycle, 
                                 args=(params, self.on_step_update))
        thread.daemon = True
        thread.start()
        
    def on_step_update(self, cycle, step):
        """步骤更新回调"""
        self.update_flowchart.emit(cycle, step)
        
        # 更新日志
        step_names = {
            1: "步骤1",
            2: "等待1", 
            3: "步骤2",
            4: "等待2",
            5: "循环判断"
        }
        
        if step > 0:
            self.log_output.append(f"循环 {cycle}: 执行{step_names[step]}")
        
        if cycle == self.controller.total_cycles and step == 5:
            self.log_output.append("所有循环执行完成！")
            self.start_btn.setEnabled(True)
            self.stop_btn.setEnabled(False)
        
    def stop_execution(self):
        """停止执行"""
        self.controller.is_running = False
        self.log_output.append("用户停止执行")
        self.start_btn.setEnabled(True)
        self.stop_btn.setEnabled(False)

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec_())
```

## 设计特点说明

### 1. **流程图可视化**
- 使用QGraphicsView框架实现流程图展示
- 不同形状表示不同类型节点：矩形（操作步骤）、菱形（判断）、椭圆（开始/结束）
- 实时高亮显示当前执行步骤

### 2. **参数输入界面**
- 循环次数控制
- 两个步骤的方向、行程、速度参数输入
- 两个等待时长的设置
- 分组布局，界面清晰

### 3. **执行控制**
- 开始/停止按钮控制循环执行
- 非阻塞执行，避免界面卡顿
- 实时日志输出执行状态

### 4. **架构设计**
- **OperationController**: 负责业务逻辑和循环控制
- **MainWindow**: 负责UI展示和用户交互
- **FlowChartItem**: 自定义流程图元素

## 扩展建议

1. **使用QThread改进**: 当前使用简单线程，建议改用PyQt5的QThread实现真正的多线程

2. **增强流程图**: 可以基于搜索结果中的Graphics View框架设计更专业的流程图组件

3. **参数验证**: 添加输入参数的有效性验证

4. **状态持久化**: 保存和加载常用参数配置

5. **进度显示**: 添加进度条显示总体执行进度

这个设计满足了在UI上展示循环流程并根据输入参数执行循环操作的需求，同时保证了良好的用户体验和可扩展性。