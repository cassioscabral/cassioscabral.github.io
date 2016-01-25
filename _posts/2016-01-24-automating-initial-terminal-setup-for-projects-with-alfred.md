---
layout:     post
title:      "Automating your project dependencies commands that run on terminal with Alfred"
excerpt: "Easy way to open several tabs with processes at once"
date:       2016-01-24 12:00:00
last_modified_at:  2016-01-24 12:00:00
author:     "cassioscabral"
header-img: "img/blog/header/post-bg-01.jpg"
thumbnail: /img/blog/thumbs/thumb01.png
tags: [automation, tools, terminal, tab, alfred, process, processes, project, dependencies, cassioscabral, cassio, cabral, cassio s cabral, cassio soares cabral]
categories: automation
comments: true
share: true
image:
  feature: getToTheChoppa.png
  topPosition: 0px
bgContrast: dark
bgGradientOpacity: darker
syntaxHighlighter: yes
---

## Do you ever needed to open a terminal and several tabs to start working on a project ?


Well, I needed. And it always gets annoying to start different projects at different time. I always get confused, "is this running MongoDB or Postgres?"

Recently I bought Alfred awesome powerpack and I can't indicate enough for you. If you love automation tools, that's the ONE.

So I started by downloading some workflows and loved the idea, then suddenly I realized that I could build a workflow for me and that's where I thought about to build this terminal-opening-several-tabs-with-several-different-processes-at-once-thing =X

I named it getToTheChoppa workflow. If you don't know the scene in Predator movie, it's from there. It's kind a meme also and it feels that I have to rush to open my workplace setup as quickly as I should get to the choppa =)

Since Alfred only works on Mac, this is focused on Mac only. And you need the powerpack to be able to create your own workflow.

### How to create a workflow

  - open Alfred settings
  - click on the '+' sign
  - give it a name and a nice icon if you want

  - click on the '+' inside the workflow builder
  - input > Keyword
  - give it the keyword(command), mine is gettothechoppa

  - click on the '+' inside the workflow builder again
  - actions > Run NSAppleScript

  - connect the two

  - build your script, example below


### The script

I tried use Ruby or Python without much success and the easiest and practical that I could found was the AppleScript.
It's quite verbose but very descriptive and human readable, so after while is intuitive.

Tried to build functions to make more generic and easy to maintain as possible but AppleScript was not kind to me, and after several tries and headache I decided to took the easiest solution.

**Script gettothechoppa:**

*Observations*: I changed the option "New tabs opens with: " from my terminal settings from 'same profile' to 'default profile'. **Without this**, the makeTab command will run the same commands over and over for every new tabs.

```AppleScript
on makeTab()
  tell application "System Events" to keystroke "t" using command down
  delay 0.2
end makeTab

on alfred_script(q)
  tell application "Terminal"
    set query to q
    activate
      if (query is equal to "myproject")  then
        set go_to_folder to "cd ~/Documents/workspace/my_project"
        do script go_to_folder
        do script "atom ." in tab 1 of front window
        my makeTab()
        do script go_to_folder in tab 2 of front window
        do script "npm start" in tab 2 of front window
      else if (query is equal to "anotherproject") then
        do script "cd ~/Documents/workspace/anotherproject"
        my makeTab()
        do script "cd ~/Documents/workspace/anotherproject" in tab 2 of front window
        do script "redis-server" in tab 2 of front window
      end if
  end tell
end alfred_script
```

The `makeTab()` function creates new tabs

The `alfred_script(q)`` is the default NSAppleScript template, it will give the user input as 'q', and I change later to 'query' for readability

The rest of the script it is pretty self-explanatory, it will `cd` to a folder and run commands on it. Then make a new tab and run another command and so on

The `cd` repetition comes from the change on the terminal settings

You **must** define the tab numeration

That's it. Simple, practical and fast. Forget about using `cmd+t` that much anymore

Cheers
