---
layout: post
title: "Cython self-Learning"
date: 2020-04-07 21:00:00 +0900
categories: [program, Python, Cython, self-learning]
tags: [Python, Cython]
---
Every Python user may hear that "Python is slower than other language (ex: C, C++, JAVA...)". There are three main theories to explain 

* Python is a dynamically typed language

* GIL (Global Interpreter Lock) [[1](https://realpython.com/python-gil/)]

* Its interpreted and not compiled

I want to survey the first point and recording the following items in this article.

* Why Python is slow from dynmically typed language view

* How slow is it?

* Future work

## Why Python is slow from dynmically typed language view
In Python, you don't have to declare ther type of variable, Python deals with this this problem for you. However, you have to declare them by yourself while you are using statically-typed language.

Therefore, statically-typed language can check whether there is type error only once in compile time, and directly execute the executing file in runtime. In dynmically-typed language, type checking happen in runtime, so  you have to check type of variables again in every executions. This slowness will be more significant if there is loop in your script. There is a vivid example on Quora [[2](https://www.quora.com/How-come-statically-typed-languages-perform-much-better-than-dynamic-ones)].

![](/img/20200406_Type Checking.png)

Fortunely, we can use Cython (an extension of Python) to declare the type of variable in advance and do type-checking at complie time.

## How slow is it?
There are many comparisions for how fast is Cython (in other hand, they show how slow is Python, too) [[4](https://blog.nelsonliu.me/2016/04/29/gsoc-week-0-cython-vs-python/)] [[5](https://notes-on-cython.readthedocs.io/en/latest/)]. However, most of experiments focus on comparision of same algorithm, and I'm curious can Cython make my reinforcement learning more faster? Therefore, I will put my attention on the efficiency of applying Cython in the project.

### Traditional comparision - Fibonacci function
On the official tutorial of Cython [[6](https://cython.readthedocs.io/en/latest/src/tutorial/index.html)], they provide an example, Fibonacci function, to introduce how to use Cython. Based on this example, we can design the first comparision.

#### Structure
```bash
.
├── Fibonacci
│   ├── cy_fib.pyx
│   ├── cy_fib_static_type.pyx
│   ├── py_fib.py
│   └── setup.py
└── test_fib.py
```
In the Fibonacci file, there are the implementation of Fibonacci function in different version. Cython files have filename extension with .pyx. Before you call the function defined in Cython file, you have to compile them with setup.py.

#### Python script (py_fib.py)
```python
def fib(n):
    """Print the Fibonacci series up to n."""
    a, b = 0, 1
    while b < n:
        a, b = b, a + b
```
#### naive Cython script (cy_fib.pyx)
```python
def fib(n):
    """Print the Fibonacci series up to n."""
    a, b = 0, 1
    while b < n:
        a, b = b, a + b
```
> "naive" is meaning there is no modifying from Python to Cython script.

#### Cython script (cy_fib_static_type.pyx)
```python
def fib(n):
    return cdef_fib(n)

cdef int cdef_fib(int n):
    """Print the Fibonacci series up to n."""
    cdef int a=0, b=1
    while b < n:
        a, b = b, a + b
```
> I declare a function to execute fibonacci function, and wrap it with a Python function.

#### Compile file (setup.py)
```python
from setuptools import setup
from Cython.Build import cythonize

setup(
    ext_modules = cythonize(['*.pyx'])
)
```
and compiled with the following order on command line
```bash
$ python setup.py build_ext --inplace
```
I got counterpart in C and executable file after executing the compile file. Then, let's start to compare the speed. With execute [test_fib.py](https://github.com/ChampDBG/PlayGround/blob/master/cython/test_fib.py), I got the following result

#### Speed comparison
```
========== pure python ==========
Record 100 times of executing statement 100000 times 
100%|███████████████████████████████| 100/100 [00:15<00:00,  6.52it/s]
the fastest result is 0.1300477999998293
the slowest result is 0.22981979999985924
the average result is 0.15290231900000437
========== naive cython  ==========
Record 100 times of executing statement 100000 times
100%|███████████████████████████████| 100/100 [00:10<00:00,  9.89it/s]
the fastest result is 0.09418899999991481
the slowest result is 0.1595972999998594
the average result is 0.10062998499999139
========== static cython ==========
Record 100 times of executing statement 100000 times
100%|███████████████████████████████| 100/100 [00:04<00:00, 20.09it/s]
the fastest result is 0.04668660000015734
the slowest result is 0.05897430000004533
the average result is 0.04952403199998571
```
![](/img/20200406_Speed Comparison Fibonacci.png)

With the above figure, we can find some interesting things.

* the fastest of Python still slower than the slowest of Cython two time.

* naive Cython can make speed faster, but still have difference to Cython.

I think the above experiment can prove that Cython can have better performance than Python. Therefore, we move to project part.

### Project Comparison - cliffwalk
The cliffwalk is a famous to show the policy difference between SARSA and Q-Learning. If you are interested about it, there is more information in Richard S. Sutton's book [7]. I used the script at [[here](https://github.com/ChampDBG/PlayGround/tree/master/cython/cliffwalk)] and got an interested result.

```
========== pure python   ==========
Record 30 times of executing cliffwalk 3 times
100%|█████████████████████████████████| 30/30 [01:22<00:00,  2.76s/it]
the fastest result is 2.2803960000019288
the slowest result is 5.677785399995628
the average result is 2.7626423766661903
========== naive cython   ==========
Record 30 times of executing cliffwalk 3 times
100%|█████████████████████████████████| 30/30 [01:14<00:00,  2.48s/it]
the fastest result is 2.4429823999962537
the slowest result is 2.5304086999967694
the average result is 2.4771759099986714
========== cython   ==========
Record 30 times of executing cliffwalk 3 times
100%|█████████████████████████████████| 30/30 [01:20<00:00,  2.68s/it]
the fastest result is 2.629339799997979
the slowest result is 2.776731000005384
the average result is 2.6813201166674845
```
![](/img/20200406_Speed Comparison cliffwalk.png)

According to the result, I maybe can said that Cython version is more stable than Python, but there is no way to said faster. I thought two possible reasons to explain this result.
1. Cython can't accelerate the script based on NumPy very much.
   
   In cliffwalk, most of my script is based on NumPy. I found some works can accelerate [[8](https://qiita.com/nena0undefined/items/730424bdffc623ab305a#cython%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%AB%E3%81%A0%E3%81%91)] [[9](https://zhuanlan.zhihu.com/p/24311879)], but some can't. I think my next purpose is finding when Cython can accelerate NumPy.

2. I declared inappropriate type for Cython.
   
   Comparing naive Cython to Cython in cliffwalk, I found that result get worse after I declared their type. To answer this problem, I should deal with the first assumption.

## Future work
1. Explain the result of cliffwalk

2. Get Deeper - release GIL in Cython [[10](https://cython.readthedocs.io/en/latest/src/userguide/parallelism.html)]

## Reference
[1] What is Python Global Interpreter Lock(GIL)? [[link](https://realpython.com/python-gil/)]

[2] How come statically typed languages perform much better than dynamic ones? [[link](https://www.quora.com/How-come-statically-typed-languages-perform-much-better-than-dynamic-ones)]

[3] Why are dynamically typed languages slow?[[link](https://stackoverflow.com/questions/761426/why-are-dynamically-typed-languages-slow)]

[4] How fast is fast, how slow is slow? A look into Python and Cython [[link](https://blog.nelsonliu.me/2016/04/29/gsoc-week-0-cython-vs-python/)]

[5] Musings on Cython[[link](https://notes-on-cython.readthedocs.io/en/latest/)]

[6] Tutorials from Cython homepage [[link](https://cython.readthedocs.io/en/latest/src/tutorial/index.html)]

[7] R. S. Sutton and A. G. Barto, Introduction to Reinforcement Learning. Cambridge, MA, USA: MIT Press, 1st ed., 1998.

[8] Cythonでnumpyを使った時の速度比較、高速化には何が必要か [[link](https://qiita.com/nena0undefined/items/730424bdffc623ab305a#cython%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%AB%E3%81%A0%E3%81%91)]

[9] Cython 基本用法 [[link](https://zhuanlan.zhihu.com/p/24311879)]

[10] Releasing from GIL in Cython [[link](https://cython.readthedocs.io/en/latest/src/userguide/parallelism.html)]

### Extension Reading
* Why is Python so slow? [[link](https://hackernoon.com/why-is-python-so-slow-e5074b6fe55b)]

* Magic lies here - Statically vs Dynamically Typed Languages[[link](https://android.jlelse.eu/magic-lies-here-statically-typed-vs-dynamically-typed-languages-d151c7f95e2b)]