---
title: 面试 - php易混淆题集锦
date: 2018-06-11 16:55:20
categories:
- 面试
tags:
- 面试
---

##### 1. 写出如下程序的输出结果（考点：变量判空）
```
$a1 = null;
$a2 = false;
$a3 = 0;
$a4 = '';
$a5 = '0';
$a6 = 'null';
$a7 = array();
$a8 = array(array());

echo empty($a1) ? 'true' : 'false';             //true
echo "<br/>";
echo empty($a2) ? 'true' : 'false';             //true
echo "<br/>";
echo empty($a3) ? 'true' : 'false';             //true
echo "<br/>";
echo empty($a4) ? 'true' : 'false';             //true
echo "<br/>";
echo empty($a5) ? 'true' : 'false';             //false  true
echo "<br/>";
echo empty($a6) ? 'true' : 'false';             //false
echo "<br/>";
echo empty($a7) ? 'true' : 'false';             //true
echo "<br/>";
echo empty($a8) ? 'true' : 'false';             //true  false
echo "<hr/>";
```
<!--more-->
---

##### 2. 写出如下程序的输出结果（考点：变量引用）
```
$test = 'aaaaaa';
$abc = &$test;
unset($test);

echo $abc;              //aaaaaa
echo $test;             //undefined valiable

echo "<hr/>";
```

---

##### 3. 写出如下程序的输出结果（考点：静态变量）
```
$count = 5;
function get_count(){
        static $count = 0;
        return $count++;
}
echo $count;            //5
++$count;

echo get_count();       //0
echo get_count();       //1

echo "<hr/>";
```

---

##### 4. 写出如下程序的输出结果（考点：全局变量作用域）
```
$GLOBALS['var1'] = 5;
$var2 = 1;
function get_value(){
        global $var2;
        $var1 = 0;
        return $var2++;
}
get_value();

echo $var1;     //5
echo $var2;     //2

echo "<hr/>";
```

**全局变量的作用域问题：**
```
function destroy_foo() 
{
        global $foo;
        unset($foo);
        // unset($GLOBALS['foo']);//清除全局变量$foo
}

$foo = 'bar';
destroy_foo();
echo $foo;      //bar
```
unset($foo) 的作用是清除函数内的 $foo变量，并没有清除全局变量的$foo，因此执行此函数后，变量仍存在。想要彻底清除$foo，需要清除掉全局变量里的$foo值。

---

##### 5. 写出如下程序的输出结果（考点：unset对于数组的作用）
```
function get_arr($arr){
        unset($arr[0]);
}
function get_arr2(&$arr){
        unset($arr[0]);
}

$arr1 = array(1, 2);
$arr2 = array(1, 2);

get_arr($arr1);
get_arr2($arr2);

echo count($arr1);      //2
echo count($arr2);      //1

echo "<hr/>";
```

---

##### 6. 字符串强制转换为整型为0
例如：
```
"a1"  => 0  //首字母a为字符，强制转换为整型为0
"1a"  => 1      //首字母1为数字，强制转换为整型为1
```

---

##### 7. & 与 && 的区别
```
$a = 10;$b = 10;

if($a < 4 && (++$a > 10)){}
if($b < 4 & (++$b > 10)){}

echo $a;
echo $b;
```
& 是按位与操作符，两边都为 1 则为 1。左侧为false，也会运行右侧的代码。
&& 是逻辑与操作符，左侧为false，不运行右侧的代码。
相同点：都可进行 and 判断。

---

##### 8. unset对于数组的作用
```
$arr = array(
        1=> 'a',
        2=> 'bb',
        3=> 'c',
        4=> 'dd'
        );
foreach($arr as $k=>&$v){
        if($k == 3){
                $v = 'x';
        }
        // unset($v);   //保持键4 的值不变
}
foreach($arr as $k=>$v){
        echo "{$k}\t{$v}<br/>";
}
```
输出结果为：array('a', 'bb', 'x', 'x');
*为什么结果是这样？*
　　因为满足条件语句之后，就把数组的值指向了'x'的内存地址了。因此后面的数组索引都是指向了'x'。
*怎么能让 键4 的值不变？*
　　使用 unset($v) 来清除 指针。

---