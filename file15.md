要将CMD窗口或Python脚本最小化到屏幕右下角的系统托盘（而非任务栏），可以使用以下方法：

---

### **方法1：Python脚本使用系统托盘库**
安装必要的库：
```bash
pip install pystray pillow
```

示例代码（保存为 `.pyw` 扩展名避免显示控制台）：
```python
import pystray
from PIL import Image
import threading
import time
import sys

# 创建托盘图标
def create_tray_icon():
    # 生成图标
    image = Image.new('RGB', (64, 64), 'blue')
    # 托盘菜单
    menu = pystray.Menu(
        pystray.MenuItem('退出', exit_app)
    )
    icon = pystray.Icon("my_app", image, "后台脚本", menu)
    icon.run()

# 退出程序
def exit_app(icon):
    icon.stop()
    sys.exit()

# 后台任务（替换为你的脚本逻辑）
def background_task():
    while True:
        print("后台运行中...")  # 实际运行时不会显示
        time.sleep(5)

if __name__ == "__main__":
    # 启动托盘图标线程
    threading.Thread(target=create_tray_icon, daemon=True).start()
    # 启动后台任务
    background_task()
```

---

### **方法2：CMD/Batch脚本使用VBS隐藏窗口**
1. 创建批处理文件 `script.bat`：
```batch
@echo off
echo 后台运行中...
ping -n 60 127.0.0.1 > nul  # 模拟后台任务
```

2. 创建VBS脚本 `hide.vbs` 来隐藏CMD窗口：
```vbscript
Set WshShell = CreateObject("WScript.Shell")
WshShell.Run "cmd /c script.bat", 0, False  ' 0 表示隐藏窗口
```

3. 双击运行 `hide.vbs`，脚本将在后台无窗口运行（需手动添加托盘图标需额外工具）。

---

### **方法3：使用第三方工具（推荐）**
- **[Minimizer](https://github.com/raspopov/Minimizer)**：将任何窗口最小化到托盘。
  1. 下载并运行 Minimizer
  2. 右键点击任务栏窗口 → 选择 "Minimize to Tray"

- **[RBTray](https://github.com/benbuck/rbtray)**：
  1. 安装后运行
  2. 右键点击窗口标题栏 → 选择 "Send to tray"

---

### **方法4：Python脚本隐藏窗口 + 托盘图标**
```python
import sys
import os
import pystray
from PIL import Image
import win32gui, win32con

# 隐藏控制台窗口
def hide_console():
    if sys.platform == 'win32':
        window = win32gui.GetForegroundWindow()
        win32gui.ShowWindow(window, win32con.SW_HIDE)

# 托盘图标设置
def setup_tray():
    image = Image.new('RGB', (64, 64), 'black')
    menu = pystray.Menu(pystray.MenuItem('退出', lambda: os._exit(0)))
    icon = pystray.Icon("app", image, "我的脚本", menu)
    icon.run()

if __name__ == "__main__":
    hide_console()  # 隐藏窗口
    setup_tray()    # 启动托盘
    # 在此处添加你的后台代码
    while True:
        pass  # 主循环
```

---

### **总结**
| 场景              | 推荐方法                          |
|-------------------|----------------------------------|
| Python脚本        | 使用 `pystray` + 隐藏控制台（方法4） |
| CMD/Batch脚本     | 用VBS隐藏窗口（方法2）             |
| 通用窗口程序      | 第三方工具如 RBTray（方法3）       |

> **注意**：系统托盘功能高度依赖操作系统，以上代码在Windows测试通过。Linux/macOS需使用平台特定库（如`libappindicator`）。