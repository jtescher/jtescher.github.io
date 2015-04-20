---
layout: post
title:  "How to set up a Rails 4.2 app on AWS with Elastic Beanstalk and PostgreSQL"
date:   2015-04-20 09:00:00
categories: rails postgresql postgres aws elastic beanstalk
---

![AWS Elastic Beanstalk]
(https://jtescher.github.io/assets/how-to-set-up-a-rails-4-2-app-on-aws-with-elastic-beanstalk-and-postgresql/aws-logo.png)

*I previously posted instructions a year ago about [how to do this in rails 4.1]
(http://jtescher.github.io/how-to-set-up-a-rails-4-1-app-on-aws-with-elastic-beanstalk-and-postgresql/) on an old 
version of Elastic Beanstalk. This is the updated version.*

Deploying Rails apps can be tricky. For simple projects, tools like [Heroku](https://www.heroku.com/) can be perfect. 
It's great for prototyping apps and testing out different ideas without a lot of hassle. However, when your project gets more 
complicated and you want to have better control of your servers, load balancers, workers, auto-scaling conditions, etc, 
you will not have the flexibility you desire.

There are many services that can be used to get a Rails app up and running quickly while still keeping full
control of your infrastructure. One reasonable option is Amazon's
[Elastic Beanstalk](http://aws.amazon.com/elasticbeanstalk/). The service is aptly described by Amazon as follows:

> AWS Elastic Beanstalk makes it even easier for developers to quickly deploy and manage applications in the AWS cloud.
Developers simply upload their application, and Elastic Beanstalk automatically handles the deployment details of
capacity provisioning, load balancing, auto-scaling, and application health monitoring.

Now that Amazon [supports PostgreSQL via RDS](http://aws.amazon.com/rds/postgresql/) having a fully-managed
postgres-backed Rails app has never been easier!

You can find all of the code for this post at
[github.com/jtescher/example-rails-4.2-elastic-beanstalk-blog]
(https://github.com/jtescher/example-rails-4.2-elastic-beanstalk-blog)

If you get stuck or have other issues the
[documentation for Elastic Beanstalk](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html) is pretty good.

## Getting Started
Sign up for an AWS account via the instructions at the
[AWS Console](https://console.aws.amazon.com/elasticbeanstalk/) and then download the
Elastic Beanstalk Command Line Tools via Homebrew (or
[here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-getting-set-up.html#eb_cli3-install-with-pip) for PC).

```bash
  $ brew update
  $ brew install aws-elasticbeanstalk
```

## Initialize the Rails app
The most current version of rails at the time of this writing is 4.2.1 so that's what we will use now.

```bash
  $ gem install rails -v 4.2.1
  $ rails new blog
  $ cd blog
  $ git init && git add -A && git commit -m "Add rails scaffold"
```

## Scaffold a `Post` resource
We will be creating a simple example app that allows you to manipulate posts. To generate this in Rails use:

```bash
  $ rails generate scaffold post title:string body:text
  $ bundle exec rake db:migrate
  $ git add -A && git commit -am "Add post resource"
```

## Initialize the Elastic Beanstalk app
Now we can initialize a new Beanstalk app through the `eb` command.

```bash
  $ eb init
```

I would choose the following settings, but for a description of each option see the AWS example
[here](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Ruby_rails.html#create_deploy_Ruby_eb_init-rds).

```
  Select a default region
  3) us-west-2 : US West (Oregon)

  Select an application to use
  [ Create new Application ]

  Enter Application Name
  blog
  Application blog has been created.

  It appears you are using Ruby. Is this correct?
  (y/n): y

  Select a platform version.
  1) Ruby 2.2 (Puma)

  Do you want to set up SSH for your instances?
  (y/n): n
```

This will set up a `.elasticbeanstalk` directory in the root of your project and add it to your .gitignore file. You
do not want your configuration stored in git because there could be private information in there. Let's
commit those changes now:

```bash
  $ git commit -am "Ignore elasticbeanstalk settings"
```

## Creating the Elastic Beanstalk environment

You can have many environments per Elastic Beanstalk application. This can be useful for having both dev and production 
environments for the same app.

To create a new environment, run the following:

```bash
  $ eb create blog-env:

  Creating application version archive "b303".
  Uploading blog/b303.zip to S3. This may take a while.
  Upload Complete.
  Environment details for: blog-env
    Application name: blog
    Region: us-west-2
    Deployed Version: b303
    Environment ID: e-g5mkeawrnz
    Platform: 64bit Amazon Linux 2015.03 v1.3.0 running Ruby 2.2 (Puma)
    Tier: WebServer-Standard
    CNAME: UNKNOWN
    Updated: 2015-04-19 23:38:50.955000+00:00
  Printing Status:
  INFO: createEnvironment is starting.
  INFO: Using elasticbeanstalk-us-west-2-83376862866 as Amazon S3 storage bucket for environment data.
  INFO: Created load balancer named: awseb-e-g-AWSEBLoa-7R0CSEMQ6W2M
  INFO: Created security group named: awseb-e-g5mkeawrnz-stack-AWSEBSecurityGroup-56IUD2ZYQ5FR
  INFO: Created Auto Scaling launch configuration named: awseb-e-g5mkeawrnz-stack-AWSEBAutoScalingLaunchConfigurat...
  INFO: Created Auto Scaling group named: awseb-e-g5mkeawrnz-stack-AWSEBAutoScalingGroup-2URXDKL0NCIJ
  INFO: Waiting for EC2 instances to launch. This may take a few minutes.
  INFO: Created Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:833768628226:scalingPolicy:02920f8b...
  INFO: Created Auto Scaling group policy named: arn:aws:autoscaling:us-west-2:833768628666:scalingPolicy:b143cea1...
  INFO: Created CloudWatch alarm named: awseb-e-g5mkeawrnz-stack-AWSEBCloudwatchAlarmHigh-APCUnlMHNIS1
  INFO: Created CloudWatch alarm named: awseb-e-g5mkeawrnz-stack-AWSEBCloudwatchAlarmLow-1UL48B2CC2OM8
  INFO: Added EC2 instance 'i-7f4b6eb7' to Auto Scaling Group 'awseb-e-g7mkeawrnz-stack-AWSEBAutoScalingGroup-2URX...
  INFO: Application available at blog-env-zckzptpdgy.elasticbeanstalk.com.
  INFO: Successfully launched environment: blog-env
```

The environment should now be running. To see the status and URL:

```bash
  $ eb status
  Environment details for: blog-env
    Application name: blog
    Region: us-west-2
    Deployed Version: b303
    Environment ID: e-g5mkeawrn
    Platform: 64bit Amazon Linux 2015.03 v1.3.0 running Ruby 2.2 (Puma)
    Tier: WebServer-Standard
    CNAME: blog-env-zckzptpdg2.elasticbeanstalk.com
    Updated: 2015-04-19 23:51:59.259000+00:00
    Status: Ready
    Health: Green
```

The last thing that we have to do to get Rails set up is to add a `SECRET_KEY_BASE` environment variable.

To generate a new secret key use:

```bash
  $ rake secret
  f655b5cfeb452e49d9182c6b5e6856704e6e1674082fa1e5f1a330782bad1833ba4cc30951e094f9250c87573dc0bbd3d46d37c5d79ff57...
```

Then in order to add the secret to your elastic beanstalk environment, use:

```bash
  $ eb setenv SECRET_KEY_BASE=f655b5cfeb452e49d9182c6b5e6856704e6e1674082fa1e5f1a330782bad1833ba4cc30951e094f9250...
```

Now if you navigate to [YOUR-ENV].elasticbeanstalk.com/posts you should see your posts index view:

![Posts Index]
(https://jtescher.github.io/assets/how-to-set-up-a-rails-4-2-app-on-aws-with-elastic-beanstalk-and-postgresql/posts-index.png)

## Using PostgreSQL with Rails

Right now our app is just using SQLite, which is not made for production use and cannot be shared across instances.
We can solve this by adding PostgreSQL to your app and to your Elastic Beanstalk environment.

### Adding the pg gem

Open your `Gemfile`. Move the `sqlite3` gem into your `development` and `test` block and add a `production` group with
the `pg` gem in it. Afterward it should look something like this:

```ruby
  source 'https://rubygems.org'


  # Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
  gem 'rails', '4.2.1'
  # Use SCSS for stylesheets
  gem 'sass-rails', '~> 5.0'
  # Use Uglifier as compressor for JavaScript assets
  gem 'uglifier', '>= 1.3.0'
  # Use CoffeeScript for .coffee assets and views
  gem 'coffee-rails', '~> 4.1.0'
  # See https://github.com/rails/execjs#readme for more supported runtimes
  # gem 'therubyracer', platforms: :ruby

  # Use jquery as the JavaScript library
  gem 'jquery-rails'
  # Turbolinks makes following links in your web application faster. Read more: https://github.com/rails/turbolinks
  gem 'turbolinks'
  # Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
  gem 'jbuilder', '~> 2.0'
  # bundle exec rake doc:rails generates the API under doc/api.
  gem 'sdoc', '~> 0.4.0', group: :doc

  # Use ActiveModel has_secure_password
  # gem 'bcrypt', '~> 3.1.7'

  # Use Unicorn as the app server
  # gem 'unicorn'

  # Use Capistrano for deployment
  # gem 'capistrano-rails', group: :development

  group :development, :test do
    # Use sqlite3 as the database for Active Record
    gem 'sqlite3', '~> 1.3.10'

    # Call 'byebug' anywhere in the code to stop execution and get a debugger console
    gem 'byebug'

    # Access an IRB console on exception pages or by using <%= console %> in views
    gem 'web-console', '~> 2.0'

    # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
    gem 'spring'
  end

  group :production do
    # Use PostgreSQL as the database for Active Record
    gem 'pg', '~> 0.18.1'
  end
```

Now run `$ bundle install` to install the gem.

### Configuring database.yml to work with RDS postgres

Database credentials should never be hard coded and Elastic Beanstalk makes setting them as environment variables quite simple.
RDS variables are configured and updated automatically so your production section of `config/database.yml` can be
updated to the following:

```yaml
  production:
    <<: *default
    adapter: postgresql
    encoding: unicode
    database: <%= ENV['RDS_DB_NAME'] %>
    username: <%= ENV['RDS_USERNAME'] %>
    password: <%= ENV['RDS_PASSWORD'] %>
    host: <%= ENV['RDS_HOSTNAME'] %>
    port: <%= ENV['RDS_PORT'] %>
```

## Getting the pg gem to work on Elastic Beanstalk

Now let's add the database to our environment. This takes a few steps but I'll walk you through it. First go to the
Elastic Beanstalk section in the AWS console: [console.aws.amazon.com/elasticbeanstalk/?region=us-west-2]
(https://console.aws.amazon.com/elasticbeanstalk/?region=us-west-2) (Note the region is us-west-2, if you deployed to
a different region, check there.)

![Blog App Environment]
(https://jtescher.github.io/assets/how-to-set-up-a-rails-4-2-app-on-aws-with-elastic-beanstalk-and-postgresql/blog-env.png)

Now click on `blog-env` and go to `configuration` on the left nav. At the bottom you should see:

![Data Tier]
(https://jtescher.github.io/assets/how-to-set-up-a-rails-4-2-app-on-aws-with-elastic-beanstalk-and-postgresql/data-tier.png)

Now click "create a new RDS database", set the DB Engine to "postgres" and create a Master Username and Master Password.

![Database Config]
(https://jtescher.github.io/assets/how-to-set-up-a-rails-4-2-app-on-aws-with-elastic-beanstalk-and-postgresql/db-config.png)

Click "Save" and you will have a fully functioning PostgreSQL instance and the environment variables will have been added to
your Beanstalk environment automatically.

Now to install the `pg` gem on your server the `postgresql-devel` yum package is required. Configuring packages on
Elastic Beanstalk instances is as simple as dropping a YAML formatted .config file in a top level .ebextensions folder.

```yaml
  # .ebextensions/packages.config
  packages:
    yum:
      postgresql-devel: []
```

Now commit this change and redeploy the app.

```bash
  $ git add -A && git commit -am "Add PostgreSQL as production database"
  $ eb deploy
```

Once this is done you can reload the `/posts` page and see your fully-functional postgres-backed Rails app! Hooray!

![Posts Index]
(https://jtescher.github.io/assets/how-to-set-up-a-rails-4-2-app-on-aws-with-elastic-beanstalk-and-postgresql/posts-index.png)
