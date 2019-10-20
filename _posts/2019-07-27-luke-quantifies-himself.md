---
layout: post
title: "Overengineering the quantified self"
date: 2019-07-27
categories: jekyll update
---

I'm a data scientist, so I like data. I'm also an optimization junkie, so I like reflecting on ways to improve. I've seen tech companies combine those affinities to produce iteratively improving apps users adore. If this idea works for killer businesses, why wouldn't it work for us? **Couldn't we also use hypotheses, metrics, and structured reflection to chart an iterative path to become better people?** Glad you asked. We totally could.

## The hipster's approach to the quantified self

Over the years I've used dozens of tools to log and synthesize my life's happenings, tasks, and habits. Most recently I ditched fancy apps and went back my curmudgeonly roots by picking up good old fashioned pen and paper.


<img src="/assets/notebook.jpg" width="300"/>

_The scary blank page won't intimidate you unless you let it intimidate you._

 For the entirety of 2018 I kept a bullet journal. The coolest part of my system was quantifying habits. Each day I'd write down, say, the number of hours I'd slept that night. At the end of the month I'd pull the weekly averages into a hand drawn line graph kind of like this:

<img src="/assets/habit_tracker.jpg" width="300"/>

_Oh. My habit tracker? You probably haven't heard of it._

## And it's discontents

The bullet journal was cool. So cool. So incredibly heavyweight. By the time I'd finished remembering each day's habits, sleep, and happenings I was generally too burned out to synthesize that raw information into actionable insights. To make matters worse, I have a prodigiously bad memory. **Bad data is worse than no data because it results in bad decisions made with high confidence.** And for the chronic optimizer writing this post, that simply will not do.

## The scientist's approach to the quantified self

It turns out that we create a lot of data just by living in the 21st Century. Our banks log transactions, our calendars log gatherings, and our heart rate monitors log heart rates. What if we pulled all that data together and treated self reflection like an analysis? For instance, looks like you missed that productivity goal you set last month. Why? Before you were guessing. Now you get to interrogate the data you've gathered to find the root cause.

<img src="/assets/interrogate_data.jpg" width="300"/>

_Who made you do it, y? Was it x? I WANT NAMES!!!_

Once you've wrapped your self reflective analysis, making a plan to hit your goal next month becomes trivial. Want to try the [Pomodoro Technique](https://en.wikipedia.org/wiki/Pomodoro_Technique) to boost that productivity goal you missed last month? Put it on the court!

<img src="/assets/chicken_timer.jpg" width="300"/>

_Experimental arm B, the scrappy competitor to the classic Pomodoro Technique_

Your automated data pipeline will handle the tedium of gathering and aggregating the information, leaving you free to engage in the fun of analyzing the results. What about that new productivity plan? Did it work? No sweat. We'll can test it empirically.

## Let's be scientists!

Enough pontification. Let's get into the code! My next post will cover using Fitbit's API to connect to health data. After that, we'll get to the good stuff - turning those raw numbers into insights and maybe a couple of visualizations.

The long run ambition here is, well, ambitious. I want to create a system of automated pipelines that aggregate all my data in a warehouse. That warehouse would support a suite of metrics and visualizations that enable scientifically informed iterative self improvement. That's a pretty intimidating vision, so we'll take it one chunk at a time, starting with my next post. See you there!
