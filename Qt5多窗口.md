在Python的PyQt5中实现多窗口功能，主要通过创建多个窗口实例，并使用`show()`和`hide()`方法，或借助`QStackedWidget`等容器来管理。**信号与槽机制**是实现窗口间数据传递和交互的强大工具。

下面这个表格概括了几种核心的实现方式及其典型应用场景，你可以快速了解：

| 实现方式 | 核心方法 | 优点 | 适用场景 |
| :--- | :--- | :--- | :--- |
| **独立窗口切换** | 在A窗口按钮点击事件中，实例化并显示B窗口，同时隐藏A窗口。 | 逻辑简单直观，窗口关系对等。 | 简单的窗口循环切换，如欢迎界面和主界面。 |
| **主窗口弹对话框** | 主窗口使用`QDialog`并调用`exec_()`方法弹出模式对话框。 | 强制用户完成对话框操作，可直接通过对话框属性或静态方法返回值。 | 获取用户输入（如设置、选择日期）。 |
| **堆叠窗口(QStackedWidget)** | 将多个窗口（`QWidget`）添加到一个`QStackedWidget`中，通过索引切换。 | 多个"页面"共享同一块显示区域，布局紧凑。 | 设置向导、功能模块集中的主界面（如左侧树形菜单+右侧内容区）。 |

### 🖼️ 多窗口的实现与切换

根据上表的分类，以下是几种常见场景的具体实现方法：

1. **实现独立的窗口切换**
    这是最基本的形式。通常在每个窗口的按钮点击事件中，创建并显示下一个窗口，同时隐藏当前窗口。
    ```python
    import sys
    from PyQt5.QtWidgets import QApplication, QMainWindow, QPushButton

    class Window1(QMainWindow):
        def __init__(self):
            super().__init__()
            self.setWindowTitle("窗口1")
            self.btn = QPushButton("打开窗口2", self)
            self.btn.clicked.connect(self.open_window2)
            
        def open_window2(self):
            self.window2 = Window2() # 创建窗口2的实例
            self.window2.show()     # 显示窗口2
            self.hide()             # 隐藏当前窗口（窗口1）

    class Window2(QMainWindow):
        def __init__(self):
            super().__init__()
            self.setWindowTitle("窗口2")
            self.btn = QPushButton("打开窗口1", self)
            self.btn.clicked.connect(self.open_window1)
            
        def open_window1(self):
            self.window1 = Window1() # 创建窗口1的实例
            self.window1.show()     # 显示窗口1
            self.hide()             # 隐藏当前窗口（窗口2）

    if __name__ == '__main__':
        app = QApplication(sys.argv)
        first_window = Window1()
        first_window.show()
        sys.exit(app.exec_())
    ```

2. **主窗口弹出对话框（子窗口）**
    在这种模式下，弹出的对话框（通常是`QDialog`）会中断用户操作，用户必须处理完对话框后才能返回主窗口。
    ```python
    from PyQt5.QtWidgets import QDialog, QVBoxLayout, QDateTimeEdit, QDialogButtonBox

    class SettingsDialog(QDialog):
        def __init__(self, parent=None):
            super().__init__(parent)
            self.setWindowTitle("设置")
            layout = QVBoxLayout(self)
            
            self.datetime_edit = QDateTimeEdit(self)
            layout.addWidget(self.datetime_edit)
            
            buttons = QDialogButtonBox(QDialogButtonBox.Ok | QDialogButtonBox.Cancel, self)
            buttons.accepted.connect(self.accept)
            buttons.rejected.connect(self.reject)
            layout.addWidget(buttons)
            
        def get_date_time(self):
            return self.datetime_edit.dateTime()
            
    # 在主窗口中的某个方法里调用
    def open_settings(self):
        dialog = SettingsDialog(self)
        if dialog.exec_() == QDialog.Accepted: # 模式方式显示对话框
            selected_datetime = dialog.get_date_time() # 获取对话框中的数据
            print(selected_datetime)
        # 只有当对话框关闭后，才会继续执行后面的代码
    ```

