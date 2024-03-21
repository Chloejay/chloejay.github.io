---
layout: post
title:  "Recap for advanced Python training from David Beazley"
date:   2024-03-21 14:00 +0100
categories: programming
image: advance_py.png
applause: true
short_description: 
--- 

<div markdown="1" id="text">
Back to 2022 August, I have attended one extensive training from Python expert David Beazley, I didn't really pick up and review what I have learnt. Now sitting in the quiet coffee shop in Helsinki, I restructure this article. This was I wrote previously, <i>I like words in a depth, which builds with curiosity, so I like use words when understanding the universe, for its the language with the creator. Maybe this is also the reason why I like programming, for it keeps me wonders the beauty of the maths, philosophy and logics. </i> I still keep the wonder of this matter.


-  Abstraction, this is the fundamental of the programming to practice you to think abstractly.
- Make use of Python built-in methods. 
    - magic method
    - not very known methods (but useful)
    - some tricks maybe
    - fp --> functools/curry/monad/lambda 
- Design layers
- Concurrency
- OOP one step more

I have divided these into 5 sub-lists, before starting write some key points. What I learnt is the experience from the teacher how he approaches given problem and what the thinking process to design the code base, and how to apply TDD in the whole development lifecycle. What one techniques I used later in the code interview is about "invariants", which means the functionality you design will never cause this happen, so you need write the code to prevent this or add logic pre check before execute the operations in the normal standards.

<h4>Built in methods</h4>
<p>
In object lifecycle, it starts with __new__() when the object is created, then for inistance apply __init__(), which is the one we use universally often. </p>
Python magic method, one of the recommended resources <span><a href="https://rszalski.github.iomagicmethods/"> guide</a></span>. 

- `__repr__`
- `__str__`
- `__new__`
- `__len__`
- `__call__`

more can refer to the guide. 

```python
class C:
    def __init__(self, name, any_val):
        self.name = name 
        self._any_val= any_val

    def __repr__(self):
        return f"C({self.name}, {self._any_val})" #good for logging/debugging

    def __call__(self, val): #let an instance to be called as func
        return self._any_val + val  

    def __len__(self):
        return len(self._any_val)

if __name__ =="__main__":
    c= C("chloe", "biker.")
    c1= C("doesn't matter", 1)
    print(c)
    print(c1(2)) #should display 3;
    print(len(c))
```

For this one `__init_subclass__`, which supported by Python 3.8+. I never saw this one before, but it looks surprising neat. 
After reading <a href= "https://peps.python.org/pep-0487/">PEP487</a> and the course, below are the example by my understanding (from the course), for `__init_subclass__`
which will initializes all subclass of give class, upon class created, the `__set_name__` hook is called on all the attributes defined in the class. 
It starts with a classmethod. 
<hr>
Use case if for adapters for registering subclass or set default attributes value for the subclass. 

```python
class Message:
    registry = { }
    def __init_subclass__(cls, *a, **kw):
        Message.registry[cls.__name__] = cls
        
    def __init__(self, sequence):
        self.sequence = sequence
        
    def __eq__(self, other):
        return type(self) == type(other) and vars(self) == vars(other)

class ChatMessage(Message):
    def __init__(self, sequence, playerid, text):
        super().__init__(sequence)
        self.playerid = playerid
        self.text = text

class PlayerUpdate(Message):
    def __init__(self, sequence, playerid, x, y):
        super().__init__(sequence)
        self.playerid = playerid
        self.x = x
        self.y = y

if __name__ == "__main__":
    sequence= 1
    m= Message(sequence)
    chat= ChatMessage(sequence, 1,2)
    print(chat.text)
    other= 2
    print(chat==2)
```

How to use arguments, by using `*, /`
The usage to use this, one reason is for arguments interference, to avoid the keywords parameters has the same name from **kwargs kind of parameters. 

```python
# passing tuple as arguments value 
def func(x, y, z):
    pass 
tuple_val= (1,2,3)
func(*tuple_val)
# or dict. 

# if arguments has * in the any pos of func. 
def func(*, x, y): 
    return x+y  

func(x=1,y=2)    

def func(x, *, y):
    return x+ y 

func(1, y=2) # a must. 

func(x, y, /, z): # works on latest version of Python.
    return x+y+z 

func(1,2,3)
func(1,2,z=3)
```

Then FP. FP is maths style to write code and no side effect, good is Python is a language FP married with OOP. Also it provides `functools` <a herf="https://docs.python.org/3/library/functools.html">package</a> which is useful to avoid "reinvent the wheel". `functools` is a useful package, you will use often when you need implement the customized decorator function.
```python
#lambda can define a function, like a function def do. 
#For function inside function will invoke earlier, 
#which will cause issue. So either define a helper function to call this function, 
#or use lambda/partial module from functools.

from functools import partial 
x,y, z= 1,2,3
def sum_it(x, y, z):
    return x+y+z 

partial_sum= partial(sum_it, 1,2)
sum_again= partial_sum(3)
lambda_sum= lambda: sum_it(x, y,z)

assert sum_it(1,2,3) == sum_again == lambda_sum()

# monad, maybe I should skip this, for I totally forget Monad concept.
```

