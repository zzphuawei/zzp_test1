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

---
# 清理HDC和Harmony OS设备的批处理脚本

以下是一个带有确认提示的批处理脚本，用于清理特定的离线设备：

```batch
@echo off
setlocal enabledelayedexpansion
chcp 65001 >nul

title HDC和Harmony OS设备清理工具
echo ========================================
echo    HDC和Harmony OS设备一键清理工具
echo ========================================
echo.

:: 检查管理员权限
net session >nul 2>&1
if %errorLevel% neq 0 (
    echo 请以管理员身份运行此脚本！
    echo 右键点击，选择"以管理员身份运行"
    pause
    exit /b
)

:: 搜索目标设备
echo 正在搜索HDC和Harmony OS设备...
echo.

set "device_count=0"
set "device_list="

:: 使用WMIC查找设备
for /f "tokens=1,2 delims==" %%a in ('wmic path Win32_PnPEntity where "Status='Error'" get Name /value 2^>nul') do (
    for /f "tokens=*" %%c in ("%%b") do (
        set "device_name=%%c"
        if not "!device_name!"=="" (
            echo !device_name! | findstr /i "HDC Harmony" >nul
            if !errorlevel! equ 0 (
                set /a device_count+=1
                echo [!device_count!] !device_name!
                set "device_list=!device_list!,"%%c""
            )
        )
    )
)

if %device_count% equ 0 (
    echo 未找到HDC或Harmony OS设备。
    pause
    exit /b
)

echo.
echo 共找到 %device_count% 个相关设备。
echo.
set /p confirm=确认要删除这些设备吗？(y/n): 

if /i not "!confirm!"=="y" (
    echo 操作已取消。
    pause
    exit /b
)

echo.
echo 开始清理设备...
echo.

:: 使用PnPUtil删除设备
set "removed_count=0"
for /f "tokens=1,2 delims==" %%a in ('wmic path Win32_PnPEntity where "Status='Error'" get Name, DeviceID /value 2^>nul') do (
    if "%%a"=="Name" (
        set "device_name=%%b"
    )
    if "%%a"=="DeviceID" (
        set "device_id=%%b"
        if not "!device_name!"=="" (
            echo !device_name! | findstr /i "HDC Harmony" >nul
            if !errorlevel! equ 0 (
                echo 正在删除: !device_name!
                pnputil /remove-device "!device_id!" >nul 2>&1
                if !errorlevel! equ 0 (
                    echo [成功] !device_name!
                    set /a removed_count+=1
                ) else (
                    echo [失败] !device_name!
                )
                set "device_name="
            )
        )
    )
)

echo.
echo 清理完成！成功删除了 %removed_count% 个设备。
echo.
echo 建议：重新连接相关USB设备以重新安装驱动。
echo.
pause
```

## 脚本功能说明

1. **权限检查**：自动检测是否以管理员权限运行
2. **设备扫描**：使用WMIC命令查找所有状态为"Error"的设备
3. **设备筛选**：筛选出名称中包含"HDC"或"Harmony"的设备
4. **确认提示**：显示找到的设备列表，并询问用户是否继续删除
5. **批量删除**：使用PnPUtil命令删除所有匹配的设备
6. **结果反馈**：显示每个设备的删除状态和最终统计

## 使用方法

1. 将上述代码保存为 `CleanHDCDevices.bat`
2. 右键点击文件，选择"以管理员身份运行"
3. 脚本会显示找到的设备列表
4. 输入 `y` 确认删除，或 `n` 取消操作

## 注意事项

- 运行前请确保关闭所有可能使用这些设备的程序
- 删除设备后，可能需要重新连接相关USB设备才能重新安装驱动
- 某些系统保护软件可能会阻止设备删除操作

这个脚本会安全地清理所有离线的HDC和Harmony OS设备，并在执行前给予确认提示，避免误操作。

----
我为您编写了一个结合Bat脚本和PowerShell命令的解决方案，可以安全清除指定的设备。这个脚本包含您要求的确认功能，让您在使用前能够确认操作。

## 批处理脚本：清除特定设备

