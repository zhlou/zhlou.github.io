---
layout: post
title: "How to Modify Variables from a Callback Function in Python"
categories: python opencv
---

I recently came across [the trackbar demo] in the [OpenCV-Python Tutorials] as part of my quest of teaching myself computer vision.
This demo shows the usage of the high level GUI features (trackbar in this case) in the OpenCV library.
For the demonstration proposes, the tutorial passes a do-nothing function as the callback handler in the creation of trackbar `cv2.createTrackbar()`.
The trackbar value is then retrived from a call to `cv2.getTrackbarPos()`.
This makes me wonder if it is possible to give a trackbar a proper callback function so the change of position can be handled directly inside the callback.
The answer turns out to be yes, but a clean implementation requires the understanding how closure in Python works, and how Python treats mutable and immutable objects differently.

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

## Use global variable
To change this behavior, we can declare the variable to be `global`.

```python
x=0
def myfunc4():
    global x
    x=1
myfunc4()
print x # 1
```

This gives the first solution to the callback problem.

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

[the trackbar demo]: https://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_gui/py_trackbar/py_trackbar.html#trackbar
[opencv-python tutorials]: https://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_tutorials.html
