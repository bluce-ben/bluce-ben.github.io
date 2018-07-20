---
title: Python 高级整理（Python中文手册）
date: 2018-02-05 13:52:23
categories:
- Python
tags:
- Python知识点
- Python
---
记住，Python把在程序中用到的任何东西都称为 对象 。这是从广义上说的。因此我们不会说“某某 东西 ”，我们说“某个 对象 ”。
**给面向对象编程用户的注释：**
>就每一个东西包括数、字符串甚至函数都是对象这一点来说，Python是极其完全地面向对象的。 

<!--more-->
### 1、模块 ###
#### （1）简介 ####
你已经学习了如何在你的程序中定义一次函数而重用代码。如果你想要在其他程序中重用很多函数，那么你该如何编写程序呢？你可能已经猜到了，答案是使用模块。模块基本上就是一个包含了所有你定义的函数和变量的文件。为了在其他程序中重用模块，模块的文件名必须以.py为扩展名。

模块可以从其他程序 输入 以便利用它的功能。这也是我们使用Python标准库的方法。
模块的用处在于它能为你在别的程序中重用它提供的服务和功能。Python附带的标准库就是这样一组模块的例子。

#### （2）字节编译的.pyc文件 ####
输入一个模块相对来说是一个比较费时的事情，所以Python做了一些技巧，以便使输入模块更加快一些。一种方法是创建 字节编译的文件 ，这些文件以.pyc作为扩展名。字节编译的文件与Python变换程序的中间状态有关。当你在下次从别的程序输入这个模块的时候，.pyc文件是十分有用的——它会快得多，因为一部分输入模块所需的处理已经完成了。另外，这些字节编译的文件也是与平台无关的。

#### （3）from..import 语句 ####
如果你想要直接输入argv变量到你的程序中（避免在每次使用它时打sys.），那么你可以使用from sys import argv语句。如果你想要输入所有sys模块使用的名字，那么你可以使用from sys import \*语句。这对于所有模块都适用。**一般说来，应该避免使用from..import而使用import语句，因为这样可以使你的程序更加易读，也可以避免名称的冲突。**

#### （4）模块的 \_\_name\_\_ ####
每个模块都有一个名称，在模块中可以通过语句来找出模块的名称。这在一个场合特别有用——就如前面所提到的，当一个模块被第一次输入的时候，这个模块的主块将被运行。假如我们只想在程序本身被使用的时候运行主块，而在它被别的模块输入的时候不运行主块，我们该怎么做呢？这可以通过模块的\_\_name\_\_属性完成。

**例8.2 使用模块的\_\_name\_\_**
```
#!/usr/bin/python
# Filename: using_name.py

if __name__ == '__main__':
    print 'This program is being run by itself'
else:
    print 'I am being imported from another module' 
```
**输出：**
```
$ python using_name.py
This program is being run by itself

$ python
>>> import using_name
I am being imported from another module
>>> 
```
每个Python模块都有它的\_\_name\_\_，如果它是'\_\_main\_\_'，这说明这个模块被用户单独运行，我们可以进行相应的恰当操作。

#### （5）dir() 函数 ####
你可以使用内建的dir函数来列出模块定义的标识符。标识符有函数、类和变量。
当你为dir()提供一个模块名的时候，它返回模块定义的名称列表。如果不提供参数，它返回当前模块中定义的名称列表。


### 2、面向对象的编程 ###
#### （1）简介 ####
在我们的程序中，我们根据操作数据的函数或语句块来设计程序的。这被称为 面向过程的 编程。还有一种把数据和功能结合起来，用称为对象的东西包裹起来组织程序的方法。这种方法称为 面向对象的 编程理念。在大多数时候你可以使用过程性编程，但是有些时候当你想要编写大型程序或是寻求一个更加合适的解决方案的时候，你就得使用面向对象的编程技术。

类和对象是面向对象编程的两个主要方面。**类**创建一个新类型，而**对象**这个类的 实例 。这类似于你有一个int类型的变量，这存储整数的变量是int类的实例（对象）。

