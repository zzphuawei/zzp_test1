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

---
# 删除系统环境变量的完整指南

在 Python 中删除环境变量也有临时删除和永久删除两种方式，具体实现取决于操作系统。以下是完整解决方案：

## 1. 临时删除环境变量（当前进程有效）

```python
import os

def delete_temp_env(var_name):
    """删除当前进程的环境变量（不影响系统设置）"""
    if var_name in os.environ:
        del os.environ[var_name]
        print(f"临时删除环境变量: {var_name}")
    else:
        print(f"环境变量 {var_name} 不存在，无需删除")

# 示例用法
delete_temp_env("MY_TEMP_VAR")
```

## 2. 永久删除环境变量

### Windows 系统（使用注册表）

```python
import winreg
import ctypes
import os

def delete_permanent_env(var_name):
    """从Windows注册表永久删除环境变量"""
    try:
        # 打开环境变量注册表键
        key = winreg.OpenKey(
            winreg.HKEY_CURRENT_USER,
            'Environment',
            0,
            winreg.KEY_READ | winreg.KEY_WRITE
        )
        
        # 检查变量是否存在
        try:
            winreg.QueryValueEx(key, var_name)
            exists = True
        except FileNotFoundError:
            exists = False
        
        if exists:
            # 删除变量
            winreg.DeleteValue(key, var_name)
            
            # 关闭注册表键
            winreg.CloseKey(key)
            
            # 通知系统更新环境变量
            HWND_BROADCAST = 0xFFFF
            WM_SETTINGCHANGE = 0x1A
            ctypes.windll.user32.SendMessageTimeoutW(
                HWND_BROADCAST, WM_SETTINGCHANGE, 0, "Environment", 0, 1000, None
            )
            
            print(f"永久删除环境变量: {var_name}")
            return True
        else:
            print(f"环境变量 {var_name} 不存在，无需删除")
            winreg.CloseKey(key)
            return False
            
    except Exception as e:
        print(f"删除环境变量失败: {str(e)}")
        return False

# 示例用法
delete_permanent_env("MY_PERMANENT_VAR")
```

### 从 PATH 中移除特定路径（Windows）

```python
def remove_from_path(path_to_remove):
    """从PATH环境变量中移除特定路径"""
    try:
        # 获取当前PATH
        key = winreg.OpenKey(
            winreg.HKEY_CURRENT_USER,
            'Environment',
            0,
            winreg.KEY_READ | winreg.KEY_WRITE
        )
        current_path, _ = winreg.QueryValueEx(key, 'PATH')
        
        # 标准化路径（解决大小写和斜杠问题）
        normalized_remove = os.path.normpath(path_to_remove).lower()
        
        # 分割路径并过滤
        paths = current_path.split(os.pathsep)
        new_paths = [
            p for p in paths 
            if os.path.normpath(p).lower() != normalized_remove
        ]
        
        # 如果路径有变化，则更新
        if len(new_paths) != len(paths):
            new_path = os.pathsep.join(new_paths)
            winreg.SetValueEx(key, 'PATH', 0, winreg.REG_EXPAND_SZ, new_path)
            winreg.CloseKey(key)
            
            # 通知系统更新
            HWND_BROADCAST = 0xFFFF
            WM_SETTINGCHANGE = 0x1A
            ctypes.windll.user32.SendMessageTimeoutW(
                HWND_BROADCAST, WM_SETTINGCHANGE, 0, "Environment", 0, 1000, None
            )
            
            print(f"已从PATH中移除: {path_to_remove}")
            return True
        else:
            print(f"路径 {path_to_remove} 不在PATH中，无需移除")
            winreg.CloseKey(key)
            return False
            
    except Exception as e:
        print(f"从PATH移除路径失败: {str(e)}")
        return False

# 示例用法
remove_from_path(r"C:\Program Files\MyApp\bin")
```

### Linux/macOS 系统

```python
def delete_permanent_env_linux(var_name, config_file="~/.bashrc"):
    """从Linux/Mac的配置文件中删除环境变量"""
    config_path = os.path.expanduser(config_file)
    temp_path = config_path + ".tmp"
    
    deleted = False
    
    try:
        with open(config_path, 'r') as fin, open(temp_path, 'w') as fout:
            # 正则匹配环境变量定义
            pattern = re.compile(rf'^\s*export\s+{var_name}=.*$')
            
            for line in fin:
                if pattern.match(line):
                    deleted = True
                    continue  # 跳过这一行（即删除）
                fout.write(line)
        
        # 如果删除了变量，替换原文件
        if deleted:
            os.replace(temp_path, config_path)
            print(f"已从 {config_file} 删除环境变量 {var_name}")
            print("请重启终端或运行 `source {}` 使更改生效".format(config_file))
            return True
        else:
            os.remove(temp_path)
            print(f"环境变量 {var_name} 未在 {config_file} 中找到")
            return False
            
    except Exception as e:
        print(f"删除环境变量失败: {str(e)}")
        if os.path.exists(temp_path):
            os.remove(temp_path)
        return False

# 示例用法
delete_permanent_env_linux("MY_LINUX_VAR")
```

## 使用说明与注意事项

1. **权限要求**：
   - Windows：修改注册表需要管理员权限
   - Linux：修改系统级配置文件需要root权限

2. **作用范围**：
   - `HKEY_CURRENT_USER` (Windows) 或用户配置文件 (Linux)：仅影响当前用户
   - 系统全局修改需使用 `HKEY_LOCAL_MACHINE` (Windows) 或 `/etc/environment` (Linux)

3. **PATH 特殊处理**：
   - 修改PATH时需保留原有路径
   - 使用 `os.pathsep` 作为分隔符确保跨平台兼容性
   - 路径比较时考虑大小写和斜杠方向差异

4. **立即生效**：
   - Windows：使用 `SendMessageTimeoutW` 广播更改通知
   - Linux：需要重启终端或运行 `source` 命令

5. **安全提示**：
   - 操作前备份注册表或配置文件
   - 避免删除系统关键环境变量（如PATH）
   - 在虚拟环境中测试后再应用到生产环境

## 完整示例：安全删除环境变量

```python
def safe_delete_env(var_name, system_level=False):
    """安全删除环境变量（跨平台）"""
    # 临时删除
    if var_name in os.environ:
        del os.environ[var_name]
    
    # 永久删除
    if os.name == 'nt':  # Windows
        if system_level:
            # 系统全局删除需要额外实现
            print("警告：系统级删除需要特殊权限和实现")
            return False
        return delete_permanent_env(var_name)
    else:  # Linux/Mac
        config_file = "/etc/environment" if system_level else "~/.bashrc"
        return delete_permanent_env_linux(var_name, config_file)

# 示例：删除当前用户的环境变量
safe_delete_env("OBSOLETE_VAR")
```

这些函数提供了在Python中删除环境变量的完整解决方案，涵盖了临时删除和永久删除的不同场景。