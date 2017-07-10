---
layout: single
title: 1Password is About to Lose a Lot of Customers
date: 2017-07-10 12:00:00
tags: [1password, password managers, security]
---

Almost 2 years ago I wrote a post calling out AgileBits on the fact that
1Password was leaking metadata. I pointed out that while they aren't leaking
passwords, there mere existence of accounts is enough to cause problems for many
people. This flaw was with their older keychain format, and my main issue wasn't
with the flaw, but the fact that they weren't encouraging people to move to the
new format which didn't have the issue. Despite this, I was still happy with the
security of the product and I had faith in the company. Today, that no longer
applies. 

A few weeks ago a
[post](https://blog.agilebits.com/2017/06/20/introducing-1password-6-6-for-windows/)
appeared on the 1Password blog. It didn't received much traction until today
though. Essentially, the post lays out a new version of 1Password for Windows
which will make the current file formats read only and force the user to pay a
subscription fee to store their passwords on 1Password's servers. Now, I want to
make it abundantly clear that I don't have a problem with their fee model being
subscription based. In fact, I practically welcome it. I paid for 1Password 3 or
4 years ago and got a licence for Windows, OS X, Android and iOS. I gave them
something like £80 for all that, once. This is a company who I trust with my
security. I don't want them to be scraping by and trying to figure out how to
pay their employees. I don't want them cutting corners in order to get sales.
Keeping the company healthy is in my best interests as a user. A subscription
model would guarantee income for them, and would help me sleep a little easier
at night. 

So where is my problem? It's the fact that I no longer have control over my
vault. My passwords will automatically be synced to 1Passwords servers and will
no longer be in my control. "But your passwords are encrypted before being sent
to them!" I hear you cry. Sure, I'm not denying that. I have faith that
AgileBits have implemented this mechanism safely and securely. The problem isn't
my passwords on their servers. It's everyone's. 

Servers full of passwords are wonderful targets for thieves. Now imagine servers
full of password vaults. What Fort Knox is to a bank robber, 1Password's servers
will be to black hats. AgileBits have just painted a target on their backs and
don't even realise it. 

So let's say that someone does compromise their servers (I said I had faith in
        AgileBits, but no one writes perfect code all the time), what happens
next? I have a strong master password, so I know that my passwords would be
safe. But what about the thousands of people who _don't_ have strong master
passwords? Their vaults are essentially ripe for the picking. Now, they do have
a secret key which is said to never leave your device and your encryption key is
derived from your master password _and_ your secret key. This is a great feature
that I don't want to put down. However, if you are a black hat, have 100,000
password vaults, 10% of which have weak passwords, and no secret keys, what do
you do? You start working on getting those keys. If a user is using a weak
master password, you can almost certainly assume their general security habits
aren't great. A targeted attack on any of these people would have a high chance
of success. 

Ok, enough about the security. What else is wrong with this approach? Well, for
a start, I now _must_ have a network connection in order to be able to add my
1Password vault to a machine. I can't keep it on a USB drive and use that on
each machine. If someone has an air-gapped machine, then it's no use. 

Next up is the fact that if I want to continue getting updates, some of which
might be critical for security, I need to upgrade, which means my vault is made
read-only. I might be left with a choice of being secure, and being able to use
my password manager. I can't speak for everyone, but it really feels like I'm
being left out in the cold on this one. 

The final thing is the worst. It's the fact that it really seems like AgileBits
just doesn't care about it's users any more. I would guess that a significant
number of people (myself included) use 1Password over competitors like LastPass
or DashLane _because_ it doesn't sync to a central server. This feels like a
money grab rather than something that has been done to make their users happier.
As I mentioned, I'm fine with a subscription for the licence, just please leave
me in control of my passwords. 


