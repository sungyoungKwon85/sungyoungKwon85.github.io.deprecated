---
layout: post
title:  "change-message-after-push"
date:   2017-09-04 10:10:53 +0900
categories: git
---

# Only want to change recent commit message after push?  
https://stackoverflow.com/questions/8981194/changing-git-commit-message-after-push-given-that-no-one-pulled-from-remote  

{% highlight ruby %}
git commit --amend
git push --force <repository> <branch>
{% endhighlight %}

<br><br><br>
# even want to include previous commits?
http://egloos.zum.com/snskshn/v/5684896  

{% highlight ruby %}
git rebase -i HEAD~3
git commit --amend와 git rebase --continue를 edit로 수정한 커밋 개수만큼 실행.
변경을 원하는 커밋을 pick에서 edit로 수정.
(도중에 취소하고 싶을 때)git rebase --abort
{% endhighlight %}
