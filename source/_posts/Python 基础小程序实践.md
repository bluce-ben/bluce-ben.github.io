---
title: Python 基础小程序实践
date: 2018-02-05 16:54:59
categories:
- Python
tags:
- Python
- Python实践
---
**要求：**
>创建你自己的命令行 地址簿 程序。在这个程序中，你可以添加、修改、删除和搜索你的联系人（朋友、家人和同事等等）以及它们的信息（诸如电子邮件地址和/或电话号码）。这些详细信息应该被保存下来以便以后提取。

**提示：**
>创建一个类来表示一个人的信息。使用字典储存每个人的对象，把他们的名字作为键。使用cPickle模块永久地把这些对象储存在你的硬盘上。使用字典内建的方法添加、删除和修改人员信息。

<font color="red">一旦你完成了这个程序，你就可以说是一个Python程序员了。</font>
<!--more-->
```
#!/usr/bin/python
#coding=utf-8
# Filename: address.py

import cPickle as p

class Person:
    '''Person class 
    save information of email,phone,name.'''
    contacts = {}
    contactfile = 'contact.data'

    def __init__(self):
        f = file(Person.contactfile)
        Person.contacts = p.load(f)
        f.close()
    
    def add(self, name, email, phone):
        Person.contacts[name] = {'email': email, 'phone': phone}
        f = file(Person.contactfile, 'w')
        p.dump(Person.contacts, f)
        f.close()
        print 'optetor success!'

    def search(self, name):
        if Person.contacts.has_key(name):
            print 'Name: %s Email: %s Phone: %s' % (name, Person.contacts[name]['email'], Person.contacts[name]['phone'])
        else:
            print 'search faild!'

    def delete(self, name):
        if Person.contacts.has_key(name):
            Person.contacts.pop(name)
            f = file(Person.contactfile, 'w')
            p.dump(Person.contacts, f)
            f.close()
            print 'delete success!'
        else:
            print 'name is not exists!'

    def modify(self, name, email, phone):
        if Person.contacts.has_key(name):
            if email:
                Person.contacts[name]['email'] = email
            if phone:
                Person.contacts[name]['phone'] = phone
            f = file(Person.contactfile, 'w')
            p.dump(Person.contacts, f)
            f.close()
            print 'modify success!'
        else:
            print 'name is not exists!'

def p_add():
    print "Pleace input add information"
    name = raw_input('name:')
    email = raw_input('email:')
    phone = raw_input('phone:')
    person = Person()
    person.add(name, email, phone)

def p_modify():
    name = raw_input('Pleace input modify name:')
    email = raw_input('email:')
    phone = raw_input('phone:')
    person = Person()
    person.modify(name, email, phone)

def p_delete():
    name = raw_input('Pleace input search name:')
    person = Person()
    person.delete(name)

def p_search():
    name = raw_input('Pleace input search name:')
    person = Person()
    person.search(name)

def main():
    print('''What would you want to do ?
    a=add, m=modify, d=delete, s=search''')
    optetor = raw_input('pleace input a character:')
    if(optetor == 'a'):
        p_add()
    elif(optetor == 'm'):
        p_modify()
    elif(optetor == 'd'):
        p_delete()
    elif(optetor == 's'):
        p_search()
    else:
        print 'input error'

if __name__ == '__main__':
    try:
        main()
    except Exception,e:
        print e
```