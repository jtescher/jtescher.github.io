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
personalization, recommendation and discovery of content. The two main components that drive PredictionIO are the 
**Event Server** and **Engine**. PredictionIO ingests data from an application and outputs prediction results.

![PredictionIO Overview](https://jtescher.github.io/assets/getting-started-with-predictionio/predictionio-overview.png)


Setting up the Event Server
---------------------------

First install PredictionIO. On a mac you can use homebrew `$ brew install predictionio` otherwise follow the 
instructions [here](http://docs.prediction.io/0.8.1/install/). 

Before you can generate and analyze data on the server, you must generate the database and create a user. Homebrew
downloads the following scripts for you to execute (replace the version number with your version):

```bash 
$ sh /usr/local/Cellar/predictionio/0.7.3/bin/setup-vendors.sh
$ sh /usr/local/Cellar/predictionio/0.7.3/bin/setup.sh
$ sh /usr/local/Cellar/predictionio/0.7.3/bin/users
```

Once you have set up your database and user you can start the server with:
`$ predictionio-start-all.sh`

Then you should be able to visit [localhost:3000](http://localhost:3000) and you will see:

![PredictionIO Login Screen](https://jtescher.github.io/assets/getting-started-with-predictionio/predictionio-login-screen.png)

