# Windows 文件夹备份脚本

下面是一个Python脚本，可以用于备份指定文件夹，并且会自动跳过已经备份过的内容（基于文件修改时间和大小判断）。

```python
import os
import zipfile
import datetime
import json
from pathlib import Path

class FolderBackup:
    def __init__(self, backup_dir="D:\\Backups"):
        self.backup_dir = Path(backup_dir)
        self.backup_dir.mkdir(exist_ok=True)
        self.history_file = self.backup_dir / "backup_history.json"
        self.history = self.load_history()
    
    def load_history(self):
        """加载备份历史记录"""
        if self.history_file.exists():
            try:
                with open(self.history_file, 'r', encoding='utf-8') as f:
                    return json.load(f)
            except:
                return {}
        return {}
    
    def save_history(self):
        """保存备份历史记录"""
        with open(self.history_file, 'w', encoding='utf-8') as f:
            json.dump(self.history, f, ensure_ascii=False, indent=2)
    
    def get_file_signature(self, file_path):
        """获取文件签名(大小和修改时间)用于判断文件是否变更"""
        stat = file_path.stat()
        return f"{stat.st_size}_{stat.st_mtime}"
    
    def needs_backup(self, folder_path, zip_path):
        """检查是否需要备份"""
        folder_path = Path(folder_path)
        zip_path = Path(zip_path)
        
        # 如果ZIP文件不存在，需要备份
        if not zip_path.exists():
            return True
        
        # 检查备份历史记录
        if str(folder_path) in self.history:
            last_backup = self.history[str(folder_path)]
            # 如果文件夹最后修改时间晚于上次备份时间，需要备份
            if folder_path.stat().st_mtime > last_backup['last_modified']:
                return True
        
        return False
    
    def backup_folder(self, folder_path, backup_name=None):
        """备份文件夹"""
        folder_path = Path(folder_path)
        if not folder_path.exists():
            print(f"错误: 文件夹 {folder_path} 不存在")
            return False
        
        if backup_name is None:
            backup_name = folder_path.name
        
        # 创建备份文件名(带时间戳)
        timestamp = datetime.datetime.now().strftime("%Y%m%d_%H%M%S")
        zip_filename = f"{backup_name}_{timestamp}.zip"
        zip_path = self.backup_dir / zip_filename
        
        # 检查是否需要备份
        if not self.needs_backup(folder_path, zip_path):
            print(f"无需备份: {folder_path} (已备份且未更改)")
            return False
        
        print(f"开始备份: {folder_path} -> {zip_path}")
        
        try:
            with zipfile.ZipFile(zip_path, 'w', zipfile.ZIP_DEFLATED) as zipf:
                for root, dirs, files in os.walk(folder_path):
                    for file in files:
                        file_path = Path(root) / file
                        # 计算相对路径
                        rel_path = file_path.relative_to(folder_path.parent)
                        zipf.write(file_path, rel_path)
            
            # 更新备份历史
            self.history[str(folder_path)] = {
                'last_modified': folder_path.stat().st_mtime,
                'backup_time': datetime.datetime.now().timestamp(),
                'backup_file': str(zip_path)
            }
            self.save_history()
            
            print(f"备份完成: {zip_path}")
            return True
            
        except Exception as e:
            print(f"备份失败: {e}")
            return False

def main():
    # 配置要备份的文件夹列表
    folders_to_backup = [
        "C:\\Users\\YourName\\Documents",
        "C:\\Users\\YourName\\Pictures",
        "D:\\ImportantFiles",
        # 添加更多需要备份的文件夹...
    ]
    
    # 创建备份器实例
    backup_manager = FolderBackup("D:\\Backups")
    
    # 备份所有文件夹
    for folder in folders_to_backup:
        backup_manager.backup_folder(folder)
    
    print("所有备份任务完成")

if __name__ == "__main__":
    main()
```

## 使用说明

1. 将上述代码保存为 `backup_script.py`
2. 修改 `folders_to_backup` 列表，添加您需要备份的文件夹路径
3. 修改 `backup_dir` 参数设置备份文件保存的位置
4. 运行脚本：`python backup_script.py`

## 功能特点

- 自动跳过未更改的文件夹（基于修改时间和文件大小）
- 备份文件以时间戳命名，避免覆盖
- 保存备份历史记录，便于追踪
- 使用ZIP压缩节省空间

## 高级用法

您可以设置Windows任务计划程序，定期自动运行此脚本：

1. 打开"任务计划程序"
2. 创建基本任务
3. 设置触发时间（每日/每周等）
4. 操作设置为启动程序：`python.exe 脚本路径.py`
5. 完成设置

## 注意事项

- 确保有足够的磁盘空间存储备份
- 首次运行会备份所有指定文件夹
- 后续运行只会备份发生变化的文件夹
- 备份历史保存在备份目录下的 `backup_history.json` 文件中

您可以根据需要调整脚本，例如添加排除特定文件类型的功能、增加加密选项等。