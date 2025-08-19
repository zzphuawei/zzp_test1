在Python脚本中获取运行它的CMD窗口句柄，可以通过以下步骤实现：

### 解决方案思路
1. **获取父进程ID** - 当前Python进程是由CMD启动的，CMD是父进程
2. **枚举窗口** - 查找属于该父进程的所有窗口
3. **识别控制台窗口** - 通过窗口类名`ConsoleWindowClass`筛选
4. **返回第一个匹配的句柄** - 通常只有一个CMD窗口关联该进程

### 完整代码 (`A.py`)
```python
import ctypes
import ctypes.wintypes
import sys

def get_console_window_handle():
    # 定义Windows API类型
    kernel32 = ctypes.WinDLL('kernel32', use_last_error=True)
    user32 = ctypes.WinDLL('user32', use_last_error=True)
    
    # 获取当前进程ID
    current_pid = kernel32.GetCurrentProcessId()
    
    # 获取父进程ID (启动Python的CMD进程)
    h_snapshot = kernel32.CreateToolhelp32Snapshot(0x2, 0)  # TH32CS_SNAPPROCESS
    if h_snapshot == -1:
        return None
        
    class PROCESSENTRY32(ctypes.Structure):
        _fields_ = [
            ('dwSize', ctypes.wintypes.DWORD),
            ('cntUsage', ctypes.wintypes.DWORD),
            ('th32ProcessID', ctypes.wintypes.DWORD),
            ('th32DefaultHeapID', ctypes.POINTER(ctypes.wintypes.ULONG)),
            ('th32ModuleID', ctypes.wintypes.DWORD),
            ('cntThreads', ctypes.wintypes.DWORD),
            ('th32ParentProcessID', ctypes.wintypes.DWORD),
            ('pcPriClassBase', ctypes.wintypes.LONG),
            ('dwFlags', ctypes.wintypes.DWORD),
            ('szExeFile', ctypes.c_char * 260)
        ]
    
    entry = PROCESSENTRY32()
    entry.dwSize = ctypes.sizeof(PROCESSENTRY32)
    
    if not kernel32.Process32First(h_snapshot, ctypes.byref(entry)):
        kernel32.CloseHandle(h_snapshot)
        return None
        
    parent_pid = None
    while True:
        if entry.th32ProcessID == current_pid:
            parent_pid = entry.th32ParentProcessID
            break
        if not kernel32.Process32Next(h_snapshot, ctypes.byref(entry)):
            break
            
    kernel32.CloseHandle(h_snapshot)
    
    if not parent_pid:
        return None
        
    # 枚举窗口查找控制台窗口
    @ctypes.WINFUNCTYPE(ctypes.wintypes.BOOL, 
                         ctypes.wintypes.HWND, 
                         ctypes.wintypes.LPARAM)
    def enum_windows_proc(hwnd, lParam):
        nonlocal target_hwnd
        pid = ctypes.wintypes.DWORD()
        user32.GetWindowThreadProcessId(hwnd, ctypes.byref(pid))
        
        # 检查进程ID和窗口类名
        if pid.value == parent_pid:
            class_name = ctypes.create_string_buffer(256)
            if user32.GetClassNameA(hwnd, class_name, 256) and \
               class_name.value == b'ConsoleWindowClass':
                target_hwnd = hwnd
                return False  # 停止枚举
        return True  # 继续枚举
    
    target_hwnd = None
    user32.EnumWindows(enum_windows_proc, 0)
    return target_hwnd

if __name__ == "__main__":
    hwnd = get_console_window_handle()
    if hwnd:
        print(f"成功获取CMD窗口句柄: 0x{hwnd:X}")
        # 示例：修改窗口标题
        ctypes.windll.kernel32.SetConsoleTitleW(f"Python控制台 [句柄: 0x{hwnd:X}]")
    else:
        print("未找到CMD窗口句柄")
```

### 关键代码说明
1. **进程快照获取父进程ID**
   - 使用`CreateToolhelp32Snapshot`获取进程列表
   - 遍历查找当前进程的父进程ID（即CMD进程）

2. **窗口枚举与过滤**
   - `EnumWindows`遍历所有窗口
   - 通过`GetWindowThreadProcessId`获取窗口所属进程ID
   - 通过`GetClassNameA`检查窗口类是否为`ConsoleWindowClass`

3. **返回第一个匹配句柄**
   - 找到属于父进程且类名正确的第一个窗口
   - 立即返回该窗口句柄

### 使用注意事项
1. **仅支持Windows系统**
2. 需要以标准方式启动（双击.bat或从CMD运行）
3. 如果从非控制台环境启动（如IDE），将返回`None`
4. 示例中添加了修改窗口标题的演示代码

### 测试方法
1. 保存为`A.py`
2. 创建`run.bat`，内容为：
   ```bat
   @echo off
   python A.py
   pause
   ```
3. 双击运行`run.bat`，窗口标题将被修改并显示句柄信息

此方案直接获取承载Python进程的原始CMD窗口句柄，而非通过虚假控制台或间接方法，确保准确性和可靠性。