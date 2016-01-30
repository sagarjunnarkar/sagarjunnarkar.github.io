---
layout:     post
title:      Git - Modifying previous commit messages
date:       2016-01-31
summary:    If you are an active git user and you want to correct mistakes in your commit messages, then this post is for you.
categories: blogs
---

If you are an active git user and you want to correct mistakes in your commit messages, then this post is for you.  

I have seen a lot of mistakes in commit messages. Sometimes they are spelling mistakes, sometimes they are incomplete sentences.  
No worries!!!. Git allows us to edit any commit message in history.

Most of the time you have to edit your recent commit message.

![Wrong commit message](/images/wrong_commit_message.png)  

Here commit message is incorrect. To resolve this we can use below command.  

    $ git commit --amend

This will open editor with last commit message. Edit your commit message and save and close.

![Corrected commit message](/images/corrected_commit_message.png)  

Notice the commit hash. It has changed.  
So If you haven't pushed your commit to remote branch then you can push it w/o any trouble. But if you have pushed the wrong commit to upstream and you try to amend it, git will reject your push.  

Git creates new commit hash whenever you create or modify an existing commit.  
Here you have to push it forcefully.  

    git push origin master -f
    
This is <span style="color:red;">Dangerous!!!  </span>  Be careful.  
Because if your teammates have already pulled your original commit, and you force pushed a new commit, they will get an error while pulling new changes.   

Another situation is when you want to edit an old commit message.  

    $ git rebase -i <hash of one commit before the wrong commit>

Here is an example.
![git log before rebase](/images/git_log_before_rebase.png)  

In above image, second commit has wrong message. So pick commit hash before wrong commit. We will pick first commit hash as our second commit message is wrong.

    $ git rebase -i 866174ca1655078b6d957b7281c2ac8b99018689

This will open up editor with all commits above selected hash as shown below.  
![rebase](/images/rebase.png)  


Replace ```pick``` to ```reword``` of the commit that you want to modify the message for, save & then close. It will again open editor with wrong commit.
Make changes and again save & close.  

![git log after rebase](/images/git_log_after_rebase.png)  

Again notice all the commit hash. Though you have changed only single commit, all the commit history above second commit has changed. As we touched parent hash i.e. second commit, all the children are going to be affected.  
To overwrite changes to your incorrect commit message, you have to push the commit forcefully.  

### <span style="color:orange;">Summary</span>  
Use ```git commit --amend``` to change recent commit message.  
Use ```git rebase -i <hash of one commit before the wrong commit>``` to change any message in the past.  
Use `amend` and `rebase` method when you haven't pushed wrong commits to upstream.  
It is not recommended to modifying past commit messages which are already on upstream and pushing it forcefully, because this is going to break your repo if other people have worked on it.  
