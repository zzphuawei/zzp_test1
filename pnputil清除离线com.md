
## 批处理脚本版本

如果你更喜欢简单的批处理文件：

```batch
@echo off
title COM端口清理工具
echo ===============================
echo      COM端口一键清理工具
echo ===============================
echo.

:: 检查管理员权限
net session >nul 2>&1
if %errorLevel% neq 0 (
    echo 请以管理员身份运行此脚本！
    echo 右键点击，选择"以管理员身份运行"
    pause
    exit /b
)

echo 正在扫描离线COM端口...
echo.

:: 使用PowerShell清理离线端口
powershell -Command "Get-PnpDevice -Class Ports -Status Unknown  | Format-Table -AutoSize FriendlyName, Status, InstanceId"

set /p confirm=确认要删除这些设备吗？(y/n): 

if /i not "!confirm!"=="y" (
    echo 操作已取消。
    pause
    exit /b
)

powershell -Command "Get-PnpDevice -Class Ports -Status Unknown | ForEach-Object { pnputil /remove-device $_.InstanceId }"

echo.
echo 离线COM端口清理完成！
echo.
echo 建议：重新插拔USB串口设备
echo.
pause
```

## 使用方法

1. **PowerShell脚本**：
   - 将代码保存为 `CleanCOMPorts.ps1`
   - 右键点击，选择"使用PowerShell运行"
   - 或打开PowerShell，输入：`.\CleanCOMPorts.ps1`

2. **批处理脚本**：
   - 将代码保存为 `CleanCOMPorts.bat`
   - 右键点击，选择"以管理员身份运行"

## 脚本功能

- ✅ 自动检测管理员权限
- ✅ 显示当前COM端口状态
- ✅ 一键清理所有离线COM端口
- ✅ 可选重置COM端口数据库
- ✅ 详细的进度和结果反馈

## 注意事项

- 运行前建议关闭所有使用COM端口的程序
- 重置COM端口数据库需要重启生效
- 某些系统保护软件可能会阻止注册表操作

选择适合你的脚本版本运行即可完成一键清理！
