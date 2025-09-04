# Python 自定义 with 语句详细教程

`with` 语句是 Python 中用于资源管理的重要特性，它通过上下文管理器自动管理资源的获取和释放。本教程将详细介绍如何创建自定义的 with 语句支持。

## 1. 理解 with 语句和上下文管理器

`with` 语句用于包装代码块的执行，它确保进入和退出代码块时执行特定的操作。最常见的用途是文件操作：

```python
with open('file.txt', 'r') as f:
    content = f.read()
# 文件在这里会自动关闭
```

## 2. 实现上下文管理器的两种方法

### 方法一：使用类实现上下文管理器

要实现一个上下文管理器，需要定义一个包含 `__enter__()` 和 `__exit__()` 方法的类。

```python
class CustomContextManager:
    def __init__(self, name):
        self.name = name
        print(f"初始化 {self.name}")
    
    def __enter__(self):
        print(f"进入上下文 {self.name}")
        # 返回的值可以被 as 子句接收
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"退出上下文 {self.name}")
        # 处理异常
        if exc_type is not None:
            print(f"发生异常: {exc_type}, {exc_val}")
        # 返回 True 表示异常已被处理，False 或 None 表示异常继续传播
        return True

# 使用示例
with CustomContextManager("测试") as manager:
    print(f"在上下文中工作，名称: {manager.name}")
    # 可以故意引发一个异常测试异常处理
    # raise ValueError("测试异常")
```

### 方法二：使用 contextlib 模块

Python 的 `contextlib` 模块提供了更简洁的方式来创建上下文管理器。

```python
from contextlib import contextmanager

@contextmanager
def custom_context_manager(name):
    print(f"进入上下文 {name}")
    try:
        yield name  # yield 的值会被 as 子句接收
    except Exception as e:
        print(f"发生异常: {e}")
        # 可以决定是否重新抛出异常
        # raise
    finally:
        print(f"退出上下文 {name}")

# 使用示例
with custom_context_manager("函数式管理器") as name:
    print(f"在上下文中工作，名称: {name}")
    # raise RuntimeError("测试异常")
```

## 3. 实际应用示例

### 示例 1：计时器上下文管理器

```python
import time
from contextlib import contextmanager

@contextmanager
def timer(name):
    start = time.time()
    try:
        yield
    finally:
        end = time.time()
        print(f"{name} 耗时: {end - start:.4f} 秒")

# 使用示例
with timer("计算任务"):
    # 模拟耗时操作
    time.sleep(1)
    result = sum(range(1000000))
```

### 示例 2：数据库连接管理器

```python
class DatabaseConnection:
    def __init__(self, db_name):
        self.db_name = db_name
        self.connection = None
    
    def __enter__(self):
        print(f"连接到数据库 {self.db_name}")
        # 这里应该是实际的连接代码
        self.connection = f"Connection to {self.db_name}"
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print(f"关闭数据库连接 {self.db_name}")
        # 这里应该是实际的关闭连接代码
        self.connection = None
        if exc_type:
            print(f"操作完成但有异常: {exc_type}: {exc_val}")
        return False  # 让异常传播

# 使用示例
with DatabaseConnection("my_database") as conn:
    print(f"使用连接: {conn}")
    # 执行数据库操作
    # 如果发生异常，连接仍会被正确关闭
```

### 示例 3：临时目录管理器

```python
import tempfile
import shutil
from contextlib import contextmanager

@contextmanager
def temporary_directory():
    temp_dir = tempfile.mkdtemp()
    try:
        yield temp_dir
    finally:
        shutil.rmtree(temp_dir)

# 使用示例
with temporary_directory() as temp_dir:
    print(f"使用临时目录: {temp_dir}")
    # 在临时目录中创建工作文件
    with open(f"{temp_dir}/test.txt", "w") as f:
        f.write("测试内容")
# 临时目录在这里已被自动删除
```

## 4. 处理异常

上下文管理器的一个重要功能是正确处理异常：

```python
class ExceptionHandlingContext:
    def __enter__(self):
        print("进入上下文")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is not None:
            print(f"捕获到异常: {exc_type.__name__}: {exc_val}")
            # 返回 True 表示异常已被处理，不会传播
            return True
        print("正常退出")
    
    def risky_operation(self):
        print("执行风险操作")
        raise ValueError("出错了!")

# 使用示例
with ExceptionHandlingContext() as manager:
    manager.risky_operation()

print("程序继续执行，异常已被处理")
```

## 5. 异步上下文管理器

Python 3.5+ 支持异步上下文管理器，使用 `async with` 语句：

```python
import asyncio

class AsyncContextManager:
    async def __aenter__(self):
        print("异步进入上下文")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("异步退出上下文")
        if exc_type:
            print(f"异步操作异常: {exc_type}")

async def main():
    async with AsyncContextManager() as manager:
        print("在异步上下文中工作")

# 运行异步示例
# asyncio.run(main())
```

