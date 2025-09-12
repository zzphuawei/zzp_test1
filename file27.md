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
# 使用WMI获取USB设备信息

在Python中，您可以使用WMI模块来获取USB设备信息。对于您提供的设备实例路径 `USB\\VID_1234&PID_5678\\1234567890`，以下是获取该设备信息的完整代码：

```python
import wmi

def get_usb_device_info(device_instance_id):
    """
    根据设备实例ID获取USB设备信息
    
    参数:
        device_instance_id: 设备实例ID，如 USB\\VID_1234&PID_5678\\1234567890
    """
    c = wmi.WMI()
    
    try:
        # 查询Win32_PnPEntity以匹配设备实例ID
        devices = c.query(f"SELECT * FROM Win32_PnPEntity WHERE DeviceID = '{device_instance_id}'")
        
        if devices:
            device = devices[0]
            print("找到匹配的USB设备:")
            print(f"设备描述: {device.Description}")
            print(f"设备ID: {device.DeviceID}")
            print(f"名称: {device.Name}")
            print(f"制造商: {device.Manufacturer}")
            print(f"状态: {device.Status}")
            print(f"PNP设备ID: {device.PNPDeviceID}")
            
            # 尝试获取更多详细信息
            try:
                usb_controllers = c.query("ASSOCIATORS OF {Win32_PnPEntity.DeviceID='" + 
                                         device_instance_id.replace("'", "''") + 
                                         "'} WHERE AssocClass=Win32_USBControllerDevice")
                if usb_controllers:
                    print(f"连接的USB控制器: {usb_controllers[0].DeviceID}")
            except Exception as e:
                print(f"获取USB控制器信息时出错: {e}")
        else:
            print("未找到匹配的USB设备")
            
    except Exception as e:
        print(f"查询设备时出错: {e}")

def get_all_usb_devices():
    """
    获取所有USB设备的信息
    """
    c = wmi.WMI()
    
    print("所有USB设备:")
    print("-" * 50)
    
    # 获取所有USB设备
    usb_devices = c.Win32_USBControllerDevice()
    
    for usb_device in usb_devices:
        try:
            # 获取关联的PnP设备
            pnp_device = c.Win32_PnPEntity(DeviceID=usb_device.Dependent.DeviceID)[0]
            
            print(f"设备ID: {pnp_device.DeviceID}")
            print(f"描述: {pnp_device.Description}")
            print(f"状态: {pnp_device.Status}")
            print("-" * 30)
        except:
            continue

if __name__ == "__main__":
    # 您提供的设备实例ID
    device_id = "USB\\VID_1234&PID_5678\\1234567890"
    
    # 查找特定设备
    get_usb_device_info(device_id)
    
    print("\n")
    
    # 列出所有USB设备
    get_all_usb_devices()
```

## 安装必要的库

在运行上述代码前，您需要安装WMI库：

```bash
pip install wmi
```

## 代码说明

1. **get_usb_device_info()** 函数根据设备实例ID查找特定USB设备
2. **get_all_usb_devices()** 函数列出所有USB设备信息
3. 代码使用WMI查询Win32_PnPEntity和Win32_USBControllerDevice类来获取USB设备信息

## 注意事项

- 设备实例ID需要精确匹配，包括大小写
- 某些设备可能需要管理员权限才能访问详细信息
- 如果设备不存在或未连接，将无法找到匹配项

您可以根据需要修改代码，提取更多设备属性或过滤特定类型的USB设备。