<h4>OOP.</h4>
Object oriented programming, as a programmer we all use it daily no matter which languages you use, for most of modern languages can use OOP design structure to implement. Object is the starting point, it starts with the class, then multi-classes, to say we have base class A, subclass or childclass class B, class C, so what's the good way to implement this concept. Class is a way to define data, or define behaviors and programming interfaces, also it expressed one core tenets of OOP is the concept of "encapsulation". 

<1.> <strong>Inheritance: Define Interfaces</strong>: For the case that having multiple classes with identical functionalities, good practice is to group them into the top-level class that defines a programming interfaces; it brings a main benefit for error checking(missing operations) happens earlier. 

```python 
from abc import ABC, abstractmethod
from dataclasses import dataclass 


class A(ABC): #define an interface; 
    @abstractmethod
    def func(self):
        raise NotImplementedError()

    @abstractmethod
    def func2():
        pass 

class B(A):
    def func(self):
        print("todo implementation")

    def func2(self):
        print("todo implementation")
```

<2.>  <strong>Composition</strong>, from principle "favor composition over inheritance", for inheritance makes code tightly couple and not flexible.

```python
class E:
    pass 

class D:
    # def __init__(self, ele= None):
    #     if ele is None:
    #         ele= E
    #     self.ele= ele() 

    def __init__(self, ele:E= None):
        if ele is None:
            ele= E() # create E instance 
        assert isinstance(ele, E), "not the same type"
        self.ele= ele 
```

<3.> So composition is a good way to combine different method in a class but deign it in a flexible way, which will apply double composition

```python 
class H:
    def __init__(self, val, val2):
        self.val= val 
        self.val2= val2

class I:
    def __init__(self, x, y):
        self.x= x
        self.y= y  

class J:
    def __init__(self, h, i):
        self.h= h  # H object
        self.i= i  # I object

    def __repr__(self):
        return f"h is {self.h} and i is {self.i}"

    def func(self):
        pass 
```

<4.> How to construct a class not using __init__, but __new__() inside a class 

```python
class F:
    def __init__(self, val, val2, val3= None):
        self.val= val 
        self.val2= val2 
        if val is None:
            ...
        self.val3= val3 

    @classmethod 
    def new_F(cls, val, val2, val3): #bypass a new 
        self= cls.__new__(cls) #create a class F without calling __init__ 
        self.val= val 
        self.val2= val2 
        self.val3= val3 
        return self 


if __name__== "__main__":
    f= F(1,2,3)
    new= f.new_F(20,30,40)
    print(new.val) #display 20;
```

<5.> For testing purpose, write __repr__, I like this point very much, one benefit is easy for debug usage. 

```python 
class G:
    def __init__(self, name: str):
        self.name= name 

    def __repr__(self):
        return f'name is {self.name}'

if __name__== "__main__":
    print(G("Chloe Ji"))
```

<6.> Multiple inheritance, concept for the "Mixin" and "MRO". `class.__mro__`
<quotes>Mixin defines the code that's meant to combined with some other class via inheritance.</quotes>
(*The example is from the course.*)

```python
class StackInterface:
    pass 
class NumericStack(StackInterface):
    def __init__(self):
        self._items = []
        
    def push(self, item):
        assert isinstance(item, (int, float)), "Number is required"
        self._items.append(item)

class Stack(StackInterface):             
    def __init__(self):        
        self._items = [ ]      # _variable meant to be "private" (to the outside)

    def push(self, item):
        self._items.append(item)
    
class NumericCheck:
    def push(self, item):
        assert isinstance(item, (int, float)), "Number is required"
        super().push(item)  
        
class NumericStack(NumericCheck, Stack):
    pass

class NumericImmutableStack(NumericCheck, ImmutableStack):
    pass

class DebugPush:
    def push(self, item):
        print("PUSH:", item)
        super().push(item)

class MyStack(DebugPush, NumericCheck, Stack):
    pass
```

<7.> Using dataclass as object value. 

```python
from dataclasses import dataclass

class M:
    pass 

@dataclass
class K(M):
    val: str 
    val2: str 
    val3: int

@dataclass
class L(M):
    x: int 
    y: int 

def process_m(m: M):
    assert isinstance(m, M), "m should be identical type as M" #type checking
    if isinstance(m, K):
        # do something 
    elif isinstance(m, L):
        # do something 
... 
```

<8.> Property to use for redefine attributes access. Alternative way is using magic method

- obj.__getattribute__(name)       # Implements: obj.name
- obj.__setattr__(name, value)     # Implements: obj.name = value

```python
class C:
    def __init__(self, name, any_val):
        self.name = name 
        self._any_val= any_val

    @property 
    def _name(self):
        return self.name

    @_name.setter 
    def _name(self, new_name):
        self.name= new_name
        return self.name 

c= C("chloe", 1)
print(c._name)
c._name= "chloejay"
print(c._name)
```

<9.> Other decorator method, the most used often one are `@staticmethod`, the reason to use this is to group the same logic or similar methods in a same class, more for code organizational management.


In the end, I keep walking in this wonder space.
</div>