*注意，即便是整数也被作为对象（属于int类）。这和C++、Java（1.5版之前）把整数纯粹作为类型是不同的。通过help(int)了解更多这个类的详情。*

对象可以使用普通的 属于 对象的变量存储数据。属于一个对象或类的变量被称为**域**。对象也可以使用 属于 类的函数来具有功能。这样的函数被称为类的方法。这些术语帮助我们把它们与孤立的函数和变量区分开来。域和方法可以合称为类的**属性**。

域有两种类型——属于每个实例/类的对象或属于类本身。它们分别被称为**实例变量**和**类变量**。

类使用class关键字创建。类的域和方法被列在一个缩进块中。

#### （2）self ####
类的方法与普通的函数只有一个特别的区别——它们必须有一个额外的第一个参数名称，但是在调用这个方法的时候你不为这个参数赋值，Python会提供这个值。这个特别的变量指对象本身，按照惯例它的名称是self。

虽然你可以给这个参数任何名称，但是 强烈建议 你使用self这个名称——其他名称都是不赞成你使用的。使用一个标准的名称有很多优点——你的程序读者可以迅速识别它，如果使用self的话，还有些IDE（集成开发环境）也可以帮助你。

你一定很奇怪Python如何给self赋值以及为何你不需要给它赋值。举一个例子会使此变得清晰。假如你有一个类称为MyClass和这个类的一个实例MyObject。当你调用这个对象的方法MyObject.method(arg1, arg2)的时候，这会由Python自动转为MyClass.method(MyObject, arg1, arg2)——这就是self的原理了。

**（注：这也意味着如果你有一个不需要参数的方法，你还是得给这个方法定义一个self参数。）**

#### （3）创建一个类 ####
```
例11.1 创建一个类

#!/usr/bin/python
# Filename: simplestclass.py

class Person:
    pass # An empty block

p = Person()
print p 
```
**输出：**
```
$ python simplestclass.py
<__main__.Person instance at 0xf6fcb18c> 
```
我们使用class语句后跟类名，创建了一个新的类。这后面跟着一个缩进的语句块形成类体。在这个例子中，我们使用了一个空白块，它由pass语句表示。

接下来，我们使用类名后跟一对圆括号来创建一个对象/实例。为了验证，我们简单地打印了这个变量的类型。它告诉我们我们已经在\_\_main\_\_模块中有了一个Person类的实例。

可以注意到存储对象的计算机内存地址也打印了出来。这个地址在你的计算机上会是另外一个值，因为Python可以在任何空位存储对象。

#### （4）__init__方法 ####
在Python的类中有很多方法的名字有特殊的重要意义。现在我们将学习\_\_init\_\_方法的意义。

\_\_init\_\_方法在类的一个对象被建立时，马上运行。这个方法可以用来对你的对象做一些你希望的 初始化 。注意，这个名称的开始和结尾都是双下划线。
```
例11.3 使用__init__方法

#!/usr/bin/python
# Filename: class_init.py

class Person:
    def __init__(self, name):
        self.name = name
    def sayHi(self):
        print 'Hello, my name is', self.name

p = Person('Swaroop')
p.sayHi()

# This short example can also be written as Person('Swaroop').sayHi() 
```
**输出：**
```
$ python class_init.py
Hello, my name is Swaroop 
```
这里，我们把\_\_init\_\_方法定义为取一个参数name（以及普通的参数self）。在这个\_\_init\_\_里，我们只是创建一个新的域，也称为name。注意它们是两个不同的变量，尽管它们有相同的名字。点号使我们能够区分它们。

最重要的是，我们没有专门调用\_\_init\_\_方法，只是在创建一个类的新实例的时候，把参数包括在圆括号内跟在类名后面，从而传递给\_\_init\_\_方法。这是这种方法的重要之处。与 *construct* 的概念类似。