## 总结

自定义 `with` 语句的关键是理解上下文管理器协议，即实现 `__enter__` 和 `__exit__` 方法。通过使用上下文管理器，你可以：

1. 确保资源被正确释放
2. 简化异常处理
3. 减少重复代码
4. 提高代码可读性

根据你的需求选择类实现或 `@contextmanager` 装饰器实现方式，前者提供更多控制，后者更加简洁。
---
# 在 `__exit__` 方法中返回什么

在 Python 的上下文管理器中，`__exit__` 方法的返回值非常重要，它决定了异常是否会被传播。让我详细解释这个问题。

## `__exit__` 方法的签名

```python
def __exit__(self, exc_type, exc_value, traceback):
    # 处理代码
    return something  # 布尔值或 None
```

## 返回值的作用

`__exit__` 方法的返回值控制着异常的处理行为：

### 1. 返回 `True` 或真值

表示异常已被处理，不会传播到 `with` 块之外。

```python
class ExceptionSwallower:
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        print(f"捕获到异常: {exc_type.__name__}: {exc_value}")
        return True  # 异常已被处理，不会传播

# 使用示例
with ExceptionSwallower():
    raise ValueError("这是一个测试异常")

print("程序继续执行，异常已被吞掉")
```

### 2. 返回 `False`、`None` 或假值

表示异常未被处理，会传播到 `with` 块之外。

```python
class ExceptionPropagator:
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        print(f"检测到异常: {exc_type.__name__}: {exc_value}")
        return False  # 异常会传播

# 使用示例
try:
    with ExceptionPropagator():
        raise ValueError("这是一个测试异常")
except ValueError as e:
    print(f"在外层捕获到异常: {e}")

print("程序继续执行")
```

### 3. 根据异常类型决定返回值

你可以根据异常的类型决定是否处理异常：

```python
class SelectiveExceptionHandler:
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is None:
            print("没有异常发生")
            return False
        
        # 只处理 ValueError，让其他异常传播
        if exc_type == ValueError:
            print(f"处理 ValueError: {exc_value}")
            return True  # 吞掉 ValueError
        
        print(f"让 {exc_type.__name__} 传播")
        return False  # 让其他异常传播

# 使用示例
try:
    with SelectiveExceptionHandler():
        # 这个异常会被处理
        raise ValueError("ValueError 会被处理")
except:
    print("这里不会执行，因为异常已被处理")

try:
    with SelectiveExceptionHandler():
        # 这个异常会传播
        raise RuntimeError("RuntimeError 会传播")
except RuntimeError as e:
    print(f"在外层捕获到: {e}")
```

## 实际应用场景

### 场景 1：数据库事务管理

```python
class DatabaseTransaction:
    def __enter__(self):
        print("开始事务")
        return self
    
    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is not None:
            print("回滚事务")
            # 返回 True 表示异常已处理，事务已回滚
            return True
        else:
            print("提交事务")
            return False

# 使用示例
with DatabaseTransaction():
    # 模拟数据库操作
    print("执行数据库操作")
    # 如果这里发生异常，事务会回滚
    # raise Exception("操作失败")
```

### 场景 2：文件操作

```python
class SafeFileWriter:
    def __init__(self, filename):
        self.filename = filename
        self.temp_filename = filename + ".tmp"
    
    def __enter__(self):
        self.file = open(self.temp_filename, 'w')
        return self.file
    
    def __exit__(self, exc_type, exc_value, traceback):
        self.file.close()
        
        if exc_type is not None:
            # 发生异常，删除临时文件
            import os
            os.remove(self.temp_filename)
            print("操作失败，已删除临时文件")
            # 返回 False 让异常传播
            return False
        else:
            # 操作成功，重命名文件
            import os
            os.rename(self.temp_filename, self.filename)
            print("操作成功")
            return True

# 使用示例
try:
    with SafeFileWriter("important.txt") as f:
        f.write("重要数据")
        # 模拟写入过程中发生异常
        # raise IOError("写入错误")
except Exception as e:
    print(f"捕获到异常: {e}")
```

## 总结

- 返回 `True`：表示异常已被处理，不会传播到 `with` 块之外
- 返回 `False` 或 `None`：表示异常未被处理，会传播到 `with` 块之外
- 可以根据异常类型有条件地决定返回值
- 如果没有异常发生（`exc_type` 为 `None`），返回值会被忽略

选择正确的返回值取决于你的使用场景：
- 如果你希望完全处理异常并继续执行，返回 `True`
- 如果你希望外层代码能够捕获和处理异常，返回 `False`
- 如果你需要根据异常类型做出不同处理，可以使用条件判断