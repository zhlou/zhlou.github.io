---
layout: post
title: "How to Modify Variables from a Callback Function in Python"
categories: python opencv
---

I recently came across [the trackbar demo] in the [OpenCV-Python Tutorials] as part of my quest of teaching myself computer vision. This demo shows the usage of the high level GUI features (trackbar in this case) in the OpenCV library. For the demonstration proposes, the tutorial passes a do-nothing function as the callback handler in the creation of trackbar `cv2.createTrackbar()`. The trackbar value is then retrived from a call to `cv2.getTrackbarPos()`. This makes me wonder if it is possible to give a trackbar a proper callback function so the change of position can be handled directly inside the callback. The answer turns out to be yes, but a clean implementation requires the understanding how Python's closure works.

 

[the trackbar demo]: https://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_gui/py_trackbar/py_trackbar.html#trackbar
[opencv-python tutorials]: https://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_tutorials.html