3. **使用堆叠窗口（QStackedWidget）**
    `QStackedWidget`就像一个相册，一次只显示一张照片，但所有照片都已准备就绪。它非常适合制作标签页、向导或根据左侧菜单切换右侧内容的应用。
    ```python
    from PyQt5.QtWidgets import QStackedWidget, QListWidget, QHBoxLayout, QWidget, QLabel

    class MainApp(QWidget):
        def __init__(self):
            super().__init__()
            
            # 创建布局和左侧列表
            layout = QHBoxLayout(self)
            self.list_widget = QListWidget()
            self.list_widget.addItems(["首页", "设置", "关于"])
            layout.addWidget(self.list_widget)
            
            # 创建堆叠窗口并添加多个页面
            self.stacked_widget = QStackedWidget()
            self.stacked_widget.addWidget(QLabel("这是首页内容")) # 页面0
            self.stacked_widget.addWidget(QLabel("这是设置页面")) # 页面1
            self.stacked_widget.addWidget(QLabel("这是关于页面")) # 页面2
            layout.addWidget(self.stacked_widget)
            
            # 连接列表点击事件到切换页面的函数
            self.list_widget.currentRowChanged.connect(self.stacked_widget.setCurrentIndex)
            
    if __name__ == '__main__':
        app = QApplication(sys.argv)
        window = MainApp()
        window.show()
        sys.exit(app.exec_())
    ```

### 📡 窗口间的数据传递

窗口之间经常需要传递数据，以下是两种常见的方法：

1. **通过属性或方法直接传递**
    这是最直接的方式，主窗口从子窗口的公开属性或方法中获取数据。
    ```python
    # 在对话框（子窗口）中定义一个方法用于获取数据
    class SettingsDialog(QDialog):
        # ... __init__ 等代码同上 ...
        def get_settings(self):
            # 返回用户设置的数据
            return {
                "datetime": self.datetime_edit.dateTime(),
                # ... 其他设置 ...
            }
            
    # 主窗口中调用
    def open_settings(self):
        dialog = SettingsDialog(self)
        if dialog.exec_() == QDialog.Accepted:
            settings = dialog.get_settings() # 调用方法获取数据
            self.apply_settings(settings)
    ```

2. **使用信号与槽（Signal & Slot）传递**
    这是PyQt中更强大和灵活的机制，特别适用于非模态窗口或需要实时通信的场景。子窗口可以定义自定义信号，在数据准备好时发射，主窗口通过槽函数接收。
    ```python
    from PyQt5.QtCore import pyqtSignal
    
    class SettingsDialog(QDialog):
        # 定义一个信号，参数类型为str
        settings_changed = pyqtSignal(str)
        
        def __init__(self):
            super().__init__()
            # ... 其他初始化代码 ...
            self.ok_button.clicked.connect(self.on_ok_clicked)
            
        def on_ok_clicked(self):
            user_input = self.some_input_widget.text()
            self.settings_changed.emit(user_input) # 发射信号，携带数据
            self.close()
            
    class MainWindow(QMainWindow):
        def __init__(self):
            super().__init__()
            self.settings_dialog = SettingsDialog()
            # 将子窗口的信号连接到主窗口的槽函数
            self.settings_dialog.settings_changed.connect(self.handle_new_settings)
            
        def open_settings(self):
            self.settings_dialog.show() # 非模态显示
            
        # 这个槽函数会在子窗口发射信号时自动调用
        def handle_new_settings(self, new_settings):
            print(f"收到新设置：{new_settings}")
            # 更新主窗口状态...
    ```

### 💡 实用技巧与注意事项

- **窗口类型选择**：主窗口通常使用`QMainWindow`，它提供了菜单栏、状态栏等标准组件。对话框或子窗口常用`QDialog`。简单的自定义内容区域可以使用`QWidget`。
- **避免窗口"一闪而过"**：确保你的窗口实例被长期引用（例如设置为`self.window`或类的属性），而不是局部变量，否则它可能会被Python的垃圾回收机制立即销毁。
- **模态与非模态**：
    - **模态对话框**：使用`exec_()`，会阻塞主窗口操作，直到对话框关闭。
    - **非模态窗口**：使用`show()`，用户可以同时与主窗口和该窗口交互。
- **内存管理**：在循环切换窗口时，注意使用`hide()`而非`close()`，或者重写`closeEvent`，以避免意外销毁窗口导致数据丢失。

### ❓ 常见问题

- **问题：点击按钮后，新窗口一闪而过，或者程序崩溃。**
    - **排查**：最常见的原因是窗口实例被创建为了局部变量，导致其生命周期过短。请将新窗口赋值给一个持久的属性，如`self.new_window = NewWindow()`。
- **问题：如何在关闭子窗口时，将数据传回主窗口？**
    - **方案1**：如果使用`exec_()`弹出的模式对话框，可以在对话框被接受后，通过其属性或方法（如`get_dateTime()`）获取数据。
    - **方案2**：使用**信号与槽**机制。这是更解耦和灵活的方式，尤其适合非模态窗口。

希望这份指南能帮助你顺利地在PyQt5中实现多窗口应用！多窗口设计的核心在于理清窗口之间的关系（是平级切换还是主从弹窗）和数据流，选择恰当的方法会让开发事半功倍。