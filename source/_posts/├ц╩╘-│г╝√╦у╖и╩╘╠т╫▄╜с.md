---
title: 面试 - 常见算法试题总结（PHP）
date: 2018-04-08 17:52:05
categories:
- 面试
tags:
- 面试
---
试题目录：
1、斐波那契数列实现
2、楼梯有s阶台阶,上楼时可以一步上1阶,也可以一步上2阶，编程计算共有多少种不同的走法。
3、输入两个整数n和m，从数列1，2.......n中随意取几个数，使其和等于m，要求将其中所有的可能组合列出来。
4、求一个数组的最长递减子序列 比如{9，4，3，2，5，4，3，2}的最长递减子序列为{9，5，4，3，2}。
5、一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。
6、输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。句子中单词以空格符隔开。为简单起见，标点符号和普通字母一样处理。例如输入“I am a student.”，则输出“student. a am I”。

<!--more-->

---

>**递归算法思想：**
1.递归过程一般通过函数或子过程实现；
2.递归算法在函数或子过程的内部，直接或间接调用自己的算法
3.递归算法实际上是把问题转化为规模缩小了的同类问题的子问题，然后再递归调用函数或过程来表示问题的解

><font color="red">注意：必须有一个明确的递归结束条件；如果递归次数过多，容易造成栈溢出。</font>

---

#### 斐波那契数列
问题描述：
斐波那契数列： 1 1 2 3 5 8 13 21 34 55 …
概念： 
前两个值都为1，该数列从第三位开始，每一位都是当前位前两位的和 
规律公式为： `Fn = F(n-1) + F(n+1) `
* F：指当前这个数列 
* n：指数列的下标

```
/* 非递归实现 */
function fbnq($n){
	if($n<=0){
		return 0;
	}
	$arr[1] = $arr[2] = 1;
	for($i=3; $i<=$n; $i++){
		$arr[$i] = $arr[$i-1] + $arr[$i-2];
	}
	return $arr;
}
// echo "<pre>";
// var_dump(fbnq(10));
/* 递归实现 */
function fbnq_2($n){
	if($n <= 0) return 0;
	static $arr;
	$arr[1] = $arr[2] = 1;
    if($n > 3){
    	fbnq_2($n-1);
    }
    $arr[$n] = $arr[$n-1] + $arr[$n-2];
    return $arr;
}
// var_dump(fbnq_2(10));
```


#### 上台阶问题
问题描述：楼梯有s阶台阶,上楼时可以一步上1阶,也可以一步上2阶，编程计算共有多少种不同的走法。

>**解题思路：**
此问题可分解为 f(n-1) + f(n-2) 种不同的走法。且结束条件为 小于等于 2（最多一步上2阶）

```
function getStepSum($s){
	if($s <= 0){
		return 0;
	}else if($s <= 2){
		return $s;
	}else{
		return getStepSum($s-1)+getStepSum($s-2);
	}
}
// var_dump(getStepSum(3));
```


#### 求数和问题
问题描述：输入两个整数n和m，从数列1，2.......n中随意取几个数，使其和等于m，要求将其中所有的可能组合列出来。

>**解题思路：**
输入sum和n，要求输出1,2...n里所有和为sum的组合
这是一个可划分子问题问题
若用f(sum, n)表示问题的界，则元素组合共有两种情况
1. 和为sum的组合里包括n，则f(sum, n) = f(sum - n, n - 1)
2. 和为sum的组合里不包括n,则 f(sum, n) = f(sum, n - 1)
所以 f(sum, n) = f(sum - n, n - 1) U f(sum, n - 1)

>注：需要传入一个空数组来接收数，用于输出

```
function combination($sum, $n, $comb) {
	if ($n < 0 || $sum < 0) {
	 #当n < 0或者sum < 0都是不符合条件的结果
	 // print_r($comb);
	 // echo "sum: {$sum} n: {$n}<br>";
	 return;
	}
	
	if ($sum < $n) {
	 combination($sum, $sum, $comb); #sum < n，则不可能需要比sum大的数来构成组合
	 return;
	}
	
	#结果求得
	if ($sum == 0) {
	 #输出元素组合
	 echo "Combination: ";
	 foreach ($comb as $val) {
	     echo $val . " ";
	 }
	 echo "<br>";
	 return;
	}

	#组合里包含n的情况
	$comb[] = $n;
	combination($sum - $n, $n - 1, $comb);
	
	#组合里不包含n的情况
	array_pop($comb);
	combination($sum, $n - 1, $comb);
}
```


#### 最长递减子序列问题
问题描述：求一个数组的最长递减子序列 比如{9，4，3，2，5，4，3，2}的最长递减子序列为{9，5，4，3，2}。

>**解题思路：**
此问题就是比较大小的问题，把列表看成栈比较容易些（且是有序的栈）。与栈头比较大小，满足条件入栈，否则出栈。
注：需要判空数组的问题。

```
// $arr = [9, 4, 3, 2, 5, 4, 3, 1];
$arr = [9, 4, 3, 2, 5, 4, 3, 1, 10, 8, 5, 3, 2, 1];
function maxDescList($arr){
	$len = count($arr);
	$list[] = $arr[0];
	for($i=1; $i<$len; $i++){
		$llen = count($list);
		for($j=$llen-1; $j>=0; $j--){
			if($arr[$i] > $list[$j]){
				array_pop($list);
				if(empty($list)){
					$list[] = $arr[$i];
					break;
				}
			}else{
				array_push($list, $arr[$i]);
				break;
			}
		}
	}
	return $list;
}
// echo "<pre>";
// var_dump(maxDescList($arr));
```


#### 找重复出现的数字
问题描述：一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

>**解题思路：**
将整型数组看成栈比较容易理解。每次出栈一个元素，然后查询该值是否存在数组中。如果存在，说明数组该值有两个，然后删除该值（原因是如果不删除，出栈元素会再次重复一次，再出现就会出现错误。因为该值已经查询过）。

```
$arr = [8, 5, 3, 4, 2, 4, 3, 5, 9, 8];
function searchNum($arr){
	$list = array();
	while(!empty($arr)){
		$val = array_pop($arr);
		if(!in_array($val, $arr)){
			$list[] = $val;
		}else{
			$key = array_search($val, $arr);
			unset($arr[$key]);
		}
	}
	return $list;
}
// var_dump(searchNum($arr));
```


#### 翻转字符串
问题描述：输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。句子中单词以空格符隔开。为简单起见，标点符号和普通字母一样处理。例如输入`“I am a student.”`，则输出`“student. a am I”`。

>**解题思路：**
此题利用PHP函数很容易实现。
注：要注意使用 `array_reverse()` 数组翻转函数

```
function strrevStr($str){
	$str_arr = explode(" ", $str);
	$reverse_arr = array_reverse($str_arr);
	return implode(" ", $reverse_arr);
}
// $str = "I am a student.";
// echo strrevStr($str);
```

问题描述：翻转字符串，但是不能使用PHP内部函数，只能使用 strlen 。
>**解题思路：**
可使用循环解决。

```
function myStrrev($str)
{
	$len = strlen($str);
	$new_str = '';
	$n = '';
	for($i=0; $i<$len-1; $i++){
		if($str[$i] != ' '){
			$n .= $str[$i];
		}else{
			$new_str = $n.' '.$new_str;
			$n = '';
		}
	}
	$new_str = $n.' '.$new_str;
	return trim($new_str);
}
$str = "your name is benwu";
echo myStrrev($str);
```
