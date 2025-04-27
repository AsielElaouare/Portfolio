---
title: "Creating my own first Social Media Platform that reduces Market Validation for Problem Solvers"
meta_title: "Solvegrid Market Validation"
description: "Why should you use Hugo"
date: 2024-12-04T05:00:00Z
image: "/images/gallery/logo-solvegrid.jpg"
categories: ["Web Aplications"]
author: "Asiel Elaouare"
tags: ["ASP.NET", "C#", "MongoDB", "Startup", "Azure",]
draft: false
---

{{< toc >}}


## You know what’s wild?

Right now in tech, if you want to build something new, you usually start by guessing what people might need. Then you spend weeks (or even months) validating that idea — interviewing users, running surveys, building MVPs — just to find out if anyone even cares. It's exhausting. And honestly? Sometimes it feels completely backwards.

I've run into this problem more times than I can count.
I build something that I think will be useful for people, but I end up realizing that others see things differently. Sometimes they don’t fully understand the product. Sometimes they just don’t need it at all.
It’s frustrating — not because the work isn’t good, but because the gap between what I thought was needed and what people actually needed was bigger than I expected.

And even though I learn while building, there’s always a part of me screaming in frustration over all the time I wasted creating something that ended up meaning nothing to anyone.

What if there was a better way?

## How it all started

One time, I was wandering in my thoughts and started thinking about how incredibly helpful Stack Overflow is and how powerful a real community can be. There are real people posting real problems and others stepping in with real solutions.
While I was on the train heading to my programming lessons, frustrated about how I kept building things that did not really matter to anyone, a light bulb suddenly went off in my head.

### What if there was a platform like Stack Overflow, but for real, unsolved pain points in everyday life?

A place where people could post the problems they genuinely struggle with, hoping that someone out there might help solve them.
At the same time, it could be a huge opportunity for builders and founders who are tired of the endless guessing game and who, like me, have struggled with building things that nobody really relates to.

Solvegrid flips the whole process on its head.
Instead of founders, builders, and makers coming up with ideas and then scrambling to find out if anyone actually has the problem they're trying to solve... Solvegrid lets people with the problems post them directly.
Daily pain points. Annoyances. Things that slow them down. Things they wish someone, anyone, would just fix already.

This allows problem solvers to gain insights into real problems that are made public through the platform. As a post is shared, people who read it can relate to the problem and provide valuable feedback by viewing, upvoting or downvoting the post, and leaving comments to suggest new ideas or features they would like to see in a solution.

{{< image src="/images/gallery/solvegrid-example.png" caption="A problem raiser posting a pain point related to what he does" alt="alter-text" height="" width="" position="center" command="" option="q100" class="img-fluid" title="image title"  webp="false" >}}


## And then?
Problem solvers, builders, entrepreneurs, developers, creators can scroll through real-world frustrations, pick the ones they resonate with, and start creating based on actual needs. No guessing. No "market validation" stage where you pray you're not wasting your time.
You know from day one: this is a real problem for real people.

Solvegrid is like a bridge between pain points and potential solutions and right now, it's zeroed in on tech. Whether it's broken workflows, missing software, outdated systems, or tiny daily irritations that add up to big headaches, it's all fair game.

At its heart, Solvegrid is about something simple:
Start with the problem. Build what matters.

Imagine a world where creators didn’t have to spend half their time validating if a problem even exists.
Imagine starting with pain points so obvious, so painful, that the only question is how fast you can solve them.

That’s the future Solvegrid is betting on.
And honestly? It sounds like the future tech needs.

## How I built the MVP of Solvegrid?

When it came time to actually build the MVP for Solvegrid, I kept things simple and focused on getting it working as fast as possible. I used ASP.NET with Razor pages and C# for the backend, which gave me a solid and familiar structure to build on. For the frontend, I used Bootstrap to move quickly without worrying too much about style or design. After all, it was an MVP, so the goal was function over looks.

For the database, I decided to go with MongoDB instead of the usual SQL setup. It was honestly a lot of fun to use something different, and MongoDB made it easier and faster to handle the kind of flexible data Solvegrid needed.

Once the MVP was ready, I hosted everything on Azure and bought a domain to make sharing it easier. I used MongoDB Atlas on the free tier to host the database, which worked perfectly for the early stages. After everything was live, I started sending the link to people to collect feedback and see how they interacted with the platform.

I shared my project on IndieVoice, which helped me gather valuable feedback like this:

<a href="https://indievoice.app/projects/solvegridio" target="_blank">
    <img src="https://1f08bbd99d1a620c734d44a7ea6c9651.cdn.bubble.io/f1732389057276x672158288395191600/featured.png" 
         alt="IndieVoice Embed Badge" 
         width="250" 
         height="60" 
style="image-rendering: -webkit-optimize-contrast; image-rendering: crisp-edges;"/>
</a>                                                                                    


> As a problem solver tired of seeking validation I say Kudos! This idea needs no validation.
This will encourage many technical founders that spend months working on ideas that seem great in their head but flop to the ground when they hit reality head first. In terms of implementation I'd say it is still rough around the edges.
Interface could use some UI/UX love to make it more appealing.
Also found a more serious bug when trying to complete my profile.
When I submit it refreshes the page and the data I entered disappears without being saved.
Also no way to go back from profile page to dashboard/main thread. I sincerely hope this products takes off. I love building, hate validating.

> The UI is clean and well-design.  A great place for both problem raisers and problem solvers. I love it and will definitely use it.

> Solvegrid seems like a useful platform for people who are creative and often come up with new ideas but don’t have the time or resources to test them and see if they actually make sense. Another great thing is that users can easily check if there is a real need for something since people with real problems share their pain points, allowing others to offer solutions if they have them. I like that the entire platform is community-based, which means time isn’t wasted on ideas that no one needs.     

                                                                                             

{{< youtube W7s9HK-l3q8 >}}
