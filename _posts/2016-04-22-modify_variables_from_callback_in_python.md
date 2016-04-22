---
layout: post
title: "How to Modify Variables from a Callback Function in Python"
categories: python opencv
date: 2016-04-22
---

I recently came across [the trackbar demo] in the [OpenCV-Python Tutorials] as part of my quest of teaching myself computer vision.
This demo shows the usage of the high level GUI features (trackbar in this case) in the OpenCV library.
For the demonstration proposes, the tutorial passes a do-nothing function as the callback handler in the creation of trackbar `cv2.createTrackbar()`.
The trackbar value is then retrieved from a call to `cv2.getTrackbarPos()`.
This makes me wonder if it is possible to give a trackbar a proper callback function so the change of position can be handled directly inside the callback.

## Using a class method

The straightforward way to enable callback function to communicate with its outside scope is to pass a class method as the callback.
Thus, the demo would look like the following without the need to call `cv2.getTrackbarPos()`.
From simplicity, I'm only using one trackbar instead of 4 in the demo.

```python
import cv2
import numpy as np

class ValWithCallback(object):
    def __init__(self, val=0):
        self.val = val
    def change(self, val):
        self.val = val

grey = ValWithCallback(0)
img = np.zeros((300, 512), np.uint8)
cv2.namedWindow('image')
cv2.createTrackbar('GREY', 'image', 0, 255, grey.change)
while (1):
    img[:] = grey.val
    cv2.imshow('image', img)
    k = cv2.waitKey(1) & 0xFF
    if k == ord('q'):
        break
cv2.destroyAllWindows()
```

However, using a dedicated class for such a simple operation seems an overkill to me.
I wonder if there are other simpler and cleaner ways to achieve the same result.
The answer turns out to be yes, but a clean implementation requires the understanding how closure in Python works, and how Python treats mutable and immutable objects differently.

## Name binding in Python

In Python, by default a function can **access** a variable outside its scope, but cannot **assign** a new value to that variable.
For **immutable** types like integer and float, this means the closure is read only.

```python
x=0
def myfunc1():
    print x
myfunc1() # 0
def myfunc2():
    x=1 # this x is now a local variable
myfunc2()
print x # global x is still 0
```
Furthermore, trying to access a global variable before a (local) assignment will raise an error

```python
x=0
def myfunc3():
    print x
    x=1
myfunc3() # UnboundLocalError
```

## Using a global variable
To change this behavior, we can declare the variable to be `global`.

```python
x=0
def myfunc4():
    global x
    x=1
myfunc4()
print x # 1
```

This gives the another solution to the callback problem.

```python
import cv2
import numpy as np
grey=0
def change(x):
    global grey
    grey = x
img = np.zeros((300, 512), np.uint8)
cv2.namedWindow('image')
cv2.createTrackbar('GREY', 'image', 0, 255, change)
while (1):
    img[:] = grey
    cv2.imshow('image', img)
    k = cv2.waitKey(1) & 0xFF
    if k == ord('q'):
        break
cv2.destroyAllWindows()
```

However, if you have nested classes or functions, using a simple `global` will be inconvenient.
In Python 3, there is `nonlocal` keyword to provide more flexible control over the variable scope.
In Python 2, we are out of luck and need to investigate other possible means to do the same.

## Using a mutable type
When a function accessing a variable of **mutable** type like list or dictionary, the behavior changes.
**Assignment** still cause the variable to bind to a local version, however modifications to the variable itself will be persistent.
Consider the following examples:

```python
x=[0]
def myfunc5():
    print x
    x[0] = 1
myfunc5() # [0]
print x # [1]
def myfunc6():
    print x
    x = [1]
myfunc6() # UnboundLoacalError
```

This behavior prompts another solution to the callback question.
To eliminate repetition, I used a more general function and then bind them use `functools.partial`.

```python
import cv2
import numpy as np
from functools import partial

def change(obj, idx, val):
    obj[idx] = val
color = [0, 0, 0]
img = np.zeros((300, 512, 3), np.uint8)
cv2.namedWindow('image')
cv2.createTrackbar('R', 'image', 0, 255, partial(change, color, 2))
cv2.createTrackbar('G', 'image', 0, 255, partial(change, color, 1))
cv2.createTrackbar('B', 'image', 0, 255, partial(change, color, 0))
while (1):
    img[:] = color
    cv2.imshow('image', img)
    k = cv2.waitKey(1) & 0xFF
    if k == ord('q'):
        break
cv2.destroyAllWindows()
```


[the trackbar demo]: https://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_gui/py_trackbar/py_trackbar.html#trackbar
[opencv-python tutorials]: https://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_tutorials.html
