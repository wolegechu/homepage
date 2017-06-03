+++
# Date this page was created.
date = "2017-03-12"

# Project title.
title = "Creating Customer Segments"

# Project summary to display on homepage.
summary = "Use unsupervised learning techniques to see if any similarities exist between customers, and how to best segment customers into distinct categories."

# Optional image to display on homepage (relative to `static/img/` folder).
image_preview = ""

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ["machine-learning", "udacity"]

# Optional external URL for project (replaces project detail page).
external_link = ""

# Does the project detail page use math formatting?
math = false

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = "My caption :smile:"

+++

In this project I applied unsupervised learning techniques on product spending data collected for customers of a wholesale distributor in Lisbon, Portugal to identify customer segments hidden in the data. I first explored the data by selecting a small subset to sample and determine if any product categories highly correlate with one another. Afterwards, I preprocessed the data by scaling each product category and then identifying (and removing) unwanted outliers. With the good, clean customer spending data, I applied PCA transformations to the data and implement clustering algorithms to segment the transformed customer data. Finally, I compared the segmentation found with an additional labeling and consider ways this information could assist the wholesale distributor with future service changes.

**The main techniques used:**

- Clustering
- More Clustering
- Feature Scaling
- Feature Selection
- PCA
- Feature Transformation

You can see the code(iPython notebook) [there](https://github.com/wolegechu/Machine_Learning_Nanodegree/blob/master/3.%20Customer%20Segments/customer_segments.ipynb).