现在，我们能够在我们的方法中使用self.name域。这在sayHi方法中得到了验证。

#### （5）类与对象的方法 ####
我们已经讨论了类与对象的功能部分，现在我们来看一下它的数据部分。事实上，它们只是与类和对象的**名称空间** 绑定 的普通变量，即这些名称只在这些类与对象的前提下有效。

有两种类型的 **域** ——类的变量和对象的变量，它们根据是类还是对象 拥有 这个变量而区分。

**类的变量** 由一个类的所有对象（实例）共享使用。只有一个类变量的拷贝，所以当某个对象对类的变量做了改动的时候，这个改动会反映到所有其他的实例上。

**对象的变量** 由类的每个对象/实例拥有。因此每个对象有自己对这个域的一份拷贝，即它们不是共享的，在同一个类的不同实例中，虽然对象的变量有相同的名称，但是是互不相关的。
```
例11.4 使用类与对象的变量

#!/usr/bin/python
# Filename: objvar.py

class Person:
    '''Represents a person.'''
    population = 0

    def __init__(self, name):
        '''Initializes the person's data.'''
        self.name = name
        print '(Initializing %s)' % self.name

        # When this person is created, he/she
        # adds to the population
        Person.population += 1

    def __del__(self):
        '''I am dying.'''
        print '%s says bye.' % self.name

        Person.population -= 1

        if Person.population == 0:
            print 'I am the last one.'
        else:
            print 'There are still %d people left.' % Person.population

    def sayHi(self):
        '''Greeting by the person.

        Really, that's all it does.'''
        print 'Hi, my name is %s.' % self.name

    def howMany(self):
        '''Prints the current population.'''
        if Person.population == 1:
            print 'I am the only person here.'
        else:
            print 'We have %d persons here.' % Person.population

swaroop = Person('Swaroop')
swaroop.sayHi()
swaroop.howMany()

kalam = Person('Abdul Kalam')
kalam.sayHi()
kalam.howMany()

swaroop.sayHi()
swaroop.howMany() 
```
**输出：**
```
$ python objvar.py
(Initializing Swaroop)
Hi, my name is Swaroop.
I am the only person here.
(Initializing Abdul Kalam)
Hi, my name is Abdul Kalam.
We have 2 persons here.
Hi, my name is Swaroop.
We have 2 persons here.
Abdul Kalam says bye.
There are still 1 people left.
Swaroop says bye.
I am the last one. 
```
这是一个很长的例子，但是它有助于说明类与对象的变量的本质。这里，population属于Person类，因此是一个类的变量。name变量属于对象（它使用self赋值）因此是对象的变量。

观察可以发现\_\_init\_\_方法用一个名字来初始化Person实例。在这个方法中，我们让population增加1，这是因为我们增加了一个人。同样可以发现，self.name的值根据每个对象指定，这表明了它作为对象的变量的本质。

记住，你只能使用self变量来参考同一个对象的变量和方法。这被称为 属性参考 。

在这个程序中，我们还看到docstring对于类和方法同样有用。我们可以在运行时使用Person.\_\_doc\_\_和Person.sayHi.\_\_doc\_\_来分别访问类与方法的文档字符串。

就如同\_\_init\_\_方法一样，还有一个特殊的方法\_\_del\_\_，它在对象消逝的时候被调用。对象消逝即对象不再被使用，它所占用的内存将返回给系统作它用。在这个方法里面，我们只是简单地把Person.population减1。

当对象不再被使用时，\_\_del\_\_方法运行，但是很难保证这个方法究竟在 什么时候 运行。如果你想要指明它的运行，你就得使用del语句，就如同我们在以前的例子中使用的那样。

**给C++/Java/C#程序员的注释**
Python中所有的类成员（包括数据成员）都是 公共的 ，所有的方法都是 有效的 。
只有一个例外：如果你使用的数据成员名称以 双下划线前缀 比如\_\_privatevar，Python的名称管理体系会有效地把它作为私有变量。
这样就有一个惯例，如果某个变量只想在类或对象中使用，就应该以单下划线前缀。而其他的名称都将作为公共的，可以被其他类/对象使用。记住这只是一个惯例，并不是Python所要求的（与双下划线前缀不同）。
同样，注意\_\_del\_\_方法与 destructor 的概念类似。

