---
title: 经典算法问题 - 最大连续子数列和与最大连续子数列乘积（PHP实现）
date: 2018-04-03 13:30:25
categories:
- 算法
tags:
- 算法
---
#### 一、最大连续子数列和问题
给出测试数列：
`$arr = [-2, 6, -1, 5, 4, -7, 2, 3];`

##### 1、穷举法
>**暴力求解法（穷举法）：**
暴力求解也是容易理解的做法，简单来说，我们只要用两层循环枚举起点和终点，这样就尝试了所有的子序列，然后计算每个子序列的和，然后找到其中最大的即可
我们可以采用一个两重的循环，假设两个循环的循环变量分别为i, j。第一层循环从第一个元素遍历到后面，第二个元素从>=第一个元素的位置开始到最后。这样就可以遍历到所有的点。然后，我们取所有从i到j的连续数组部分和再比较。这样最终就可以得到最大的连续和以及最大子序列的起始与结束点。

<!--more-->
代码实现：
```
function maxSubSum1($arr){
	$len = count($arr);
	$maxSum = 0;
	for($i=0; $i<$len; $i++){
		for($j=$i; $j<$len; $j++){
			$thisSum = 0;
			for($k=$i; $k<=$j; $k++){
				$thisSum += $arr[$k];
			}
			if($thisSum > $maxSum){
				$maxSum = $thisSum;
				$seqStart = $i;
				$seqEnd = $j;
			}
		}
	}
	$length = $seqEnd-$seqStart+1;
	return array_slice($arr, $seqStart, $length);
}
```

##### 2、优化穷举法
>**一个简单的优化：**
前面那个最简单暴力的方法虽然看起来能解决问题，但是循环遍历的次数太多了。里面还是有不少可以改进的空间。比如说，每次我们用变量k遍历i到j的时候，都要计算求和。实际上当每次j增加1时，k把前面计算的结果在循环里又计算了一遍。这是完全没必要的，完全可以重复利用前面的计算结果。

代码实现：
```
function maxSubSum2($arr){
	$len = count($arr);
	$maxSum = 0;
	for($i=1; $i<$len; $i++){
		$thisSum = 0;
		for($j=$i; $j<$len; $j++){
			$thisSum += $arr[$j];
			if($thisSum > $maxSum){
				$maxSum = $thisSum;
				$seqStart = $i;
				$seqEnd = $j;
			}
		}
	}
	$length = $seqEnd-$seqStart+1;
	return array_slice($arr, $seqStart, $length);
}
```

##### 3、线性算法
>这个问题还是存在着一个线性时间复杂度的解法。需要我们对数组的序列进行进一步的分析。我们在数组中间找到的连续子序列，可能存在和为负的序列。如果需要找到一个最大的子数组的话，肯定该序列不是在最大子序列当中的。我们可以通过反证的方式来证明。
假设数组A[i...j]，里面的元素和为负。如果A[i....j]在一个最大子序列的数组中间，假定为A[i...k]，k > j。那么既然从i到j这一段是负的，我把这一段去掉剩下的部分完全比我们假定的这个最大子序列还要大。这就和我的假设矛盾了。
这个假设还有一个限制，就是该数组就是从i开头的。否则有人可能会这么问，如果我A[i...j]这一部分确实是一个负数，但是在A[i]前面是一个很大的正数，使得他们的和为正数。那不就使得我们的结果不成立了么？如果我们是从某个数组的开头i开始的话，就不存在这个情况。
结合前面的讨论，我们就可以发现一个有意思的事情，就是假设我们从数组的开头A[0]开始，不断的往后面走，每一步判断是否当前和最大，并保存结果。当发现当前字串和为负数的时候，我们可以直接跳过。假设当前的索引为i的话，从0到i这一段的和是负数，可以排除。然后再从当前元素的后面开始找。这样可以得到最终最大子串和以及串的起点和终点。

代码实现：
```
function maxSubSum3($arr){
	$len = count($arr);
	$maxSum = 0;
	$thisSum = 0;
	for($i=0, $j=0; $j<$len; $j++){
		$thisSum += $arr[$j];
		if($thisSum > $maxSum){
			$maxSum = $thisSum;
			$seqStart = $i;
			$seqEnd = $j;
		}else if($thisSum < 0){
			$i = $j+1;
			$thisSum = 0;
		}
	}
	$length = $seqEnd-$seqStart+1;
	return array_slice($arr, $seqStart, $length);
}
```

#### 二、最大连续子数列乘积问题
给出测试数列：
`$arr = [2, 3, 0, -3, 3, -1, 0, 9];`

##### 1、穷举法
```
function maxSubProduct1($arr){
	$len = count($arr);
	$maxPro = 0;
	for($i=0; $i<$len; $i++){
		$thisPro = 1;
		for($j=$i; $j<$len; $j++){
			$thisPro *= $arr[$j];
			if($thisPro > $maxPro){
				$maxPro = $thisPro;
				$seqStart = $i;
				$seqEnd = $j;
			}
		}
	}
	$length = $seqEnd-$seqStart+1;
	return array_slice($arr, $seqStart, $length);
}
```

##### 2、线性算法
```
function maxSubProduct2($arr){
	$len = count($arr);
	$maxPro = 0;
	$thisPro = 1;
	for($i=0, $j=0; $j<$len; $j++){
		$thisPro *= $arr[$j];
		if($thisPro > $maxPro){
			$maxPro = $thisPro;
			$seqStart = $i;
			$seqEnd = $j;
		}else if($thisPro == 0){
			$thisPro = 1;
			$i = $j+1;
		}
	}
	$length = $seqEnd-$seqStart+1;
	return array_slice($arr, $seqStart, $length);
}
```

**总结：**
通过对“最大连续子数列和与最大连续子数列乘积”的算法实现，可发现共通之处，即<font color="red">算法原理都是一样的</font>。最简单、最暴力的解决方法就是穷举法，同时，算法时间复杂度也是最差的。现实中使用必须要优化，否则特别影响性能。
因此，就出现了线性算法，此算法时间复杂度只有 O(n)，极大的提高了性能。