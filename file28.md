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


---
通过Python（特别是`pywin32`）获取Windows中USB设备的详细信息，并**根据设备名称进行筛选**是可行的。下面是一个简洁的代码示例和说明。

下面的代码演示了如何根据名称（支持模糊匹配）获取USB设备详细信息：

```python
import win32com.client
import re

def get_usb_devices_by_name(device_name_pattern):
    """
    根据设备名称模式获取USB设备详细信息

    Args:
        device_name_pattern (str): 要匹配的设备名称模式（不区分大小写，支持部分匹配）

    Returns:
        list: 包含匹配设备信息的字典列表，每个字典代表一个设备及其属性。
    """
    wmi = win32com.client.GetObject("winmgmts:")
    # 使用 Win32_PnPEntity 获取更友好的设备名称
    all_devices = wmi.InstancesOf("Win32_PnPEntity") 
    usb_devices = []

    for device in all_devices:
        # 检查是否为USB设备
        if device.PNPDeviceID and 'USB' in device.PNPDeviceID:
            device_name = device.Name or ''
            description = device.Description or ''

            # 检查设备名称或描述是否包含目标字符串（不区分大小写）
            pattern_lower = device_name_pattern.lower()
            if (pattern_lower in device_name.lower() or 
                pattern_lower in description.lower()):

                device_info = {
                    'Name': device_name,
                    'Description': description,
                    'DeviceID': device.DeviceID,
                    'PNPDeviceID': device.PNPDeviceID,
                    'Status': device.Status,
                    'Manufacturer': device.Manufacturer,
                    'Service': device.Service, # 设备使用的驱动服务
                    'ClassGuid': device.ClassGUID,
                    # 可以根据需要添加更多属性
                    # 'HardwareID': device.HardwareID, # 某些情况下可能需要尝试其他属性获取VID/PID
                }
                # 尝试从PNPDeviceID或HardwareID中提取VID和PID
                vid_pid_info = extract_vid_pid(device.PNPDeviceID)
                device_info.update(vid_pid_info)

                usb_devices.append(device_info)

    return usb_devices

def extract_vid_pid(pnp_device_id):
    """
    尝试从PNPDeviceID字符串中提取USB设备的VID和PID。

    Args:
        pnp_device_id (str): 设备的PNPDeviceID字符串

    Returns:
        dict: 包含VID和PID的字典（如果找到）
    """
    vid, pid = None, None
    # 常见的格式如 USB\VID_1234&PID_5678\...
    vid_match = re.search(r'VID_([0-9A-Fa-f]{4})', pnp_device_id)
    pid_match = re.search(r'PID_([0-9A-Fa-f]{4})', pnp_device_id)
    if vid_match:
        vid = vid_match.group(1)
    if pid_match:
        pid = pid_match.group(1)
    return {'VID': vid, 'PID': pid}

# 使用示例
if __name__ == "__main__":
    # 搜索名称中包含"HID"的USB设备（例如键盘、鼠标）
    matching_devices = get_usb_devices_by_name("HID")

    print(f"找到 {len(matching_devices)} 个匹配的设备:")
    for idx, device in enumerate(matching_devices, 1):
        print(f"\n设备 {idx}:")
        for key, value in device.items():
            print(f"  {key}: {value}")
```

### 代码说明

1.  **核心函数**：
    *   `get_usb_devices_by_name(device_name_pattern)`: 该函数查询WMI中的`Win32_PnPEntity`类，获取所有即插即用设备信息，并筛选出`PNPDeviceID`包含"USB"的设备（表明是USB设备）。它还会检查设备名称或描述是否包含用户提供的模式（不区分大小写）。匹配设备的详细信息（如名称、描述、设备ID、状态等）会被收集到一个字典列表中返回。

2.  **VID/PID提取**：
    *   `extract_vid_pid(pnp_device_id)`: 这是一个辅助函数，尝试使用正则表达式从设备的`PNPDeviceID`中提取供应商ID（VID）和产品ID（PID）。VID和PID是USB设备的重要标识符。

3.  **使用示例**：
    *   在`if __name__ == "__main__":`块中，代码演示了如何搜索名称中包含"HID"（通常指人体接口设备，如键盘、鼠标）的USB设备，并打印出找到的每个设备的详细信息。

### 注意事项

*   **运行权限**：某些WMI信息可能需要**管理员权限**才能访问。
*   **设备名称的局限性**：Windows系统提供的设备名称有时可能不够直观（例如，可能显示为“USB输入设备”而不是具体的品牌型号）。`Win32_PnPEntity`提供的名称通常比`Win32_USBHub`更友好一些。
*   **`pywin32`安装**：确保已通过`pip install pywin32`安装pywin32库。
*   **模糊匹配**：当前实现是简单的字符串包含匹配（不区分大小写）。你可以根据需要修改匹配逻辑（例如，使用正则表达式进行更复杂的模式匹配）。

### 其他参考方法

除了使用`Win32_PnPEntity`，你也可以尝试查询`Win32_USBControllerDevice`或`Win32_USBHub`，但它们提供的设备名称信息往往更偏向于控制器或集线器本身，而不是连接的外部设备。

如果`pywin32`获取的信息不足以满足需求，特别是无法获取到足够清晰的“友好名称”时，也可以考虑通过**PowerShell命令**（`Get-PnpDevice`）来获取设备列表，该命令获取的`FriendlyName`属性有时可能更直观。

希望这能帮助你有效地找到所需的USB设备信息。

----
----
# 使用 pywin32 获取串口设备信息

如果您想获取串口设备（COM端口）的信息，可以使用类似的方法。以下是专门用于获取串口设备信息的代码：

