---
title: How do I know if it is an NSObject?
date: 2016-08-18 08:40:00
tags: [Objective-C, Programming, iOS]
---

While I spend most of my time trying to write Swift code these days, there is at
least one thing Swift can't do: Interop with C++. For most of you, that isn't a
problem and you don't need to worry about it. Unfortunately for me, that is my
day job. I work in integrating a C++ library into an app that is mainly Swift
code, and therefore spend a lot of time writing Objective-C++ code. 

Yesterday I was looking at a delegate which returned an item from a cache. Going
into the cache it had to conform to the `NSCopying` protocol, but coming out, I
could say that the type is going to be `id<NSCopying>` but Swift was just
treating this as `AnyObject?` so I thought why not just leave it as `id`? Then,
in a particular use of this method, I knew that the data I returned was going to
be a `UIImage`. Actually, I knew that it should be a `UIImage`. In order to
avoid bugs down the road, I checked the type first:

```objc
id myObject = [myCache getItem:@"someKey"];
if (![myObject isKindOfClass:[UIImage class]]) 
{
    NSLog(@"Things went wrong!");
}
```

Now, if it's not a `UIImage` we can detect it. Right? Well, no. 

One of the things about Objective-C that isn't obvious to everyone is the
various differences between `id`, `id<NSObject>` and `NSObject`. As it turns
out, `NSObject` is the root class of most Objective-C classes. _Most_. Classes
don't have to inherit from `NSObject` and are free to choose what they like to
inherit from. The most common example is `NSProxy`. 

OK, so if it's an object which inherits from `NSProxy` then the above check
won't work? Actually, it will. While `NSProxy` isn't a subclass of `NSObject`,
it does implement the `NSObject`
[protocol](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Protocols/NSObject_Protocol/).
This protocol actually declares that an object implementing it must have the
methods we would expect to find when trying to check the type of the object,
such as `isKindOfClass:`, `isMemberOfClass:`, `respondsToSelector:` and
`conformsToProtocol:`. 

As far as I am aware, there are no other root classes available in Foundation so
we are going to have to go somewhere else for one. That is a little beyond the
scope of this blog post, so if you don't already know how this works, you
basically have to do it all manually. Allocating memory, clearing it up, the
whole thing. Let's not do that and imagine we have our own root class and
various other classes can inherit from it. 

Back to that variable `myObject`, what happens if it is not something which
implements the `NSObject` protocol? Using `isKindOfClass:` isn't going to work as
the selector won't be recognised (and that's if you can even get it to compile). 
Neither will any of the other methods defined in the NSObject protocol. We need
a way to check what the class is without relying on these methods. 

Here is where the Objective-C runtime comes to the rescue! In the runtime there
is a method named `object_getClassName()`. This takes the instance of a class
(anything with type `id`) and returns the name of the class as a `const char*`.
While it makes checking inheritance a little awkward, we can definitely use it
on unknown types to make our check look something more like:

```objc
id myObject = [myCache getItem:@"someKey"];
const char* objectName = object_getClassName(myObject);
if (strcmp(objectName, "MyRootClass") != 0)
{
    NSLog(@"Things went wrong"!);
}
```

On a final note, please don't use custom base classes. You basically don't ever
have a good reason. I'm not saying that good reasons don't exist, I'm just
saying yours almost certainly isn't one of them.