```batch
@echo off
chcp 65001 >nul
title Device Cleanup Script

echo ===============================================
echo           设备清理脚本
echo ===============================================
echo 此脚本将清理以下设备：
echo   - HDC Device
echo   - Harmony OS Device  
echo   - HDC Interface
echo.
echo 注意：请以管理员身份运行此脚本
echo ===============================================
echo.

:choice
set /p choice=是否继续执行清理操作？(Y/N): 
if /i "%choice%"=="y" goto execute_cleanup
if /i "%choice%"=="n" goto exit_script
echo 请输入 Y 或 N
goto choice

:execute_cleanup
echo.
echo 开始清理设备...
echo.

powershell -Command "& {"
powershell -Command "    Write-Host '正在扫描和清理 HDC Device...' -ForegroundColor Yellow"
powershell -Command "    Get-PnpDevice | Where-Object {$_.FriendlyName -like '*HDC Device*'} | ForEach-Object {"
powershell -Command "        Write-Host ('正在移除: ' + $_.FriendlyName) -ForegroundColor Cyan"
powershell -Command "        pnputil.exe /remove-device $_.InstanceId"
powershell -Command "    }"
powershell -Command "    "
powershell -Command "    Write-Host '正在扫描和清理 Harmony OS Device...' -ForegroundColor Yellow"
powershell -Command "    Get-PnpDevice | Where-Object {$_.FriendlyName -like '*Harmony OS Device*'} | ForEach-Object {"
powershell -Command "        Write-Host ('正在移除: ' + $_.FriendlyName) -ForegroundColor Cyan"
powershell -Command "        pnputil.exe /remove-device $_.InstanceId"
powershell -Command "    }"
powershell -Command "    "
powershell -Command "    Write-Host '正在扫描和清理 HDC Interface...' -ForegroundColor Yellow"
powershell -Command "    Get-PnpDevice | Where-Object {$_.FriendlyName -like '*HDC Interface*'} | ForEach-Object {"
powershell -Command "        Write-Host ('正在移除: ' + $_.FriendlyName) -ForegroundColor Cyan"
powershell -Command "        pnputil.exe /remove-device $_.InstanceId"
powershell -Command "    }"
powershell -Command "}"

echo.
echo 设备清理操作完成！
echo 建议重新启动计算机以使更改生效。
pause
goto :eof

:exit_script
echo 操作已取消。
pause
```

## 脚本功能说明

这个脚本主要实现以下功能：

### 🔍 **设备清理目标**
- **HDC Device**
- **Harmony OS Device**  
- **HDC Interface**

### ⚙️ **技术实现**
1. **用户确认机制**：使用批处理的choice功能实现Y/N确认
2. **PowerShell集成**：通过bat调用PowerShell命令
3. **设备枚举和移除**：使用`Get-PnpDevice`配合`pnputil /remove-device`来识别和移除设备

### 🛡️ **使用前重要提示**

1. **管理员权限**：务必右键点击bat文件，选择"以管理员身份运行"
2. **设备识别**：脚本使用通配符匹配设备名称，确保准确识别目标设备
3. **操作可逆**：清理后如需恢复设备，通常重新连接硬件即可自动重新安装驱动

## 使用步骤

1. 将上述代码保存为 `.bat` 文件（如 `CleanupDevices.bat`）
2. **右键点击文件**，选择"以管理员身份运行"
3. 查看显示的设备列表，确认无误后输入 `Y` 继续
4. 脚本会自动扫描并清理指定的三类设备
5. 完成后建议重启计算机

这个脚本在Windows 10和Windows 11系统上都应该可以正常工作。如果您在执行过程中遇到任何问题，或者需要清理其他类型的设备，请告诉我，我可以帮您调整脚本。


---
您提出了一个很好的改进建议。我将修改脚本，先扫描并显示所有匹配的设备，然后再让用户确认是否删除。

## 改进后的批处理脚本（先扫描后确认）

```batch
@echo off
chcp 65001 >nul
title 设备清理脚本 - 扫描模式

echo ===============================================
echo           设备清理脚本（扫描模式）
echo ===============================================
echo 此脚本将先扫描以下设备：
echo   - HDC Device
echo   - Harmony OS Device  
echo   - HDC Interface
echo.
echo 扫描完成后会显示找到的设备，并询问是否删除
echo ===============================================
echo.
echo 正在扫描设备，请稍候...
echo.

set "DEVICE_COUNT=0"

:: 扫描设备并计数
for /f "tokens=*" %%i in ('powershell -Command "Get-PnpDevice | Where-Object {$_.FriendlyName -like '*HDC Device*' -or $_.FriendlyName -like '*Harmony OS Device*' -or $_.FriendlyName -like '*HDC Interface*'} | ForEach-Object { Write-Output ('『' + $_.FriendlyName + '』 - InstanceId: ' + $_.InstanceId) }"') do (
    echo %%i
    set /a DEVICE_COUNT+=1
)

echo.
echo ===============================================
echo 扫描完成！共找到 %DEVICE_COUNT% 个匹配的设备
echo ===============================================
echo.

if %DEVICE_COUNT% equ 0 (
    echo 未找到任何匹配的设备，无需清理。
    pause
    exit /b 0
)

:choice
set /p choice=是否要删除以上所有设备？(Y/N): 
if /i "%choice%"=="y" goto execute_cleanup
if /i "%choice%"=="n" goto exit_script
echo 请输入 Y 或 N
goto choice

:execute_cleanup
echo.
echo 开始删除设备...
echo.

powershell -Command "& {"
powershell -Command "    Write-Host '正在删除 HDC Device...' -ForegroundColor Red"
powershell -Command "    Get-PnpDevice | Where-Object {$_.FriendlyName -like '*HDC Device*'} | ForEach-Object {"
powershell -Command "        Write-Host ('正在删除: ' + $_.FriendlyName) -ForegroundColor Yellow"
powershell -Command "        try {"
powershell -Command "            pnputil.exe /remove-device $_.InstanceId"
powershell -Command "            Write-Host '   删除成功' -ForegroundColor Green"
powershell -Command "        } catch {"
powershell -Command "            Write-Host '   删除失败' -ForegroundColor Red"
powershell -Command "        }"
powershell -Command "    }"
powershell -Command "    "
powershell -Command "    Write-Host '正在删除 Harmony OS Device...' -ForegroundColor Red"
powershell -Command "    Get-PnpDevice | Where-Object {$_.FriendlyName -like '*Harmony OS Device*'} | ForEach-Object {"
powershell -Command "        Write-Host ('正在删除: ' + $_.FriendlyName) -ForegroundColor Yellow"
powershell -Command "        try {"
powershell -Command "            pnputil.exe /remove-device $_.InstanceId"
powershell -Command "            Write-Host '   删除成功' -ForegroundColor Green"
powershell -Command "        } catch {"
powershell -Command "            Write-Host '   删除失败' -ForegroundColor Red"
powershell -Command "        }"
powershell -Command "    }"
powershell -Command "    "
powershell -Command "    Write-Host '正在删除 HDC Interface...' -ForegroundColor Red"
powershell -Command "    Get-PnpDevice | Where-Object {$_.FriendlyName -like '*HDC Interface*'} | ForEach-Object {"
powershell -Command "        Write-Host ('正在删除: ' + $_.FriendlyName) -ForegroundColor Yellow"
powershell -Command "        try {"
powershell -Command "            pnputil.exe /remove-device $_.InstanceId"
powershell -Command "            Write-Host '   删除成功' -ForegroundColor Green"
powershell -Command "        } catch {"
powershell -Command "            Write-Host '   删除失败' -ForegroundColor Red"
powershell -Command "        }"
powershell -Command "    }"
powershell -Command "}"

echo.
echo 设备删除操作完成！
echo 建议重新启动计算机以使更改生效。
pause
goto :eof

:exit_script
echo 操作已取消。
pause
```

