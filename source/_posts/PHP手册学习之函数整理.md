---
title: PHP手册学习之函数整理
date: 2019-07-19 16:59:43
categories:
- PHP
tags:
- PHP知识点
- 面试
---
下面是此段时间看手册整理之后的函数：

<!--more-->

#### 数组函数：
array_chunk(array, size, bool) 对数组进行分组，每个数组包含size个元素
array_column(array, null, key) 获取数组中某一列数据，或以某列为键，某列为值
array_combine(array1, array2)  合并两个数组，以array1为键，以array2为值，如果两个数组元素个数不一样，返回false
array_diff(array1, array2,...) 比较数组的值，返回差集，存在array1,且不存在其它数组中的值。保持索引不变
array_diff_key(array1, array2, ...) 比较数组的键，返回差集，存在array1,且不存在其它数组中，保持索引不变
array_intersect(array1, array2, ...) 比较数组的值，返回交集，返回即存在array1,又存在array2种，保持索引不变
array_intersect_key(array1, array2, ...) 比较数组的键，返回交集，返回即存在array1,又存在array2种，保持索引不变
array_filter(array, [callback]) 用回调函数过滤数组中的单元，如果没有回调函数，则过滤数组中false的单元
array_flip(array)  交换数组中的键值
array_keys(array)  返回数组中的键
array_values(array)   返回数组中的值
array_map(callback, array)  返回数组，对array中的每个元素都应用callback之后的数组
array_walk(&array, callback, [userdata])  返回bool，对数组中的每个元素应用callback
array_merge(array1, array2, ...) 合并两个数组，关联数组后面的会覆盖前面。索引数组会重置索引值，从0开始。（比较 “+” 合并数组，索引数组会保持索引值，前面的会覆盖后面。）
array_pop()、array_push()、array_shift()、array_unshift()
array_rand(array, [int]) 从数组中随机取出一个或多个单元。只取出一个，返回键名。否则返回包含键名的数组。
shuffle(array) 打乱数组，返回bool。
array_reverse(array, [bool=false]) 返回单元顺序相反的数组。如果第二个值为true，会保留数字索引。
array_key_exists(key, array)  检查数组中是否包含key
in_array(needle, array) 检查数组中是否存在某个值
array_search(needle, array) 在数组中搜索给定的值，成功返回键名
array_slice(array, offset, [length, bool=false]) 返回根据offset和length指定的数组序列。第四个值设为true，保持数字索引不变。
array_sum(array)  对数组中所有值求和
array_unique(array)  移除数组中重复的值，默认按照字符串排序
sort()、rsort()、asort()、arsort()、ksort()、krsort()、usort(array, callback)、uksort()、uasort()
compact(varname1) 建立一个数组，包含变量名和它们的值。会在当前符号表中查找该变量名并将它添加到输出的数组中，变量名成为键名而变量的内容成为该键的值。
extract(array)  从数组中将变量导入到当前的符号表。 与compact正好相反
count(array, [int = 0]) 获取数组的单元个数。 如果第二个值为1，则获取多维数组中单元个数。
current() 返回数组当前单元，并不移动指针
key() 从关联数组中取得键名，返回当前单元的键名，并不移动指针
end() 返回数组最后一个单元，返回值
next() 将数组中的内部指针向前移动一位，返回值
prev() 将数组的内部指针倒回一位，返回值
reset() 将数组的内部指针指向第一个单元，返回值
each() 返回数组中当前的键/值对并将数组指针向前移动一位
list(var1, ...) 把数组中的值赋给一组变量
range(start, end, [step=1]) 根据范围创建数组，包含指定的元素。


#### 字符串函数：
addslashes($str):str	- 使用反斜线引用字符串
stripcslashes($str):str	- 反引用一个使用addslashes转义的字符串
html_entity_decode	- 转化html实体
htmlspecialchars()	- 将特殊字符转换为HTML实体
htmlspecialchars_decode() - 将特殊的HTML实体转换回普通字符
explode($delimiter, $str):array		- 使用一个字符串分隔另一个字符串
implode($glue, $pieces):string		- 将一个一维数组的值转化为字符串
trim()、ltrim()、rtrim()	- 删除字符串两边的空白字符（或其它字符）
md5($str, [$raw_output]):str		- 计算字符串的MD5散列值
sha1($str, [$raw_output]):str		- 计算字符串的sha1散列值
crc32($str):int		- 计算一个字符串的crc32多项式
sprintf($format, [$args,...]):str 	- 格式化字符串
printf($format, [$args, ...]):int	- 输出格式化字符串
number_format($number, $decimals, $dec_point='.', $thousands_sep=',') - 以千位分隔符方式格式化一个数字（注：本函数接受1个、2个或者4个参数，不能是3个）
str_pad($input, $pad_length, [$pad_string, $pad_type='left']):str	- 使用另一个字符串填充字符串为指定长度
str_shuffle($str):str	- 随机打乱一个字符串
strlen($str):str	- 获取字符串长度
strrev($str):str	- 反转字符串
strpos($haystack, $needle, [$offset]):int	- 查找字符串首次出现的位置
stripos($haystack, $needle, [$offset]):int	- 查找字符串首次出现的位置（不区分大小写）
strrpos($haystack, $needle, [$offset]):int	- 计算指定字符串在目标字符串中最后一次出现的位置
strtolower($str):str	- 将字符串转化为小写
strtoupper($str):str	- 将字符串转化为大写
substr_replace($str, $replace, $start, [$length]):mixed	- 替换字符串的子串
substr($str, $start, $length):str	- 返回字符串的子串
ucfirst($str):str	- 将字符串的首字母转换为大写
ucwords($str):str	- 将字符串中每个单词的首字母转换为大写


