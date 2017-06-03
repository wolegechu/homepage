+++
# Date this page was created.
date = "2017-03-16"

# Project title.
title = "A Digit Recognition Program"

# Project summary to display on homepage.
summary = "Train a DNN model that can decode sequences of digits from natural images by using the [SVHN dataset](http://ufldl.stanford.edu/housenumbers/)."

# Optional image to display on homepage (relative to `static/img/` folder).
image_preview = ""

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ["deep-learning", "udacity"]

# Optional external URL for project (replaces project detail page).
external_link = ""

# Does the project detail page use math formatting?
math = false

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = "My caption :smile:"

+++

In this project, I used deep neural networks and convolutional neural networks to create a program that prints numbers it observes in real time from images it is given. First, I designed and tested a model architecture that can identify sequences of digits in an image. Next, I trained that model so it could decode sequences of digits from natural images by using the Street View House Numbers (SVHN) dataset. After the model was properly trained, I then tested my model using a program on newly-captured images. Finally, I refined your implementation to also localize where numbers are on the image, and test this localization on newly-captured images.

**The main techniques used:**

- Deep Neural Networks
- Convolutional Neural Networks
- Adadelta
- Keras based on TensorFlow


You can see the code(iPython notebook) [there](https://github.com/wolegechu/Machine_Learning_Nanodegree/blob/master/5.%20Digit%20Recognition/digit_recognition.ipynb).