**一些特殊的方法：**
<style type="text/css">
	table th:first-of-type {
		width: 100px;
	}
</style>

名称                  |           说明 
----------------------|------------------------------------------------------
__init__(self,...)    | 这个方法在新建对象恰好要被返回使用之前被调用。 
__del__(self)         | 恰好在对象要被删除之前调用。 
__str__(self)         | 在我们对对象使用print语句或是使用str()的时候调用。 
__lt__(self,other)    | 当使用 小于 运算符（<）的时候调用。类似地，对于所有的运算符（+，>等等）都有特殊的方法。 
__getitem__(self,key) | 使用x[key]索引操作符的时候调用。 
__len__(self)         | 对序列对象使用内建的len()函数的时候调用。 

#### （6）继承 ####
面向对象的编程带来的主要好处之一是代码的重用，实现这种重用的方法之一是通过 继承 机制。继承完全可以理解成类之间的 类型和子类型 关系。

假设你想要写一个程序来记录学校之中的教师和学生情况。他们有一些共同属性，比如姓名、年龄和地址。他们也有专有的属性，比如教师的薪水、课程和假期，学生的成绩和学费。

你可以为教师和学生建立两个独立的类来处理它们，但是这样做的话，如果要增加一个新的共有属性，就意味着要在这两个独立的类中都增加这个属性。这很快就会显得不实用。

一个比较好的方法是创建一个共同的类称为SchoolMember然后让教师和学生的类 继承 这个共同的类。即它们都是这个类型（类）的子类型，然后我们再为这些子类型添加专有的属性。

使用这种方法有很多优点。如果我们增加/改变了SchoolMember中的任何功能，它会自动地反映到子类型之中。例如，你要为教师和学生都增加一个新的身份证域，那么你只需简单地把它加到SchoolMember类中。然而，在一个子类型之中做的改动不会影响到别的子类型。另外一个优点是你可以把教师和学生对象都作为SchoolMember对象来使用，这在某些场合特别有用，比如统计学校成员的人数。一个子类型在任何需要父类型的场合可以被替换成父类型，即对象可以被视作是父类的实例，这种现象被称为多态现象。

另外，我们会发现在 重用 父类的代码的时候，我们无需在不同的类中重复它。而如果我们使用独立的类的话，我们就不得不这么做了。

在上述的场合中，SchoolMember类被称为 基本类 或 超类 。而Teacher和Student类被称为 导出类 或 子类 。
```
例11.5 使用继承

#!/usr/bin/python
# Filename: inherit.py

class SchoolMember:
    '''Represents any school member.'''
    def __init__(self, name, age):
        self.name = name
        self.age = age
        print '(Initialized SchoolMember: %s)' % self.name

    def tell(self):
        '''Tell my details.'''
        print 'Name:"%s" Age:"%s"' % (self.name, self.age),

class Teacher(SchoolMember):
    '''Represents a teacher.'''
    def __init__(self, name, age, salary):
        SchoolMember.__init__(self, name, age)
        self.salary = salary
        print '(Initialized Teacher: %s)' % self.name

    def tell(self):
        SchoolMember.tell(self)
        print 'Salary: "%d"' % self.salary

class Student(SchoolMember):
    '''Represents a student.'''
    def __init__(self, name, age, marks):
        SchoolMember.__init__(self, name, age)
        self.marks = marks
        print '(Initialized Student: %s)' % self.name

    def tell(self):
        SchoolMember.tell(self)
        print 'Marks: "%d"' % self.marks

t = Teacher('Mrs. Shrividya', 40, 30000)
s = Student('Swaroop', 22, 75)

print # prints a blank line

members = [t, s]
for member in members:
    member.tell() # works for both Teachers and Students 
```
**输出：**
```
$ python inherit.py
(Initialized SchoolMember: Mrs. Shrividya)
(Initialized Teacher: Mrs. Shrividya)
(Initialized SchoolMember: Swaroop)
(Initialized Student: Swaroop)

Name:"Mrs. Shrividya" Age:"40" Salary: "30000"
Name:"Swaroop" Age:"22" Marks: "75" 
```
为了使用继承，我们把基本类的名称作为一个元组跟在定义类时的类名称之后。然后，我们注意到基本类的__init__方法专门使用self变量调用，这样我们就可以初始化对象的基本类部分。这一点十分重要——Python不会自动调用基本类的constructor，你得亲自专门调用它。

