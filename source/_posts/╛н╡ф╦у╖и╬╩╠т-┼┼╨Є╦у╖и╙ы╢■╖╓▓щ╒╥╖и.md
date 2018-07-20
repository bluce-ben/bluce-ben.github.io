---
title: 经典算法问题 - 排序算法与二分查找法（PHP实现）
date: 2018-04-03 13:53:33
categories:
- 算法
tags:
- 算法
---
#### 排序算法
测试数组：
`$arr = [3,5,7,1,8,4,9,6,2];`

##### 1、冒泡排序
冒泡排序是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果它们的顺序错误就把它们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。这个算法的名字由来是因为越大的元素会经由交换慢慢“浮”到数列的顶端。 
<!--more-->
**算法描述**
* 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
* 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
* 针对所有的元素重复以上的步骤，除了最后一个；
* 重复步骤1~3，直到排序完成。

**代码实现：**
```
function bubbleSort($arr){
	if(!is_array($arr)) return false;
	$len = count($arr);
	if($len <= 1) return $arr;
	for($i=0; $i<$len; $i++){
		for($j=1; $j<$len-$i; $j++){
			if($arr[$j-1] > $arr[$j]){
				$tmp = $arr[$j-1];
				$arr[$j-1] = $arr[$j];
				$arr[j] = $tmp;
			}
		}
	}
	return $arr;
}
```

##### 2、选择排序
选择排序(Selection-sort)是一种简单直观的排序算法。它的工作原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。 

**算法描述**
n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：
* 初始状态：无序区为R[1..n]，有序区为空；
* 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
* n-1趟结束，数组有序化了。

**代码实现：**
```
function selectSort($arr){
	if(!is_array($arr)) return false;
	$len = count($arr);
	if($len <= 1) return $arr;
	for($i=0; $i<$len; $i++){
		$k = $i;
		for($j=$i+1; $j<$len; $j++){
			if($arr[$k] > $arr[$j]){
				$k = $j;
			}
		}
		if($k != $i){
			$tmp = $arr[$k];
			$arr[$k] = $arr[$i];
			$arr[$i] = $tmp;
		}
	}
	return $arr;
}
```

##### 3、插入排序
插入排序（Insertion-Sort）的算法描述是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

**算法描述**
一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：
* 从第一个元素开始，该元素可以认为已经被排序；
* 取出下一个元素，在已经排序的元素序列中从后向前扫描；
* 如果该元素（已排序）大于新元素，将该元素移到下一位置；
* 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
* 将新元素插入到该位置后；
* 重复步骤2~5。

**代码实现：**
```
function insertSort($arr){
	if(!is_array($arr)) return false;
	$len = count($arr);
	if($len <= 1) return $arr;
	for($i=1; $i<$len; $i++){
		$tmp = $arr[$i];
		for($j=$i-1; $j>=0; $j--){
			if($tmp < $arr[$j]){
				$arr[$j+1] = $arr[$j];
				$arr[$j] = $tmp;
			}else{
				break;
			}
		}
	}
	return $arr;
}
```

##### 4、快速排序
快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。

 **算法描述**
快速排序使用分治法来把一个串（list）分为两个子串（sub-lists）。具体算法描述如下：
* 从数列中挑出一个元素，称为 “基准”（pivot）；
* 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
* 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

**代码实现：**
```
function quickSort($arr){
	if(!is_array($arr)) return false;
	$len = count($arr);
	if($len <= 1) return $arr;
	$left = $right = array();
	for($i=1; $i<$len; $i++){
		if($arr[$i] > $arr[0]){
			$right[] = $arr[$i];
		}else{
			$left[] = $arr[$i];
		}
	}
	$left = quickSort($left);
	$right = quickSort($right);
	return array_merge($left, array($arr[0]), $right);
}
```

<font color="red">排序参考文章：https://www.cnblogs.com/onepixel/articles/7674659.html</font>


#### 二分查找法
**算法描述**
1. 二分查找又称折半查找，它是一种效率较高的查找方法。
2. 二分查找要求：
  （1）必须采用顺序存储结构 
  （2）.必须按关键字大小有序排列
3. 原理：将数组分为三部分，依次是中值（所谓的中值就是数组中间位置的那个值）前，中值，中值后；将要查找的值和数组的中值进行比较，若小于中值则在中值前 面找，若大于中值则在中值后面找，等于中值时直接返回。然后依次是一个递归过程，将前半部分或者后半部分继续分解为三部分。
4. 实现：二分查找的实现用递归和循环两种方式

**（1）代码实现：（非递归实现）**
```
function binarySearch1($arr, $searchV){
	$high = count($arr);
	$low = 0;
	while($low <= $high){
		$middle = floor(($high+$low)/2);
		if($arr[$middle] > $searchV){
			$high = $middle - 1;
		}else if($arr[$middle] < $searchV){
			$low = $middle +1;
		}else{
			return $middle;
		}
	}
	return false;
}
```
**（2）代码实现：（递归实现）**
```
function binarySearch2($arr, $searchV, $low, $high){
	if($low <= $high){
		$middle = floor(($low+$high)/2);
		if($arr[$middle] > $searchV){
			return binarySearch2($arr, $searchV, $low, $middle-1);
		}else if($arr[$middle] < $searchV){
			return binarySearch2($arr, $searchV, $middle+1, $high);
		}else{
			return $middle;
		}
	}
	return false;
}
```