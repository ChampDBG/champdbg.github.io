---
layout: post
title: "staticmethod 與 classmethod 的差異"
date: 2020-04-01 13:00:00 +0900
categories: [program, Python, Transfer]
tags: [staticmethod, classmethod, decorator, Python]
---
在 Python 的 class 裡面，除了可以直接定義 method，透過添加 decorator (裝飾器)，可以定義出具有特殊功能的 method。

### 閱讀本文的先備知識
1. 基本 Python 語法
2. 一般的 class 的使用方式

### 本文將帶給你的內容
1. 如何使用 @staticmethod 與 @classmethod
2. 各 method 的使用時機

---

## 如何使用 @staticmethod 與 @classmethod
通常一個 class 會包含以下的部分

```python
class classA():
    def __init__(self, num):
        self.num = num
    
    def print_number(self, x):
        print("printing by general method (%s, %s)" % self, x)
```
而 staticmethod 與 classmethod 與其他 decorator 一樣，在 method 加上即可。
```python
class classA():
    def __init__(self, num):
        self.num = num
    
    def print_number(self, x):
        print("printing by general method (%s, %s)" % (self, x))
    
    @classmethod
    def class_print_number(cls, x):
        print("printing by class method (%s, %s)" % (cls, x))
    
    @staticmethod
    def static_print_number(x):
        print("printing by static method (%s)" % x)
```
恩對，使用上沒有什麼特別的，只是要注意 method 的輸入
* classmethod 的輸入會從 self 變成 cls，後續會再說明
* staticmethod 的輸入相較於一般 method，少了 self 的部分

執行以下的結果會看到各自的差異
```
> a = classA(10)
> a.print_number(1)
printing by general method (<__main__.classA object at 0x7fb6998e9080>, 1)

> a.class_print_number(1)
printing by class method (<class '__main__.classA'>, 1)

> a.static_print_number(1)
printing by static method (1)
```

可以發現在使用上，三種 method 都只吃一個參數，而一般 method 與 classmehtod 有設定時就產生的預設參數。那麼下個問題是，這三個 method 的使用時機。

## 各 method 的使用時機
這邊的時機，指的是使用上的需求，當然也可以什麼情況都使用一般的 method。下面先定義另一個 class，方便後續說明
```python
class classEX():
    def __init__(self, num):
        self.num = num
    
    def ex_print_number(self, x):
        print("printing by general method (%s, %s)" % (self.num, x))
    
    @classmethod
    def ex_class_print_number(cls, x):
        print("printing by class method (%s, %s)" % (cls.square(x), x))
    
    @staticmethod
    def ex_static_print_number(x):
        print("printing by static method (%s)" % x)
    
    @staticmethod
    def square(x):
        return x*x
```

### 繼承 class 的輸入參數時
選擇一般的 method，並透過 self 直接調用 class 的輸入。
```
> example.ex_print_number(2)
printing by general method (10, 2)
```
### 調用 class 裡其他的 method，甚至是整個 class 時
選擇 classmethod，並透過 cls 調用，例如下面這個例子，就直接調用同一 class 中 square 這個 method。
```
> example.ex_class_print_number(2)
printing by class method (4, 2)
```
### 什麼都不需要，只是包在 class 內部
使用 staticmethod 即可
```
> example.ex_static_print_number(2)
printing by static method (2)
```

## 延伸閱讀
* [medium - decorators in Python](https://medium.com/citycoddee/python%E9%80%B2%E9%9A%8E%E6%8A%80%E5%B7%A7-3-%E7%A5%9E%E5%A5%87%E5%8F%88%E7%BE%8E%E5%A5%BD%E7%9A%84-decorator-%E5%97%B7%E5%97%9A-6559edc87bc0)

  介紹什麼是 decorator，以及如何自訂 decorator。
* [Python AST explorer](https://python-ast-explorer.com/)
  
  可以看看這幾種 method 在結構上的差異。

## References
* [stackoverflow - Difference between staticmethod and classmethod](https://stackoverflow.com/questions/136097/difference-between-staticmethod-and-classmethod)
* [GeeksforGeeks - class method vs static method in Python](https://www.geeksforgeeks.org/class-method-vs-static-method-python/)
