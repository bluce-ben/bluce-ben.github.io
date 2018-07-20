---
title: 面试 - 常见函数试题总结（PHP）
date: 2018-04-03 15:14:51
categories:
- 面试
tags:
- 面试
---
试题目录：
1、写一个能创建多级目录的PHP函数
2、请写一段代码，确保多个进程同时写入一个文件成功
3、写一个函数，尽可能高效的，从一个标准url里取出文件的扩展名
4、写一个函数，能够遍历一个文件夹下的所有文件和子文件夹。
5、无限级分类
<!--more-->

#### 1、写一个能创建多级目录的PHP函数
```
function create_multidir($path, $mode=0777){
	if(is_dir($path)){
		return "目录已经存在！";
	}else if(mkdir($path, $mode, true)){
		return true;
	}
	return false;
}
```

#### 2、请写一段代码，确保多个进程同时写入一个文件成功
```
$fp = fopen($file, 'w+');
if(flock($fp, LOCK_EX)){
	//执行逻辑
	fwrite($fp, "content");
	flock($fp, LOCK_UN);
}else{
	echo "服务器繁忙...";
}
fclose($fp);
```

#### 3、写一个函数，尽可能高效的，从一个标准url里取出文件的扩展名
例如:http://www.sina.com.cn/abc/de/fg.php?id=1需要取出php或.php
```
function getFileExtension($url){
	$url_arr = parse_url($url);
	$filename = basename($url_arr['path']);
	$file_arr = explode('.', $filename);
	return $file_arr[1];
}
其中，最后获取扩展名，还可使用字符串截取函数。
return strstr($filename, '.');
```

#### 4、写一个函数，能够遍历一个文件夹下的所有文件和子文件夹。
```
function getAllDirAndFile($path){
	$files = array();
	if(is_dir($path)){
		$handle = opendir($path);
		while(($file=readdir($handle)) !== false){
			if($file != '.' && $file != '..'){
				if(is_dir($path.'/'.$file)){
					$files[$file] = getAllDirAndFile($path.'/'.$file);
				}else{
					$files[] = $file;
				}
			}
		}
		closedir($handle);
	}
	return $files;
}
```

#### 5、无限级分类
创建类别表如下：
```
CREATE TABLE category(
cat_id smallint unsigned not null auto_increment primary key comment'类别ID',
cat_name VARCHAR(30)NOT NULL DEFAULT''COMMENT'类别名称',
parent_id SMALLINT UNSIGNED NOT NULL DEFAULT 0 COMMENT'类别父ID'
)engine=MyISAM charset=utf8;
```
编写一个函数，递归遍历，实现无限分类。

代码实现：（结果显示层级分明）
```
function getAllCategory($arr, $pid=0){
	$categories = array();
	foreach($arr as $val){
		if($val['parent_id'] == $pid){
			$categories[$val['cat_id']] = $val;
			$categories[$val['cat_id']]['childs'] = getAllCategory($arr, $val['cat_id']);
		}
	}
	return $categories;
}
```
代码实现：（结果显示在一个层级，通过level标明层级）
```
function getAllCategory($arr, $pid=0, $level){
	static $categories = array();
	foreach($arr as $val){
		if($val['parent_id'] == $pid){
			$categories[] = $val;
			$categories['level'] = $level;
			getAllCategory($arr, $val['cat_id'], $level++);
		}
	}
	return $categories;
}
```