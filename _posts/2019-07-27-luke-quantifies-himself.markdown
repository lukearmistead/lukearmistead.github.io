---
layout: post
title:  "Overengineering the quantified self: Background and motivation"
date:   2019-07-27
categories: jekyll update
---

As a data scientist, I like data. As an optimization junkie, I like reflecting to find ways to improve. I've seen tech companies combine those affinities to produce iteratively improving apps users adore. If the scientific method works for business (and science), why wouldn't it work for us? **Couldn't we also use hypotheses, metrics, and structured reflection to chart an iterative path to become better people?** We totally could, rhetorical question asker. We totally could.

## The hipster's approach to the quantified self

Over the years, I've used dozens of tools to log and synthesize my life's happenings, tasks, and habits. Most recently, I ditched fancy apps and went back my curmudgeonly roots by picking up good old fashioned pen and paper.

<img src="/assets/notebook.jpg" width="300"/>

_The scary blank page won't intimidate you unless you let it intimidate you._

 For the entirety of 2018 I kept a bullet journal. The coolest part of my system was quantifying habits. Each day I'd write down, say, the number of hours I'd slept that night. At the end of the month, I'd pull the weekly averages into a hand drawn line graph kind of like this.

<img src="/assets/habit_tracker.jpg" width="200"/>

_Oh my habit tracker? You probably haven't heard of it._

## And it's discontents

The bullet journal is cool. So cool. So incredibly heavyweight. By the time I'd finished remembering and entering each day's habits, moods, sleep hours, and happenings I'd be too burned out to actually synthesize the information into insights I could act on. To make matters worse, I have a prodigiously bad memory. **Bad data is worse than no data because it results in bad decisions made with high confidence.** And for this optimizer, that will not do.

## The scientist's approach to the quantified self

It turns out that we create a lot of data just by living in the 21st Century. Our banks log transactions, our calendars log gatherings, and our heart rate trackers log activity. What if we pulled all that stuff together and treated self reflection like an analysis? Did you miss that productivity goal you set last month? Now you can analytically interrogate whatever data you've gathered for the root cause.

<img src="/assets/interrogate_data.jpg" width="200"/>

_Who made you do it, y? Was it x? I want NAMES!!!_

Once you're done, making a plan to hit your goal next month becomes trivial. Even cooler, what if we tried random experiments? Want to try the Pomodoro Technique? Randomly assign days with either a tomato timer or with your existing method and measure which show has the highest productivity.

<img src="/assets/chicken_timer.jpg" width="200"/>

_Experimental arm B, the scrappy competitor to the classic Pomodoro Technique_

Your automated data pipeline will handle the tedium of gathering and aggregating the information, leaving you free to engage in the fun of analyzing the results and turning them into a plan.

## Let's be scientists!

Enough pontification. Let's get into the code! My next post will cover using Fitbit's API to connect to health data. After that, we'll get to the good stuff - turning those raw numbers into insights and maybe a couple of visualizations.

The long run ambition here is, well, ambitious. I want to create a system of automated pipelines that aggregate all my data in a warehouse. That warehouse would support a suite of metrics and visualizations that enable scientifically informed iterative self improvement. That's a pretty intimidating vision, so we'll take it one chunk at a time, starting with my next post. See you there!
