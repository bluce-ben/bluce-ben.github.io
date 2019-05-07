---
title: PHP异常处理机制
date: 2019-05-06 20:55:32
categories:
- PHP
tags:
- PHP
---
#### 异常与错误的区别 ####
首先要明白异常跟错误是两个不一样的概念，异常是出现正常逻辑之外的情况，而错误是指运行时出错了，比如，使用了一个未定义的变量等。异常需要抛出（throw）才能被捕捉到，而错误会导致程序执行终止。

PHP默认情况下，在代码出现了错误，如notice warning等消息时，错误信息会被直接打印到浏览器上，这个时候你通过 try catch是捕获不到错误信息的。php的try catch只能捕获到你自己 throw new Exception(" ")抛出的错误，通过throw之后，程度会直接进入到catch中继续执行。如果你想抛弃php自身的错误处理机制，这个时候可以通过set_error_handler自定义一个函数用来处理，在这个函数中你可以抛出异常，然后再通过catch捕捉到异常。
<!--more-->

#### 异常介绍 ####
  PHP异常一般是指在业务逻辑上出现的不合预期、与正常流程不同的状况，不是语法错误。

  PHP异常处理机制借鉴了java  c++等，但是PHP的异常处理机制是不健全的。异常处理机制目的是将程序正常执行的代码与出现异常如何处理的代码分离。异常主要有检测（try）、抛出（throw）和捕获（catch）等操作。

  PHP异常处理中需要注意的有，**当代码中有throw出来的异常，则必须要catch到，也即是一个 try 至少要有一个与之对应的 catch。**可以定义多个 catch 可以捕获不同的对象，php会按这些 catch 被定义的顺序执行，直到完成最后一个为止。而在这些 catch 内，又可以抛出新的异常。php的异常也像JAVA的异常的一样，可以在最外层catch捕捉，也可以在throw的地方捕捉。

  当一个异常被抛出时，其后的代码将不会继续执行，PHP 会尝试查找匹配的 "catch" 代码块。如果一个异常没有被捕获，而且又没用使用set_exception_handler()作相应的处理的话，那么 PHP 将会产生一个严重的错误，并且输出未能捕获异常(Uncaught Exception ...)的提示信息。

  PHP是无法自动捕获异常的（绝大多数），只有主动抛出异常并捕捉。也就是说，对于异常，是可预见的。目前PHP能自动抛出的异常不多，如：PDO类。


#### 异常相关函数（可自定义处理异常函数） ####
1. set_exception_handler - 设置一个用户定义的异常处理函数
1）callable set_exception_handler(callable $exception_handler)
  该函数设置默认的异常处理程序，用于没有用 try/catch 块来捕获的异常。在 exception_handler调用后异常会中止。
2）如果把自定义的异常封装到一个类上，则可以使用数组的方式调用：
  `set_exception_handler(array('MyExceptionHander', 'deal'));`

2. register_shutdown_function - 设置一个当执行关闭时可以调用的一个函数
  当脚本执行完成或意外死掉导致PHP执行即将关闭时，我们的这个函数将会被调用。
  该函数使用场景：1）页面被强制停止；2）程序代码意外终止或超时。
说明：该函数接收一个回调函数作为参数（注意：如果函数里面有写路径一定要写绝对路径，因为执行回调函数时已经脱离了脚本，是从内存中调用该函数）

3. Exception
  Exception 是系统自带的异常处理类，是所有异常的基类。（这个在PHP7中有变化）


#### PHP7中的错误和异常 ####
在 PHP7 之前的 PHP 版本一个很大的痛点就是：发生了 E_ERROR 错误，无法捕获，导致数据库的事务无法回滚造成数据不一致。
PHP 7 改变了大多数错误的报告方式。不同于传统（PHP 5）的错误报告机制，现在大多数错误被作为 Error 异常抛出（在 PHP7 中，只有 fatal error 和 recoverable error 抛出异常，其他 error 比如 warning 和 notice 的表现不变）。PHP7 中的 Error 和 Exception 的关系如图：
```
interface Throwable
    |- Exception implements Throwable
        |- ...
    |- Error implements Throwable
        |- TypeError extends Error
        |- ParseError extends Error
        |- ArithmeticError extends Error
            |- DivisionByZeroError extends ArithmeticError
        |- AssertionError extends Error
```
值得注意的是，Error 类表现上和 Exception 基本一致，可以像 Exception 异常一样被第一个匹配的 try / catch 块所捕获，如果没有匹配的 catch 块，则调用异常处理函数（事先通过 set_exception_handler() 注册7）进行处理。 如果尚未注册异常处理函数，则按照传统方式处理，被报告为一个致命错误（Fatal Error）。但并非继承自 Exception 类（要考虑到和 PHP5 的兼容性），所以不能用 catch (Exception $e) { ... } 来捕获，而需要使用 catch (Error $e) { ... }，当然，也可以使用 set_exception_handler 来捕获。

