我来为您介绍几种在PyCharm中设置自定义函数注释模板的方法：

## 方法一：使用Live Template（推荐）

### 步骤：
1. **打开设置**：`File → Settings` (Windows/Linux) 或 `PyCharm → Preferences` (Mac)
2. **进入Live Templates**：`Editor → Live Templates`
3. **添加Python组**：点击右侧`+`，选择`Template Group`，命名为"Python"
4. **创建新模板**：选中"Python"组，点击`+`，选择`Live Template`

### 常用模板示例：

#### 1. **Google风格文档字符串**
```python
"""
$NAME$

$PARAMS$
$RETURNS$
$RAISES$
"""
```
变量设置：
- `$NAME$`：`variableOfType("")`
- `$PARAMS$`：自定义groovyScript（见下文）
- `$RETURNS$`：`methodReturnType()`
- `$RAISES$`：可选的异常说明

#### 2. **Sphinx风格**
```python
"""
$NAME$

:$PARAMS$
:$RETURNS$
:$RAISES$
"""
```

#### 3. **详细参数模板**
```
"""
$NAME$

Args:
$ARGS$

Returns:
$RETURNS$

Raises:
$RAISES$

Examples:
    >>>
"""
```
在`Edit variables`中设置：
- `$ARGS$`：使用以下groovyScript：
```groovy
def result = '';
def params = _1.replaceAll('[\\[\\]\\s]', '').split(',').toList();
for(i = 0; i < params.size(); i++) {
    if(params[i] != '') {
        result += '    ' + params[i] + ' (type): description\n';
    }
};
return result;
```
表达式填写：`groovyScript("上述代码", methodParameters())`

## 方法二：使用File and Code Templates

对于新建文件时的模板：
1. `Settings → Editor → File and Code Templates`
2. 选择"Python Script"
3. 添加自定义模板，如：
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
@filename: ${NAME}.py
@author: YourName
@date: ${DATE} ${TIME}
@description: 
"""

## 方法三：使用Docstring插件

### 1. **安装插件**：
- 搜索并安装 "Python Docstring Generator"

### 2. **配置插件**：
- 支持多种风格：Google、NumPy、Sphinx
- 设置快捷键生成文档字符串

## 实用配置示例

### 我的常用模板：
```python
"""
功能描述

Args:
    $args$

Returns:
    $returns$: 返回值描述

Raises:
    Exception: 异常描述

Examples:
    >>> function_name(arg1, arg2)
    result
"""
```
缩写：`doc`
上下文：选择`Python`

### 使用技巧：

1. **快捷键**：
   - 在函数定义下方输入`"""`并回车，自动生成模板
   - 或输入缩写（如`doc`）按Tab键

2. **参数自动填充**：
```groovy
// 自动提取参数的groovyScript
groovyScript("""
def result='';
def params=_1.replaceAll('[\\\\[\\\\]\\\\s]', '').split(',').toList();
for(i=0; i<params.size(); i++) {
    if(params[i]!='') {
        result+='    ' + params[i].replaceFirst('(\\w+)\\s*:.*', '$1') + ' (' + params[i].replaceFirst('\\w+\\s*:(.*)', '$1').trim() + '): \\n';
    }
};
return result;
""", methodParameters())
```

3. **应用范围设置**：
   - 在模板设置中点击`Change`
   - 选择`Python → Function`和`Python → Class`

## 最佳实践建议

1. **保持一致性**：团队使用同一种文档字符串风格
2. **包含关键信息**：
   - 功能描述
   - 参数类型和说明
   - 返回值说明
   - 异常说明
   - 使用示例

3. **配置示例文件**：
```python
# .pycharm/template_settings.py
DOCSTRING_TEMPLATE = {
    'google': '''
    {function_name}
    
    Args:
        {args}
        
    Returns:
        {returns}
    ''',
    'numpy': '''
    {function_name}
    
    Parameters
    ----------
    {args}
    
    Returns
    -------
    {returns}
    '''
}
```

这样设置后，输入预设的缩写（如`doc`）即可快速生成规范的函数注释模板。