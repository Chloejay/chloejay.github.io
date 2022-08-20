---
layout: post
title:  "The thoughts after programming course - Part 2"
date:   2022-08-13 16:00 +0800
categories: programming
image: 
applause: true
short_description: 
--- 

<div markdown="1" id="text">
<title>Pure thinking is fun.</title>
Reading is the fun thing which leads me to the space and time that I can talk to the wise minds. This morning I started reading the 3rd chapter of "Software mistakes and tradeoffs", the author illustrates using Java to express error handling strategy, so on the process of reading, I plan to continue writing the 2nd part of the programming course I took last week as a closure. 

For varied angles can go for the closure, I think I will focus on below two points, 
- Handle the error/exception 
- Concurrency: asyncio and threading. 

For the software is like the science, which can't to be proved how "correct" it is, but if can't be proved is false, then I know its logically correct.
But if code can't be tested, then its either too complicated or the logic is wrong is another case.

<h6>Writing assumption and test</h6>
Besides writing unit test, I gained some suggestions that how to test logic during the development, 
1. Make things observable(`__repr__` methods)
2. Make it configurable. make it easy to create any state. It's useful for very specific tests, such as checking the edge case. 
3. Defense programming. Use of asserts/invariants. (Invariants== Never supposed to happen.)
4. Use mocks/fake objects. Also useful in testing. Could be used to enforce further defensive coding. 

<h6>Concurrency</h6>
Asynchrony programming is means when compiler execute the code, it doesn't need to wait for the current task to complete until execute other tasks, which will make execution efficiently. For asyncio case, which is based on IO bound and it backbone is using generator and `sleep()`. For threading, the key idea that a thread runs a function independently, and concurrently, with other code that happens to be running. Why use this, to make code runs faster in run time, common case is to be used to call API data, like data ingestion. 

Below use `asyncio` module, 
```python
import asyncio 

async def main():
    task1= asyncio.create_task(get_name("chloe"))
    task2= asyncio.create_task(func("any val")) 
    await asyncio.sleep(2)
    print("finishded")

async def get_name(name: str):
    print(name)
    return name 

async def func(text):
    print(text)
    await asyncio.sleep(1) #waiting for some operation; 

asyncio.run(main(), debug= True)
```
</div>