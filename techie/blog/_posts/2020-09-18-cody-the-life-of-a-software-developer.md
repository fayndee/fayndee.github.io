---
title: Cody - The Life of a Software Developer
tags: architecture
excerpt: The story of Cody, a fellow software developer who is passionate about his job. After many years of up-and-downs, Cody has learned so much that it comes almost natural for a nice guy like him to summarize the learnings and share them with others.
---

Today, let me tell you the story of Cody, a fellow software developer who is passionate about his job. Here he is ↓

{% include article_image.html image='article_cody.jpg' format='original' %}

## It's Magic

The company where Cody works has a big legacy system. Cody's team had a tough time to get their features delivered as many others teams also work on the same code base. Every second build was failing due to conflicts or broken tests. Even if Cody gets it through once, chances are he has to roll it back as somebody changed a setting which tears his feature apart. Cody felt he has to do something, but he didn't know what exactly.

He went to a local meetup and learned the magic word "*Microservices*". Since then, Cody and this team did nothing but cutting the big monolith in pieces.  Much later, instead of release 1 monolith, Cody had to release 10 microservices to roll out a feature. He was kind of happy about the cut, when he didn't think too much about it.

Just another day at cafe, Cody met an old fellow named Eric. They spoke for a long time. The old Eric talked a lot about "*Domain*" and told Cody that it is not about the cut, but the context your business domain lives in and the boundary surround it. Impatient guy he is, Cody ran back to his desk and wrote down "*__Domain-Oriented Services__*". The next day, he was on a different challenge. 

{% include article_image.html image='article_microservices.jpg' format='original' %}

## Somebody Else's Problem

Things had improved. Cody and his team were able to shape their features around a coherent domain. One day his PO came to him asking for help. It turned out their last feature rollout wasn't financially a success and the boss is unhappy. They had no idea what went wrong. Cody pulled out the data they used for backing up their early assumptions. Only then he found their data was outdated. Cody's team was using data from a shared database, but the owner already moved their data into a new data store where nobody else has access. 

In order to fix the mistake, Cody's team need to run a new analysis and adjust their product accordingly. The good thing is the company has an expensive BI tool capable of doing that. The bad, not much data was ever made available to it as everybody think that was somebody else's problem. Realised the issue or stressed by pressure, Cody went team after team yelling "*Guys, __Data is a shared asset__, by making it available centrally makes everybody's life easier. Helping me, helping you!*"

A few weeks had to pass until Cody's team finally got the data they need to fix their last feature.

{% include article_image.html image='article_data.gif' %}

## Oups, Sorry

Cody's team owns a couple of applications. One of them deals with their customer's data using nothing fancy but only simple CRUD operations. A good colleague of Cody wanted to make fun of him so wrote a small website that picks a random customer's data using Cody's application and put it under Cody's picture. Everything was wrapped in an email with a good clickbait and sent to everybody in IT. 

As soon as Cody opened it, he knew who did that. Joke aside, there was a big WTF in Cody's head: how did this guy manage to put our customer's data on this website? Before Cody figures it out, the boss stepped in and it was the worst day of Cody's life.

It turned out the application in question didn't have any cross-origin check or authorisation whatsoever. Anybody can trick an internal employee to click open such website which then can steal data from any unprotected internal resources. The security officer got quickly involved and told Cody's team that they should trust nobody, their apps more so. It was a hard lesson for Cody and his team. Purely for memorial purpose, Cody wrote down "*Zero Trust Room. __Zero Trust Architecture__.*" and a skull on a post-it and put it on the team's front door.

{% include article_image.html image='article_trust.jpg' %}

## No one wants Wheels

As Cody's product became more important and attracted more traffic, the team wanted to improve the stability of its build pipeline and find a better way to deal with its high-volume internal messaging flow. After some research, Cody decided to derive from company's shared pipeline library to add a new feature. Also he convinced the team to roll out a whole message processing cluster, completely managed by the team itself, to handle the load.

At first, those measures seem solved the team's problem. Soon enough, Cody found himself busy patching his pipeline library with the latest updates from the company shared one in order to keep it compatible with the build platform. Moreover, all team members were regularly involved in making the new message processing cluster alive. Everybody was overloaded, so did the system.

Cody asked help from the platform team, they quickly pointed out: The core issue isn't the tools, but the process. The team should have consulted the platform team first before making those decisions. There are other fully managed alternatives the platform team can help Cody's team to set up to deal with the load. Also deviate from standard library means additional maintenance effort to the team. In the end, Cody's team don't want to build wheels, they want to build a car. So they should "*__Re-use over Re-invent__*" the wheels.

{% include article_image.html image='article_reinvent.png' format='original' %}

## Cody, the Nice Guy

After many years of up-and-downs, Cody is still strong in coding. He has learned so much that it comes almost natural for a nice guy like him to summarise the learnings and share them with more people. Cody put them down and named it the "*__Architectural Principles__*", hoping that it will help developers like him to continue to do what they truly love.

* *__Domain-Oriented Services__ - Build loosely coupled and highly cohesive services around comprehensive business domains.*
* *__Data is a shared Asset__ - Embrace data ownership in services and share data through the central data lake.*
* *__Zero Trust Architecture__ - Take security seriously and apply the least privilege principle in services.*
* *__Re-use over Re-invent__ - Re-use and generalize existing knowledge, software, platform rather than re-inventing them.*

{% include article_image.html image='article_the_life.png' %}
