---
layout:     post
title:      How to accept user input in rake task
date:       2015-05-01
summary:    Recently I wrote a rake task in my rails project where I have to accept user inputs and do some stuff on that. My rake task looks like below.
categories: blogs
---

Recently I wrote a rake task in my rails project where I have to accept user inputs and do some stuff on that. My rake task looks like below.

    namespace :enter do
      task :name do
        p 'Enter your name'
        name = gets.chomp
      end
    end

Then run rake task

    $ rake enter:name

And I astonished! I got an error.

    No such file or directory - setup:name

What's wrong here?

The gets is method of Kernel module. When we pass any argument to rake, 'gets' assumes that there is file named as 'enter:name' (i.e. our rake task) and tries to return content of the file, which is undoubtedly does not exist.

Try ```STDIN.gets.chomp```

```Kernel.gets``` first tries to read the contents of files passed in through ARGV. If there are no files in ARGV, it will use standard input at which point it's the same as ```STDIN.gets```
