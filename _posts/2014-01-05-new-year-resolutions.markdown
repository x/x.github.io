---
layout: post
title: New Year Resolutions
date: 2014-01-05 13:58:07.000000000 -05:00
---
'Tis the season to make empty promises of self improvement. But this year, instead of resolutions to go to the gym, cook at home, or not horde shiny gadgets, I thought I'd make resolutions that would help me improve myself as a developer.

__2015 Update:__ _It's now been almost two years since I wrote this post. I decided to follow up and see how I did. Future notes are in italics._


## 1. Put my dot files on a diet
I love the "Unix as an IDE" philosophy. I use Vim as my code editor, tmux to manage my sessions, and a myriad of Unix tools to poke and prod my code.

But, over the years, my desires for the next best plugins, more _elegant_ shortcuts, and everything to have Vim-like keybindings have made my dot files fat slobs. __By the end of this year, my dot files will be smaller, not larger.__

__2015 Update:__ _I did do this, but then new stuff just came in. At the end of the day, I've actually only net-trimmed 13 lines over two years. Here's the git diff --numstat against my last commit of 2013:_

```
added	deleted	name
87      100		.bashrc
4       2		.gitconfig
17      15		.tmux.conf
123     127		.vimrc
```

__2015 Results:__ 👍


## 2. Improve my Linux-foo
Since graduating school and working in the real world, my ability to weaponize basic Unix tools has skyrocketed. I find myself regularly crafting long chains of tools that, until recently, I had not known existed.

However, there are a few tools that, even though I know are useful, avoid using properly due to their steep learning curves. To approach this I've come up with three sub-resolutions.

1. __Use find like a boss.__ Up until now I've only every used find by doing ```find . | grep filename```. I know it can do much more, but it's unnatural flags have scared me. I will conquer these fears and learn to use find properly.
2. __Learn Sed & Awk.__ My father once told me he writes all his code in either assembly or Awk. Its terrified me ever since. But I know these tools are the next step in my path to pipe-chain enlightenment. I will master these languages to use them prominently in my tool set.
3. __Write tar commands without referencing the man page.__ This has been a goal of mine ever since I was a naive teenage Linux user who attempted to install things from source. [Even XKCD knows my pain](https://xkcd.com/1168/)

__2015 Update:__ _This ones funny, because I think I have improved my linux-foo in some amazing ways, but they weren't the ways I thought they'd be._

1. _I still do `find . | grep filename`. Screw find. It's too confusing. Also, I've since realized that saying "like a man" is the wrong way to refer to programming._
2. _I've written a litte, but still not much. I've gotten much better with cut and just started using python to do anything more complicated._
3. _This was a fools quest._

_On the other hand, here are some awesome things I have learned..._

1. _How to integrate vim into my shell-life by `ctrl+x,ctrl+e` and the shell into my vim life using `:<selector>!`._
2. _Better understand how a machine is being used by using new `top` filter, sorts, and views, `bwm-ng`, `jstat`, and `strace`._
3. _How to debug network and load with `tcpdump`, `ngrep`, and `ab`._

__2015 Results:__ 👎x3 + 👍x3

## 3. Get better at setting and reaching goals
This is a big one. It's a problem I find myself facing more and more in my growth as a developer. When I was in school, I had grades to keep me motivated to reach assignment goals. In my free time I went to hackathons which motivated me to reach personal project goals.

I've grown as a developer since then, and so have my goals. The things I do at work are more challenging and open-ended then anything I did in school. The things I desire to build outside of work have vastly outgrown a caffeine-fueled weekend.

At work, I've decided to embrace the tools provided to us. For Chartbeat, this namely means Asana. I will keep everything I do in Asana as a goal, pace myself with deadlines, and most importantly, only do things that I've set in those goals. This is to quell my desires to refactor and optimize.

Outside of work it becomes much harder. I struggle with balancing my desire to learn iOS with my need to clean my apartment. To face this, each week I'm setting a goal and blocking out time to work on that goal. The hardest part will be to keep focus on one project over many weeks. I find that the rate at which I have and become excited about new ideas greatly exceeds my realistic rate of execution.

__2015 Update:__ _This is an interesting one I completely forgot about. Since writing this, my company has grown and transitioned to a more traditional scrum process. I'm now the lead of a small team, which really just means I help a couple other developers, who work on stuff relevant to me, reach goals. Like most programming problems, this seemed trivial from the outside, but revealed it's self to be more and more difficult the deeper I dive._

_Goals, as I've learned, are almost never independent of other goals. They have dependency goals. And often those dependencies are on other engineers. And those dependency goals have their own dependency goals. And every engineer has a priortirty of their goals. And all of them have fluctuating levels of difficulty and time. It's really an NP-hard problem that we're trying to solve with TODO lists and google spreadsheets. I don't think I've learned all I have on this yet, I'm not sure I ever will, but one thing I've gained is a big improvement on setting and reaching goals for myself._

__2015 Results:__ 👍

---

I don't think I've ever actually kept a new year resolution for the entire year. It's all too easy to fall back into the same old habits. But, at the very least I'm identifying my weaknesses and defining ways to improve them. I'm looking forward to this year.