我们还观察到我们在方法调用之前加上类名称前缀，然后把self变量及其他参数传递给它。

注意，在我们使用SchoolMember类的tell方法的时候，我们把Teacher和Student的实例仅仅作为SchoolMember的实例。

另外，在这个例子中，我们调用了子类型的tell方法，而不是SchoolMember类的tell方法。可以这样来理解，Python总是首先查找对应类型的方法，在这个例子中就是如此。如果它不能在导出类中找到对应的方法，它才开始到基本类中逐个查找。基本类是在类定义的时候，在元组之中指明的。

一个术语的注释——如果在继承元组中列了一个以上的类，那么它就被称作 多重继承 。


### 3、输入/输出 ###
在很多时候，你会想要让你的程序与用户（可能是你自己）交互。你会从用户那里得到输入，然后打印一些结果。我们可以分别使用raw_input和print语句来完成这些功能。对于输出，你也可以使用多种多样的str（字符串）类。例如，你能够使用rjust方法来得到一个按一定宽度右对齐的字符串。利用help(str)获得更多详情。

另一个常用的输入/输出类型是处理文件。创建、读和写文件的能力是许多程序所必需的，我们将会在这章探索如何实现这些功能。
#### （1）文件 ####
你可以通过创建一个file类的对象来打开一个文件，分别使用file类的read、readline或write方法来恰当地读写文件。对文件的读写能力依赖于你在打开文件时指定的模式。最后，当你完成对文件的操作的时候，你调用close方法来告诉Python我们完成了对文件的使用。
```
例12.1 使用文件

#!/usr/bin/python
# Filename: using_file.py

poem = '''\
Programming is fun
When the work is done
if you wanna make your work also fun:
        use Python!
'''

f = file('poem.txt', 'w') # open for 'w'riting
f.write(poem) # write text to file
f.close() # close the file

f = file('poem.txt')
# if no mode is specified, 'r'ead mode is assumed by default
while True:
    line = f.readline()
    if len(line) == 0: # Zero length indicates EOF
        break
    print line,
    # Notice comma to avoid automatic newline added by Python
f.close() # close the file 
```
**输出：**
```
$ python using_file.py
Programming is fun
When the work is done
if you wanna make your work also fun:
        use Python! 
```
首先，我们通过指明我们希望打开的文件和模式来创建一个file类的实例。模式可以为读模式（'r'）、写模式（'w'）或追加模式（'a'）。事实上还有多得多的模式可以使用，你可以使用help(file)来了解它们的详情。

我们首先用写模式打开文件，然后使用file类的write方法来写文件，最后我们用close关闭这个文件。

接下来，我们再一次打开同一个文件来读文件。如果我们没有指定模式，读模式会作为默认的模式。在一个循环中，我们使用readline方法读文件的每一行。这个方法返回包括行末换行符的一个完整行。所以，当一个 空的 字符串被返回的时候，即表示文件末已经到达了，于是我们停止循环。

注意，因为从文件读到的内容已经以换行符结尾，所以我们在print语句上**使用逗号来消除自动换行**。最后，我们用close关闭这个文件。

现在，来看一下poem.txt文件的内容来验证程序确实工作正常了。


