
[TOC]

### 旧类和新类

旧类 深度优先
class A:
    pass

新类 广度优先
class A(object):
    pass


### 类方法 静态方法
```python
class A(object):
    __num = 0
    @classmethod
    def method(cls):
        cls.__num +=1

class B(object):
    __num = 0
    @staticmethod
    def method():
        print(123)
```


