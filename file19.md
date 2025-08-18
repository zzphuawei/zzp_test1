在Windows系统中，任务栏托盘图标的隐藏行为由系统设置控制。要使某个托盘图标始终显示（而非隐藏），可以通过修改注册表或使用Windows API实现。以下是两种方法的Python实现：

### 方法1：修改注册表（推荐）
此方法通过修改注册表强制图标始终显示。需要知道应用程序的**进程名**或**窗口类名**。

```python
import winreg

def set_tray_icon_visible(app_name, always_show=True):
    """
    设置指定应用的托盘图标在任务栏始终显示
    :param app_name: 应用程序的进程名（如notepad.exe）或窗口类名
    :param always_show: True表示始终显示，False表示恢复默认
    """
    # 注册表路径
    key_path = r"Software\Microsoft\Windows\CurrentVersion\Explorer\TrayNotify"
    
    try:
        # 打开注册表键
        with winreg.OpenKey(winreg.HKEY_CURRENT_USER, key_path, 0, winreg.KEY_READ | winreg.KEY_WRITE) as key:
            # 读取二进制值
            icon_streams, _ = winreg.QueryValueEx(key, "IconStreams")
            past_icon_streams, _ = winreg.QueryValueEx(key, "PastIconsStream")
            
            # 修改二进制数据（核心操作）
            # 注意：这里需要根据实际结构解析二进制数据，以下为示意代码
            modified_icon_streams = modify_binary_data(icon_streams, app_name, always_show)
            modified_past_icon_streams = modify_binary_data(past_icon_streams, app_name, always_show)
            
            # 写回注册表
            winreg.SetValueEx(key, "IconStreams", 0, winreg.REG_BINARY, modified_icon_streams)
            winreg.SetValueEx(key, "PastIconsStream", 0, winreg.REG_BINARY, modified_past_icon_streams)
            
        print("修改成功！请重启资源管理器或注销后生效")
        return True
    except Exception as e:
        print(f"操作失败: {str(e)}")
        return False

def modify_binary_data(data, app_name, always_show):
    """
    修改二进制数据（此处需要逆向工程知识）
    实际应用中需根据Windows版本解析数据结构
    """
    # 伪代码：此处应包含解析二进制数据的逻辑
    # 1. 在data中搜索app_name的标识
    # 2. 修改对应的标志位（通常是一个字节）
    # 3. 返回修改后的二进制数据
    # 注意：不同Windows版本数据结构不同，需自行逆向分析
    return data  # 此处返回原始数据示意

# 使用示例
set_tray_icon_visible("your_app.exe")
```

> **重要警告**：  
> - 直接修改二进制注册表值风险极高，可能导致系统不稳定  
> - 需自行逆向分析`IconStreams`数据结构（不同Windows版本不同）  
> - 更安全的方法见方法2

### 方法2：使用Windows设置API（更安全）
通过系统提供的`IShellDispatch`接口操作任务栏设置：

```python
import pythoncom
import win32com.client

def set_tray_visibility(app_name, visible=True):
    """
    通过Windows Shell API设置图标可见性
    :param app_name: 应用程序名称（如"Telegram"）
    :param visible: True为始终显示，False为仅通知时显示
    """
    try:
        # 初始化COM
        pythoncom.CoInitialize()
        
        # 创建Shell对象
        shell = win32com.client.Dispatch("Shell.Application")
        tray_settings = shell.Namespace(0x27)  # 0x27是系统托盘的特殊文件夹
        
        # 遍历系统托盘图标
        for item in tray_settings.Items():
            if app_name.lower() in item.Name.lower():
                # 设置可见性属性
                item.InvokeVerb("Properties")
                # 此处需要模拟UI操作（见下方说明）
                break
    except Exception as e:
        print(f"操作失败: {str(e)}")
    finally:
        pythoncom.CoUninitialize()

# 使用示例
set_tray_visibility("Telegram")
```

> **注意**：  
> 此方法只能打开设置对话框，需结合UI自动化完成勾选操作（如使用`pywinauto`）

### 完整解决方案（UI自动化）
结合方法2和`pywinauto`实现全自动设置：

