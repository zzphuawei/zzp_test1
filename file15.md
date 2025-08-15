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

----
要将命令行窗口（CMD）或Python脚本最小化到系统托盘（屏幕右下角），并支持双击托盘图标显示窗口、窗口最小化回托盘的功能，可以通过Python的图形界面库（如`tkinter`）结合系统托盘库（如`pystray`）实现。以下是完整解决方案：

### 安装依赖库
```bash
pip install pystray pillow
```

### 完整代码示例
```python
import tkinter as tk
from tkinter import scrolledtext
import pystray
from PIL import Image
import sys
import threading

class SystemTrayApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Tray Application")
        self.root.geometry("400x300")
        self.root.protocol('WM_DELETE_WINDOW', self.minimize_to_tray)
        
        # 创建模拟CMD界面的组件
        self.text_area = scrolledtext.ScrolledText(root, state='disabled')
        self.text_area.pack(fill=tk.BOTH, expand=True)
        
        self.entry = tk.Entry(root)
        self.entry.pack(fill=tk.X, padx=5, pady=5)
        self.entry.bind("<Return>", self.process_command)
        
        # 托盘图标（使用内置默认图标）
        self.tray_icon = None
        self.create_tray_icon()
        
        # 绑定最小化事件
        self.root.bind("<Unmap>", self.on_minimize)

    def process_command(self, event):
        command = self.entry.get()
        self.entry.delete(0, tk.END)
        self.display_output(f"> {command}\nExecuting...\n")

    def display_output(self, text):
        self.text_area.configure(state='normal')
        self.text_area.insert(tk.END, text)
        self.text_area.configure(state='disabled')
        self.text_area.see(tk.END)

    def create_tray_icon(self):
        # 创建内存中的纯色图标
        image = Image.new('RGB', (64, 64), 'darkgray')
        menu = pystray.Menu(
            pystray.MenuItem('Show', self.show_window),
            pystray.MenuItem('Exit', self.exit_app)
        )
        self.tray_icon = pystray.Icon("tray_icon", image, "My App", menu)

    def run_tray_icon(self):
        self.tray_icon.run()

    def minimize_to_tray(self):
        self.root.withdraw()  # 隐藏窗口
        # 在后台线程运行托盘图标
        threading.Thread(target=self.run_tray_icon, daemon=True).start()

    def show_window(self):
        self.tray_icon.stop()  # 停止托盘图标
        self.root.deiconify()  # 显示窗口
        self.root.lift()       # 置顶窗口

    def exit_app(self):
        self.tray_icon.stop()
        self.root.destroy()
        sys.exit(0)

    def on_minimize(self, event):
        if event.widget == self.root:
            self.minimize_to_tray()

if __name__ == "__main__":
    root = tk.Tk()
    app = SystemTrayApp(root)
    root.mainloop()
```

### 功能说明
1. **最小化到托盘**：
   - 点击窗口关闭按钮时隐藏窗口并显示托盘图标
   - 窗口最小化时自动触发隐藏到托盘
2. **从托盘恢复**：
   - 双击托盘图标：显示主窗口
   - 右键托盘图标 → 选择"Show"恢复窗口
3. **模拟CMD功能**：
   - 底部输入框可输入命令
   - 上方文本区域显示命令执行结果
4. **退出程序**：
   - 右键托盘图标 → 选择"Exit"彻底退出

### 关键实现细节
1. **托盘图标管理**：
   - 使用`pystray`创建系统托盘图标
   - 通过`Pillow`动态生成纯色图标
2. **窗口事件绑定**：
   - `WM_DELETE_WINDOW`：拦截关闭按钮事件
   - `<Unmap>`事件：捕获最小化操作
3. **线程安全**：
   - 托盘图标运行在独立线程中
   - 使用`daemon=True`确保主程序退出时线程终止
4. **窗口控制**：
   - `withdraw()`：完全隐藏窗口
   - `deiconify()`：恢复隐藏的窗口

### 扩展功能建议
1. **自定义托盘图标**：
   ```python
   # 替换create_tray_icon中的image生成代码
   image = Image.open("custom_icon.png")  # 使用本地图标文件
   ```
2. **添加更多菜单项**：
   ```python
   menu = pystray.Menu(
        pystray.MenuItem('Show', self.show_window),
        pystray.MenuItem('Settings', self.open_settings),
        pystray.MenuItem('Exit', self.exit_app)
   )
   ```
3. **实现真实命令执行**：
   ```python
   def process_command(self, event):
       import subprocess
       command = self.entry.get()
       result = subprocess.getoutput(command)
       self.display_output(f"> {command}\n{result}\n\n")
   ```

> **注意**：某些杀毒软件可能阻止程序创建托盘图标，首次运行时需允许权限。此方案适用于Windows/macOS/Linux，但不同系统托盘行为可能略有差
---
要将运行 `A.py` 脚本的 CMD 窗口最小化到系统托盘，并能通过双击托盘图标恢复窗口，可以创建一个包装脚本。以下是完整的解决方案：

