---
layout: post
title: How to change the past – Changing Git committer.
categories: []
tags: []
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
---
<p>There’s a war going on. A war between Subversion users and Git users on the issue of changing history. Subversion supporters believe that the commit history shouldn’t be changes and this brakes the contract between the developer and the svc. On the other hand there’s Git where history modification is possible and fairly easy - only your moral rules that can stop your from altering the past :)</p>  <p>I’m a Git user, but still I think that rewriting history is a bad thing. Even if you committed an awful bug – nobody is perfect, and this commit will remind you about the mistake. Hiding mistakes is almost never good. Recently however I found myself in a situation when I needed to rewrite history. I was working on my project for work and just before committing it to the central repository I’ve realized that I was using my personal user account to do the changes (every commit was by user kyrisu which I don’t think is very professional ;) ). However thanks to some Git magic it’s an easy fix:</p>  <p></p>
{% highlight bash %}
git filter-branch --commit-filter '
        if [ &quot;$GIT_COMMITTER_NAME&quot; = &quot;OLD_COMMITTER&quot; ];
        then
                GIT_COMMITTER_NAME=&quot;NEW_COMMITTER&quot;;
                GIT_AUTHOR_NAME=&quot;NEW_COMMITTER&quot;;
                GIT_COMMITTER_EMAIL=&quot;NEW_COMMITTER_EMAIL&quot;;
                GIT_AUTHOR_EMAIL=&quot;NEW_COMMITTER_EMAIL&quot;;
                git commit-tree &quot;$@&quot;;
        else
                git commit-tree &quot;$@&quot;;
        fi' HEAD
{% endhighlight %}

<p></p>

<p>.. and that's all. Now all my commits are submitted by company_login@company.com ;) </p>
