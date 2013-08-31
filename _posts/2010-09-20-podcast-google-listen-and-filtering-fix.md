---
layout: post
title: Podcast – Google Listen and Filtering Fix
categories:
- misc
tags:
- rss; podcast
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
---
<p>Hi dear reader. Long time no see ;) but lets get straight to the point.</p>  
<p>I’m a huge podcast fan. I listen them on a daily basis and I’m always trying something new. Recently I’ve started learning Korean. The best thing in learning languages those days is that you have lots of free resources – from articles about the language – to whole curses with audio. One of those curses is provided by <a href="http://www.talktomeinkorean.com/">TalkToMeinKorean</a>. It’s in a format of podcast with companion pdfs. The problem that I had was that I was just starting and the beginner series were at the very beginning (or at the end – depending on the point of view) of the RSS feed. Getting to them them on my Google Listen Android app was impossible. It would choke on <em>Load more episodes…</em> after about 20 of them. Having in mind that mentioned RSS at the time of this writing on Level 3 Lesson 14.. well let’s just say scrolling is not an option.</p>  
<h4>Quick Fix</h4>  
<p>After evaluating some filtering solutions on the web I realized that even though I’m able to filter the feed – FeedBurner only returns about 10 episodes, and I don’t know how to force it to give me more (I’m still looking for an answer if anyone knows). I took a different approach. I took the feed straight from the website – where I could get it filtered by category. Then using Yahoo! Pipes I connected feed from several sites and ordered them appropriately. It sounds complicated but it looks like that:</p> 
![]({{site_url}}/images/20100920_image.png)
<p>You can actually add several URLs to the <em>Fetch Feed</em> module but I had some troubles with the sorting afterwards. </p>  
<p>At this point it looked good. The problem was that both Google Listen and Google Reader didn’t recognize the podcast feed. After looking on the internet I’ve found that the difference between the normal RSS feed and Podcast RSS feed is the Enclosure element. I had that … but..</p>
<p>Apparently the blog feed didn’t set media type on the Enclosure element so both readers didn’t see it as an audio feed. Solution:</p>
![](/images/20100920_image1.png)
<p>After this little Regex fix everything works fine and I can enjoy Korean lessons when I commute to work and back (well at least in between all those programming podcasts I’m subscribing to).</p>  <p>That’s all folks :) If you ever need to filter or sort RSS feeds Yahoo! Pipes is a great tool.</p>
