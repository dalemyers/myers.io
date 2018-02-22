---
layout: single
title: Stop Suggesting Git Aliases
date: 2018-02-22 12:00:00
tags: [git]
---

Sometimes it feels like every second day there is a post on Hacker News or Reddit about great new git aliases that you totally need today, and if you don't you're the devil. To users who have just gotten to grips with git, these can seem like the coolest new thing to have. How do I know this? Because I fell for it when I was in that situation. 

Git is one of the most difficult tools for people to wrap their heads around. Interspersed with the git alias blog posts (yes, I do love irony, why do you ask?) are the git tutorial posts. Sure, years on it all makes sense to us, but for beginners, understanding what each command does is the most important thing. 

Aliases break that. Well, that's not true. YOUR aliases break that. Here are 4 different categories of alias and reasons why each are bad. 

The first is the set of typing savers:

`git p` -> `git pull`<br/>
`git co` -> `git checkout` 

These teach new users to use these commands rather than the real ones. When it comes to learning about something else, there is a separation between the commands they are learning how to use more effectively and the commands that they know how to use. It can be difficult to reconcile that `pull` is `p` each time you see it. Sticking with the original commands, at least until you are quite comfortable with git teaches you how to think about it in a consistent way which is supported by the rest of the community. 

In slot number 2 we have the aliases which string multiple commands together:

`git cm` -> `git add -A && git commit -m`<br/>
`git cp` -> `git commit && git push`

These commands can simplify the flow for beginners, I won't deny that. But they do it at a cost. In this case, it can be difficult to reason about what is going on under the hood when a user runs `git cp`. It doesn't force you to face the idea of there being a staging area, committed code and pushed code. Instead, it treats it more like SVN does, where you have changes locally or you have them on the server. It ties up the usefulness of git into a less functional wrapper, and makes it harder to learn the individual components. 

For number 3, we have the commands which provide flags to existing commands:

`git rbc` -> `git rebase --continue`<br/>
`git fix` -> `git commit --amend --reset-author --no-edit`

This is the hardest to argue against. The flags can be hard to remember. That's a fact. Having aliases helps with these. In this case though, it takes from you the configuration possibilities, as well as understanding of individual components as above. The `fix` command for example makes it easy to make a change and roll it into the previous commit. However, without the flags, you don't really understand the mechanism by which it does this. Not to mention the fact that if you are ever on a machine without your aliases, you'll have no idea what you are doing. 

The final category is workflow specific aliases:

`git up` -> `git pull --rebase --prune $@ && git submodule update --init --recursive`<br/>
`git wip` -> `git add -A; git ls-files --deleted -z | xargs -0 git rm; git commit -m "wip"`<br/>
`git unwip` -> `git log -n 1 | grep -q -c wip && git reset HEAD~1`

Each of these are tailored to a specific person or teams workflow. They may not be useful to a new user and pushing them to use it is giving them a nice shiny new hammer and saying "Look at all these things which can be treated like nails!". A user needs to decide their own flow. Git lets you customise it with aliases so you can adapt your tooling to your flow. It's a terrible idea to ignore all that and make your flow fit someone else's tools when you have your own ones. 

So, I'm not against git aliases. In fact, 2 of the aliases above are from my own config files. What I'm against is confusing users with new things, and hiding the truth from them. Once a user has used git on it's own and understands their own flow, that knowledge, combined with the knowledge that you can add lines like:

    alias_name = some_sub_command -with -flags
    other_alias = !git whatever | other_command

is enough for them to build their own tools, customised to their own knowledge that suit their own flows. 

Let users learn their own aliases. 

P.S. Except log aliases. Share those as much as you want. No one is going to remember how to write those each time. 
