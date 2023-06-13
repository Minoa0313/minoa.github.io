---
layout: post
title:  "解决了git push失败的问题"
date:   2023-06-13
category: Git
---
# 出现的问题
使用TortoiseGit可以Pull，但是无法Push

错误如下（隐私起见，模糊了部分信息）
```
git.exe push --progress "origin" master:refs/for/master
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 12 threads
Compressing objects: 100% (5/5), done.
Writing objects: 100% (6/6), 35.88 KiB | 2.11 MiB/s, done.
Total 6 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2)
remote: Counting objects: 14726, done
remote: Processing changes: refs: 1, done
remote: ERROR: commit 5640255: email address <my_email_addressA> is not registered in your account, and you lack 'forge committer' permission.
remote: The following addresses are currently registered:
remote:    <my_email_addressB>
remote: To register an email address, visit:
remote: http://gerrit...................
remote:
remote:
To ssh://gerrit.....................
! [remote rejected]   master -> refs/for/master (commit 5640255: invalid committer)
error: failed to push some refs to 'ssh://gerrit...................'
```

# 错误原因
公司的邮箱有两个，一个是公司内部用，一个是对外用。
我之前某次提交， 用了A邮箱，导致这份提交无法被push。
git设定中登录的是B邮箱，我尝试添加A邮箱也失败，大概我没有权限吧，也或许A邮箱不可被登录在git上。

# 解决方案
赞美chatgpt指引我找到答案！
步骤如下：
1. `git log --pretty=format:'%H %ae %ce'`

     查看每次提交者的信息，查看错误邮箱在哪个提交里面。

2. `git rebase -i <commit-hash>`

     把commit-hash 改成想要修改的提交的哈希值，运行上面的代码，不用加尖括号。

3. 接下来就会出现提交时的信息，把commit前面的pick改为edit，保存并退出。

   插播一句：按`i`进入修改模式，按`esc`返回，按`:wq`保存并退出。

4. 接下来就是修改邮箱啦

   `git commit --amend --author="Author Name <email@address.com>"`

   把自己的用户名和邮箱改为正确的，这次需要尖括号啦

   这样就OK啦！

5. 但是我的同事已经替我push了，我要把自己之前的提交删掉。

   `git reset --hard HEAD~1`

   因为我只动了最后一次提交，这句代码就够用了，可以撤销我所有的更改和暂存。

  




