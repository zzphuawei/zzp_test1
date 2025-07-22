### Python 装饰器详细教程

装饰器（Decorator）是 Python 中一种强大的语法特性，**允许在不修改原函数代码的前提下，动态扩展函数功能**。它基于高阶函数和闭包实现，广泛应用于日志记录、性能测试、权限校验等场景。

---

#### **一、装饰器核心概念**
1. **本质**：装饰器是一个**返回函数的高阶函数**
2. **原理**：使用 `@decorator` 语法糖，等价于 `func = decorator(func)`
3. **核心思想**：遵循 **开放封闭原则**（对扩展开放，对修改封闭）

---

#### **二、基础装饰器实现**
```python
# 1. 定义装饰器
def simple_decorator(func):
    def wrapper():
        print("函数执行前操作")
        func()  # 执行原函数
        print("函数执行后操作")
    return wrapper

# 2. 使用装饰器
@simple_decorator
def say_hello():
    print("Hello!")

# 3. 调用函数
say_hello()
```
**输出**：
```
函数执行前操作
Hello!
函数执行后操作
```

---

#### **三、处理带参数的函数**
使用 `*args` 和 `**kwargs` 接收任意参数：
```python
def param_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"参数: {args}, {kwargs}")
        return func(*args, **kwargs)
    return wrapper

@param_decorator
def greet(name, greeting="Hi"):
    print(f"{greeting}, {name}!")

greet("Alice", greeting="Hello")
```
**输出**：
```
参数: ('Alice',), {'greeting': 'Hello'}
Hello, Alice!
```

---

#### **四、带参数的装饰器**
需要三层嵌套函数：
```python
def repeat(num):  # 装饰器参数
    def decorator(func):  # 被装饰函数
        def wrapper(*args, **kwargs):  # 函数参数
            for _ in range(num):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def say_hi():
    print("Hi!")

say_hi()
```
**输出**：
```
Hi!
Hi!
Hi!
```

---

#### **五、类装饰器**
通过实现 `__call__` 方法使类可调用：
```python
class ClassDecorator:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        print("类装饰器前置操作")
        result = self.func(*args, **kwargs)
        print("类装饰器后置操作")
        return result

@ClassDecorator
def calculate(a, b):
    return a + b

print(calculate(2, 3))
```
**输出**：
```
类装饰器前置操作
类装饰器后置操作
5
```

---

#### **六、保留元信息**
使用 `functools.wraps` 保持原函数属性：
```python
from functools import wraps

def meta_decorator(func):
    @wraps(func)  # 保留元信息
    def wrapper(*args, **kwargs):
        """Wrapper Docstring"""
        return func(*args, **kwargs)
    return wrapper

@meta_decorator
def example():
    """Original Docstring"""
    pass

print(example.__name__)  # 输出: example
print(example.__doc__)   # 输出: Original Docstring
```

---

#### **七、装饰器链**
多个装饰器按**从下往上**的顺序执行：
```python
def decorator1(func):
    def wrapper():
        print("Decorator 1")
        func()
    return wrapper

def decorator2(func):
    def wrapper():
        print("Decorator 2")
        func()
    return wrapper

@decorator1
@decorator2
def chain():
    print("Main Function")

chain()
```
**输出**：
```
Decorator 1
Decorator 2
Main Function
```

---

#### **八、实际应用场景**
1. **计时装饰器**：
```python
import time

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} 耗时 {end - start:.4f}秒")
        return result
    return wrapper
```

2. **权限验证装饰器**：
```python
def login_required(func):
    def wrapper(user, *args, **kwargs):
        if user.is_authenticated:
            return func(user, *args, **kwargs)
        else:
            raise PermissionError("需要登录")
    return wrapper
```

3. **日志记录装饰器**：
```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"[LOG] 调用 {func.__name__}，参数: {args}, {kwargs}")
        return func(*args, **kwargs)
    return wrapper
```

---

#### **九、总结**
| 特性             | 说明                          |
|------------------|-----------------------------|
| 核心组件         | 高阶函数 + 闭包               |
| 语法糖           | `@decorator`                 |
| 参数处理         | 使用 `*args, **kwargs`       |
| 元信息保留       | `functools.wraps`            |
| 装饰器参数       | 三层嵌套函数                  |
| 执行顺序         | 从下往上（靠近函数定义的先执行）|

掌握装饰器能大幅提升代码的**可重用性**和**可维护性**，是 Python 高级编程的必备技能！