---
title: Python JSON
toc: true
date: 2018-06-22 22:20:02
---


# 可以补充进来的

- 最好都介绍下，不仅仅是 Json
- 常用的用法也要整理下。

# Json

Json 在传输数据的时候，或者使用不同的语言之间来传输数据的时候经常会用到。因此还是要好好整理下的。常用的用法也要整理下。

# Json 的编码解码

在使用 Python 编码或解码 JSON 数据前，我们需要先安装 JSON 模块。本教程我们会下载 [Demjson](https://deron.meranda.us/Python/demjson/) 并安装：

```shell
$tar xvfz demjson-1.6.tar.gz
$cd demjson-1.6
$Python setup.py install
```

## JSON 函数


<table class="reference" >
<tbody >
<tr >
函数
描述
</tr>
<tr >

<td >encode
</td>

<td >将 Python 对象编码成 JSON 字符串
</td>
</tr>
<tr >

<td >decode
</td>

<td >将已编码的 JSON 字符串解码为 Python 对象
</td>
</tr>
</tbody>
</table>


## encode
Python encode() 函数用于将 Python 对象编码成 JSON 字符串。
### 语法
    demjson.encode(self, obj, nest_level=0)

### 实例

以下实例将数组编码为 JSON 格式数据：

```Python
#!/usr/bin/Python
import demjson

data = [ { 'a' : 1, 'b' : 2, 'c' : 3, 'd' : 4, 'e' : 5 } ]

json = demjson.encode(data)
print json
```


以上代码执行结果为：

```text
    [{"a":1,"b":2,"c":3,"d":4,"e":5}]
```






* * *





## decode


Python 可以使用 demjson.decode() 函数解码 JSON 数据。该函数返回 Python 字段的数据类型。


### 语法




    demjson.decode(self, txt)





### 实例


以下实例展示了 Python 如何解码 JSON 对象：


    #!/usr/bin/Python
    import demjson

    json = '{"a":1,"b":2,"c":3,"d":4,"e":5}';

    text = demjson.decode(json)
    print  text



以上代码执行结果为：


    {u'a': 1, u'c': 3, u'b': 2, u'e': 5, u'd': 4}


# 相关

- [Python基础教程 w3cschool](https://www.w3cschool.cn/Python/)
- [Python 3 教程 菜鸟教程](http://www.runoob.com/Python3/Python3-tutorial.html)
- [Python JSON](http://www.runoob.com/Python/Python-json.html)
