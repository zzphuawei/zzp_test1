# Windows USB设备信息获取接口

下面是一个专门针对Windows系统的简洁Python接口，用于获取USB设备信息：

```python
import subprocess

def get_windows_usb_devices():
    """
    获取Windows系统下的USB设备信息
    
    返回:
        str: USB设备信息的字符串表示
    """
    try:
        # 获取USB控制器设备信息
        result = "USB控制器设备:\n"
        result += subprocess.check_output(
            ['wmic', 'path', 'Win32_USBControllerDevice', 'get', 'Dependent'],
            stderr=subprocess.STDOUT,
            text=True,
            creationflags=subprocess.CREATE_NO_WINDOW
        )
        
        # 获取USB设备详细信息
        result += "\nUSB设备详细信息:\n" + "="*50 + "\n"
        result += subprocess.check_output(
            ['wmic', 'path', 'Win32_PnPEntity', 'where', "DeviceID like 'USB%'", 'get', 'Name,DeviceID'],
            stderr=subprocess.STDOUT,
            text=True,
            creationflags=subprocess.CREATE_NO_WINDOW
        )
        
        # 获取USB主机控制器信息
        result += "\nUSB主机控制器:\n" + "="*50 + "\n"
        result += subprocess.check_output(
            ['wmic', 'path', 'Win32_USBHub', 'get', 'Name,DeviceID'],
            stderr=subprocess.STDOUT,
            text=True,
            creationflags=subprocess.CREATE_NO_WINDOW
        )
        
        return result
        
    except subprocess.CalledProcessError as e:
        return f"命令执行失败: {e.output}"
    except Exception as e:
        return f"获取USB设备信息时出错: {str(e)}"

def get_usb_devices_by_vid_pid(vid=None, pid=None):
    """
    根据供应商ID(Vendor ID)和产品ID(Product ID)筛选USB设备
    
    参数:
        vid (str): 供应商ID (十六进制，如"8087")
        pid (str): 产品ID (十六进制，如"0024")
    
    返回:
        str: 匹配的USB设备信息
    """
    try:
        # 构建查询条件
        where_clause = "DeviceID like 'USB%'"
        if vid:
            where_clause += f" and DeviceID like '%VID_{vid.upper()}%'"
        if pid:
            where_clause += f" and DeviceID like '%PID_{pid.upper()}%'"
        
        # 执行查询
        result = subprocess.check_output(
            ['wmic', 'path', 'Win32_PnPEntity', 'where', where_clause, 'get', 'Name,DeviceID'],
            stderr=subprocess.STDOUT,
            text=True,
            creationflags=subprocess.CREATE_NO_WINDOW
        )
        
        if not result.strip() or "No Instance(s) Available" in result:
            return f"未找到匹配VID={vid}, PID={pid}的USB设备"
        
        return result
        
    except subprocess.CalledProcessError as e:
        return f"命令执行失败: {e.output}"
    except Exception as e:
        return f"获取USB设备信息时出错: {str(e)}"

# 使用示例
if __name__ == "__main__":
    # 获取所有USB设备信息
    usb_info = get_windows_usb_devices()
    print(usb_info)
    
    # 根据VID和PID筛选USB设备
    print("\n查找特定VID和PID的USB设备:")
    specific_usb = get_usb_devices_by_vid_pid("8087", "0024")  # 示例: Intel USB 3.0根集线器
    print(specific_usb)
```

## 使用说明

1. 调用 `get_windows_usb_devices()` 函数获取所有USB设备信息
2. 调用 `get_usb_devices_by_vid_pid(vid, pid)` 函数根据供应商ID和产品ID筛选特定USB设备

## 功能特点

- 获取USB控制器设备信息
- 获取USB设备详细信息
- 获取USB主机控制器信息
- 支持根据供应商ID(Vendor ID)和产品ID(Product ID)筛选设备
- 简洁的API接口
- 自动错误处理

## 注意事项

- 需要Windows系统
- 在某些情况下可能需要管理员权限才能获取完整的USB设备信息
- VID和PID通常是十六进制格式，例如Intel设备的VID可能是"8087"

您可以直接调用这些函数来获取USB设备信息，或者将上述代码导入到您的项目中使用。