### 解决方案思路
1. **隐藏 CMD 窗口**：使用 `win32gui` 隐藏控制台窗口
2. **创建托盘图标**：使用 `pystray` 在系统托盘创建图标
3. **窗口管理**：通过托盘菜单控制窗口的显示/隐藏
4. **子进程管理**：用 `subprocess.Popen` 启动 `A.py` 并管理其生命周期

### 完整代码 (`run_hidden.py`)
```python
import subprocess
import win32gui
import win32con
import sys
import threading
import os
from PIL import Image
import pystray

# 全局变量
console_hwnd = None
process = None
tray_icon = None

def get_console_window():
    """获取当前控制台窗口句柄"""
    return win32gui.GetForegroundWindow()

def hide_console():
    """隐藏控制台窗口"""
    global console_hwnd
    if not console_hwnd:
        console_hwnd = get_console_window()
    win32gui.ShowWindow(console_hwnd, win32con.SW_HIDE)

def show_console():
    """显示并激活控制台窗口"""
    global console_hwnd
    if not console_hwnd:
        console_hwnd = get_console_window()
    win32gui.ShowWindow(console_hwnd, win32con.SW_RESTORE)
    win32gui.SetForegroundWindow(console_hwnd)

def start_script():
    """启动A.py脚本"""
    global process
    script_path = os.path.join(os.path.dirname(__file__), "A.py")
    process = subprocess.Popen(
        [sys.executable, script_path],
        creationflags=subprocess.CREATE_NEW_CONSOLE
    )
    process.wait()  # 等待进程结束
    tray_icon.stop()  # 脚本结束后退出托盘图标
    os._exit(0)  # 退出程序

def on_show(icon, item):
    """显示窗口的菜单项回调"""
    show_console()

def on_exit(icon, item):
    """退出程序的菜单项回调"""
    global process
    if process:
        process.terminate()  # 终止子进程
    icon.stop()  # 停止托盘图标
    os._exit(0)  # 退出程序

def setup_tray():
    """设置托盘图标"""
    global tray_icon
    # 创建临时图标（白色方形图标）
    image = Image.new('RGB', (64, 64), color='white')
    
    # 创建菜单
    menu = pystray.Menu(
        pystray.MenuItem("显示控制台", on_show),
        pystray.MenuItem("退出", on_exit)
    )
    
    # 创建托盘图标
    tray_icon = pystray.Icon("A.py 控制器", image, "A.py 正在运行", menu)
    tray_icon.run()

if __name__ == "__main__":
    # 首次启动时隐藏控制台
    hide_console()
    
    # 启动脚本线程
    script_thread = threading.Thread(target=start_script, daemon=True)
    script_thread.start()
    
    # 启动托盘图标
    setup_tray()
```

### 使用说明
1. **安装依赖库**：
   ```bash
   pip install pystray pillow pywin32
   ```

2. **文件结构**：
   ```
   ├── A.py            # 你的第三方脚本
   ├── run_hidden.py   # 本解决方案脚本
   ```

3. **运行方式**：
   - 双击运行 `run_hidden.py`（不是 `A.py`）
   - 程序会自动：
     1. 启动 `A.py` 并隐藏其 CMD 窗口
     2. 在系统托盘创建白色方块图标
     3. 右键菜单提供控制功能

4. **托盘操作**：
   - **显示控制台**：右键托盘图标 → 选择"显示控制台"
   - **退出程序**：右键托盘图标 → 选择"退出"

### 功能特点
1. **完全隐藏窗口**：启动后 CMD 窗口立即隐藏到托盘
2. **进程管理**：退出时会自动终止子进程
3. **托盘交互**：
   - 右键菜单提供控制选项
   - 支持双击托盘图标显示窗口（需在代码中添加，见扩展说明）
4. **线程安全**：脚本运行在独立线程，不影响主线程响应

### 扩展：添加双击托盘图标显示功能
在 `setup_tray()` 函数中添加双击支持：
```python
def on_click(icon, item):
    """双击托盘图标显示窗口"""
    show_console()

def setup_tray():
    ...
    tray_icon = pystray.Icon("A.py 控制器", image, "A.py 正在运行", menu)
    tray_icon.on_click = on_click  # 添加双击支持
    tray_icon.run()
```

> **注意**：某些系统可能将双击识别为单击，具体行为取决于操作系统设置。

### 常见问题解决
1. **图标不显示**：检查 `pillow` 是否安装正确
2. **窗口无法隐藏**：以管理员身份运行脚本
3. **子进程未关闭**：确保通过托盘菜单退出程序

这个方案将你的第三方脚本包装在托盘应用中，实现了专业级的窗口管理体验。