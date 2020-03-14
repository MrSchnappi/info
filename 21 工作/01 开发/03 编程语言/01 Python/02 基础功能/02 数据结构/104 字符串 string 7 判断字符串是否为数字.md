---
title: 104 字符串 string 7 判断字符串是否为数字
toc: true
date: 2018-06-11 08:14:49
---


# 判断字符串是否为数字





在从文本中读取数据的时候，有时候会要判断读进来的 str 是不是数字。而：

- Python isdigit() 方法检测字符串是否只由数字组成。 但是对于 0.02 这种就会认为是 False
- Python isnumeric() 方法检测字符串是否只由数字组成。这种方法是只针对 unicode 对象。


因此可以自己进行判断。

举例：

```py
def is_number(s):
    try:
        float(s)
        return True
    except ValueError:
        pass

    try:
        import unicodedata
        unicodedata.numeric(s)
        return True
    except (TypeError, ValueError):
        pass

    return False

# 测试字符串和数字
print(is_number('foo'))   # False
print(is_number('1'))     # True
print(is_number('1.3'))   # True
print(is_number('-1.37')) # True
print(is_number('1e3'))   # True

# 测试 Unicode
# 阿拉伯语 5
print(is_number('٥'))  # True
# 泰语 2
print(is_number('๒'))  # True
# 中文数字
print(is_number('四')) # True
# 版权号
print(is_number('©'))  # False
```

输出：


```
False
True
True
True
True
True
True
True
False
```







# 相关

- [Python 判断字符串是否为数字](http://www.runoob.com/Python3/Python3-check-is-number.html)