#### 数学函数：
ceil()	- 进一法取整
floor()	- 舍去法取整
round()	- 对浮点数进行四舍五入
fmod()	- 返回除法的浮点数余数
max()	- 找出最大值
min()	- 找出最小值
rand()	- 产生一个随机整数（闭合区间）
mt_rand()	- 生成更好的随机数（闭合区间）
pow()	- 指数表达式
abs()	- 绝对值


#### BC数学函数：
bcadd($left_operand, $right_operand, [$scale])	- 2个任意精度数字的加法计算
bcsub($left_operand, $right_operand, [$scale])	- 2个任意精度数学的减法
bcmul($left_operand, $right_operand, [$scale])	- 2个任意精度数字乘法计算
bcdiv($left_operand, $right_operand, [$scale])		- 2个任意精度的数字除法计算
bccomp($left_operand, $right_operand, [$scale])	- 比较两个任意精度的数字
bcmod($left_operand, $modulus)		- 对一个任意精度数字取模
bcpow($left_operand, $right_operand, [$scale])		- 任意精度数字的乘方
bcsqrt($operand, [$scale])	- 任意精度数学的二次方根
bcscale($scale) 	- 设置所有bc数学函数的默认小数点保留位数


#### 时间函数：
date()		- 格式化一个本地时间/日期
microtime()	- 返回当前Unix时间戳和微秒数
mktime()	- 取得一个日期的Unix时间戳
strtotime()	- 将任何字符串的日期时间描述解析为Unix时间戳
time()		- 返回当前的Unix时间戳
localtime()	- 取得本地时间
date_default_timezone_set - 设定用于一个脚本中所有日期时间函数的默认时区
date_default_timezone_get - 获取


#### 文件函数：
opendir($path):resource	- 打开目录句柄
closedir($dir_handle)	- 关闭目录句柄
readdir($dir_handle):str	- 从目录句柄中读取条目
scandir($directory, [$sorting_order]):array	- 列出指定路径中的文件和目录
chdir($directory):bool		- 改变目录
chroot($directory):bool	- 改变根目录
getcwd():str	- 取得当前工作目录


#### 文件系统函数；
file_get_contents($filename):str - 将整个文件读取一个字符串
file_put_contents($filename, $data):int - 将一个字符串写入文件
basename($path, [$suffix]):str	- 返回路径中的文件名部分（如果文件名是以suffix结尾，也会被去掉）
dirname($path):str	- 返回路径中的目录部分
pathinfo($path):mixd	- 返回文件路径信息（包含：surname,basename,extension,filename）
fopen($filename, $mode):resource	- 打开文件或者URL
fread($handle, $length):str		- 读取文件
fgetc($handle):str		- 从文件指针中读取字符
fgets($handle, [$length]):str	- 从文件指针中读取一行（如果没有指定length，则默认1K，即1024字节）
fclose($handle):bool	- 关闭一个已打开的文件指针
feof($handle):bool		- 测试文件指针是否到了文件结束的位置
fwrite($handle, $str, [$length]):int	- 写入文件
mkdir($pathname, [$mode=0777, $recursive=false]):bool		- 新建目录
rmdir($dirname):bool		- 删除目录（目录必须是空的）
unlink($filename):bool	- 删除文件
fseek()		- 在文件指针中定位
move_uploaded_file($filename, $destination):bool	- 将上传的文件移动到新位置
filesize($filename):int	- 取得文件大小
filetype($filename):type	- 取得文件类型
fileatime($filename):int	- 文件访问时间
filectime($filename):int	- 文件inode修改时间
filemtime($filename):int	- 取得文件修改时间
is_dir($filename):bool	- 是否是一个目录
is_file($filename):bool	- 是否为一个正常的文件
is_executable($filename):bool	- 文件是否可执行
is_writable($filename):bool	- 文件是否可写
is_readable($filename):bool	- 文件是否可读
chgrp($filename, $group):bool		- 改变文件所属的组
chmod($filename, $mode):bool		- 改变文件模式
chown($filename, $user):bool		- 改变文件的所有者
copy($source, $dest):bool		- 拷贝文件
rename()	- 重命名一个文件或目录
realpath()	- 返回规范化的绝对路径名
touch()		- 设定文件的访问和修改时间
