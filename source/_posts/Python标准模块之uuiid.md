---
title: Python 标准模块之 uuiid
date: 2018-04-23 18:52:51
categories:
- Python
tags:
- Python
- Python模块
---
uuid是128位的全局唯一标识符（univeral unique identifier），通常用32位的一个字符串的形式来表现。有时也称guid(global unique identifier)。python中自带了uuid模块来进行uuid的生成和管理工作。

python中的uuid模块基于信息如MAC地址、时间戳、命名空间、随机数、伪随机数来uuid。具体方法有如下几个：
>**uuid.getnode()：**获取硬件地址为48位正整数。这是第一次运行，它可能会启动一个单独的程序，可能会很慢。如果所有尝试获取硬件地址都失败，我们选择一个随机的48位数字，其第8位设置为1，如RFC 4122中推荐的那样。“硬件地址”表示网络接口的MAC地址，以及具有多个网络接口可以返回其中任何一个的MAC地址。
**uuid.uuid1([ node [，clock_seq ] ])：**从主机ID，序列号和当前时间生成一个UUID。如果 没有给出节点，getnode()则用于获取硬件地址。如果 给出clock_seq，它将用作序列号; 否则选择一个随机的14位序列号。（可以保证全球范围内的唯一性。但是可能会危害隐私，因为它会创建一个包含计算机网络地址的UUID。）
**uuid.uuid3(名称空间，名称)：**根据名称空间标识（这是一个UUID）和一个名称（这是一个字符串）的MD5散列生成一个UUID。
**uuid.uuid4()：**生成一个随机的UUID。（注：有一定概率重复的）
**uuid.uuid5 (名称空间，名称)：**根据名称空间标识（这是一个UUID）和名称（它是一个字符串）的SHA-1散列生成一个UUID。（注：和uuid3基本相同，只不过采用的散列算法是sha1）

<!--more-->

该uuid模块定义了以下用于uuid3()或的名称空间标识符 uuid5()。
>**uuid.NAMESPACE_DNS**
当指定此名称空间时，名称字符串是完全限定的域名。
**uuid.NAMESPACE_URL**
当这个名字空间被指定时，名字字符串就是一个URL。
**uuid.NAMESPACE_OID**
当这个名字空间被指定时，名字字符串就是一个ISO OID。
**uuid.NAMESPACE_X500**
当指定此名称空间时，名称字符串是DER中的X.500 DN或文本输出格式。


UUID 实例具有这些只读属性：
>**UUID.bytes**
UUID作为一个16字节的字符串（包含以big-endian字节顺序的六个整数字段）。
**UUID.bytes_le**
UUID作为16字节的字符串（以 little-endian字节顺序包含time_low，time_mid和time_hi_version）。
**UUID.fields**
UUID的六个整数字段的元组，它们也可用作六个单独的属性和两个派生属性：

领域					| 含义
------------------------|-----------------
time_low				| UUID的前32位
time_mid				| UUID的接下来的16位
time_hi_version			| UUID的接下来的16位
clock_seq_hi_variant	| UUID的接下来的8位
clock_seq_low			| UUID的接下来的8位
node					| UUID的最后48位
time					| 60位时间戳
clock_seq				| 14位序列号

>**UUID.hex**
UUID作为32个字符的十六进制字符串。
**UUID.int**
UUID是一个128位整数。
**UUID.urn**
UUID作为RFC4122中规定的URN。
**UUID.variant**
UUID变体，它确定UUID的内部布局。这将是一个常量RESERVED_NCS，RFC_4122， RESERVED_MICROSOFT，或RESERVED_FUTURE。
**UUID.version**
UUID版本号（1到5，仅在变体时才有意义 RFC_4122）。



以下是uuid模块典型用法的一些示例：
```
>>> import uuid

>>> # make a UUID based on the host ID and current time
>>> uuid.uuid1()
UUID('a8098c1a-f86e-11da-bd1a-00112444be1e')

>>> # make a UUID using an MD5 hash of a namespace UUID and a name
>>> uuid.uuid3(uuid.NAMESPACE_DNS, 'python.org')
UUID('6fa459ea-ee8a-3ca4-894e-db77e160355e')

>>> # make a random UUID
>>> uuid.uuid4()
UUID('16fd2706-8baf-433b-82eb-8c7fada847da')

>>> # make a UUID using a SHA-1 hash of a namespace UUID and a name
>>> uuid.uuid5(uuid.NAMESPACE_DNS, 'python.org')
UUID('886313e1-3b8a-5372-9b90-0c9aee199e5d')
```