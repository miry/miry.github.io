---
url: https://medium.com/notes-and-tips-in-full-stack-development/run-tests-against-multiple-ruby-versions-on-circleci-60e1e523e245
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/run-tests-against-multiple-ruby-versions-on-circleci-60e1e523e245
title: Run Tests Against Multiple Ruby Versions On CircleCI
subtitle: Using multiple MRI ruby versions is not very hard. There are some ruby versions
  that have already been installed, and if you would like to…
slug: run-tests-against-multiple-ruby-versions-on-circleci
description: ""
tags:
- ruby
- docker
- circleci
- rails
- ruby-on-rails
author: Michael Nikitochkin
username: miry
---

![Unsplash Photo: Daniel Alvarez Sanchez Diaz](/assets/2017-06-13-run-tests-against-multiple-ruby-versions-on-circleci-1_3pdZBNSUulxomF-fHSsTcQ.jpeg)

# Run Tests Against Multiple Ruby Versions On CircleCI

Using multiple MRI ruby versions is not very hard. There are some ruby versions that have already been installed, and if you would like to add, for example, `2.2.0-dev` you need to have a closer look. [CircleCI](https://circleci.com) adds a lot of abilities to run different commands during all processes.

# 1. Run the same tests against multiple ruby versions for the same platform

```
dependencies:
  override:
    - 'rvm-exec 2.1.0 bundle install'
    - 'rvm-exec 2.2.0-preview1 bundle install'

test:
  override:
    - 'rvm-exec 2.1.0 bundle exec rake'
    - 'rvm-exec 2.2.0-preview1 bundle exec rake'

```
> *[circleci-multiple-platform-tests.yaml view raw](https://gist.githubusercontent.com/marchi-martius/3bf1a9e3ed36ea7aa0627098354c3dff/raw/16ef6d7788d89309afc9f10940b79c2cdd273e19/circleci-multiple-platform-tests.yaml)*

It was easy. The main feature that CircleCI uses is [rvm] and it helps us to choose the correct ruby version. There is a list of preinstalled ruby [versions](https://circleci.com/docs/environment#ruby). In the first section we install gems for each ruby and in the second we run a test.

# 2. Install custom ruby version

Before runnung tests against a new version of ruby, we should install it. [rvm] helps us to do that.

```
dependencies:
  pre:
    - 'rvm install ruby-head'
  
  override:
    - 'rvm-exec ruby-head bundle install'
    #....

test:
  override:
    - 'rvm-exec ruby-head bundle exec rake'
    #....
```
> *[circleci-multiple-ruby-install.yaml view raw](https://gist.githubusercontent.com/marchi-martius/352e94fea2cd90cf70fa0a61c472a82d/raw/c739c464cc8826663bdb52fac111c0db304f3a41/circleci-multiple-ruby-install.yaml)*

Now you should see the logs with ruby installation.

# 3. Jruby + MRI

We already know how to run test for multiple ruby versions. Now let’s add supporting of [JRuby]. [Latest version](https://circleci.com/docs/environment#ruby) that CircleCI currently supports is `jruby-1.7.13`. Let’s update our config to use the latest available by rvm.

```
dependencies:
  pre:
    - 'rvm install jruby'
    #....
  
  override:
    - 'rvm-exec jruby bundle install'
    #....

test:
  override:
    - 'rvm-exec jruby bundle exec rake'
    #....

```
> *[circleci-multiple-jruby.yaml view raw](https://gist.githubusercontent.com/marchi-martius/dab18a48ffe8b591e860c5c83c13d452/raw/5b7dd5f36d9b3906c4712fb20c0837d18d702cea/circleci-multiple-jruby.yaml)*

Let’s add other options to specify what JVM should use and Jruby options:

```
machine:
  java:
    version: 'openjdk7'
  environment:
    RAILS_ENV: 'test'
    JRUBY_OPTS: '-J-XX:+TieredCompilation -J-XX:TieredStopAtLevel=1 -J-noverify -X-C -Xcompile.invokedynamic=false --1.9 -J-Xmx1g'

```
> *[circleci-multiple-jruby_env.yaml view raw](https://gist.githubusercontent.com/marchi-martius/aaeae63a3670f29cc9d32d2fd41462f1/raw/1a7f2a6a3f4fb96a906a8f423bb8c44191b5a668/circleci-multiple-jruby_env.yaml)*

# 4. What if Java gems are different from MRI

As you know, the real world projects are cruel, and mostly you have different versions of gems or even different gems to support Jruby and MRI. A simple solution is just to add a separate `Gemfile` for Java. And to update our config file accordingly:

```
dependencies:
  override:
    - 'rvm-exec jruby bundle install --gemfile Gemfile.java'
    #....

test:
  override:
    - 'BUNDLE_GEMFILE=Gemfile.java rvm-exec jruby bundle exec rake'
    #....
```
> *[circleci-multiple-bundle.yaml view raw](https://gist.githubusercontent.com/marchi-martius/e2639072f8367df8b39b4d9730ff69ee/raw/9437d636a042e3d237236f7306be06995c10590c/circleci-multiple-bundle.yaml)*

# 5. Parallelism

I was tired of waiting for all tests to be finished. And decided to split all processes. I’ve found a good [manual](https://circleci.com/docs/parallel-manual-setup) and what we have:

```
machine:
  java:
    version: openjdk7
  environment:
    RAILS_ENV: test
    JRUBY_OPTS: -J-XX:+TieredCompilation -J-XX:TieredStopAtLevel=1 -J-noverify -X-C -Xcompile.invokedynamic=false --1.9 -J-Xmx2g

dependencies:
  cache_directories:
    - '~/.rvm/rubies'
    - 'vendor'

  override:
    - >
      case $CIRCLE_NODE_INDEX in
       0)
         rvm-exec 2.1.0 bash -c "bundle check --path=vendor/bundle_2.1 || bundle install --path=vendor/bundle_2.1"
         ;;
       1)
         rvm-exec 2.2.0-preview1 bash -c "bundle check --path=vendor/bundle_2.2 || bundle install --path=vendor/bundle_2.2"
         ;;
       2)
         rvm-exec ruby-head gem install bundler
         rvm-exec ruby-head bash -c "bundle check --path=vendor/bundle_head || bundle install --path=vendor/bundle_head"
         ;;
       3)
         rvm-exec jruby gem install bundler
         rvm-exec jruby bash -c "bundle check --path=vendor/bundle_jruby --gemfile Gemfile.java || bundle install --path=vendor/bundle_jruby --gemfile Gemfile.java"
         ;;
      esac

test:
  override:
    - case $CIRCLE_NODE_INDEX in 0) rvm-exec 2.1.0 bundle exec rake ;; 1) rvm-exec 2.2.0-preview1 bundle exec rake ;; 2) rvm-exec ruby-head bundle exec rake ;; 3) BUNDLE_GEMFILE=Gemfile.java rvm-exec jruby bundle exec rake ;; esac:
        parallel: true

```
> *[circleci-multiple-parallelism.yaml view raw](https://gist.githubusercontent.com/marchi-martius/025bdb14c65853e435bbee3843604d61/raw/f1b29f9f54185bd82dc30cdf233b71157a3305b0/circleci-multiple-parallelism.yaml)*

You can move these scripts to files.

You can find the final version of `circle.yml` with some tricks [here](https://github.com/miry/multiple_ruby_for_circleci/blob/master/circle.yml).

![That’s all Folks](/assets/2017-06-13-run-tests-against-multiple-ruby-versions-on-circleci-1_iifqfnqorqkZVMpCyq1BjA.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


