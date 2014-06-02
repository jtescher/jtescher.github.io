---
layout: post
title:  "Writing Command Line Applications In Ruby With Thor"
date:   1970-01-01 00:00:00
categories:
---

Writing small command line utilities and bash scripts can save you a lot of time as a developer. People often don't take 
advantage of them though because they feel intimidated and confused by the ones they use every day like `$ git status` 
or `$ rails new` and it seems like there is just too much to learn. While it's true that some utilities are pretty 
complicated, writing simple scripts is fairly painless and can help you greatly with repetitive tasks or when using an
application that doesn't need a graphical interface. 
 
In this post I'll show you how to create a simple calculator app/utility starting from in-line shell scripts in ruby and 
moving up to using the [Thor gem](https://github.com/erikhuda/thor). Even though my examples with Thor are pretty simple,
it can be used to build quite powerful and expressive projects like the [Twitter CLI](https://github.com/sferik/t) and 
the Rails CLI.

I'm going to use a simple calculator with just add and subtract functions as examples but they are just placeholders for
whatever work you want to do. I'll show how this functionality can be implemented in three versions: a simple inline 
version, a more complicated version with command line options, and finally a version that uses Thor.


Starting Small: Inline scripts
------------------------------

To get started, we want to be able to add a group of numbers and subtract a group of numbers. This is so simple that it
can be expressed as a one line Ruby script and evaluated directly. The `ruby` command lets you pass in arbitrary Ruby 
code that can be executed with the `-e` flag. The simplest version of our program then would be to add numbers
in Ruby and use `puts` to print out the results:

```bash
$ ruby -e "puts 2 + 2"                    
4

```

And we can implement our subtraction solution the same way:

```bash
$ ruby -e "puts 11 - 6"                    
5
```

That's pretty simple. If your needs can be fulfilled with that then there is no need to go on. But most requirements are
not that basic. If you want to be able to let someone who does not understand Ruby use this then you want a simpler, 
more well-defined interface for them to use.


Adding Complexity: Command Line Options
---------------------------------------

A step up from having your code evaluated in-line would be to have a defined set of functions that a user could call, 
passing the arguments in from the terminal. This would allow them to find their answers without all the knowledge of 
how the results are calculated and returned.

To do this, you can save your methods to a file (call it whatever you like, I am going with `calculator` for this 
example). Remember to make your file executable with `$ chmod +x calculator`.

In Ruby all command line options are available to scripts as `ARGV` so we can use this to allow options to be passed in. 

Below are the two methods that will take the options and perform the operations on them (the first line just says that 
this file should be interpreted as ruby code):

```ruby
#!/usr/bin/env ruby

def add(args)
  puts args.map(&:to_f).inject(:+)
end

def subtract(args)
  puts args.map(&:to_f).inject(:-)
end

send(ARGV.shift, ARGV) if ARGV.length

```

The two functions first convert the user input from strings, and then either sum with "+" or take the difference with 
"-". The last line grabs the first argument as the name of the method to use and the rest of the arguments as the inputs.

Then you can execute this application passing in the arguments with:

```bash
$ ./calculator add 1 2 3
6.0
```

```bash
$ ./calculator subtract 10 6
4.0
```

This is still a very simple example, but you can see how this technique could be used to encapsulate more complicated 
ideas into scripts with cleaner interfaces.

However, once you want to package your application for others to use or develop lots of complicated options that need 
explaining and option parsing this can get messy and repetitive. I've found that the best way to create powerful and 
well documented applications and utilities in Ruby is to use Thor.


Creating Command Line Interfaces With Thor
------------------------------------------

[Thor](https://github.com/erikhuda/thor) is built exactly for the purpose of writing command line applications like
this. Here is how it is described by the creators:

> Thor is a simple and efficient tool for building self-documenting command line utilities. It removes the pain of 
> parsing command line options, writing "USAGE:" banners, and can also be used as an alternative to the Rake build tool. 
> The syntax is Rake-like, so it should be familiar to most Rake users.

You can install Thor with `gem install thor`, and then replace your `calculator` file with:

```ruby
#!/usr/bin/env ruby

require 'thor'
 
class Calculator < Thor
  
  desc "add ...ARGS", "Calculate the sum of all numbers in ARGS"
  def add(*args)
    say args.map(&:to_f).inject(:+)
  end
  
  desc "subtract ...ARGS", "Calculate the difference of all numbers in ARGS"
  def subtract(*args)
    say args.map(&:to_f).inject(:-)
  end
  
end
 
Calculator.start(ARGV)
```

This should look very familiar by now, the difference is that you now have a class that inherits from Thor, and Thor 
will parse the options and build the output for you.

It also gives you a convenient way to list all options by passing no arguments when you execute calculator:

```bash
$ ./calculator
Commands:
  calculator add ...ARGS       # Calculate the sum of all numbers in ARGS
  calculator help [COMMAND]    # Describe available commands or one specific command
  calculator subtract ...ARGS  # Calculate the difference of all numbers in ARGS

```

And you can pass in arguments as usual:

```bash
$ ./calculator add 1 2 3
6.0
```

It is as simple as that. Thor gives you the power to create well documented and full-featured utilities simply and 
quickly. If you want to know more about Thor and all of it's fancy features like sub commands you can go on to read the 
helpful [whatisthor.com](http://whatisthor.com/) site.
