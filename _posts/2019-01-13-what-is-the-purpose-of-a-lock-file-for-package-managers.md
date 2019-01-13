---
layout: single
title: What is the Purpose of a Lock File for Package Managers?
date: 2019-01-13 12:00:00
tags: [programming, ci, build]
---

Whatever language and package manager you use, be it Ruby Gems, CocoaPods, NPM, Cargo, etc. there's a good chance that if you have a file specifying your dependencies (such as `Gemfile`, `Podfile`, `package.json` or `Cargo.toml`), there's a corresponding `.lock` file. It's not always clear what the purpose of these files are, and whether or not they should be checked in to your repo.

I'm going to use CocoaPods as the example for this, but most package managers are the same and the same logic applies.

Here's an example `Podfile`:

```ruby
platform :ios, '10.0'

source 'https://github.com/CocoaPods/Specs.git'

target 'MyApp' do
  pod 'SwiftLint', '~> 0.29.1'
  pod 'OCMock', '~> 2.0.1'
end
```

This file basically says, let's install a version of SwiftLint that is at least 0.29.1 and up to, but not including 0.30.0. Similarly, it wants OCMock from at least 2.0.1 up to, but not including 2.1.0. Different tools use slightly different operators for these sorts of things, so make sure you are using the right one for your tool.

When we run `pod install`, CocoaPods analyses this file, finds the dependencies, figures out what versions it can possibly install, and then does so. By default, most package managers will take the latest possible version that meets the requirements specified. So if there was a version 0.29.7 of SwiftLint as the latest available one, that's what would be installed. OCMock is still on 2.0.1 so that's what you get.

You then go and create some project using these dependencies, check in your `Podfile` and everything looks good. Someone else on your team checks out the commit, and runs `pod install` and they end up with OCMock 2.0.1, however they get SwiftLint 0.29.8. That could be problematic. In theory, the two versions should be compatible, but despite everyone's best efforts, mistakes can still be made. How do you make sure that everyone on your team gets the same version as you? Well, that's where the lock file comes in.

When you run `pod install`, not only does it resolve the dependencies and install, it generates a `Podfile.lock`. This file contains the _exact_ versions of all the dependencies (and their dependencies) that were installed. If you run `pod install` and there is a `Podfile.lock`, then instead of resolving the dependencies and taking the latest possible, it instead looks at the versions in the lock file and install those. Now, if you check in your `Podfile.lock`, when your team mates run `pod install`, they get the exact same versions that you have.

So, it's clear where lock files help. But do you always need them? What if you pin your versions exactly. Instead of `pod 'SwiftLint', '~> 0.29.1'` you use `pod 'SwiftLint', '0.29.1'`. If that's the case, you will end up with exactly the same versions, right? That's true for the most part. If you specify the versions explicitly, then you will get the same versions. However, if you have 30 dependencies, it can be annoying to update each one manually in turn. Using [SemVer](https://semver.org) you can pin compatible versions of each tool, get the most benefits, and then, importantly, just run `pod update` and it will resolve the latest possible dependencies and install them, updating your lock file as it goes. So, you don't have to use a lock file, but it has its advantages.

One additional benefit of using lock files, even if you pin exact versions, depends on your exact tool. Lots of package registries let you change the version already in place. That means that some developer might upload version 1.2.3 of their tool, but realise they made a mistake. Then instead of pushing 1.2.4, they just "fix" 1.2.3. This means that you could have two different versions of 1.2.3. In theory, this shouldn't happen, but it does (and sometimes maliciously). Using a lock file lets you verify that the version you have is the same as the version others have since the hash of the dependency needs to match. But this depends on your tool.
