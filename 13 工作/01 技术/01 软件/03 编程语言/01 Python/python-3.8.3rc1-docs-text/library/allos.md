通用操作系统服务
****************

本章中描述的各模块提供了在（几乎）所有的操作系统上可用的操作系统特性的
接口，例如文件和时钟。这些接口通常以 Unix 或 C 接口为参照对象设计，不
过在大多数其他系统上也可用。这里有一个概述：

* "os" --- 多种操作系统接口

  * 文件名，命令行参数，以及环境变量。

  * 进程参数

  * 创建文件对象

  * 文件描述符操作

    * 查询终端的尺寸

    * 文件描述符的继承

  * 文件和目录

    * Linux 扩展属性

  * 进程管理

  * 调度器接口

  * 其他系统信息

  * 随机数

* "io" --- 处理流的核心工具

  * 概述

    * 文本 I/O

    * 二进制 I/O

    * 原始 I/O

  * 高阶模块接口

    * 内存中的流

  * 类的层次结构

    * I/O 基类

    * 原始文件 I/O

    * 缓冲流

    * 文本 I/O

  * 性能

    * 二进制 I/O

    * 文本 I/O

    * 多线程

    * 可重入性

* "time" --- 时间的访问和转换

  * 函数

  * Clock ID 常量

  * 时区常量

* "argparse" --- 命令行选项、参数和子命令解析器

  * 示例

    * 创建一个解析器

    * 添加参数

    * 解析参数

  * ArgumentParser 对象

    * prog

    * usage

    * description

    * epilog

    * parents

    * formatter_class

    * prefix_chars

    * fromfile_prefix_chars

    * argument_default

    * allow_abbrev

    * conflict_handler

    * add_help

  * add_argument() 方法

    * name or flags

    * action

    * nargs

    * const

    * default

    * type

    * choices

    * required

    * help

    * metavar

    * dest

    * Action classes

  * parse_args() 方法

    * Option value syntax

    * 无效的参数

    * 包含 "-" 的参数

    * 参数缩写（前缀匹配）

    * Beyond "sys.argv"

    * 命名空间对象

  * 其它实用工具

    * 子命令

    * FileType 对象

    * 参数组

    * Mutual exclusion

    * Parser defaults

    * 打印帮助

    * Partial parsing

    * 自定义文件解析

    * 退出方法

    * Intermixed parsing

  * 升级 optparse 代码

* "getopt" --- C-style parser for command line options

* "logging" --- Python 的日志记录工具

  * 记录器对象

  * 日志级别

  * 处理器对象

  * 格式器对象

  * Filter Objects

  * LogRecord Objects

  * LogRecord 属性

  * LoggerAdapter 对象

  * 线程安全

  * 模块级别函数

  * 模块级属性

  * 与警告模块集成

* "logging.config" --- 日志记录配置

  * Configuration functions

  * Configuration dictionary schema

    * Dictionary Schema Details

    * Incremental Configuration

    * Object connections

    * User-defined objects

    * Access to external objects

    * Access to internal objects

    * Import resolution and custom importers

  * Configuration file format

* "logging.handlers" --- 日志处理

  * StreamHandler

  * FileHandler

  * NullHandler

  * WatchedFileHandler

  * BaseRotatingHandler

  * RotatingFileHandler

  * TimedRotatingFileHandler

  * SocketHandler

  * DatagramHandler

  * SysLogHandler

  * NTEventLogHandler

  * SMTPHandler

  * MemoryHandler

  * HTTPHandler

  * QueueHandler

  * QueueListener

* "getpass" --- 便携式密码输入工具

* "curses" --- 终端字符单元显示的处理

  * 函数

  * Window Objects

  * 常量

* "curses.textpad" --- Text input widget for curses programs

  * 文本框对象

* "curses.ascii" --- Utilities for ASCII characters

* "curses.panel" --- A panel stack extension for curses

  * 函数

  * Panel Objects

* "platform" ---  获取底层平台的标识数据

  * 跨平台

  * Java平台

  * Windows平台

  * Mac OS平台

  * Unix Platforms

* "errno" --- Standard errno system symbols

* "ctypes" --- Python 的外部函数库

  * ctypes 教程

    * 载入动态连接库

    * 操作导入的动态链接库中的函数

    * 调用函数

    * 基础数据类型

    * 调用函数，继续

    * 使用自定义的数据类型调用函数

    * Specifying the required argument types (function prototypes)

    * Return types

    * Passing pointers (or: passing parameters by reference)

    * Structures and unions

    * Structure/union alignment and byte order

    * Bit fields in structures and unions

    * Arrays

    * Pointers

    * Type conversions

    * Incomplete Types

    * Callback functions

    * Accessing values exported from dlls

    * Surprises

    * Variable-sized data types

  * ctypes reference

    * Finding shared libraries

    * Loading shared libraries

    * Foreign functions

    * Function prototypes

    * Utility functions

    * Data types

    * 基础数据类型

    * Structured data types

    * Arrays and pointers
