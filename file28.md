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


----
# Windows USB设备信息获取接口 - 按名称查询

下面是一个专门针对Windows系统的Python接口，用于根据设备名称获取USB设备的详细信息：

```python
import subprocess
import re

def get_usb_devices_by_name(device_name):
    """
    根据设备名称获取USB设备的详细信息
    
    参数:
        device_name (str): 设备名称或名称的一部分（支持模糊匹配）
    
    返回:
        list: 包含匹配设备详细信息的字典列表
    """
    try:
        # 获取所有USB设备信息
        result = subprocess.check_output(
            ['wmic', 'path', 'Win32_PnPEntity', 'where', "DeviceID like 'USB%'", 'get', '/format:list'],
            stderr=subprocess.STDOUT,
            text=True,
            creationflags=subprocess.CREATE_NO_WINDOW
        )
        
        # 解析WMIC输出
        devices = []
        current_device = {}
        
        for line in result.splitlines():
            line = line.strip()
            if not line:
                if current_device and _device_matches_name(current_device, device_name):
                    devices.append(current_device)
                current_device = {}
            else:
                if '=' in line:
                    key, value = line.split('=', 1)
                    current_device[key] = value
        
        # 检查最后一个设备
        if current_device and _device_matches_name(current_device, device_name):
            devices.append(current_device)
            
        return devices
        
    except subprocess.CalledProcessError as e:
        return [{"Error": f"命令执行失败: {e.output}"}]
    except Exception as e:
        return [{"Error": f"获取USB设备信息时出错: {str(e)}"}]

def _device_matches_name(device, name_pattern):
    """检查设备名称是否匹配模式"""
    if not name_pattern:
        return True
        
    name = device.get('Name', '')
    description = device.get('Description', '')
    
    # 不区分大小写的匹配
    pattern = name_pattern.lower()
    return pattern in name.lower() or pattern in description.lower()

def get_detailed_usb_info(device_id):
    """
    根据设备ID获取更详细的USB设备信息
    
    参数:
        device_id (str): 设备ID
    
    返回:
        dict: 包含设备详细信息的字典
    """
    try:
        # 获取设备硬件ID
        result = subprocess.check_output(
            ['wmic', 'path', 'Win32_PnPEntity', 'where', f"DeviceID='{device_id}'", 'get', 'HardwareID', '/format:list'],
            stderr=subprocess.STDOUT,
            text=True,
            creationflags=subprocess.CREATE_NO_WINDOW
        )
        
        # 解析硬件ID
        hardware_ids = []
        for line in result.splitlines():
            line = line.strip()
            if line.startswith('HardwareID='):
                ids = line.split('=', 1)[1].split(';')
                hardware_ids.extend(ids)
        
        # 从硬件ID中提取VID和PID
        vid, pid = None, None
        for hw_id in hardware_ids:
            vid_match = re.search(r'VID_([0-9A-Fa-f]{4})', hw_id)
            pid_match = re.search(r'PID_([0-9A-Fa-f]{4})', hw_id)
            
            if vid_match:
                vid = vid_match.group(1)
            if pid_match:
                pid = pid_match.group(1)
            
            if vid and pid:
                break
        
        # 获取设备状态
        status_result = subprocess.check_output(
            ['wmic', 'path', 'Win32_PnPEntity', 'where', f"DeviceID='{device_id}'", 'get', 'Status', '/format:list'],
            stderr=subprocess.STDOUT,
            text=True,
            creationflags=subprocess.CREATE_NO_WINDOW
        )
        
        status = "Unknown"
        for line in status_result.splitlines():
            line = line.strip()
            if line.startswith('Status='):
                status = line.split('=', 1)[1]
                break
        
        # 构建详细信息字典
        detailed_info = {
            "DeviceID": device_id,
            "VID": vid,
            "PID": pid,
            "Status": status,
            "HardwareIDs": hardware_ids
        }
        
        return detailed_info
        
    except subprocess.CalledProcessError as e:
        return {"Error": f"命令执行失败: {e.output}"}
    except Exception as e:
        return {"Error": f"获取USB设备详细信息时出错: {str(e)}"}

# 使用示例
if __name__ == "__main__":
    # 根据名称搜索USB设备
    device_name = "USB"  # 可以替换为具体的设备名称，如"Arduino"、"Logitech"等
    devices = get_usb_devices_by_name(device_name)
    
    print(f"找到 {len(devices)} 个匹配 '{device_name}' 的USB设备:")
    print("=" * 80)
    
    for i, device in enumerate(devices, 1):
        print(f"\n设备 {i}:")
        for key, value in device.items():
            print(f"  {key}: {value}")
        
        # 获取更详细的信息
        if 'DeviceID' in device:
            detailed_info = get_detailed_usb_info(device['DeviceID'])
            print("  详细信息:")
            for key, value in detailed_info.items():
                if key == 'HardwareIDs':
                    print(f"    {key}:")
                    for hw_id in value:
                        print(f"      - {hw_id}")
                else:
                    print(f"    {key}: {value}")
```

## 使用说明

1. 调用 `get_usb_devices_by_name(device_name)` 函数根据设备名称搜索USB设备
   - 支持模糊匹配（不区分大小写）
   - 返回包含匹配设备信息的字典列表

2. 调用 `get_detailed_usb_info(device_id)` 函数根据设备ID获取更详细的USB设备信息
   - 包括VID、PID、状态和硬件ID等信息

## 功能特点

- 根据设备名称模糊搜索USB设备
- 返回结构化的设备信息（字典格式）
- 提取设备的VID（供应商ID）和PID（产品ID）
- 获取设备状态信息
- 列出所有硬件ID

## 注意事项

- 需要Windows系统
- 在某些情况下可能需要管理员权限才能获取完整的USB设备信息
- 设备名称搜索是模糊匹配，不区分大小写

您可以直接调用这些函数来根据名称搜索USB设备并获取详细信息，或者将上述代码导入到您的项目中使用。