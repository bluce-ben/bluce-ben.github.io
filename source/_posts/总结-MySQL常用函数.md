---
title: 总结-MySQL常用函数
date: 2017-12-08 11:59:00
categories:
- MySQL
tags:
- MySQL
---
### 一、聚合函数(常用于GROUP BY从句的SELECT查询中)
　下面五个函数会忽略值为NULL的行

函数名 | 说明
-------|-------------------
AVG(col)   | 返回指定列的平均值
COUNT(col) |   返回指定列中非NULL值/行的个数（当函数参数为星号*时不会忽略）
MIN(col)   | 返回指定列的最小值
MAX(col)   | 返回指定列的最大值
SUM(col)   | 返回指定列的所有值之和
GROUP_CONCAT(col) |    返回由属于一组的列值连接组合而成的结果
<!--more-->
### 二、数学函数
函数名 | 说明
-------|-------------------
ABS(x)  |  返回x的绝对值
BIN(x)  |   返回x的二进制（OCT返回八进制，HEX返回十六进制）
EXP(x)  |  返回值e（自然对数的底）的x次方
GREATEST(x1,x2,...,xn) |  返回集合中最大的值
LEAST(x1,x2,...,xn)    |    返回集合中最小的值
LN(x)       |       返回x的自然对数
LOG(x,y)    |     返回x的以y为底的对数
MOD(x,y)    |     返回x/y的模（余数）
PI()        |       返回pi的值（圆周率）
RAND()      |     返回0到1内的随机值,可以通过提供一个参数(种子)使RAND()随机数生成器生成一个指定的值。
FLOOR(x)    |    返回小于x的最大整数值，（去掉小数取整）
CEILING(x)  |     返回大于x的最小整数值，（进一取整）
ROUND(x,y)  |    返回参数x的四舍五入的有y位小数的值，（四舍五入）
TRUNCATE(x,y) |     返回数字x截短为y位小数的结果
SIGN(x)       |         返回代表数字x的符号的值（正数返回1，负数返回-1，0返回0）
SQRT(x)       |        返回一个数的平方根

### 三、字符串函数
函数名 | 说明
-------|-------------------
ASCII(char)     |    返回字符的ASCII码值
BIT_LENGTH(str) | 返回字符串的比特长度
CONCAT(s1,s2...,sn)  |  将s1,s2...,sn连接成字符串
CONCAT_WS(sep,s1,s2...,sn) |   将s1,s2...,sn连接成字符串，并用sep字符间隔
INSERT(str,x,y,instr)  |   将字符串str从第x位置开始，y个字符长的子串替换为字符串instr，返回结果
FIND_IN_SET(str,list)  |  分析逗号分隔的list列表，如果发现str，返回str在list中的位置
LCASE(str)或LOWER(str) |    返回将字符串str中所有字符改变为小写后的结果
UCASE(str)或UPPER(str) |    返回将字符串str中所有字符转变为大写后的结果
LEFT(str,x)   | 返回字符串str中最左边的x个字符
RIGHT(str,x)  |   返回字符串str中最右边的x个字符
LENGTH(str)   | 返回字符串str中的字符数
POSITION(substr,str)   |  返回子串substr在字符串str中第一次出现的位置
QUOTE(str)   |  用反斜杠转义str中的单引号
REPEAT(str,srchstr,rplcstr) |   返回字符串str重复x次的结果
REVERSE(str) |    返回颠倒字符串str的结果
LTRIM(str)   |  去掉字符串str开头的空格
RTRIM(str)   |  去掉字符串str尾部的空格
TRIM(str)   | 去除字符串首部和尾部的所有空格

