互联网数据处理
**************

本章介绍了支持处理互联网上常用数据格式的模块。

* "email" --- 电子邮件与 MIME 处理包

  * "email.message": Representing an email message

  * "email.parser": Parsing email messages

    * FeedParser API

    * Parser API

    * Additional notes

  * "email.generator": Generating MIME documents

  * "email.policy": Policy Objects

  * "email.errors": 异常和缺陷类

  * "email.headerregistry": Custom Header Objects

  * "email.contentmanager": Managing MIME Content

    * Content Manager Instances

  * "email": 示例

  * "email.message.Message": Representing an email message using the
    "compat32" API

  * "email.mime": Creating email and MIME objects from scratch

  * "email.header": Internationalized headers

  * "email.charset": Representing character sets

  * "email.encoders": 编码器

  * "email.utils": 其他工具

  * "email.iterators": 迭代器

* "json" --- JSON 编码和解码器

  * 基本使用

  * 编码器和解码器

  * 异常

  * 标准符合性和互操作性

    * 字符编码

    * Infinite 和 NaN 数值

    * 对象中的重复名称

    * 顶级非对象，非数组值

    * 实现限制

  * 命令行界面

    * 命令行选项

* "mailcap" --- Mailcap 文件处理

* "mailbox" --- Manipulate mailboxes in various formats

  * "Mailbox" 对象

    * "Maildir"

    * "mbox"

    * "MH"

    * "Babyl"

    * "MMDF"

  * "Message" objects

    * "MaildirMessage"

    * "mboxMessage"

    * "MHMessage"

    * "BabylMessage"

    * "MMDFMessage"

  * 异常

  * 示例

* "mimetypes" --- Map filenames to MIME types

  * MimeTypes Objects

* "base64" --- Base16, Base32, Base64, Base85 数据编码

* "binhex" --- 对binhex4文件进行编码和解码

  * 注释

* "binascii" --- 二进制和 ASCII 码互转

* "quopri" --- 编码与解码经过 MIME 转码的可打印数据

* "uu" --- 对 uuencode 文件进行编码与解码
