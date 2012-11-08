---
layout: post
title: "Use Capistrano"
date: 2012-11-08 15:18
comments: true
categories: 
published: false
---

## Goals
### Assumptions

When get called it loads `deploy.rb` in `config` directory.
It searches up the file tree to find the `Capfile`.

## Building Blocks
### Variables
Variables are used to:
1. config the capistrano
2. reduce the duplicates strings

- `set` is used to to create and assign a variable or a proc
- cap variable has global scope
- variable could be retrieved through `fetch` or string 
- `fetch` or string interpolation is used to retrieve the variable
- `exist?` is used to check existense of variable
- use `unset` to destroy a variable
- Cap implements an `_cset` to undestructive assignment.
- use `set` with a block to use lazy-assignment

    set :username, 'Capistrano Wizard'
    set(:user) do
       Capistrano::CLI.ui.ask "Give me a ssh user: "
    end

### Tasks
Task is the foundation of capistrano. 
- Task could be invoked with `cap task_name`.
- Task associates a block of commands which will be run for roles .
- A `desc` could be defined as the description of this tasks.
- Task has a name which could be used to invoke the tasks elsewhere.
- Task with the same name as exist task will overwrite it.

    desc "Search Remote Application Server Libraries"
    task :search_libs, :roles => :app do
      run "ls -x1 /usr/lib | grep -i xml"
    end
    
### Namespace
Namespace is used to group related tasks. 
- It is called with a block of tasks. 
- Tasks with namespace could be invoked with `cap namespace:task_name`.
- The default and omitted namespace is `top`.

    namespace :backup do

      task :default do
        web
        db
      end

      task :web, :roles => :web do
        puts "Backing Up Web Server"
      end

      task :db, :roles => :db do
        puts "Backing Up DB Server"
      end

    end

### Chains
- Task could invoke other tasks by refer its name.
- Callback is used to chain task or group of tasks together in sequence.

    namespace :deploy do
      task :default do
        update
        restart           # <= v2.5.5
      end
    end

    after("deploy:symlink", "notifier:email_the_boss")

### Transactions
Transaction is used to rollback a failed task.
- Use `transaction` to run group of tasks. They will run in a transactin.
- The rollback task is defined by `on_rollback` with in a task.

    task :symlink do
      on_rollback do
        run <<-EOC 
          rm #{current_path};
          ln -s #{previous_release} #{current_path}
        EOC
      end
      run "rm #{current_path}; ln -s #{release_path} #{current_path}"
    end

### Execution
Use `cap` from the command line to execute the capistrano tasks you defined.
- tasks in the arguments are invoked in order.

## Main Configuration
Check out [Configuration Variable][]

### Server Management
`role` is used to associate a server with a particular role.

- The server string could contain `user@` or `:port`, this has the highest priority.
- Additional attributes could be specified as a Hash. It is used by tasks to filter server to run commands on.
- When pass a block, the evaluation will be delayed to the first time a command run.

`server` is used to assign multiple roles to one host.

## Main Action API
`run` is mostly used to execute arbitrary shell commands on all matched servers.

    You could embed `#{sudo}` to run command as sudoer.

`capture` is used to return the **stdout** to the command as a string.

`stream` is used to stream the stdout locally.

`download`, `upload` and `put` to transfer files between remotes and local machine.

Use `$CAPISTRANO:HOST$` variable to distinguish different servers.

## References
[capistrano handbook][]

[capistrano handbook](https://github.com/leehambley/capistrano-handbook/blob/master/index.markdown)
[Configuration Variable](https://github.com/capistrano/capistrano/wiki/2.x-Significant-Configuration-Variables)
[DSL Configuration](https://github.com/capistrano/capistrano/wiki/2.x-DSL-Documentation-Configuration-Module)
[DSL Action](https://github.com/capistrano/capistrano/wiki/2.x-DSL-Documentation-Action-Module)

## Customized Tasks
All installation is management by `apt-get`.

`check:revision`: compare the HEAD in the local and remove repository, make sure the remote code base is up-to-date.

`dev_lib:install`: install C libraries on which application depends.

`nginx:install`: install the lastest release of nginx.

`nginx:setup`: generate nginx conf for the application using unicorn as web server.

`nginx:start/stop/restart/reload`: commands mapped to `sudo service nginx start/stop/restart/reload`.

`nodejs:install`: install node and npm from offcial recommended source.

`rbenv:install`: install rbenv, ruby, patches and libraries which ruby depends on.

`pg:install`: install postgresql.

`pg:create_db`: create database from prompt.

`pg:setup`: generate the database.yml and copied to the shared path.

`pg:symlink`: generate symlink in the current path for the database.yml file.
