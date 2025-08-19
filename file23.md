# PyStray 动态刷新 MenuItem 的 text 参数

在 PyStray 中，要实现动态刷新 MenuItem 的 text 参数，你需要使用回调函数来返回菜单项的文本。这样每次菜单显示时都会调用该函数来获取最新的文本内容。

下面是一个完整的示例，展示如何创建系统托盘图标并动态更新菜单项的文本：

```python
import pystray
from pystray import MenuItem, Menu
from PIL import Image, ImageDraw
import threading
import time

# 创建一个变量用于存储动态文本
dynamic_text = "初始文本"
counter = 0

# 创建托盘图标图像
def create_image():
    # 生成一个简单的图像
    image = Image.new('RGB', (64, 64), color='white')
    dc = ImageDraw.Draw(image)
    dc.rectangle((0, 0, 64, 64), fill='blue')
    dc.text((10, 25), "Tray", fill='white')
    return image

# 动态文本的获取函数
def get_dynamic_text(item):
    return f"计数: {counter}"

# 更新动态文本的函数
def update_dynamic_text():
    global counter
    while True:
        time.sleep(1)  # 每秒更新一次
        counter += 1
        # 通知托盘图标更新菜单
        icon.update_menu()

# 菜单项点击处理函数
def on_clicked(icon, item):
    global dynamic_text
    if str(item) == "静态菜单项":
        print("静态菜单项被点击")
    elif str(item).startswith("计数:"):
        print("动态菜单项被点击")

# 创建菜单
menu = Menu(
    MenuItem(lambda text: get_dynamic_text(text), on_clicked),
    MenuItem("静态菜单项", on_clicked),
    MenuItem("退出", lambda icon, item: icon.stop())
)

# 创建系统托盘图标
icon = pystray.Icon("test_icon", create_image(), "动态菜单示例", menu)

# 启动一个线程来定期更新计数器
update_thread = threading.Thread(target=update_dynamic_text)
update_thread.daemon = True
update_thread.start()

# 运行图标
icon.run()
```

## 关键点说明：

1. **动态文本函数**：`get_dynamic_text` 函数返回菜单项的最新文本内容
2. **Lambda 表达式**：使用 `lambda text: get_dynamic_text(text)` 作为菜单项的文本参数
3. **定期更新**：通过后台线程定期更新计数器变量
4. **菜单刷新**：调用 `icon.update_menu()` 强制刷新菜单显示

## 更复杂的示例

如果你需要更复杂的动态菜单，可以这样实现：

```python
import pystray
from pystray import MenuItem, Menu
from PIL import Image, ImageDraw
import threading
import time

# 存储动态数据
class AppState:
    def __init__(self):
        self.counter = 0
        self.status = "运行中"
    
    def get_status_text(self, item):
        return f"状态: {self.status}"
    
    def get_counter_text(self, item):
        return f"计数器: {self.counter}"
    
    def increment_counter(self, icon, item):
        self.counter += 1
        icon.update_menu()
    
    def change_status(self, icon, item):
        self.status = "暂停" if self.status == "运行中" else "运行中"
        icon.update_menu()

# 创建托盘图标图像
def create_image():
    image = Image.new('RGB', (64, 64), color='white')
    dc = ImageDraw.Draw(image)
    dc.rectangle((0, 0, 64, 64), fill='green')
    dc.text((10, 25), "App", fill='white')
    return image

# 创建应用状态实例
app_state = AppState()

# 创建菜单
menu = Menu(
    MenuItem(lambda text: app_state.get_status_text(text), 
             lambda icon, item: app_state.change_status(icon, item)),
    MenuItem(lambda text: app_state.get_counter_text(text), 
             lambda icon, item: app_state.increment_counter(icon, item)),
    MenuItem("退出", lambda icon, item: icon.stop())
)

# 创建系统托盘图标
icon = pystray.Icon("dynamic_icon", create_image(), "动态菜单高级示例", menu)

# 运行图标
icon.run()
```

这种方法允许你创建完全动态的菜单系统，菜单项可以根据应用程序的当前状态实时更新。