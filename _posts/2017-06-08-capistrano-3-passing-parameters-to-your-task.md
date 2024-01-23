---
url: https://medium.com/notes-and-tips-in-full-stack-development/capistrano-3-passing-parameters-to-your-task-e22cc9f659c3
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/capistrano-3-passing-parameters-to-your-task-e22cc9f659c3
title: 'Capistrano 3: Passing Parameters to Your Task'
subtitle: The new way of passing parameters in Capistrano v3 is to use the same solution
  as Rake (in some sort Capistrano 3 is totally based on…
slug: capistrano-3-passing-parameters-to-your-task
description: ""
tags:
- ruby-on-rails
- capistrano
- testing
- deployment
- ruby
author: Michael Nikitochkin
username: miry
---

![Unsplash Photo: Miguel Ibáñez](/assets/2017-06-08-capistrano-3-passing-parameters-to-your-task-1_7n_mlixEJwpezggQn1PrYQ.jpeg)

# Capistrano 3: Passing Parameters to Your Task

The new way of passing parameters in Capistrano v3 is to use the same solution as Rake (in some sort Capistrano 3 is totally based on Rake).

A little example, let us create a task to run any specific rake task with options:

```
namespace :task do
  desc 'Execute the specific rake task'
  task :invoke, :command do |task, args|
    on roles(:app) do
      execute :rake, args[:command]
    end
  end
end
```

and now we can run `rake db:migrate` on remote hosts:

```
$ cap staging "task:invoke[db:migrate]"
INFO [397d776e] Running rake db:migrate on 8.8.8.8
DEBUG [397d776e] Command: ( RAILS_ENV=staging rake db:migrate )
...
```

I used the quotes, because for `zsh` the brackets are used for some shell features.

Some more information with good examples of passing parameters to rake task can be found [here](http://viget.com/extend/protip-passing-parameters-to-your-rake-tasks).

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