```python
import time
import pythoncom
import win32com.client
from pywinauto import Application

def force_tray_icon_visible(app_name):
    """全自动设置托盘图标始终显示"""
    try:
        # 打开设置对话框
        pythoncom.CoInitialize()
        shell = win32com.client.Dispatch("Shell.Application")
        tray = shell.Namespace(0x27)
        for item in tray.Items():
            if app_name.lower() in item.Name.lower():
                item.InvokeVerb("Properties")
                time.sleep(1)  # 等待对话框弹出
                
                # 使用pywinauto操作对话框
                app = Application().connect(title="属性")
                dlg = app.window(title="属性")
                
                # 勾选"始终在任务栏显示所有图标"
                if dlg["始终在任务栏上显示所有图标和通知(R)"].exists():
                    dlg["始终在任务栏上显示所有图标和通知(R)"].check()
                
                # 或者取消勾选特定应用的隐藏
                elif dlg[app_name].exists():
                    dlg[app_name].uncheck()  # 取消勾选隐藏
                
                dlg["确定"].click()
                print(f"{app_name} 设置成功！")
                return True
        print("未找到指定应用")
        return False
    except Exception as e:
        print(f"操作失败: {e}")
        return False
    finally:
        pythoncom.CoUninitialize()

# 使用示例
force_tray_icon_visible("Telegram")
```

### 注意事项
1. **管理员权限**：需要以管理员身份运行脚本
2. **应用名称**：使用任务管理器中的精确进程名
3. **系统兼容性**：
   - 方法1适用于Windows 7/10/11
   - 方法2/3适用于Windows 10+
4. **重启生效**：修改后可能需要重启资源管理器：
   ```python
   import os
   os.system("taskkill /f /im explorer.exe")
   os.system("start explorer.exe")
   ```

### 备选方案
如果上述方法复杂，可直接调用系统设置：
```python
import os
os.system("start ms-settings:notifications")  # 打开通知设置页
```

> **建议**：优先考虑方法3（UI自动化），它更安全可靠且避免直接操作注册表的风险。实际使用时请根据目标应用的名称调整参数。

---
# 在创建托盘图标时指定始终显示（不隐藏）

在Python中使用pystray库时，它本身并没有提供直接在创建图标时指定"始终显示"（不隐藏）的功能。这是因为托盘图标的隐藏行为是由操作系统（如Windows）的任务栏设置控制的，而不是由应用程序代码直接控制的。

不过，你可以通过以下方法实现在创建托盘图标后将其设置为始终显示：

## 解决方案

