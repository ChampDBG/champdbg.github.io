---
layout: post
title: "Cython self-Learning (3)"
date: 2020-04-10 17:00:00 +0900
categories: [program, Python, Cython, self-learning]
tags: [Python, Cython]
---
> If you haven't seen the previous [article](https://champdbg.github.io/program/python/cython/self-learning/Cython-Self-Study2/), it is better to read it before you read this article.

Yesterday, I found there is a discussion [[1](https://stackoverflow.com/questions/49090448/how-can-i-use-cython-to-speed-up-the-numpy)] implies the possibility to speed up more in Cython with class. Therefore, I wrote a class version for cliffwalk.

I refer tutroial [[2](https://cython.readthedocs.io/en/latest/src/tutorial/cdef_classes.html)] and Dr. Stefan Behnel's slide [[3](http://www.behnel.de/cython200910/talk.html)] and program the class version of cliffwalk is [[here](https://github.com/ChampDBG/PlayGround/tree/master/cython/cliffwalk_v3)]. There is a extra function (exec_cliffwalk) in the cy_cliffwalk.pyx for easy to call from outside. Let's observe the result.

## Project Comparison - cliffwalk_v3
``` 
========== pure python ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:49<00:00,  1.66s/it]
the fastest result is 1.565133900003275
the slowest result is 1.8010003000090364
the average result is 1.662423889998657
========== naive cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:48<00:00,  1.62s/it]
the fastest result is 1.5775447000050917
the slowest result is 1.801998300012201
the average result is 1.622433603334745
========== cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:39<00:00,  1.33s/it]
the fastest result is 1.2863884999824222
the slowest result is 1.3819133999932092
the average result is 1.3261936766687237
```
![](/img/20200410_Speed Comparison cliffwalk_v3.png)

Although Cython version is still the fastest one, we got worse performance than function version (average 0.82). Comparing to function, there is no siginficant performance difference in Python version (average 1.68) and naive Cython version (average 1.65).

Actually, I have no idea about why performance get worse so much, so I use cython buildin function to observe where may can modify. With the following command
```bash
$ cython -a ./cy_cliffwalk.pyx
```

and it will generate a html file. After open the html, there are some lines were highlighted. It means that cython and python interact at this line. In other word, there are the places that may make your script slow down, so it is better to turn them into to Cython version. 
![](/img/20200410_Interacting Place.png)

I did lots of accesses from the array, using memroyview may be a better choice [[4](https://stackoverflow.com/questions/50086564/cython-index-should-be-typed-for-more-efficient-access)]. Therefore, I modified the class script with memoryview's tutorial [[5](http://docs.cython.org/en/latest/src/userguide/memoryviews.html)] [[6](https://docs.python.org/3/library/struct.html#format-characters)], revised the declaration of related variables and replaced some numpy function to buildin function. There is the [new version](https://github.com/ChampDBG/PlayGround/blob/master/cython/cliffwalk_v3/cy_cliffwalk_v2.pyx).

## Project Comparison - cliffwalk_v3.2
```
========== pure python   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:48<00:00,  1.62s/it]
the fastest result is 1.5564553000149317
the slowest result is 1.8868544999859296
the average result is 1.6155082133307588
========== naive cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:47<00:00,  1.57s/it]
the fastest result is 1.4714686999795958
the slowest result is 1.905901600024663
the average result is 1.5702071633325734
========== cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:26<00:00,  1.12it/s]
the fastest result is 0.8554559000185691
the slowest result is 0.9498729000333697
the average result is 0.8884332499990706
```
![](/img/20200410_Speed Comparison cliffwalk_v3.2.png)
It looks better, but still have a little distance to function version. I tried to simpify some unnecessary part in class version and there is the [version 3](https://github.com/ChampDBG/PlayGround/blob/master/cython/cliffwalk_v3/cy_cliffwalk_v3.pyx).

## Project Comparison - cliffwalk_v3.3
```
========== pure python   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:48<00:00,  1.60s/it]
the fastest result is 1.5414092999999411
the slowest result is 1.7876132999663241
the average result is 1.6017421900003683
========== naive cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:47<00:00,  1.58s/it]
the fastest result is 1.519462399999611
the slowest result is 1.8125073999981396
the average result is 1.5822270499949809
========== cython   ==========
Record 30 times of executing cliffwalk 3000 times
100%|█████████████████████████████████| 30/30 [00:24<00:00,  1.23it/s]
the fastest result is 0.7949378999765031
the slowest result is 0.8382730000303127
the average result is 0.8154112533375155
```
![](/img/20200410_Speed Comparison cliffwalk_v3.3.png)
I think there is little difference between version 2 and 3, but it finally has similar performance to function version. However, I think class version is more convenient for me to handle whole project, so I will use class version if it has similar speed as function version.

## All in all
According to this cython series, I think fine-tuned cython script can give you large bonus on heavy calculation. The fine-tuned may be
* appropriate declaration

* builtin function in cython (ex: memoryview, declare contiguous array...)

* Turn off bounds checking and warparound [[7](https://cython.readthedocs.io/en/latest/src/tutorial/numpy.html#tuning-indexing-further)] (I didn't test this part)

* find the bottleneck with ```cython -a FILENAME.pyx```

There are lots of things I haven't tried yet. For example, calling C/C++ function [[8](https://cython.readthedocs.io/en/latest/src/tutorial/external.html)][[9](https://cython.readthedocs.io/en/latest/src/tutorial/clibraries.html)], import C/C++ script [[10](https://cython.readthedocs.io/en/latest/src/userguide/external_C_code.html)] into cython or release cython from GIL [[11](https://cython.readthedocs.io/en/latest/src/userguide/parallelism.html)]. It means that, my cython script might be faster than now, but you have to balance whether it is worth it or not.

## Future work
1. GIL (Global Interpreter Lock)

2. Making my life harder with importing C/C++ into cython

3. [One day] Making life easier with Julia 

## Reference
[1] How can I use cython to speed up the numpy? [[link](https://stackoverflow.com/questions/49090448/how-can-i-use-cython-to-speed-up-the-numpy)]

[2] Cython tutorial - Extension types (aka. cdef classes) [[link](https://cython.readthedocs.io/en/latest/src/tutorial/cdef_classes.html)]

[3] Using the Cython Compiler to write fast Python code [[link](http://www.behnel.de/cython200910/talk.html)]

[4] Cython: Index should be typed for more efficient access [[link](https://stackoverflow.com/questions/50086564/cython-index-should-be-typed-for-more-efficient-access)]

[5] Cython tutorial - Typed Memoryviews [[link](http://docs.cython.org/en/latest/src/userguide/memoryviews.html)]

[6] The Python Standard Library - struct#Format Characters [[link](https://docs.python.org/3/library/struct.html#format-characters)]

[7] Cython tutorial - Tuning indexing further [[link](https://cython.readthedocs.io/en/latest/src/tutorial/numpy.html#tuning-indexing-further)]

[8] Cython tutorial - Calling C functions [[link](https://cython.readthedocs.io/en/latest/src/tutorial/external.html)]

[9] Cython tutorial - Using C libraries [[link](https://cython.readthedocs.io/en/latest/src/tutorial/clibraries.html)]

[10] Cython tutorial - Interfacing with External C Code [[link](https://cython.readthedocs.io/en/latest/src/userguide/external_C_code.html)]

[11] Cython tutorial - Using Parallelism [[link](https://cython.readthedocs.io/en/latest/src/userguide/parallelism.html)]

### Reading related to GIL
* Python - Global Interpreter Lock [[link](https://wiki.python.org/moin/GlobalInterpreterLock)]

* 深入 GIL: 如何寫出快速且 thread-safe 的 Python – Grok the GIL: How to write fast and thread-safe Python [[link](https://blog.louie.lu/2017/05/19/%E6%B7%B1%E5%85%A5-gil-%E5%A6%82%E4%BD%95%E5%AF%AB%E5%87%BA%E5%BF%AB%E9%80%9F%E4%B8%94-thread-safe-%E7%9A%84-python-grok-the-gil-how-to-write-fast-and-thread-safe-python/)]

### Extension Reading
* Function in C [[link](https://www.geeksforgeeks.org/functions-in-c/)]

* C-Function [[link](https://fresh2refresh.com/c-programming/c-function/)]

* Why Numba and Cython are not substitutes for Julia [[link](http://www.stochasticlifestyle.com/why-numba-and-cython-are-not-substitutes-for-julia/)]

* Python を高速化する Numba, Cython 等を使って Julia Micro-Benchmarks してみた [[link](https://qiita.com/yniji/items/b7acffa02f03a94882e5)]

* Python Function versus Class: what is the difference between using either methods? [[link](https://www.quora.com/Python-Function-versus-Class-what-is-the-difference-between-using-either-methods)]