## 可选版本：更详细的设备信息显示

如果您希望看到更详细的设备信息，这里还有一个增强版本：

```batch
@echo off
chcp 65001 >nul
title 设备清理脚本 - 详细扫描模式

echo ===============================================
echo           设备清理脚本（详细扫描模式）
echo ===============================================
echo.
echo 正在扫描设备，请稍候...
echo.

:: 使用PowerShell获取更详细的设备信息
powershell -Command "& {
    Write-Host '扫描结果：' -ForegroundColor Cyan
    Write-Host '==========' -ForegroundColor Cyan
    
    $devices = Get-PnpDevice | Where-Object {
        $_.FriendlyName -like '*HDC Device*' -or 
        $_.FriendlyName -like '*Harmony OS Device*' -or 
        $_.FriendlyName -like '*HDC Interface*'
    }
    
    if ($devices.Count -eq 0) {
        Write-Host '未找到任何匹配的设备。' -ForegroundColor Yellow
        exit
    }
    
    $counter = 1
    foreach ($device in $devices) {
        Write-Host ('设备 ' + $counter + ':') -ForegroundColor White
        Write-Host ('  名称: ' + $device.FriendlyName) -ForegroundColor Gray
        Write-Host ('  实例ID: ' + $device.InstanceId) -ForegroundColor Gray
        Write-Host ('  状态: ' + $device.Status) -ForegroundColor Gray
        Write-Host ('  类: ' + $device.Class) -ForegroundColor Gray
        Write-Host ''
        $counter++
    }
    
    Write-Host ('总计找到 ' + $devices.Count + ' 个设备') -ForegroundColor Cyan
}"

echo.
set /p choice=是否要删除以上所有设备？(Y/N): 
if /i "%choice%"=="y" (
    echo 开始删除设备...
    powershell -Command "Get-PnpDevice | Where-Object {$_.FriendlyName -like '*HDC Device*' -or $_.FriendlyName -like '*Harmony OS Device*' -or $_.FriendlyName -like '*HDC Interface*'} | ForEach-Object { pnputil.exe /remove-device $_.InstanceId }"
    echo 删除操作完成！
) else (
    echo 操作已取消。
)

pause
```

## 🆕 脚本改进特点

### 🔍 **扫描阶段**
- **显示设备详情**：在删除前显示所有匹配设备的详细信息
- **设备计数**：明确告知用户找到的设备数量
- **实例ID显示**：显示每个设备的唯一标识符

### ✅ **确认机制**
- **二次确认**：用户在看到具体设备列表后再决定是否删除
- **清晰提示**：显示"共找到 X 个设备"的明确信息

### 🛡️ **安全增强**
- **零设备处理**：如果没有找到设备，直接退出而不询问
- **操作反馈**：显示每个设备的删除状态（成功/失败）

## 使用流程

1. **以管理员身份运行**脚本
2. **查看扫描结果** - 脚本会列出所有匹配的设备
3. **做出决定** - 基于显示的设备列表选择 Y/N
4. **执行操作** - 只有确认后才执行删除

这样的设计让您能够清楚地知道将要删除哪些设备，避免误操作，更加安全可靠。