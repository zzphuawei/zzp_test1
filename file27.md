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