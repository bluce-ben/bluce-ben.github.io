---
title: Python 模块之cPickle
date: 2018-02-05 17:28:00
categories:
- Python
tags:
- Python
- Python模块
---
在python中，一般可以使用pickle类来进行python对象的序列化，而cPickle提供了一个更快速简单的接口，如python文档所说的：“cPickle – A faster pickle”。使用cPickle模块永久地把这些对象储存在你的硬盘上。

>**对象持久化：**
如果希望透明地存储 Python 对象，而不丢失其身份和类型等信息，则需要某种形式的对象序列化：它是一个将任意复杂的对象转成对象的文本或二进制表示的过程。同样，必须能够将对象经过序列化后的形式恢复到原有的对象。在 Python 中，这种序列化过程称为 pickle，可以将对象 pickle 成字符串、磁盘上的文件或者任何类似于文件的对象，也可以将这些字符串、文件或任何类似于文件的对象 unpickle 成原来的对象。

<!--more-->
cPickle可以对任意一种类型的python对象进行序列化操作，比如list，dict，甚至是一个类的对象等。 pickle 模块提供了以下函数对： dumps(object) 返回一个字符串，它包含一个 pickle 格式的对象； loads(string) 返回包含在 pickle 字符串中的对象； dump(object, file) 将对象写到文件，这个文件可以是实际的物理文件，但也可以是任何类似于文件的对象，这个对象具有 write() 方法，可以接受单个的字符串参数； load(file) 返回包含在 pickle 文件中的对象。

##### dump() 和 load()
```
>>> a1 = 'apple'
>>> b1 = {1: 'One', 2: 'Two', 3: 'Three'}
>>> c1 = ['fee', 'fie', 'foe', 'fum']
>>> f1 = file('temp.pkl', 'wb')
>>> pickle.dump(a1, f1, True)
>>> pickle.dump(b1, f1, True)
>>> pickle.dump(c1, f1, True)
>>> f1.close()
>>> f2 = file('temp.pkl', 'rb')
>>> a2 = pickle.load(f2)
>>> a2
'apple'
>>> b2 = pickle.load(f2)
>>> b2
{1: 'One', 2: 'Two', 3: 'Three'}
>>> c2 = pickle.load(f2)
>>> c2
['fee', 'fie', 'foe', 'fum']
>>> f2.close()
```
>**dump： 将python对象序列化保存到本地的文件**
dump函数需要指定两个参数，第一个是需要序列化的python对象名称，第二个是本地的文件，需要注意的是，在这里需要使用open函数打开一个文件，并指定“写”操作 
**load：载入本地文件，恢复python对象**
同dump一样，这里需要使用open函数打开本地的一个文件，并指定“读”操作 


##### dumps() 和 loads()
```
>>> import cPickle as pickle
>>> t1 = ('this is a string', 42, [1, 2, 3], None)
>>> t1
('this is a string', 42, [1, 2, 3], None)
>>> p1 = pickle.dumps(t1)
>>> p1
"(S'this is a string'/nI42/n(lp1/nI1/naI2/naI3/naNtp2/n."
>>> print p1
(S'this is a string'
I42
(lp1
I1
aI2
aI3
aNtp2
.
>>> t2 = pickle.loads(p1)
>>> t2
('this is a string', 42, [1, 2, 3], None)
>>> p2 = pickle.dumps(t1, True)
>>> p2
'(U/x10this is a stringK*]q/x01(K/x01K/x02K/x03eNtq/x02.'
>>> t3 = pickle.loads(p2)
>>> t3
('this is a string', 42, [1, 2, 3], None)
```
>**dumps：将python对象序列化保存到一个字符串变量中**
**loads：从字符串变量中载入python对象**