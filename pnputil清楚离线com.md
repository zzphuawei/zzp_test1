以下是一键清理离线COM端口的PowerShell脚本：

## PowerShell 脚本

```powershell
# 以管理员权限运行COM端口清理脚本
Write-Host "=== COM端口清理工具 ===" -ForegroundColor Green
Write-Host "开始清理离线COM端口..." -ForegroundColor Yellow

# 检查是否以管理员身份运行
if (-NOT ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    Write-Host "请以管理员身份运行此脚本！" -ForegroundColor Red
    Write-Host "右键点击PowerShell，选择'以管理员身份运行'" -ForegroundColor Yellow
    pause
    exit
}

# 显示当前COM端口状态
Write-Host "`n当前COM端口状态：" -ForegroundColor Cyan
Get-PnpDevice -Class Ports | Format-Table FriendlyName, Status, InstanceId -AutoSize

# 清理离线COM端口
Write-Host "`n正在清理离线COM端口..." -ForegroundColor Yellow
$offlinePorts = Get-PnpDevice -Class Ports -Status Unknown

if ($offlinePorts.Count -eq 0) {
    Write-Host "未找到离线的COM端口。" -ForegroundColor Green
} else {
    Write-Host "找到 $($offlinePorts.Count) 个离线COM端口，正在清理..." -ForegroundColor Yellow
    $removedCount = 0
    
    foreach ($port in $offlinePorts) {
        try {
            $result = pnputil /remove-device $port.InstanceId 2>$null
            if ($LASTEXITCODE -eq 0) {
                Write-Host "已清理: $($port.FriendlyName)" -ForegroundColor Green
                $removedCount++
            } else {
                Write-Host "清理失败: $($port.FriendlyName)" -ForegroundColor Red
            }
        } catch {
            Write-Host "清理错误: $($port.FriendlyName)" -ForegroundColor Red
        }
    }
    Write-Host "成功清理 $removedCount 个离线COM端口" -ForegroundColor Green
}

# 可选：重置COM端口数据库
Write-Host "`n是否要重置COM端口数据库？(y/n)" -ForegroundColor Cyan -NoNewline
$resetChoice = Read-Host

if ($resetChoice -eq 'y' -or $resetChoice -eq 'Y') {
    Write-Host "正在重置COM端口数据库..." -ForegroundColor Yellow
    try {
        Remove-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\COM Name Arbiter" -Name "ComDB" -ErrorAction Stop
        Write-Host "COM端口数据库已重置，需要重启生效" -ForegroundColor Green
        
        Write-Host "是否立即重启计算机？(y/n)" -ForegroundColor Cyan -NoNewline
        $rebootChoice = Read-Host
        if ($rebootChoice -eq 'y' -or $rebootChoice -eq 'Y') {
            Restart-Computer -Confirm
        }
    } catch {
        Write-Host "重置COM端口数据库失败: $_" -ForegroundColor Red
    }
}

Write-Host "`n操作完成！" -ForegroundColor Green
Write-Host "如果问题仍然存在，请重新插拔USB设备或重启计算机" -ForegroundColor Yellow
pause
```

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