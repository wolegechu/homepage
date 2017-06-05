+++
# Date this page was created.
date = "2017-03-14"

# Project title.
title = "Train a Smartcab How to Drive"

# Project summary to display on homepage.
summary = "Use reinforcement learning to aid a self-driving agent effectively reaching its destinations in the allotted time."

# Optional image to display on homepage (relative to `static/img/` folder).
image_preview = ""

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ["reinforcement-learning", "udacity"]

# Optional external URL for project (replaces project detail page).
external_link = ""

# Does the project detail page use math formatting?
math = false

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = "My caption :smile:"

+++

## Overview
In this project I applied reinforcement learning techniques for a self-driving agent in a simplified world to aid it in effectively reaching its destinations in the allotted time. I first investigated the environment the agent operates in by constructing a very basic driving implementation. Once the agent is successful at operating within the environment, I then identified each possible state the agent can be in when considering such things as traffic lights and oncoming traffic at each intersection. With states identified, I then implemented a Q-Learning algorithm for the self-driving agent to guide the agent towards its destination within the allotted time. Finally, I improved upon the Q-Learning algorithm to find the best configuration of learning and exploration factors to ensure the self-driving agent is reaching its destinations with consistently positive results.

## Description
In the not-so-distant future, taxicab companies across the United States no longer employ human drivers to operate their fleet of vehicles. Instead, the taxicabs are operated by self-driving agents, known as *smartcabs*, to transport people from one location to another within the cities those companies operate. In major metropolitan areas, such as Chicago, New York City, and San Francisco, an increasing number of people have come to depend on *smartcabs* to get to where they need to go as safely and reliably as possible. Although *smartcabs* have become the transport of choice, concerns have arose that a self-driving agent might not be as safe or reliable as human drivers, particularly when considering city traffic lights and other vehicles. To alleviate these concerns, my task is to use reinforcement learning techniques to construct a demonstration of a *smartcab* operating in real-time to prove that both safety and reliability can be achieved.

### Below is the final score:
![](/img/posters/smartcab.png)

### The main techniques used:

- Markov Decision Processes
- Reinforcement Learning
- Game Theory
- More Game Theory beginning at Stochastic games and Multi-agent RL

You can see the code(iPython notebook) [there](https://github.com/wolegechu/Machine_Learning_Nanodegree/blob/master/4.%20Smartcab/smartcab.ipynb).