#### （2）存储器 ####
Python提供一个标准的模块，称为pickle。使用它你可以在一个文件中储存任何Python对象，之后你又可以把它完整无缺地取出来。这被称为 持久地 储存对象。

还有另一个模块称为cPickle，它的功能和pickle模块完全相同，只不过它是用C语言编写的，因此要快得多（比pickle快1000倍）。你可以使用它们中的任一个，而我们在这里将使用cPickle模块。记住，我们把这两个模块都简称为pickle模块。
```
例12.2 储存与取储存

#!/usr/bin/python
# Filename: pickling.py

import cPickle as p
#import pickle as p

shoplistfile = 'shoplist.data'
# the name of the file where we will store the object

shoplist = ['apple', 'mango', 'carrot']

# Write to the file
f = file(shoplistfile, 'w')
p.dump(shoplist, f) # dump the object to a file
f.close()

del shoplist # remove the shoplist

# Read back from the storage
f = file(shoplistfile)
storedlist = p.load(f)
print storedlist 
```
**输出：**
```
$ python pickling.py
['apple', 'mango', 'carrot'] 
```
首先，请注意我们使用了import..as语法。这是一种便利方法，以便于我们可以使用更短的模块名称。在这个例子中，它还让我们能够通过简单地改变一行就切换到另一个模块（cPickle或者pickle）！在程序的其余部分的时候，我们简单地把这个模块称为p。

为了在文件里储存一个对象，首先以写模式打开一个file对象，然后调用储存器模块的dump函数，把对象储存到打开的文件中。这个过程称为 储存 。

接下来，我们使用pickle模块的load函数的返回来取回对象。这个过程称为 取储存 。



### 4、异常 ###
当你的程序中出现某些 异常的 状况的时候，异常就发生了。例如，当你想要读某个文件的时候，而那个文件不存在。或者在程序运行的时候，你不小心把它删除了。上述这些情况可以使用异常来处理。

假如你的程序中有一些无效的语句，会怎么样呢？Python会引发并告诉你那里有一个错误，从而处理这样的情况。
#### （1）处理异常 ####
我们可以使用try..except语句来处理异常。我们把通常的语句放在try-块中，而把我们的错误处理语句放在except-块中。
```
例13.1 处理异常

#!/usr/bin/python
# Filename: try_except.py

import sys

try:
    s = raw_input('Enter something --> ')
except EOFError:
    print '\nWhy did you do an EOF on me?'
    sys.exit() # exit the program
except:
    print '\nSome error/exception occurred.'
    # here, we are not exiting the program

print 'Done' 
```
**输出：**
```
$ python try_except.py
Enter something -->
Why did you do an EOF on me?

$ python try_except.py
Enter something --> Python is exceptional!
Done
```
我们把所有可能引发错误的语句放在try块中，然后在except从句/块中处理所有的错误和异常。except从句可以专门处理单一的错误或异常，或者一组包括在圆括号内的错误/异常。如果没有给出错误或异常的名称，它会处理 所有的 错误和异常。对于每个try从句，至少都有一个相关联的except从句。

如果某个错误或异常没有被处理，默认的Python处理器就会被调用。它会终止程序的运行，并且打印一个消息，我们已经看到了这样的处理。

你还可以让try..catch块关联上一个else从句。当没有异常发生的时候，else从句将被执行。

我们还可以得到异常对象，从而获取更多有个这个异常的信息。