```python
import pystray
import os
import time
import threading
from PIL import Image
import win32gui
import win32con
import win32api
import ctypes
from ctypes import wintypes

# 创建一个简单的图标
def create_image():
    # 生成一个简单的红色图标
    image = Image.new('RGB', (64, 64), color='red')
    return image

# 获取窗口句柄的函数
def get_tray_icon_hwnd(process_id):
    """获取指定进程ID的托盘图标窗口句柄"""
    hwnd_list = []
    
    def enum_windows_callback(hwnd, _):
        # 检查窗口是否可见
        if win32gui.IsWindowVisible(hwnd):
            # 获取窗口的进程ID
            _, pid = win32process.GetWindowThreadProcessId(hwnd)
            if pid == process_id:
                # 检查是否是托盘图标窗口
                class_name = win32gui.GetClassName(hwnd)
                if class_name == "SysPager" or class_name == "ToolbarWindow32":
                    hwnd_list.append(hwnd)
    
    win32gui.EnumWindows(enum_windows_callback, None)
    return hwnd_list

# 设置托盘图标始终显示的函数
def set_tray_icon_always_show(process_id, icon_title):
    """设置托盘图标始终显示"""
    try:
        # 等待图标出现
        time.sleep(2)
        
        # 获取托盘图标句柄
        hwnd_list = get_tray_icon_hwnd(process_id)
        
        if not hwnd_list:
            print("未能找到托盘图标窗口")
            return
            
        # 使用Shell API设置图标可见性
        shell = win32com.client.Dispatch("Shell.Application")
        tray = shell.Namespace(0x27)  # 系统托盘的特殊文件夹
        
        # 查找我们的图标
        found = False
        for i in range(tray.Items().Count):
            item = tray.Items().Item(i)
            if icon_title in item.Name:
                # 打开属性对话框
                item.InvokeVerb("Properties")
                time.sleep(1)
                
                # 查找属性窗口
                hwnd = win32gui.FindWindow(None, "属性")
                if hwnd:
                    # 找到"始终显示"复选框
                    hwnd_checkbox = win32gui.FindWindowEx(hwnd, 0, "Button", "始终在任务栏上显示所有图标和通知(&R)")
                    if not hwnd_checkbox:
                        # 尝试其他版本的标题
                        hwnd_checkbox = win32gui.FindWindowEx(hwnd, 0, "Button", "始终在任务栏上显示所有图标和通知(R)")
                    
                    if hwnd_checkbox:
                        # 确保复选框已勾选
                        state = win32gui.SendMessage(hwnd_checkbox, win32con.BM_GETCHECK, 0, 0)
                        if state != win32con.BST_CHECKED:
                            win32gui.SendMessage(hwnd_checkbox, win32con.BM_SETCHECK, win32con.BST_CHECKED, 0)
                        
                        # 点击确定按钮
                        hwnd_ok = win32gui.FindWindowEx(hwnd, 0, "Button", "确定")
                        if hwnd_ok:
                            win32gui.SendMessage(hwnd_ok, win32con.BM_CLICK, 0, 0)
                            print(f"已设置 '{icon_title}' 图标始终显示")
                            found = True
                            break
                break
        
        if not found:
            print("未能设置图标为始终显示")
            
    except Exception as e:
        print(f"设置图标可见性时出错: {e}")

# 主程序
if __name__ == "__main__":
    # 获取当前进程ID
    process_id = os.getpid()
    
    # 创建托盘图标
    icon = pystray.Icon(
        "test_icon",
        icon=create_image(),
        title="我的托盘图标 - 始终显示",
        menu=pystray.Menu(
            pystray.MenuItem("退出", lambda: icon.stop())
        )
    )
    
    # 在单独的线程中设置图标始终显示
    threading.Thread(
        target=set_tray_icon_always_show, 
        args=(process_id, "我的托盘图标 - 始终显示"),
        daemon=True
    ).start()
    
    # 运行托盘图标
    icon.run()
```

## 说明

1. **创建托盘图标**：使用pystray创建一个简单的托盘图标
2. **获取进程ID**：获取当前Python进程的ID，用于识别我们的托盘图标
3. **后台线程**：创建一个后台线程来设置图标的可见性
4. **设置始终显示**：
   - 等待图标出现在系统托盘中
   - 使用Windows Shell API打开图标的属性对话框
   - 勾选"始终在任务栏上显示所有图标和通知"选项
   - 点击"确定"按钮保存设置

## 注意事项

1. **权限要求**：此脚本需要以管理员权限运行才能修改系统设置
2. **系统兼容性**：适用于Windows 10及更高版本
3. **延迟处理**：需要等待图标出现在系统托盘后再尝试修改设置
4. **UI自动化**：此方法通过自动化UI操作实现，如果系统UI发生变化可能需要调整
5. **图标标题**：确保传递给`set_tray_icon_always_show`的标题与图标创建时使用的标题一致

## 备选方案

如果上述方法过于复杂，可以考虑引导用户手动设置：

```python
import pystray
from PIL import Image
import webbrowser
import threading

def create_image():
    image = Image.new('RGB', (64, 64), color='blue')
    return image

def show_instructions(icon, item):
    # 打开系统设置说明
    webbrowser.open("https://support.microsoft.com/zh-cn/windows/windows-%E4%B8%AD%E7%9A%84%E9%80%9A%E7%9F%A5%E5%92%8C%E8%A1%8C%E5%8A%A8%E4%B8%AD%E5%BF%83-8940c744-6198-fe56-4639-34320cf9444e")

# 创建带说明菜单的图标
icon = pystray.Icon(
    "my_icon",
    icon=create_image(),
    title="我的应用程序",
    menu=pystray.Menu(
        pystray.MenuItem("设置图标可见性", show_instructions),
        pystray.MenuItem("退出", lambda: icon.stop())
    )
)

# 在单独的线程中运行，避免阻塞
threading.Thread(target=icon.run, daemon=True).start()
```

这种方法虽然不能自动设置，但可以指导用户如何手动配置图标始终显示。

总之，虽然pystray本身不支持在创建时直接指定图标始终显示，但通过结合Windows API和UI自动化，我们可以实现类似的效果。