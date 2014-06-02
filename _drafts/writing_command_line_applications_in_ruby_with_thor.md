---
layout: post
title:  "Writing Command Line Applications In Ruby With Thor"
date:   1970-01-01 00:00:00
categories:
---

Command line applications can be as simple as one-off scripts or more complicated than git.
 
In this post I'll show you how to create simple calculator app starting from in-line shell scripts in ruby and moving up 
to using the [thor gem](https://github.com/erikhuda/thor).


Starting Small: Inline scripts
------------------------------

For the most basic requirements, ruby code can be passed and evaluated directly in the terminal using the `-e` flag:

```bash
$ ruby -e "puts 2 + 2"                    
4

```


Adding Complexity: Command Line Options
---------------------------------------

When your application moves beyond what a one line script can accomplish, you can save your methods to a `calculator`
file and pass your arguments in as command line options. Remember to make your file executable with 
`$ chmod +x calculator`

In ruby all command line options are available to scripts as `ARGV`. 

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

And you can execute this application passing in the arguments with:

```bash
$ ./calculator add 1 2 3
6.0
```

```bash
$ ./calculator subtract 10 6
4.0
```

This is clearly an overly simplistic example, but you can see how this could be used to encapsulate tasks into 
relatively clean scripts.

However, once you want to package your application for others to use, or develop lots of complicated options that need 
explaining and option parsing this can get complicated. The best way to create powerful and documented applications is
to use Thor.


Creating Powerful Applications With Thor
----------------------------------------

[Thor](https://github.com/erikhuda/thor) is built exactly for the purpose of writing command line applications like
this. Here is how it is described by the creators:

> Thor is a simple and efficient tool for building self-documenting command line utilities. It removes the pain of 
> parsing command line options, writing "USAGE:" banners, and can also be used as an alternative to the Rake build tool. 
> The syntax is Rake-like, so it should be familiar to most Rake users.

You can install thor with `gem install thor`, and then replace your `calculator` file with

```ruby
#!/usr/bin/env ruby

require 'thor'
 
class Calculator < Thor
  
  desc "add ...ARGS", "Add all numbers in ARGS"
  def add(*args)
    say args.map(&:to_f).inject(:+)
  end
  
  desc "subtract ...ARGS", "Subtract all numbers in ARGS"
  def subtract(*args)
    say args.map(&:to_f).inject(:-)
  end
  
end
 
Calculator.start(ARGV)
```

Thor will build the output for you, so to list options, you can pass no arguments:

```bash
$ ./calculator
Commands:
  calculator add ...ARGS       # Add all numbers in ARGS
  calculator help [COMMAND]    # Describe available commands or one specific command
  calculator subtract ...ARGS  # Subtract all numbers in ARGS

```

And you can pass in arguments as usual:

```bash
$ ./calculator add 1 2 3
6.0
```

It is as simple as that. Thor gives you the power to create well documented and full-featured utilities simply and 
quickly.
