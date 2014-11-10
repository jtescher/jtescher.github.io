---
layout: post
title:  "Getting Started With PredictionIO"
date:   2014-11-10 09:00:00
categories: predictionio machine-learning
---

![PredictionIO Logo](https://jtescher.github.io/assets/getting-started-with-predictionio/predictionio.jpg)

The concept of machine learning can seem scary, like something you need a PhD and a neckbeard to understand. In this 
post I will show you how the nice gents at PredictionIO have made the whole process approachable to beginners and easy
to set up.


What is PredictionIO
--------------------

PredictionIO is an open source machine learning server that allows developers to create predictive features such as
content personalization, recommendation and discovery. The two main components that drive PredictionIO are the 
**Event Server** and **Engine**. It ingests data from an application and outputs prediction results.

![PredictionIO Overview](https://jtescher.github.io/assets/getting-started-with-predictionio/predictionio-overview.png)


Creating Your First Engine
--------------------------

First install PredictionIO, on a mac you can use homebrew `$ brew install predictionio` otherwise follow the 
instructions [here](http://docs.prediction.io/0.8.1/install/).  
