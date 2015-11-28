---
layout: post
title: Flappy Bird in 1K
date: 2014-02-22 21:48:32.000000000 -05:00
---
For a guy who's paid to write Python and C, I sure write a lot of blog posts about JavaScript.

Over this past week, I worked on a simple Flappy Bird clone as a 1K challenge. That is, fit the entire game into 1024 bytes of HTML and JavaScript. I actually did it 996 including the comment that points to my github. I'm still thinking about what I might want to do with those extra 28 bytes.

Replicating the game actually ended up being a little tricky. I tried to do realistic physics. I treated gravity as constant acceleration, gave my bird a weight, and assumed the bounce to be an impulse (continuous force for some interval of time). But the feeling of the jump just never worked out. What finally did do it for me was giving up on the idea of weight and force and just treating the bounce as constant velocity over some interval of time. It was a nice insight into how the original developer might have wrote.

Minifying the code resulted in some serious ugliness. Especially when it became a fight for the little optimizations. The actual bounce call back tied to the space bar, for example, started looking something like this
```
if (e.keyCode === 32) {
  if (!f) {
    r();
  }
  t = 0.1
}
```
but ended as
```
e.keyCode === 32 && (t = 0.1) && !f && r();
```

[Demo it here.](http://devon.peticol.as/Flappy-Bird-1k/flappy-min.html)

Source at [github.com/x/Flappy-Bird-1K](https://github.com/x/Flappy-Bird-1k). I'd love suggestions.
