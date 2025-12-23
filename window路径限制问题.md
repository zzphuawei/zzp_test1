## **Windows路径长度限制的解决方案**

Windows系统默认有**260字符的路径长度限制**（MAX_PATH），这确实是导致HDC传输失败的主要原因。以下是几种有效的解决方案：

## 1. **启用Windows长路径支持（推荐）**

### **方法一：通过组策略（Windows 10/11专业版/企业版）**
```batch
@echo off
:: 检查是否已启用长路径
reg query "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled

:: 启用长路径需要管理员权限
if exist "%SystemRoot%\System32\gpedit.msc" (
    echo 如果是专业版/企业版，可以通过组策略启用长路径
    echo 1. 运行 gpedit.msc
    echo 2. 计算机配置 -> 管理模板 -> 系统 -> 文件系统
    echo 3. 启用 "启用 Win32 长路径"
)
```

### **方法二：通过注册表（所有Windows版本）**
```batch
@echo off
:: 以管理员身份运行此脚本
reg add "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled /t REG_DWORD /d 1 /f
echo 请重启计算机使设置生效
pause
```

### **方法三：通过PowerShell**
```powershell
# 以管理员身份运行PowerShell
New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
    -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

## 2. **使用短文件名（8.3格式）**

Windows为每个文件和文件夹都保留了短名称（8.3格式）：

```batch
@echo off
:: 获取长路径的短名称
set "LONG_PATH=C:\非常长的文件夹名\另一个长文件夹\..."
dir /x %LONG_PATH%

:: 使用for循环获取短路径
for %%I in ("C:\非常长的文件夹名\另一个长文件夹\...") do (
    echo 短路径：%%~sI
    set "SHORT_PATH=%%~sI"
)

:: 使用短路径执行HDC命令
hdc file recv /data/local/tmp/file.txt "%SHORT_PATH%\file.txt"
```

## 3. **使用UNC路径前缀（立即生效）**

在路径前添加`\\?\`可以突破260字符限制：

```batch
@echo off
set "LONG_PATH=C:\超长路径\多层文件夹\..."
set "UNC_PATH=\\?\C:\超长路径\多层文件夹\..."

:: 方法1：直接使用UNC路径
hdc file recv /data/local/tmp/file.txt "\\?\C:\超长路径\...\file.txt"

:: 方法2：如果路径中有空格或特殊字符
set "ESCAPED_PATH=%UNC_PATH:\=\\%"
hdc file recv /data/local/tmp/file.txt "%ESCAPED_PATH%\file.txt"
```

## 4. **使用SUBST命令创建虚拟驱动器**

```batch
@echo off
:: 将长路径映射为虚拟驱动器
subst Z: "C:\非常非常长的文件夹路径\多层子文件夹\..."

:: 使用虚拟驱动器
hdc file recv /data/local/tmp/file.txt "Z:\file.txt"

:: 传输完成后解除映射
subst Z: /d
```

## 5. **使用符号链接**

```batch
@echo off
:: 创建符号链接（需要管理员权限）
mklink /D C:\shortlink "C:\非常长的文件夹路径\多层子文件夹\..."

:: 使用短链接路径
hdc file recv /data/local/tmp/file.txt "C:\shortlink\file.txt"

:: 删除符号链接
rmdir C:\shortlink
```

## 6. **修改接收路径到更短的位置**

最简单的临时解决方案：

```batch
@echo off
:: 使用根目录或较短的路径
set "SHORT_RECEIVE_PATH=C:\receive\"

:: 如果原路径太长，先接收到短路径
hdc file recv /data/local/tmp/file.txt "%SHORT_RECEIVE_PATH%temp_file.txt"

:: 然后移动到最终位置（移动操作通常不受路径长度限制）
move "%SHORT_RECEIVE_PATH%temp_file.txt" "C:\非常长的路径\...\file.txt"
```

## 7. **完整的批处理脚本示例**

```batch
@echo off
setlocal enabledelayedexpansion