### 四、日期和时间函数
函数名 | 说明
-------|-------------------
DATE_ADD(date,INTERVAL int keyword)  |  返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：SELECTDATE_ADD(CURRENT_DATE,INTERVAL 6 MONTH);
DATE_SUB(date,INTERVAL int keyword)  | 返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：SELECTDATE_SUB(CURRENT_DATE,INTERVAL 6 MONTH);
DATE_FORMAT(date,fmt)  |    依照指定的fmt格式格式化日期date值
FROM_UNIXTIME(ts,fmt)  |   根据指定的fmt格式，格式化UNIX时间戳ts
MONTHNAME(date)   |    返回date的月份名(英语月份，如October)
DAYNAME(date)    |   返回date的星期名(英语星期几，如Saturday)
NOW()    |    返回当前的日期和时间 如：2016-10-08 18:57:39
CURDATE()或CURRENT_DATE()  |   返回当前的日期
CURTIME()或CURRENT_TIME()  |   返回当前的时间
QUARTER(date)   |    返回date在一年中的季度(1~4)
WEEK(date)     |  返回日期date为一年中第几周(0~53)
DAYOFYEAR(date)   |    返回date是一年的第几天(1~366)
DAYOFMONTH(date)  |    返回date是一个月的第几天(1~31)
DAYOFWEEK(date)   |    返回date所代表的一星期中的第几天(1~7)
YEAR(date)     |  返回日期date的年份(1000~9999)
MONTH(date)    |   返回date的月份值(1~12)
DAY(date)   |  返回date的天数部分
HOUR(time)  |   返回time的小时值(0~23)
MINUTE(time)  |     返回time的分钟值(0~59)
SECOND(time)  |   返回time的秒值（0-59）
DATE(datetime) |    返回datetime的日期值
TIME(datetime)  |   返回datetime的时间值

### 五、加密函数
函数名 | 说明
-------|-------------------
AES_ENCRYPT(str,key)  |    返回用密钥key对字符串str利用高级加密标准算法加密后的结果，调用AES_ENCRYPT的结果是一个二进制字符串，以BLOB类型存储
AES_DECRYPT(str,key)  |    返回用密钥key对字符串str利用高级加密标准算法解密后的结果
DECODE(str,key)    |   使用key作为密钥解密加密字符串str
ENCRYPT(str,salt)   |    使用UNIXcrypt()函数，用关键词salt(一个可以惟一确定口令的字符串，就像钥匙一样)加密字符串str
ENCODE(str,key)     |  使用key作为密钥加密字符串str，调用ENCODE()的结果是一个二进制字符串，它以BLOB类型存储
MD5()     |   计算字符串str的MD5校验和
PASSWORD(str)  |     返回字符串str的加密版本，这个加密过程是不可逆转的，和UNIX密码加密过程使用不同的算法。
SHA()     |   计算字符串str的安全散列算法(SHA)校验和

### 六、格式化函数
函数名 | 说明
-------|-------------------
DATE_FORMAT(date,fmt)  |    依照字符串fmt格式化日期date值
FORMAT(x,y)     |  把x格式化为以逗号隔开的数字序列，y是结果的小数位数
INET_ATON(ip)    |  返回IP地址的数字表示
INET_NTOA(num)    |   返回数字所代表的IP地址
TIME_FORMAT(time,fmt)  |    依照字符串fmt格式化时间time值,其中最简单的是FORMAT()函数，它可以把大的数值格式化为以逗号间隔的易读的序列。

### 七、数据类型转换函数
　CAST()函数，将一个值转换为指定的数据类型（类型有：BINARY,CHAR,DATE,TIME,DATETIME,SIGNED,UNSIGNED）
`SELECT CAST(NOW() AS SIGNED INTEGER),CURDATE()+0;`

### 八、系统信息函数
函数名 | 说明
-------|-------------------
DATABASE()   |  返回当前数据库名
BENCHMARK(count,expr) |   将表达式expr重复运行count次
CONNECTION_ID()  |   返回当前客户的连接ID
FOUND_ROWS()   |  返回最后一个SELECT查询进行检索的总行数
USER()或SYSTEM_USER()  |  返回当前登陆用户名
VERSION()   |  返回MySQL服务器的版本

### 九、条件判断函数
函数名 | 说明
-------|-------------------
isnull(expr)  |	如expr为null，那么isnull()的返回值为1，否则返回值为0。
ifnull(expr1,expr2) |	假如expr1不为NULL，则IFNULL()的返回值为expr1; 否则其返回值为expr2。IFNULL()的返回值是数字或是字符串，具体情况取决于其所使用的语境。
nullif(expr1,expr2) |	如果expr1=expr2成立，那么返回值为NULL，否则返回值为expr1。
IF(expr1,expr2,expr3) |  如果 expr1 是TRUE (expr1 <> 0 and expr1 <> NULL)，则 IF()的返回值为expr2; 否则返回值则为 expr3。IF() 的返回值为数字值或字符串值，具体情况视其所在语境而定。