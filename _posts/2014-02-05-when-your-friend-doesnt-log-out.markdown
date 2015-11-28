---
layout: post
title: When your friend doesn't log out
date: 2014-02-05 16:25:54.000000000 -05:00
---
In many Unix systems, users have something called a bashrc file located at ```$HOME/.bashrc``` which runs every time someone starts a new session.

Imagine your friend has walked away having not logged out, leaving this file vulnerable. As their friend, you care about them and feel the need to teach them the importance of security. Here are some suggestions for some simple one-liner _lessons_ you could stick in that file.

If it's their first time, start with something simple.
```
alias ls="cd"
```

Maybe make their favorite tool unfamiliar.
```
alias vim="emacs"; alias emacs="vim"
```

Create a problem that grows when inspected.
```
alias ls="touch \`fortune\`; ls"
```

Take away their sense of control.
```
alias cd="cd \`ls -d */ | shuf -n 1\`"
```

Play the long game.
```
echo \"sleep 1\" >> ~/.bashrc"
```

Leave something to surprise them.

```
sh -c "sleep $RANDOM; cat /dev/random" &
```

Fuck them.
```
mv ~/.bash_history ~/.bashrc
```

_If you would like to do this on OS X, just keep in mind that by default you won't have fortune or shuf, and the .bashrc isn't loaded on new sessions. You'll have to attack their .bash\_profile instead._