但是，用户不能自己定义类实现 Throwable，这是为了保证只有 Exception 和 Error 才可以抛出。

>注：平常使用想要自定义处理错误的时候，可直接选择继承 Exception就可以。


#### 对错误和异常的一种实践 ####
根据以上所述，我们提炼了一个对错误和异常处理较好的实践。
对于业务中不应该出现错误的地方，抛出 InternalException，而不是 Error
```
<?php
class InternalException extends Exception { /*...*/ }

function find(Array $ids) {
  if (empty($ids)) {
    throw new InternalException('ids should not be empty');
  }
  ...
}
```

**1. 只在需要清理现场的时候 catch Error**
```
<?php
try { /*...*/ }
catch (Throwable $t) {
  // log, transaction rollback, cleanup...
}
```
1. 未捕获的 Error 和Exception 通过 set_exception_handler 做后续清理和log
2. 其他错误仍然通过 set_error_handler来处理，在处理的时候使用更加明确的 FriendlyErrorType，并抛出ErrorException 记录调用栈。


**FriendlyErrorType:**
```
<?php
function FriendlyErrorType($type) 
{ 
    switch($type) 
    { 
        case E_ERROR: // 1 // 
            return 'E_ERROR'; 
        case E_WARNING: // 2 // 
            return 'E_WARNING'; 
        case E_PARSE: // 4 // 
            return 'E_PARSE'; 
        case E_NOTICE: // 8 // 
            return 'E_NOTICE'; 
        case E_CORE_ERROR: // 16 // 
            return 'E_CORE_ERROR'; 
        case E_CORE_WARNING: // 32 // 
            return 'E_CORE_WARNING'; 
        case E_COMPILE_ERROR: // 64 // 
            return 'E_COMPILE_ERROR'; 
        case E_COMPILE_WARNING: // 128 // 
            return 'E_COMPILE_WARNING'; 
        case E_USER_ERROR: // 256 // 
            return 'E_USER_ERROR'; 
        case E_USER_WARNING: // 512 // 
            return 'E_USER_WARNING'; 
        case E_USER_NOTICE: // 1024 // 
            return 'E_USER_NOTICE'; 
        case E_STRICT: // 2048 // 
            return 'E_STRICT'; 
        case E_RECOVERABLE_ERROR: // 4096 // 
            return 'E_RECOVERABLE_ERROR'; 
        case E_DEPRECATED: // 8192 // 
            return 'E_DEPRECATED'; 
        case E_USER_DEPRECATED: // 16384 // 
            return 'E_USER_DEPRECATED'; 
    } 
    return ""; 
}
```


**error_handler:**
```
<?php
function exception_error_handler($severity, $message, $file, $line) {
    if (!(error_reporting() & $severity)) {
        // This error code is not included in error_reporting
        return;
    }
 	log FriendlyErrorType($severity);
    throw new ErrorException($message, 0, $severity, $file, $line);
}
set_error_handler("exception_error_handler");
```


**总结：**
要注意，PHP中异常与错误是有区别的，并且处理机制也都是不一样的。因此，要区别对待。



参考博文：
1. [PHP异常处理机制](https://www.jianshu.com/p/fd3683407993)
2. [PHP的错误和异常处理机制](https://juejin.im/entry/5987d2ff6fb9a03c314fe732)
3. [我们什么时候应该使用异常？](http://www.laruence.com/2012/02/02/2515.html)
4. [深入理解PHP原理之异常机制](http://www.laruence.com/2010/08/03/1697.html)
