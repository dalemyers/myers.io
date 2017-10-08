---
layout: post
title: 1Password Update 
date: 2015-10-24 14:19:00
tags: [software, security, passwords, password managers, 1Password]
---

My previous [post](/2015/10/22/1password-leaks-your-data/) turned out to be
rather popular in much the way I didn't intend. Originally I assumed I would
post it to [Hacker News](https://news.ycombinator.com/), get maybe a hundred
views at the most, and then email AgileBits again, pointing out that some people
cared. That's not what happened. Not even close. The post made #1 on Hacker News within an hour
and people started taking it completely the wrong way. As such, I want to make a
few things clear. 

1. 1Password is still safe as a password manager. Your passwords are still safe.
   It is your metadata that is not if you still use the older Agile Keychain
   format. Everything else is completely secure.
2. This issue does not affect anyone who does not use the Agile Keychain format.
   For instance, if you use 1Password on OS X and don't sync with Dropbox, then
   you will be using the newer OPVault which does not have this issue. 
3. AgileBits were not only aware of this, but have publicly published articles
   about this. They have also explained their reasoning for it on their forums.
4. AgileBits have publically stated since that, while all passwords are still
   safe, they are already making changes to make OPVault the default format. 
5. Again, 1Password is still safe as a password manager, it's just the metadata
   that can be leaked if someone has access to your keychain (whether that's
           becuase you host it in Dropbox, or they have access to your machine).


Over the last few days many people have taken this completely out of context.
For that, I take full blame. I failed to consider how sensationally internet
users can perceive information. I should have made the title and article as
boring as possible so only those who this affects would pay attention to it. I
actually felt so bad about this that I wrote a letter of apology to AgileBits.
They assured me that there are no hard feeling and have thanked me for reminding
them that a significant proportion of security is perception. It doesn't matter
how secure something is if users don't use it because they don't understand the
risks. 

For one final time, 1Password is still safe as a password manager. I will
continue to use it. I have no intention of switching to any other manager. 

