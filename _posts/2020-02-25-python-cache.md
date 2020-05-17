---
layout: post
title:  "Python lru_cache"
date:   2020-02-25 08:00:00 +0800
categories: python packages
tags: python
image: cache.png
applause: true
short_description: cache lru_cache 
--- 

<div markdown="1" id="text">
I just read and inspired by this medium article <a href='https://medium.com/better-programming/every-python-programmer-should-know-lru-cache-from-the-standard-library-8e6c20c6bc49'>Every Python Programmer Should Know Lru_cache From the Standard Library.</a> From this article, it uses cache function to speed up Python code. The concept is similar to Redis case, which saves the first query result to the dictionary as key/value pairs or pre define the key value, which are stored on Redis database as a cache layer/memory, set TTL and other parameters. The purpose to do this is to save execution time when the same result needs to be called more than once, the second time call can be looked up through cache database, then to the main database, which saves time to run high volume dataset during run time, to reduce time cost and money (code cheap not code expensive). 

#### use functools.lru_cache
Lru (least recently used), one of most popular cache metrics, which keeps the most recently input pairs. (Slightly) TL;DR, you can read the article I list above from Medium, which is a very good article to introduce this cache function. 

Lru_cache <a href='https://docs.python.org/3/library/functools.html#functools.lru_cache'>doc</a> is released since Python 3.2+, which is a decorator, so you can just place it on top of the function you will call multiply times. As comparing the perf for Python running time, fibonacci function is the best candidate for its simplifity, which can be done with few lines of code. The concept can be derived that if one function needs to run multiply time, like fibonacci function, it benefits from caching because it does the same workflow multiple times through a recursion call.

Generally, lur_cache is cache return value of function based on arguments, to save time when a I/O bound function is called multiply times with the same arguments. 

To implement code here, let's first create a `decorator` function so that can avoid to write the `time log` more than once, save total execution time on log.txt. Decorator function is very interesting one, I like to use it, for it makes me feel that Python actually is creative or flexible.

```Python
from functools import lru_cache 
from time import time 
from datetime import datetime 
import logging 
logging.basicConfig(level=logging.INFO)


def logtime(if_cache_func):
    def wrapper(*args, **kwargs):
        start = time() 
        result = if_cache_func(*args, **kwargs)
        exc_time = time()- start

        with open (f'{if_cache_func.__name__}_eventLog.txt', 'a') as outf:
            outf.write(f'{datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")}\t{if_cache_func.__name__}\t{exc_time:.4f}\n')
        return result 

    return wrapper 

@logtime
def fib(number: int) -> int:
    if number == 0: 
        return 0
    elif number == 1: 
        return 1
    return fib(number-1) + fib(number-2)


#maxsize is size of cache, how many recent return value to cache
@lru_cache(maxsize = 32)
@logtime
def fib_memoization(n: int)-> int:
    if n == 0:
        return 0
    elif n == 1:
        return 1

    return fib_memoization(n-1) + fib_memoization(n-2)


def main(n: int)-> n:
    fib(n)
    fib_memoization(n)
    logging.info(fib_memoization.cache_info())


if __name__ == '__main__':
    main(20)
```
Below is the end of result of the running time, obviously lru_cache is far more efficiently.  

![fib_slow](/assets/fib_slow.png)
Function fib_slow costs 2.3s. 
![fib_memoization_result](/assets/fib_cache.png)
Function fib_memoization costs 0.002s with few lines of code. 

By using lru_cache to optimize it, the execution time decrease. On CS field, this optimization technique is called <a href='https://en.wikipedia.org/wiki/Memoization'>memoization.</a> Memorization is a specific type of caching that is used as a software optimization technique.
</div> 