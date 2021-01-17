---
title: Python static method, class method, instance method

summary: Definitions of static method, class method, instance method and their application.

# Link this post with a project
projects: []

# Date published
date: "2020-11-10T00:00:00Z"

# Date updated
lastmod: "2020-11-10T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

authors:
- admin

tags:
- Python

categories:
- Python
---

python类方法、静态方法、实例方法

- 静态方法：通过类直接调用，不用创建对象，参数不需要self，不能调用类的所有属性和方法
- 实例方法：只能被类的对象调用
- 类方法：第一个参数不是self，而是cls，也可以通过类名直接调用

静态方法和类方法都用来做什么？
    类方法用在java定义多个构造函数的时候
    但是python不能定义多个初始化方法，可以借用静态方法或类方法来定义多个初始化类型,如下：
```python
# coding:utf-8
class Book(object):
 
    def __init__(self, title):
        self.title = title
 
    @classmethod
    def class_method_create(cls, title):
        book = cls(title=title) #好处是不用每次都写类名，cls代替
        return book
 
    @staticmethod
    def static_method_create(title):
        book= Book(title)
        return book
 
book1 = Book("use instance_method_create book instance")
book2 = Book.class_method_create("use class_method_create book instance")
book3 = Book.static_method_create("use static_method_create book instance")
print(book1.title)
print(book2.title)
print(book3.title)
```

实际工作中很少用到类方法和静态方法，但是在工厂模式下，使用这些是很好的选择
什么是工厂模式？有一些base class（比如不同型号的汽车）再一个生产汽车的工厂类（负责create 汽车），增加新的汽车只需要增加base class，不用改工厂类


