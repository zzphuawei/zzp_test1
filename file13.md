您遇到了 `close key() 需要一个参数` 的错误，这是因为在 Windows 注册表操作中，`CloseKey()` 方法需要传入要关闭的键对象。让我修复并提供完整的代码：

```python
import winreg
import os
import ctypes

def set_permanent_env(var_name, var_value):
    # 打开环境变量注册表键
    try:
        key = winreg.OpenKey(
            winreg.HKEY_CURRENT_USER,
            'Environment',
            0, 
            winreg.KEY_WRITE | winreg.KEY_READ
        )
    except Exception as e:
        print(f"无法打开注册表键: {e}")
        return False

    try:
        # 设置变量
        winreg.SetValueEx(key, var_name, 0, winreg.REG_EXPAND_SZ, var_value)
        
        # 正确关闭注册表键 - 这里修复了错误
        winreg.CloseKey(key)
        
        # 通知系统更新环境变量
        HWND_BROADCAST = 0xFFFF
        WM_SETTINGCHANGE = 0x1A
        SendMessageTimeout = ctypes.windll.user32.SendMessageTimeoutW
        SendMessageTimeout(HWND_BROADCAST, WM_SETTINGCHANGE, 0, "Environment", 0, 1000, None)
        
        print(f"成功设置环境变量: {var_name}={var_value}")
        return True
        
    except Exception as e:
        print(f"设置环境变量失败: {e}")
        winreg.CloseKey(key)  # 确保在出错时也关闭键
        return False

# 示例：添加路径到 PATH
try:
    # 获取当前PATH
    current_path = os.environ['PATH']
    new_path = r'C:\my\custom\path'
    
    # 检查路径是否已存在
    if new_path not in current_path.split(os.pathsep):
        # 设置新的PATH
        set_permanent_env('PATH', current_path + os.pathsep + new_path)
    else:
        print("路径已在PATH中存在")
except KeyError:
    # 如果PATH不存在（极少数情况）
    set_permanent_env('PATH', new_path)
```

### 关键修复点：

1. **修复了 `CloseKey()` 调用**：
   ```python
   winreg.CloseKey(key)  # 正确方式：传入key对象作为参数
   ```

2. **添加了错误处理**：
   - 使用 try-except 捕获可能的异常
   - 确保在任何情况下都正确关闭注册表键

3. **改进的广播通知**：
   ```python
   SendMessageTimeout(HWND_BROADCAST, WM_SETTINGCHANGE, 0, "Environment", 0, 1000, None)
   ```

4. **避免重复添加PATH**：
   ```python
   if new_path not in current_path.split(os.pathsep):
   ```

### 使用说明：

1. **仅修改当前用户的PATH**：
   - 此代码修改 `HKEY_CURRENT_USER\Environment`
   - 若要修改系统全局PATH（所有用户），需：
     - 管理员权限运行
     - 替换为 `winreg.HKEY_LOCAL_MACHINE`
     - 路径改为 `'SYSTEM\CurrentControlSet\Control\Session Manager\Environment'`

2. **路径格式**：
   - 使用原始字符串避免转义问题：`r'C:\my\path'`
   - 使用 `os.pathsep`（Windows上是`;`）作为分隔符

3. **权限要求**：
   - 修改注册表需要程序以管理员权限运行

4. **测试建议**：
   ```python
   # 测试设置一个临时变量
   set_permanent_env('MY_TEST_VAR', 'test_value')
   # 重启后检查是否生效
   ```

如果仍然遇到问题，可能是权限或路径格式问题，请提供具体错误信息以便进一步调试。