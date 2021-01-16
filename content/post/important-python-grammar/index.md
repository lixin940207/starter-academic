---
title: some important python grammar
subtitle: list, dict, set, collections, yield, decrator

# Link this post with a project
projects: []

# Date published
date: "2016-04-20T00:00:00Z"

# Date updated
lastmod: "2020-12-13T00:00:00Z"

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

#### list：
- 优先用列式推导，相比于for loop
    + 因为list comprehension更快，代码整洁
    + 不用每次循环都做if 判断
- list.append() 普通情况下是O(1)，但是，最坏是O(n)
    + 是因为CPython的数组是过度分配的数组作为内部存储，而不是链表，如果长度超过，那么以后再append，就是O(n)

#### dict：
- 创建字典也可以用字典推导
- 可哈希的，意味着不可变的（也可比较的），也就是可以作为字典的key来用的
- 字典的获取，改变和删除操作都是O(1), 但是遍历和复制是O(n)，而且这个n是曾经出现的所有n，如果曾经有过很多值后来删了很多，建议使用新的
- 字典的key的顺序不是一定的，如果需要有序可以使用collections中的OrderedDict

#### set
- set-可变的
- fronzenset-不可变的，所以可以作为dict的键

#### collections moudule
- deque
- defaultdict-有默认value的dict
- Counter-计数
- OrderedDict-有序字典

#### yield
- what it is？
a generator， it can stop a function and return a intermediate result， make code simple and preformance
- 举例：
    ```python
    //fibonacci
    def fibonacci():
        a,b = 0,1
        while True:
            yield b
            a,b = b, a+b
    ```

    可以使用next()或者for循环来从generator中获取元素，比如：
    fib = fibonacci()
    next(fib) => 1
    [next(fib) for _ in range(10)] => [1,2,3,5,8...]
- 为什么高效？
    + 当序列元素被传递到另一个函数中以进行后续处理时，一次返回一个会更高效
    + 这样就相当于提供了一个缓冲区，由另外的代码去控制，像上面的函数不需要无限大的内存去存结果
    + 当两个generator函数组合使用的时候，每次next()都处理一个元素调用两个结果然后返回，详见P42

#### decorator
- 包装已有的函数或类，使之补充功能
- 最简单的写自定义装饰器的方法是用一个函数，模版如下：
	
    ```python
    def my_decorator(function):
        def wrapped(*args, **kwargs):
            ...
            result = function(*args, **kwargs)
            ...
            return result
        return wrapped
    ```

- 如果需要参数化装饰器的话，只需要在上面模版再包一层
    ```python
    def repeat(number=3):
        def my_decorator(function):
            def wrapped(*args, **kwargs):
                ...
                for _ in range(number):
                    result = function(*args, **kwargs)
                ...
                return result
            return wrapped
    ```
- 使用场景
    + 引入日志
    + 统计时间
    + before或after函数执行前后要做的事
    + 缓存
    
#### 其他语法：
- for 后面接else，在for循环自然结束（而不是中途break）后执行
    ```python
    for i in range(10):
        print(i)
    else:
        print("for loop finished")
    ```
