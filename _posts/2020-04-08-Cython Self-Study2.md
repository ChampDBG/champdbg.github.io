---
layout: post
title: "Cython self-Learning (2)"
date: 2020-04-08 21:00:00 +0900
categories: [program, Python, Cython, self-learning]
tags: [Python, Cython]
---
> If you haven't seen the previous [article](https://champdbg.github.io/program/python/cython/self-learning/Cython-Self-Study/), it is better to read it before you read this article.

In this article, I tried to prove my explaination, the reason why my Cython script did so bad, in last article and I found this words.

> Because static typing is often the key to large speed gains, beginners often have a tendency to type everything in sight. This cuts down on both readability and flexibility, and can even slow things down (e.g. by adding unnecessary type checks, conversions, or slow buffer unpacking). 
> 
> \- Cython tutorial - Faster code via static typing - Determining where to add types [[1](https://cython.readthedocs.io/en/latest/src/quickstart/cythonize.html#determining-where-to-add-types)]

It implies that if you declare type too more, your script may slower than Python version. Btw, there are some limits about declaring Numpy in Cython.

* Cython at a glance, and decide where to add type [[2](https://cython.readthedocs.io/en/latest/src/userguide/numpy_tutorial.html#numpy-tutorial)]

* No module type variable with Numpy [[3](https://stackoverflow.com/questions/23838241/cython-says-buffer-types-only-allowed-as-function-local-variables-even-for-ndarr)]

* Equivalent declaration between Cython and NumPy [[4](https://stackoverflow.com/questions/21851985/difference-between-np-int-np-int-int-and-np-int-t-in-cython)]

With these material, I start to modify my script and the new one is [[here](https://github.com/ChampDBG/PlayGround/tree/master/cython/cliffwalk_v2)]. The following is the place is what I refined:

* Merge function

  Remove __run_cliffwalk__ function and merge train-related parameters into __qlearn__ function. This is just for simplify the script. I add the related loop into testing, so the total calculations won't decrease.

* Declare in the right place

  In the first version, I declare lots of types on functions, but ignored the variables in these functions. In my understanding, first version had no help to speed up because calculating is happened in variables rather than output. (You can compare the __GetAcion__ function and __ValueUpdate__ function in [[v1](https://github.com/ChampDBG/PlayGround/blob/master/cython/cliffwalk/cy_cliffwalk.pyx)] and [[v2](https://github.com/ChampDBG/PlayGround/blob/master/cython/cliffwalk_v2/cy_cliffwalk.pyx)] for more information.)

## Project Comparison - cliffwalk_v2
With the refined script, I got the reasonable result.
``` 
========== pure python ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:50<00:00,  1.69s/it]
the fastest result is 1.5165109999943525
the slowest result is 3.2652387999987695
the average result is 1.6859406133327866
========== naive cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:49<00:00,  1.65s/it]
the fastest result is 1.5883004000061192
the slowest result is 1.838901100010844
the average result is 1.6543756700023853
========== cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:24<00:00,  1.21it/s]
the fastest result is 0.7837197999760974
the slowest result is 0.8648693999857642
the average result is 0.8234472199973728
```
![](/img/20200408_Speed Comparison cliffwalk_v2.png)

Although there is no dramatic progression like [[5](https://qiita.com/nena0undefined/items/730424bdffc623ab305a)], but this result finally meet my expectation. Therefore, I think my assumption in last article about "inappropriate declaration" is right.

Here is some methods like calling C function or writing pyd header could speed up. However, as the quota in the top of this article "it will cut down on both readability and flexibility", so I think the better is using them on heavy calculation instead of using it comprehensively.

### Another assuption
I had another reason to explain in last artcle. It's about "Cython can't accelerate the script based on NumPy very much". After reading some discussions, I think this reason is incorrect. Despite the fact that NumPy is fast, but you can still speed up the script with some advanced skills.

First, there are lots of functions in my script, and __Cython has to check the types of the passed arrays during the run-time__ [[6](https://stackoverflow.com/questions/49090448/how-can-i-use-cython-to-speed-up-the-numpy)]. According to that discussion, if you have lots of functions and pass arrays very often, __using class is a better choice__.

Besides, there are couple of details can make Cython speed [[7](https://stackoverflow.com/questions/42349587/cython-slow-numpy-arrays)]. In this discussion, he mentioned three methods (Turn off bounds checking and wraparound [[8](https://cython.readthedocs.io/en/latest/src/tutorial/numpy.html#tuning-indexing-further)], typed memoryview [[9](https://cython.readthedocs.io/en/latest/src/userguide/memoryviews.html)][[10](https://stackoverflow.com/questions/37432076/cython-typed-memoryviews-what-they-really-are/37497998)] and declare contiguous array [[9](https://cython.readthedocs.io/en/latest/src/userguide/memoryviews.html#c-and-fortran-contiguous-copies)]). 

However, I think [[8](https://cython.readthedocs.io/en/latest/src/tutorial/numpy.html#tuning-indexing-further)] is dangerous in common usage, but it still a chioce if necessary. Memoryview is similar to NumPy and almost no difference in usage and speed. Thus, I am not sure when/why should I use this skill. 

Btw, lots of discussions suggest to use [Julia](https://julialang.org/) if you want to make your life easier.

## Future work
1. class version cliffwalk

2. Move to next topic - GIL

3. Making life easier with Julia

## Reference
[1] Faster code via static typing [[link](https://cython.readthedocs.io/en/latest/src/quickstart/cythonize.html#determining-where-to-add-types)]

[2] Cython for NumPy users [[link](https://cython.readthedocs.io/en/latest/src/userguide/numpy_tutorial.html#numpy-tutorial)]

[3] Cython says buffer types only allowed as function local variables even for ndarray.copy() [[link](https://stackoverflow.com/questions/23838241/cython-says-buffer-types-only-allowed-as-function-local-variables-even-for-ndarr)]

[4] Difference between np.int, np.int_, int, and np.int_t in cython? [[link](https://stackoverflow.com/questions/21851985/difference-between-np-int-np-int-int-and-np-int-t-in-cython)]

[5] Cythonでnumpyを使った時の速度比較、高速化には何が必要か [[link](https://qiita.com/nena0undefined/items/730424bdffc623ab305a)]

[6] How can I use cython to speed up the numpy? [[link](https://stackoverflow.com/questions/49090448/how-can-i-use-cython-to-speed-up-the-numpy)]

[7] Cython: slow numpy arrays [[link](https://stackoverflow.com/questions/42349587/cython-slow-numpy-arrays)]

[8] Cython tutorial - Tuning indexing further [[link](https://cython.readthedocs.io/en/latest/src/tutorial/numpy.html#tuning-indexing-further)] 

[9] Cython tutorial - Typed Memoryviews [[link](https://cython.readthedocs.io/en/latest/src/userguide/memoryviews.html)]

[10] Cython typed memoryviews: what they really are? [[link](https://stackoverflow.com/questions/37432076/cython-typed-memoryviews-what-they-really-are/37497998)]

### Reading related to GIL
* Python - Global Interpreter Lock [[link](https://wiki.python.org/moin/GlobalInterpreterLock)]

* 深入 GIL: 如何寫出快速且 thread-safe 的 Python – Grok the GIL: How to write fast and thread-safe Python [[link](https://blog.louie.lu/2017/05/19/%E6%B7%B1%E5%85%A5-gil-%E5%A6%82%E4%BD%95%E5%AF%AB%E5%87%BA%E5%BF%AB%E9%80%9F%E4%B8%94-thread-safe-%E7%9A%84-python-grok-the-gil-how-to-write-fast-and-thread-safe-python/)]

* Releasing from GIL in Cython [[Link](https://cython.readthedocs.io/en/latest/src/userguide/parallelism.html)]

### Extension Reading
* Function in C [[link](https://www.geeksforgeeks.org/functions-in-c/)]

* C-Function [[link](https://fresh2refresh.com/c-programming/c-function/)]

* Why Numba and Cython are not substitutes for Julia [[link](http://www.stochasticlifestyle.com/why-numba-and-cython-are-not-substitutes-for-julia/)]

* Python を高速化する Numba, Cython 等を使って Julia Micro-Benchmarks してみた [[link](https://qiita.com/yniji/items/b7acffa02f03a94882e5)]