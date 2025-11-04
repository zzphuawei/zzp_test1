下面这个表格整理了 Qt Designer 中常用的控件，并附上了简要的功能说明和代码示例，希望能帮你快速上手。

| 控件类别 | 控件名称 (英文) | 主要功能 | 简单代码示例 (PyQt5) |
| :--- | :--- | :--- | :--- |
| **布局** | 垂直布局 (Vertical Layout) | 组件**垂直**排列。 | `layout = QVBoxLayout()` <br> `layout.addWidget(button)` |
| | 水平布局 (Horizontal Layout) | 组件**水平**排列。 | `layout = QHBoxLayout()` <br> `layout.addWidget(button)` |
| | 栅格布局 (Grid Layout) | 组件在**网格**中排列。 | `layout = QGridLayout()` <br> `layout.addWidget(button, 0, 0)` <br> `# 第0行第0列` |
| | 表单布局 (Form Layout) | 常以 **"标签: 输入框"** 成对布局。 | `layout = QFormLayout()` <br> `layout.addRow("姓名:", line_edit)` |
| **按钮** | 按压按钮 (Push Button) | **最常用**的按钮，如"确定"、"取消"。 | `button = QPushButton("点击我")` <br> `button.clicked.connect(self.on_button_click)` |
| | 单选框 (Radio Button) | **多选一**，同一组内互斥。 | `radio1 = QRadioButton("选项1")` <br> `radio2 = QRadioButton("选项2")` |
| | 多选框 (Check Box) | **可多选**，表示开关状态。 | `checkbox = QCheckBox("同意条款")` <br> `checkbox.stateChanged.connect(self.on_checkbox_change)` |
| **输入控件** | 单行文本 (Line Edit) | 输入**单行**文本。 | `line_edit = QLineEdit()` <br> `text = line_edit.text()` <br> `# 获取文本` |
| | 纯文本编辑 (Plain Text Edit) | 输入和显示**多行**纯文本。 | `text_edit = QPlainTextEdit()` <br> `text = text_edit.toPlainText()` |
| | 数字调节框 (Spin Box) | 通过按钮微调**整数**。 | `spinbox = QSpinBox()` <br> `spinbox.setValue(50)` |
| | 下拉框 (Combo Box) | **下拉列表**中选择一项。 | `combo = QComboBox()` <br> `combo.addItems(["选项A", "选项B"])` |
| | 水平滑块 (Horizontal Slider) | 通过**拖动滑块**输入。 | `slider = QSlider(Qt.Horizontal)` <br> `slider.valueChanged.connect(self.on_value_change)` |
| **显示控件** | 标签 (Label) | **显示文本**或图片。 | `label = QLabel("你好，世界!")` <br> `label.setPixmap(QPixmap("image.png"))` |
| | 文本浏览器 (Text Browser) | 显示**富文本**，支持超链接。 | `text_browser = QTextBrowser()` <br> `text_browser.setHtml("<b>加粗</b>文字")` |
| | 进度条 (Progress Bar) | **可视化**显示任务进度。 | `progress = QProgressBar()` <br> `progress.setValue(75)` <br> `# 进度75%` |
| | LCD 数字 (LCD Number) | 以 **LCD** 样式显示数字。 | `lcd = QLCDNumber()` <br> `lcd.display(123)` |
| **容器** | 分组框 (Group Box) | 提供**带标题的分组框**，逻辑上组合控件。 | `group_box = QGroupBox("分组")` <br> `group_layout = QVBoxLayout()` <br> `group_box.setLayout(group_layout)` |
| | 选项卡 (Tab Widget) | **多页签**容器，点击标签切换内容。 | `tab_widget = QTabWidget()` <br> `tab_widget.addTab(widget1, "标签1")` |
| | 滚动区域 (Scroll Area) | 为其他组件提供**滚动条**。 | `scroll_area = QScrollArea()` <br> `scroll_area.setWidget(large_widget)` |

### 🛠️ 一个简单的完整示例

这个例子演示了如何创建一个带有按钮和标签的窗口，点击按钮会更新标签文本。它结合使用了上面表格里的 `QPushButton` 和 `QLabel`。

