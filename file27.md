# Python USB 控制脚本

以下是一个简洁的Python脚本，用于控制指定USB设备的断开和重新连接：

```python
import time
import subprocess
import sys

def control_usb(device_id, action):
    """
    控制USB设备的连接状态
    
    参数:
    device_id: USB设备的标识符（Windows为硬件ID，Linux为总线/设备号）
    action: 操作类型 - 'disconnect' 或 'connect'
    """
    if sys.platform.startswith('win'):
        # Windows系统使用devcon工具
        if action == 'disconnect':
            cmd = f'devcon disable "{device_id}"'
        else:
            cmd = f'devcon enable "{device_id}"'
    elif sys.platform.startswith('linux'):
        # Linux系统使用uhubctl工具
        if action == 'disconnect':
            cmd = f'uhubctl -a off -p {device_id}'
        else:
            cmd = f'uhubctl -a on -p {device_id}'
    else:
        raise OSError("Unsupported operating system")
    
    try:
        result = subprocess.run(cmd, shell=True, check=True, 
                              capture_output=True, text=True)
        print(f"USB设备 {action} 操作成功: {device_id}")
        return True
    except subprocess.CalledProcessError as e:
        print(f"USB设备 {action} 操作失败: {e.stderr}")
        return False

def disconnect_and_reconnect(device_id, wait_minutes=2):
    """
    断开USB设备，等待指定时间后重新连接
    
    参数:
    device_id: USB设备的标识符
    wait_minutes: 等待时间（分钟）
    """
    # 断开设备
    if not control_usb(device_id, 'disconnect'):
        return False
    
    # 等待指定时间
    wait_seconds = wait_minutes * 60
    print(f"等待 {wait_minutes} 分钟...")
    time.sleep(wait_seconds)
    
    # 重新连接设备
    return control_usb(device_id, 'connect')

if __name__ == "__main__":
    # 示例使用
    # 请根据您的系统替换以下设备ID
    
    # Windows设备ID示例 (在设备管理器中查看硬件ID)
    # windows_device_id = "USB\\VID_1234&PID_5678\\1234567890"
    
    # Linux设备端口示例 (使用 lsusb 或 uhubctl 查看)
    # linux_device_port = "2"  # USB端口号
    
    # 根据系统选择设备ID
    if sys.platform.startswith('win'):
        device_id = "USB\\VID_1234&PID_5678\\1234567890"  # 替换为您的设备ID
    else:
        device_id = "2"  # 替换为您的USB端口号
    
    # 执行断开和重新连接操作
    success = disconnect_and_reconnect(device_id, wait_minutes=2)
    
    if success:
        print("操作完成成功")
    else:
        print("操作失败")
        sys.exit(1)
```

## 使用说明

1. **Windows系统**:
   - 需要下载并安装 [devcon工具](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/devcon)
   - 在设备管理器中找到您的USB设备，查看其硬件ID
   - 将脚本中的`device_id`替换为您的设备硬件ID

2. **Linux系统**:
   - 需要安装 `uhubctl` 工具: `sudo apt install uhubctl`
   - 使用 `uhubctl` 命令查看可控制的USB端口
   - 将脚本中的`device_id`替换为您的USB端口号

3. **运行脚本**:
   - 需要管理员/root权限运行
   - 直接执行: `sudo python usb_control.py` (Linux) 或以管理员身份运行 (Windows)

## 注意事项

- 此脚本需要根据您的具体USB设备和系统进行配置
- 实际操作USB设备可能需要特定的硬件支持
- 在某些系统上，可能需要额外的驱动程序或工具
- 请谨慎操作，错误的设备ID可能导致系统不稳定

如果您需要更具体的实现，请提供您的操作系统和USB设备信息。

------
# 获取 USB 设备 VID 和 PID 信息

在 Python 中，您可以使用 `pyudev` 库来获取 USB 设备的 VID (Vendor ID) 和 PID (Product ID) 信息。下面是一个简单的实现：

```python
import pyudev

def get_usb_devices():
    """获取所有USB设备信息"""
    context = pyudev.Context()
    devices = []
    
    for device in context.list_devices(subsystem='usb', DEVTYPE='usb_device'):
        vid = device.get('ID_VENDOR_ID', 'N/A')
        pid = device.get('ID_MODEL_ID', 'N/A')
        serial = device.get('ID_SERIAL_SHORT', 'N/A')
        vendor = device.get('ID_VENDOR', 'N/A')
        product = device.get('ID_MODEL', 'N/A')
        
        devices.append({
            'vid': vid,
            'pid': pid,
            'serial': serial,
            'vendor': vendor,
            'product': product,
            'device_path': device.device_path
        })
    
    return devices

def find_device_by_vid_pid(vid, pid):
    """根据VID和PID查找特定设备"""
    context = pyudev.Context()
    
    for device in context.list_devices(subsystem='usb', DEVTYPE='usb_device'):
        device_vid = device.get('ID_VENDOR_ID')
        device_pid = device.get('ID_MODEL_ID')
        
        if device_vid == vid and device_pid == pid:
            return {
                'vid': device_vid,
                'pid': device_pid,
                'serial': device.get('ID_SERIAL_SHORT', 'N/A'),
                'vendor': device.get('ID_VENDOR', 'N/A'),
                'product': device.get('ID_MODEL', 'N/A'),
                'device_path': device.device_path
            }
    
    return None

# 示例使用
if __name__ == "__main__":
    # 获取所有USB设备
    devices = get_usb_devices()
    print("所有USB设备:")
    for i, device in enumerate(devices, 1):
        print(f"{i}. VID: {device['vid']}, PID: {device['pid']}, 序列号: {device['serial']}")
        print(f"   厂商: {device['vendor']}, 产品: {device['product']}")
        print()
    
    # 查找特定设备 (示例VID和PID)
    target_vid = "1234"
    target_pid = "5678"
    specific_device = find_device_by_vid_pid(target_vid, target_pid)
    
    if specific_device:
        print(f"找到设备 VID:{target_vid} PID:{target_pid}:")
        print(f"序列号: {specific_device['serial']}")
        print(f"厂商: {specific_device['vendor']}")
        print(f"产品: {specific_device['product']}")
    else:
        print(f"未找到设备 VID:{target_vid} PID:{target_pid}")
```

## 安装必要的库

在运行代码前，需要先安装 `pyudev` 库：

```bash
pip install pyudev
```

## 代码说明

1. `get_usb_devices()` 函数返回所有连接的USB设备信息
2. `find_device_by_vid_pid(vid, pid)` 函数根据指定的VID和PID查找特定设备
3. 返回的信息包括：
   - VID (Vendor ID)
   - PID (Product ID)
   - 序列号
   - 厂商名称
   - 产品名称
   - 设备路径

## 注意事项

- 在Linux系统上运行此代码可能需要管理员权限
- Windows系统可能需要额外的驱动或库支持
- 示例中的VID(1234)和PID(5678)是示例值，您需要替换为实际要查找的值

如果您需要更详细的信息或有特定的使用场景，请提供更多细节。