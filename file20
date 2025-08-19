在Python中，可以通过`win32gui`和`psutil`库实现窗口句柄（HWND）与进程ID（PID）的相互查找。以下是详细实现：

### 安装依赖库
```bash
pip install pywin32 psutil
```

### 代码实现
```python
import win32gui
import win32process
import psutil

def get_pid_from_hwnd(hwnd):
    """通过窗口句柄获取进程ID"""
    try:
        _, pid = win32process.GetWindowThreadProcessId(hwnd)
        return pid
    except Exception as e:
        print(f"获取PID失败: {e}")
        return None

def get_hwnds_from_pid(pid):
    """通过进程ID获取所有关联的窗口句柄列表"""
    def callback(hwnd, hwnd_list):
        if win32gui.IsWindowVisible(hwnd) and win32gui.GetWindowText(hwnd):
            _, found_pid = win32process.GetWindowThreadProcessId(hwnd)
            if found_pid == pid:
                hwnd_list.append(hwnd)
        return True

    hwnd_list = []
    win32gui.EnumWindows(callback, hwnd_list)
    return hwnd_list

# 示例用法
if __name__ == "__main__":
    # 示例：通过记事本窗口标题查找
    target_hwnd = win32gui.FindWindow(None, "无标题 - 记事本")
    
    if target_hwnd:
        print(f"找到窗口句柄: 0x{target_hwnd:X}")
        
        # 通过HWND获取PID
        pid = get_pid_from_hwnd(target_hwnd)
        print(f"对应的进程ID: {pid}")
        
        # 通过PID重新查找HWND
        if pid:
            hwnds = get_hwnds_from_pid(pid)
            print(f"该进程的所有窗口句柄: {[f'0x{h:X}' for h in hwnds]}")
    else:
        print("未找到记事本窗口")
```

### 关键函数说明
1. **`get_pid_from_hwnd(hwnd)`**
   - 使用`GetWindowThreadProcessId`获取窗口所属进程PID
   - 返回整数类型的进程ID

2. **`get_hwnds_from_pid(pid)`**
   - 通过`EnumWindows`遍历所有窗口
   - 使用回调函数过滤可见窗口且匹配指定PID
   - 返回该进程所有可见窗口的句柄列表

### 注意事项
1. **窗口可见性**：只检索可见窗口（`IsWindowVisible`）
2. **多窗口进程**：一个进程可能有多个窗口（如浏览器多标签）
3. **权限要求**：需以足够权限运行（管理员权限可能更全面）
4. **窗口标题**：示例使用记事本窗口，实际使用时替换目标窗口标题

### 应用场景
- 自动化测试中定位应用窗口
- 进程监控工具开发
- 游戏/应用外挂检测
- 窗口管理工具开发

> 提示：若需查找后台窗口，可移除`IsWindowVisible`过滤条件，但可能包含大量隐藏窗口。