:: 设置参数
set "DEVICE_FILE=/data/local/tmp/test.apk"
set "LONG_TARGET_PATH=C:\超长路径\多层文件夹\..."
set "SHORT_TEMP_PATH=C:\temp\"

:: 检查并创建目录
if not exist "%SHORT_TEMP_PATH%" mkdir "%SHORT_TEMP_PATH%"

:: 方法1：先尝试直接传输
echo 尝试直接传输...
hdc file recv %DEVICE_FILE% "%LONG_TARGET_PATH%\test.apk"

if %errorlevel% neq 0 (
    echo 直接传输失败，可能由于路径长度限制
    
    :: 方法2：使用短路径临时接收
    echo 使用临时路径接收...
    hdc file recv %DEVICE_FILE% "%SHORT_TEMP_PATH%temp.apk"
    
    if %errorlevel% equ 0 (
        :: 移动到目标位置
        echo 移动文件到目标位置...
        move "%SHORT_TEMP_PATH%temp.apk" "%LONG_TARGET_PATH%\test.apk"
        echo 文件传输并移动完成！
    ) else (
        echo 传输失败，请检查设备连接
    )
) else (
    echo 文件传输成功！
)

:: 清理临时文件
if exist "%SHORT_TEMP_PATH%temp.apk" del "%SHORT_TEMP_PATH%temp.apk"
endlocal
```

## 8. **检测路径长度并自动选择方案**

```batch
@echo off
setlocal enabledelayedexpansion

set "TARGET_PATH=%~1"
set "MAX_LENGTH=260"

:: 计算路径长度
set "TEST_PATH=%TARGET_PATH%\test.txt"
call :strlen TEST_PATH LENGTH

echo 目标路径长度：%LENGTH%
if %LENGTH% gtr %MAX_LENGTH% (
    echo 路径超过%MAX_LENGTH%字符，将使用替代方案
    
    :: 使用UNC路径
    set "UNC_PATH=\\?\%TARGET_PATH%"
    set "CMD_PATH=!UNC_PATH:\=\\!"
    
    hdc file recv %DEVICE_FILE% "!CMD_PATH!\file.txt"
) else (
    hdc file recv %DEVICE_FILE% "%TARGET_PATH%\file.txt"
)

endlocal
goto :eof

:strlen <stringVar> <resultVar>
(
    setlocal enabledelayedexpansion
    set "s=!%1!#"
    set "len=0"
    for %%P in (4096 2048 1024 512 256 128 64 32 16 8 4 2 1) do (
        if "!s:~%%P,1!" NEQ "" ( 
            set /a "len+=%%P"
            set "s=!s:~%%P!"
        )
    )
)
( 
    endlocal
    set "%2=%len%"
)
exit /b
```

## **最佳实践建议**

1. **首选方案**：启用Windows长路径支持（注册表方法）
2. **临时方案**：使用SUBST创建虚拟驱动器
3. **应急方案**：先接收到短路径，再移动到长路径
4. **开发建议**：在应用设计阶段就避免过深的目录结构

## **验证长路径是否生效**
```batch
@echo off
:: 创建一个超长路径测试
set "LONG_DIR=C:\%random%%random%%random%_%random%%random%%random%_%random%%random%%random%_%random%%random%%random%_%random%%random%%random%_%random%%random%%random%_%random%%random%%random%_%random%%random%%random%_%random%%random%%random%_%random%%random%%random%"

echo 测试路径：%LONG_DIR%
mkdir "%LONG_DIR%"
if exist "%LONG_DIR%" (
    echo 长路径创建成功！
    rmdir "%LONG_DIR%"
) else (
    echo 长路径创建失败，请启用长路径支持
)
```

启用长路径支持后，Windows可以支持最多**32,767字符**的路径长度，基本可以解决所有路径长度问题。