```python
import sys
from PyQt5.QtWidgets import QApplication, QMainWindow, QPushButton, QLabel, QVBoxLayout, QWidget
from PyQt5.QtCore import Qt

# 创建一个主窗口类
class MainWindow(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        # 设置窗口
        self.setWindowTitle('Qt控件示例')
        self.setGeometry(300, 300, 280, 150) # (x, y, width, height)

        # 创建中央部件和布局
        central_widget = QWidget()
        self.setCentralWidget(central_widget)
        layout = QVBoxLayout(central_widget)

        # 创建一个标签 (QLabel)
        self.label = QLabel('初始文字')
        self.label.setAlignment(Qt.AlignCenter) # 文字居中
        layout.addWidget(self.label)

        # 创建一个按钮 (QPushButton)
        self.button = QPushButton('点我改变文字')
        self.button.clicked.connect(self.on_button_click) # 连接信号和槽
        layout.addWidget(self.button)

    # 自定义的槽函数
    def on_button_click(self):
        self.label.setText('你好，PyQt5!')

# 程序入口
if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = MainWindow()
    window.show()
    sys.exit(app.exec_())
```

**代码说明**：
- 我们创建了一个 `MainWindow` 类，它继承自 `QMainWindow`。
- 在 `initUI` 方法中，我们设置了窗口标题、大小，并创建了一个垂直布局。
- 我们实例化了 `QLabel` 和 `QPushButton`，并将它们添加到布局中。
- **关键一步**：使用 `button.clicked.connect(self.on_button_click)` 将按钮的 `clicked` 信号连接到自定义的 `on_button_click` 槽函数。这样，当按钮被点击时，就会调用这个函数来改变标签的文本。

### 🔧 Qt Designer 基础工作流程

对于桌面GUI开发，Qt Designer能帮你通过拖拽快速设计界面。

1.  **打开 Qt Designer**：通常可以在PyCharm的`External Tools`中找到，或通过命令行输入`designer`启动。
2.  **选择模板**：初次打开，通常会选择 **"Main Window"**（带菜单栏、状态栏的主窗口）或 **"Widget"**（通用窗口）。
3.  **设计界面**：从左侧的 **"Widget Box"** (窗口部件盒) 中，将需要的控件拖拽到中间的窗体上。
4.  **设置属性**：在右下方的 **"Property Editor"** (属性编辑器) 中，可以修改选中控件的各种属性，如对象名称(`objectName`)、文本(`text`)、大小等。**注意**：在代码中访问控件主要通过其`objectName`。
5.  **布局管理**：使用布局控件（如垂直、水平布局）来排列和管理控件，确保窗口缩放时界面能自适应。
6.  **预览**：使用 `Ctrl + R` 快捷键预览设计好的窗口效果。
7.  **保存文件**：将设计保存为 `.ui` 文件。
8.  **转换 `.ui` 为 `.py`**：使用命令行工具 `pyuic5` 将XML格式的 `.ui` 文件转换为Python代码。
    ```bash
    pyuic5 your_design.ui -o ui_mainwindow.py
    ```
9.  **在主程序中使用**：在主要的Python代码中导入生成的UI类，并设置到主窗口上。
    ```python
    import sys
    from PyQt5.QtWidgets import QApplication, QMainWindow
    from ui_mainwindow import Ui_MainWindow # 导入生成的UI类

    class MyMainWindow(QMainWindow, Ui_MainWindow): # 多重继承
        def __init__(self):
            super().__init__()
            self.setupUi(self) # 调用生成的UI类中的方法设置界面
            # 在这里添加你的自定义逻辑，比如信号槽连接

    if __name__ == '__main__':
        app = QApplication(sys.argv)
        window = MyMainWindow()
        window.show()
        sys.exit(app.exec_())
    ```

### 💡 学习建议

- **从模仿开始**：先尝试用Qt Designer复现一些常见的简单界面（如计算器、设置对话框）。
- **善用属性编辑器**：多尝试修改控件的不同属性，直观感受它们的效果。
- **掌握布局**：这是构建整洁、自适应界面的关键。
- **理解信号与槽**：这是Qt编程的核心机制，务必掌握如何连接信号与槽来实现交互。

希望这份详解和示例能帮助你顺利开启PyQt5和Qt Designer的图形界面开发之旅。如果在学习过程中遇到具体问题，比如某个控件的特定属性如何使用，欢迎随时提问。