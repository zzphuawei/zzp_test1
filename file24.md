# Windows Python 文件夹增量备份脚本

下面是一个Python脚本，用于备份指定的文件夹，只备份自上次备份以来新增或修改过的文件，并将它们压缩到一个ZIP文件中。

```python
import os
import zipfile
import json
from datetime import datetime
import argparse

class IncrementalBackup:
    def __init__(self, backup_zip_path, record_file_path):
        self.backup_zip_path = backup_zip_path
        self.record_file_path = record_file_path
        self.backup_record = self.load_backup_record()
        
    def load_backup_record(self):
        """加载备份记录文件"""
        if os.path.exists(self.record_file_path):
            try:
                with open(self.record_file_path, 'r') as f:
                    return json.load(f)
            except:
                return {}
        return {}
    
    def save_backup_record(self):
        """保存备份记录文件"""
        with open(self.record_file_path, 'w') as f:
            json.dump(self.backup_record, f, indent=4)
    
    def get_file_hash(self, file_path):
        """获取文件的哈希值（使用文件大小和修改时间作为简易哈希）"""
        try:
            stat = os.stat(file_path)
            return f"{stat.st_size}_{stat.st_mtime}"
        except OSError:
            return None
    
    def should_backup(self, file_path):
        """检查文件是否需要备份"""
        file_hash = self.get_file_hash(file_path)
        if file_hash is None:
            return False
        
        # 如果文件不在记录中，或者哈希值不同，则需要备份
        if file_path not in self.backup_record or self.backup_record[file_path] != file_hash:
            return True
        
        return False
    
    def backup_folders(self, folders_to_backup):
        """备份文件夹列表"""
        # 确保备份目录存在
        os.makedirs(os.path.dirname(self.backup_zip_path), exist_ok=True)
        
        # 打开或创建ZIP文件
        with zipfile.ZipFile(self.backup_zip_path, 'a', compression=zipfile.ZIP_DEFLATED) as backup_zip:
            for folder in folders_to_backup:
                if not os.path.exists(folder):
                    print(f"警告: 文件夹不存在: {folder}")
                    continue
                
                print(f"正在备份文件夹: {folder}")
                
                # 遍历文件夹中的所有文件
                for root, dirs, files in os.walk(folder):
                    for file in files:
                        file_path = os.path.join(root, file)
                        relative_path = os.path.relpath(file_path, os.path.dirname(folder))
                        
                        if self.should_backup(file_path):
                            try:
                                backup_zip.write(file_path, relative_path)
                                # 更新备份记录
                                self.backup_record[file_path] = self.get_file_hash(file_path)
                                print(f"已备份: {file_path}")
                            except Exception as e:
                                print(f"备份失败 {file_path}: {str(e)}")
                        else:
                            print(f"跳过(已备份): {file_path}")
        
        # 保存备份记录
        self.save_backup_record()
        print(f"备份完成! 备份文件保存在: {self.backup_zip_path}")

def main():
    parser = argparse.ArgumentParser(description='增量备份文件夹到ZIP文件')
    parser.add_argument('folders', nargs='+', help='要备份的文件夹路径')
    parser.add_argument('-o', '--output', default='backup.zip', help='输出ZIP文件路径(默认: backup.zip)')
    parser.add_argument('-r', '--record', default='backup_record.json', help='备份记录文件路径(默认: backup_record.json)')
    
    args = parser.parse_args()
    
    # 创建备份实例
    backup = IncrementalBackup(args.output, args.record)
    
    # 执行备份
    backup.backup_folders(args.folders)

if __name__ == "__main__":
    main()
```

## 使用方法

1. 将上述代码保存为 `incremental_backup.py`

2. 通过命令行运行脚本：
   ```
   python incremental_backup.py 文件夹路径1 文件夹路径2 ... -o 备份文件.zip -r 记录文件.json
   ```

3. 参数说明：
   - 位置参数：要备份的文件夹路径（可以指定多个）
   - `-o` 或 `--output`: 指定输出的ZIP文件路径（默认为backup.zip）
   - `-r` 或 `--record`: 指定备份记录文件路径（默认为backup_record.json）

## 示例

```bash
# 备份两个文件夹到默认的backup.zip文件
python incremental_backup.py C:\重要文档 D:\项目文件

# 备份指定文件夹到自定义位置
python incremental_backup.py C:\用户数据 -o D:\备份\我的备份.zip -r D:\备份\备份记录.json
```

## 功能特点

1. **增量备份**：只备份自上次备份以来新增或修改过的文件
2. **记录管理**：使用JSON文件记录已备份文件的状态
3. **简单哈希**：使用文件大小和修改时间判断文件是否变化
4. **错误处理**：跳过无法访问的文件并显示警告
5. **相对路径**：在ZIP文件中保持文件的相对路径结构

## 注意事项

1. 首次运行时会备份所有文件，后续运行只会备份新增或修改的文件
2. 备份记录文件不应删除，否则会导致重新备份所有文件
3. 脚本不会删除ZIP中已不存在于源文件夹中的文件

如果需要更强大的备份功能，可以考虑使用专门的备份工具如rsync、Duplicati等。