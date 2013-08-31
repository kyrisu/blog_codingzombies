---
layout: post
title: ! 'Paying back to open source community: DB Schema Extractor - Part 1'
categories:
- coding
tags:
- DB Schem Extractor
- GitHub
- open source
status: publish
type: post
published: true
meta:
  aktt_notify_twitter: 'no'
  _aktt_hash_meta: ''
---
<p>In life of every developer there is a time when he needs to pay back to the community that brought you up. I believe most of code should be open – this is the best way to tap into endless community knowledge and resources, the fastest way to find bugs and fix them and foremost the best way to become an awesome developer. If I only could I would publish all the code my clients are paying for – but most of the line of business apps that I’m writing are giving my clients some sort of
competitive advantage and they would never agree to such a thing .</p>
<p>Fortunately I can disclose utilities that I’m writing to make my life easier. Recently I needed to generate an excel file with a database schema for one of my clients. Seems easy and straight forward. Unfortunately I didn’t find any app that would do exactly that. After looking at some forums I decided that writing to Excel file would require to have Office installed due to lack of good and free XLS library (well there’s one called <a href="http://code.google.com/p/excellibrary/">excellibrary</a> but it lacks an option to merge cells), but wait.. Wasn’t HTML used to create documentation? Well.. Yes Sir! 
</p>  <p>HTML solution has many advantages: 
</p>  <ul>   <li>Output would be a single, small, easy to open file. </li>    <li>It’s multiplatform (you have a browser pretty much everywhere), and doesn’t require any office suite </li>    <li>It’s really easy to copy results to Excel or any other spreadsheet editor </li>    <li>Additionally it will allow us to use some JavaScript magic for filtering purposes. </li> </ul>  <p>After an evening of coding the end result looks like that:
</p>
![](/images/20101018_image.png)
<p>For now it’s not very pretty, but I just wanted to put it out the door, hoping I will get some feedback. You can of course find all the code on <a href="http://github.com/kyrisu/DBSchemaExtractor">GitHub</a> – have fun.
</p>