#### （2）引发异常 ####
你可以使用raise语句 引发 异常。你还得指明错误/异常的名称和伴随异常 触发的 异常对象。你可以引发的错误或异常应该分别是一个Error或Exception类的直接或间接导出类。
```
例13.2 如何引发异常

#!/usr/bin/python
# Filename: raising.py

class ShortInputException(Exception):
    '''A user-defined exception class.'''
    def __init__(self, length, atleast):
        Exception.__init__(self)
        self.length = length
        self.atleast = atleast

try:
    s = raw_input('Enter something --> ')
    if len(s) < 3:
        raise ShortInputException(len(s), 3)
    # Other work can continue as usual here
except EOFError:
    print '\nWhy did you do an EOF on me?'
except ShortInputException, x:
    print 'ShortInputException: The input was of length %d, \
          was expecting at least %d' % (x.length, x.atleast)
else:
    print 'No exception was raised.' 
```
**输出：**
```
$ python raising.py
Enter something -->
Why did you do an EOF on me?

$ python raising.py
Enter something --> ab
ShortInputException: The input was of length 2, was expecting at least 3

$ python raising.py
Enter something --> abc
No exception was raised.
```
这里，我们创建了我们自己的异常类型，其实我们可以使用任何预定义的异常/错误。这个新的异常类型是ShortInputException类。它有两个域——length是给定输入的长度，atleast则是程序期望的最小长度。

在except从句中，我们提供了错误类和用来表示错误/异常对象的变量。这与函数调用中的形参和实参概念类似。在这个特别的except从句中，我们使用异常对象的length和atleast域来为用户打印一个恰当的消息。

#### （3）try..finally ####
假如你在读一个文件的时候，希望在无论异常发生与否的情况下都关闭文件，该怎么做呢？这可以使用finally块来完成。注意，在一个try块下，你可以同时使用except从句和finally块。如果你要同时使用它们的话，需要把一个嵌入另外一个。
```
例13.3 使用finally

#!/usr/bin/python
# Filename: finally.py

import time

try:
    f = file('poem.txt')
    while True: # our usual file-reading idiom
        line = f.readline()
        if len(line) == 0:
            break
        time.sleep(2)
        print line,
finally:
    f.close()
    print 'Cleaning up...closed the file' 
```
**输出：**
```
$ python finally.py
Programming is fun
When the work is done
Cleaning up...closed the file
Traceback (most recent call last):
  File "finally.py", line 12, in ?
    time.sleep(2)
KeyboardInterrupt
```
我们进行通常的读文件工作，但是我有意在每打印一行之前用time.sleep方法暂停2秒钟。这样做的原因是让程序运行得慢一些（Python由于其本质通常运行得很快）。在程序运行的时候，按Ctrl-c中断/取消程序。

我们可以观察到KeyboardInterrupt异常被触发，程序退出。但是在程序退出之前，finally从句仍然被执行，把文件关闭


### 5、其它知识点 ###
#### （1）在函数中接收元组和列表 ####
当要使函数接收元组或字典形式的参数的时候，有一种特殊的方法，它分别使用*和**前缀。这种方法在函数需要获取可变数量的参数的时候特别有用。

由于在args变量前有*前缀，所有多余的函数参数都会作为一个元组存储在args中。如果使用的是**前缀，多余的参数则会被认为是一个字典的键/值对。

#### （2）lambda形式 ####
lambda语句被用来创建新的函数对象，并且在运行时返回它们。
```
例15.2 使用lambda形式

#!/usr/bin/python
# Filename: lambda.py

def make_repeater(n):
    return lambda s: s*n

twice = make_repeater(2)

print twice('word')
print twice(5) 
```
**输出：**
```
$ python lambda.py
wordword
10
```
这里，我们使用了make_repeater函数在运行时创建新的函数对象，并且返回它。lambda语句用来创建函数对象。本质上，lambda需要一个参数，后面仅跟单个表达式作为函数体，而表达式的值被这个新建的函数返回。注意，即便是print语句也不能用在lambda形式中，只能使用表达式。

#### （3）exec 和 eval 语句 ####
exec语句用来执行储存在字符串或文件中的Python语句。例如，我们可以在运行时生成一个包含Python代码的字符串，然后使用exec语句执行这些语句。下面是一个简单的例子。
```
>>> exec 'print "Hello World"'
Hello World 
```
eval语句用来计算存储在字符串中的有效Python表达式。下面是一个简单的例子。
```
>>> eval('2*3')
6 
```