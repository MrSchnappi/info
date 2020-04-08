
# Shell 变量

## 变量

### 定义变量

举例：

```sh
your_name="tom"
echo $your_name
your_name="alibaba"
echo $your_name
```

说明：

- 定义变量时，变量名不加美元符号。
- 第二次赋值的时候不能写 `$your_name="alibaba"`，使用变量的时候才加美元符。


注意：

- 变量名和等号之间不能有空格，这可能和你熟悉的所有编程语言都不一样。

变量名的命名须遵循如下规则：

- 首个字符必须为字母（a-z，A-Z）。
- 中间不能有空格，可以使用下划线（_）。
- 不能使用标点符号。
- 不能使用 bash 里的关键字（可用 help 命令查看保留关键字）。


除了显式地直接赋值，还可以用语句给变量赋值，如：


```sh
for file in `ls /etc`
```

以上语句将 `/etc` 下目录的文件名循环出来。


### 使用变量


使用一个定义过的变量，只要在变量名前面加美元符号即可，如：


```sh
your_name="qinjx"
echo $your_name
echo ${your_name}
```

说明：

- 变量名外面的花括号是可选的，加不加都行，推荐给所有变量加上花括号，这是个好的编程习惯。加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

```sh
for skill in Ada Coffe Action Java do
    echo "I am good at ${skill}Script"
done
```


## 字符串

### 字符串使用


字符串是 shell 编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了）。

字符串可以用单引号，也可以用双引号，也可以不用引号。


举例：

```sh
str='this is a string'

your_name='qinjx'
str="Hello, I know your are \"$your_name\"! \n"
```


注意：

- 单引号：
  - 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
  - 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。
- 双引号的优点：
  - 双引号里可以有变量。
  - 双引号里可以出现转义字符。

### 字符串拼接

```sh
your_name="qinjx"
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting $greeting_1
```

输出：

```

```




### 获取字符串长度


{% raw %}

string="abcd"
echo ${#string} #输出 4

{% endraw %}




### 提取子字符串




    string="alibaba is a great company"
    echo ${string:1:4} #输出 liba





### 查找子字符串




    string="alibaba is a great company"
    echo `expr index "$string" is`



**注意：** 以上脚本中 "`" 是反引号，而不是单引号 "'"，不要看错了哦。



* * *





## Shell 数组


bash支持一维数组（不支持多维数组），并且没有限定数组的大小。

类似与 C 语言，数组元素的下标由 0 开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式，其值应大于或等于 0。


### 定义数组


在 Shell 中，用括号来表示数组，数组元素用"空格"符号分割开。定义数组的一般形式为：


    数组名=(值 1 值 2 ... 值 n)



例如：


    array_name=(value0 value1 value2 value3)



或者


    array_name=(
    value0
    value1
    value2
    value3
    )



还可以单独定义数组的各个分量：


    array_name[0]=value0
    array_name[1]=value1
    array_name[n]=valuen



可以不使用连续的下标，而且下标的范围没有限制。


### 读取数组


读取数组元素值的一般格式是：


    ${数组名[下标]}



例如：


    valuen=${array_name[n]}



使用@符号可以获取数组中的所有元素，例如：


    echo ${array_name[@]}





### 获取数组的长度


获取数组长度的方法与获取字符串长度的方法相同，例如：

{% raw %}


    # 取得数组元素的个数
    length=${#array_name[@]}
    # 或者
    length=${#array_name[*]}
    # 取得数组单个元素的长度
    lengthn=${#array_name[n]}

{% endraw %}



## Shell 注释


以"#"开头的行就是注释，会被解释器忽略。

sh里没有多行注释，只能每一行加一个#号。只能像这样：

```
#--------------------------------------------
# 这是一个自动打 ipa 的脚本，基于 webfrogs 的 ipa-build书写：
# https://github.com/webfrogs/xcode_shell/blob/master/ipa-build
# 功能：自动为 etao ios app打包，产出物为 14 个渠道的 ipa 包
# 特色：全自动打包，不需要输入任何参数
#--------------------------------------------
##### 用户配置区 开始 #####
#
#
# 项目根目录，推荐将此脚本放在项目的根目录，这里就不用改了
# 应用名，确保和 Xcode 里 Product 下的 target_name.app名字一致
#
##### 用户配置区 结束  #####
```


如果在开发过程中，遇到大段的代码需要临时注释起来，过一会儿又取消注释，怎么办呢？

每一行加个#符号太费力了，可以把这一段要注释的代码用一对花括号括起来，定义成一个函数，没有地方调用这个函数，这块代码就不会执行，达到了和注释一样的效果。




# 相关

- [Linux教程](https://www.w3cschool.cn/linux/)