```python
import win32com.client
import re

def get_serial_ports():
    """
    获取所有串口设备信息
    
    Returns:
        list: 包含所有串口设备信息的字典列表
    """
    wmi = win32com.client.GetObject("winmgmts:")
    # 获取所有PnP设备
    all_devices = wmi.InstancesOf("Win32_PnPEntity")
    serial_ports = []
    
    for device in all_devices:
        device_name = device.Name or ''
        pnp_device_id = device.PNPDeviceID or ''
        
        # 检查是否为串口设备
        if ('COM' in device_name.upper() or 
            'SERIAL' in device_name.upper() or
            re.search(r'COM\d+', device_name) or
            'UART' in device_name.upper() or
            (pnp_device_id and ('SERIAL' in pnp_device_id.upper() or 'COM' in pnp_device_id.upper()))):
            
            device_info = {
                'Name': device_name,
                'Description': device.Description or '',
                'DeviceID': device.DeviceID or '',
                'PNPDeviceID': pnp_device_id,
                'Status': device.Status or '',
                'Manufacturer': device.Manufacturer or '',
                'Service': device.Service or '',
                'ClassGuid': device.ClassGUID or ''
            }
            
            # 尝试从名称中提取COM端口号
            com_port_match = re.search(r'COM(\d+)', device_name.upper())
            if com_port_match:
                device_info['COMPort'] = f"COM{com_port_match.group(1)}"
            
            # 尝试提取VID和PID
            vid_pid_info = extract_vid_pid(pnp_device_id)
            device_info.update(vid_pid_info)
            
            serial_ports.append(device_info)
    
    return serial_ports

def get_serial_ports_by_name(device_name_pattern):
    """
    根据设备名称模式获取串口设备信息
    
    Args:
        device_name_pattern (str): 要匹配的设备名称模式（不区分大小写，支持部分匹配）
    
    Returns:
        list: 包含匹配设备信息的字典列表
    """
    all_serial_ports = get_serial_ports()
    matching_ports = []
    
    pattern_lower = device_name_pattern.lower()
    
    for port in all_serial_ports:
        name = port.get('Name', '').lower()
        description = port.get('Description', '').lower()
        
        if pattern_lower in name or pattern_lower in description:
            matching_ports.append(port)
    
    return matching_ports

def extract_vid_pid(pnp_device_id):
    """
    尝试从PNPDeviceID字符串中提取USB设备的VID和PID
    
    Args:
        pnp_device_id (str): 设备的PNPDeviceID字符串
    
    Returns:
        dict: 包含VID和PID的字典（如果找到）
    """
    vid, pid = None, None
    # 常见的格式如 USB\VID_1234&PID_5678\...
    vid_match = re.search(r'VID_([0-9A-Fa-f]{4})', pnp_device_id or '')
    pid_match = re.search(r'PID_([0-9A-Fa-f]{4})', pnp_device_id or '')
    if vid_match:
        vid = vid_match.group(1)
    if pid_match:
        pid = pid_match.group(1)
    return {'VID': vid, 'PID': pid}

# 使用示例
if __name__ == "__main__":
    # 获取所有串口设备
    all_serial_ports = get_serial_ports()
    print(f"找到 {len(all_serial_ports)} 个串口设备:")
    for idx, port in enumerate(all_serial_ports, 1):
        print(f"\n设备 {idx}:")
        for key, value in port.items():
            print(f"  {key}: {value}")
    
    print("\n" + "="*50 + "\n")
    
    # 搜索特定名称的串口设备
    search_term = "USB"  # 可以替换为 "FTDI", "Prolific", "Arduino" 等
    matching_ports = get_serial_ports_by_name(search_term)
    print(f"找到 {len(matching_ports)} 个名称包含 '{search_term}' 的串口设备:")
    for idx, port in enumerate(matching_ports, 1):
        print(f"\n设备 {idx}:")
        for key, value in port.items():
            print(f"  {key}: {value}")
```

## 获取系统COM端口列表的替代方法

除了使用WMI，还可以使用Python的`serial.tools.list_ports`模块来获取COM端口列表：

```python
import serial.tools.list_ports

def get_com_ports_list():
    """
    使用pyserial获取COM端口列表
    
    Returns:
        list: 包含COM端口信息的列表
    """
    ports = serial.tools.list_ports.comports()
    port_list = []
    
    for port in ports:
        port_info = {
            'Device': port.device,
            'Name': port.name,
            'Description': port.description,
            'HardwareID': port.hwid,
            'VID': port.vid,
            'PID': port.pid,
            'SerialNumber': port.serial_number,
            'Location': port.location,
            'Manufacturer': port.manufacturer,
            'Product': port.product,
            'Interface': port.interface
        }
        port_list.append(port_info)
    
    return port_list

# 使用pyserial的示例
if __name__ == "__main__":
    com_ports = get_com_ports_list()
    print("使用pyserial获取的COM端口信息:")
    for port in com_ports:
        print(f"\n端口: {port['Device']}")
        for key, value in port.items():
            if value:  # 只显示有值的属性
                print(f"  {key}: {value}")
```

## 安装依赖

要运行上述代码，您需要安装以下Python库：

```bash
pip install pywin32 pyserial
```

## 注意事项

1. **串口设备的识别**：串口设备可能通过多种方式连接到计算机（USB转串口、PCI串口卡、主板内置串口等），它们的标识方式可能不同。

2. **COM端口号**：代码尝试从设备名称中提取COM端口号，但并非所有串口设备都会在名称中明确显示COM端口号。

3. **pyserial的优势**：`pyserial`库专门用于串口通信，它提供的端口信息通常更加准确和完整，特别是对于USB转串口设备。

4. **权限问题**：在某些系统上，可能需要管理员权限才能访问某些串口设备信息。

这些代码应该能够帮助您获取系统中所有串口设备的信息，并根据设备名称进行筛选。