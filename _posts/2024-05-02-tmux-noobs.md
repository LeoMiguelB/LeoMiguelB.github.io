---
title:  "Minimum Working Tmux"
date:    2024-06-16 12:06:00 -04:00
published: true
---

## Intro
Today I will show you how to get started with Tmux - a terminal multiplexer. I am no expert but I believe the following steps if followed correctly will yield a minimal working workflow for tmux. Let delve into it.


## Setup
Firstly install tmux.

```
sudo apt-get install tmux
```

You can now type tmux in the commandline, and enter the wonderful interface it presents. This will effectively make you look like a try hard programmer.

## Commands
Something that tmux does is that you can "detach" sessions on demand. What this basically means is that you can leave the session running in the background of the server. This way the session persists, and you can hop back into it any time you want.

Assuming you haven't touched the config file yet your [prefix](https://waylonwalker.com/tmux-prefix/) will likely be `ctrl + b`. In order to detach from your sesions you must enter the prefix and then press `d` - `ctrl + b and then d`.

Now in order to enter back into that session you will need to ls tmux sessions, and then attach back into it.

```
1. list tmux sessions
tmux ls

output:
3: 1 windows (created Sun Jun 16 11:57:26 2024) (attached)

2. attach into the session
tmux attach -t 3

```

Now I will present you the minimum controls you need to be competant in this tool.

```
1. create a new pane
prefix + %

2. delete the pane
ctrl + d
or
type exit

3. adjust width of pane
prefix + alt + left/right arrow keys

4. switch between panes using index
prefix + q -> this will show a temporary UI that shows indices of panes -> you must enter the indices in that time frame

5. check key binding
prefix + ?

```

## Useful Mode
As you might have already noticed tmux is a modal editor. The mode that I find myself using the most is the copy mode. This allows me to copy different content across panes simply by doing `prefix + [`, then pressing space to start the highlight and finally press enter to copy highlighted section. Something to note is you need to add the following line to your tmux config file. This enables the key binding that I mentioned.

```
setw -g mode-keys vi
```

## Conclusion
As with any tool, it takes practice and determination to get good at it. Hence practice...


