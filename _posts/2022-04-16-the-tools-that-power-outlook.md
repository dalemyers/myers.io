---
layout: single
title: The tools that power Outlook
date: 2022-04-16 18:21:00
tags: [outlook, ios, python]
---

I've been responsible for the developer tooling for Outlook iOS for 6 years now. Back when I started, we had just a single Bash script that covered everything we though we needed at the time. Now, we have 30,000+ lines of Python code (including tests I have to admit) that depend _directly_ on 50 Python packages. Most of these packages are ones you might expect such as `requests`, `pylint`, or `black`. However, 13 are our own and shared with others. Of those 13, 9 are open source and available for the general community. A further 3 out of the 50 packages we use are my own creation from outside work. While some are personal, and some were created at work, all were created by me.

Without these packages, Outlook iOS wouldn't be where it is today. Personally, I think these various tools were paramount to allowing developers to focus on what really matters: Developing the app. Knowing that these tools were available and could handle the various day to day issues removes a massive burden and improves the results we see.

These various tools have been so successful that many other teams at Microsoft contact me asking to use them. I have to admit that it gives me the greatest pleasure when I can point out that not only can they use them, but they are open-source so _anyone_ can use them. 

Here is a brief description of each of these tools that I created and how you can use them for your app/library/etc. (Note: order does not imply importance)

## Foundations

### 1. deserialize

First up is a personal creation, [deserialize](https://pypi.org/project/deserialize/). This library takes a dictionary or list and a type and creates an instance of that type using the data supplied. 

For example, if you want to convert this data:

```python
{"a": 1, "b": 2}
```

Into an object with `a` and `b` as properties, you'd have to do something like this:

```python
class MyThing:

    def __init__(self, a, b):
        self.a = a
        self.b = b

    @staticmethod
    def from_json(json_data):
        a_value = json_data.get("a")
        b_value = json_data.get("b")

        if a_value is None:
            raise Exception("'a' was None")
        elif b_value is None:
            raise Exception("'b' was None")
        elif type(a_value) != int:
            raise Exception("'a' was not an int")
        elif type(b_value) != int:
            raise Exception("'b' was not an int")

        return MyThing(a_value, b_value)

my_instance = MyThing.from_json(json_data)
```

With `deserialize`, all you need to do is:

```python
import deserialize

class MyThing:
    a: int
    b: int

my_instance = deserialize.deserialize(MyThing, json_data)
```

`deserialize` will run all the checks for you and give you a nice new shiny object from it. It of course works to any depth of types, and not just primitives. 

The reason this comes up as #1 is because it a foundational building block of so many other packages in here. If I ever consume from a REST API, load data from disk, or even query a database, you can be sure I'll have at the very least considered using this package to make it easy and error free. 

### 2. protool

[protool](https://pypi.org/project/protool/) removes all the pain from dealing with provisioning profiles. Instead of being mysterious binary files, `protool` makes them easy to use, understand, and work with. Some examples of what it can do:

* Easily diff between two profiles using `protool diff --profiles /path/to/profile1 /path/to/profile2`
* Get a property from a profile: `protool read --profile /path/to/profile --key UUID`
* See the raw XML without having to memorise the obscure parameters for the `security` command: `protool decode --profile /path/to/profile`

These commands are actually based around the full Python API it provides. Some examples:

```python
import protool
profile = protool.ProvisioningProfile("/path/to/profile")

# Get the diff of two profiles
diff = protool.diff("/path/to/first", "/path/to/second", tool_override="diff")

# Get the UUID of a profile
print profile.uuid

# Get the full XML of the profile
print profile.xml

# Get the parsed contents of the profile as a dictionary
print profile.contents()
```

Personally, the start feature of protool is as a diff driver for git. Normally if you change profiles you see "Binary files differ" from git. With `protool` you can edit your git config (at any level) and add:

```
[diff "mobileprovision"]
    external = protool gitdiff -g
```

This will let you see the differences in XML format. However, that on its own isn't particularly helpful. You could just have easily used `security cms -D -i` on in the config and it would do the same thing. The real power is in being able to ignore keys. For example:

```
[diff "mobileprovision"]
    external = protool gitdiff -i TimeToLive UUID -g
```

This will ignore the time to live value, as well as the UUID in the diff. You know those will be different between any two profiles, so why bother cluttering your diff with them?

### 3. dotstrings

Dealing with localization can be tough, but [dotstrings](https://pypi.org/project/dotstrings/) makes it just that little bit easier. 

This tiny tool does one thing and one thing only: It reads your `.strings` files. Here's the entirety of what it does:

```python
import dotstrings

entries = dotstrings.load("/path/to/file.strings")

for entry in entries:
    print("Key: " + entry.key)
    print("Value: " + entry.value)
    print("Comments: " + "\n".join(entry.comments))
```

Why is that useful you ask? Well, it allows you to test your strings easily! We use it directly for a bunch of checks, but you'll see later how we integrate it with another tool for even better testing.

### 4. xcodeproj

One of the most annoying and difficult things to comprehend as an Apple developer is the Xcode project format. Testing it to ensure that developers haven't accidentally broken anything, or moved files where they shouldn't be, etc. can be a real nightmare. Especially when coupled with the fact that the `pbxproj` format is inscrutable to most. This is where [xcodeproj](https://pypi.org/project/xcodeproj/) comes in. It aims to solve all of those woes. By simply running:

```python
import xcodeproj
project = xcodeproj.XcodeProject("/path/to/project.xcodeproj")
```

you now have a nice, easy to understand, simple to use, project object which you can test directly. 

Let's look at a trivial example where you are sick of seeing Xcode have those files highlighted in red because they exist in the project but no longer exist on disk. How would you make sure no one is accidentally committing changes with that? Easy!

```python
import xcodeproj

project = xcodeproj.XcodeProject("/path/to/project.xcodeproj")

for item in project.fetch_type(xcodeproj.PBXFileReference).values():
    assert os.path.exists(item.absolute_path())
```

This library makes Xcode projects something which can be part of your code reviews and no longer some mysterious black box where people automatically approve changes to pbxproj files. 


### 5. xcresult

Another personal creation here. [xcresult](https://pypi.org/project/xcresult/) does exactly what it sounds like. It lets you work with xcresult bundles. When you buiild, run tests, etc. Xcode will generate an xcresult bundle with the, you guessed it, results of the operation in there. Reading it though to get the data out is a whole different story. 

For example, let's say you run snapshot tests and one is failing. You know there are two images in there somewhere, how do you get them out? There's absolutely no hint in the logs. Thankfully, it's relatively easy:

```python
results_bundle = xcresult.Xcresults(results_bundle_path)
attachments_path = "/some/output/folder"
os.makedirs(attachments_path, exist_ok=True)
results_bundle.export_attachments(attachments_path)
```

Now all the images, etc. that are in this bundle are available as PNG images. These can then be easily surfaced to what ever CI system you are using so that developers can easily see exactly what went wrong. For example, if you use Azure DevOps, you might see something like this attached to your build:

![Example of snapshot differences in the ADO UI](/images/posts/2022-04-16-xcresult-example.png)

## Testing

### 6. isim

Dealing with simulators can be tricky at the best of times. So many questions around things like "Do you wipe them after each test run?", "If so, how?", "How do I create a simulator for a test for a particular device?", etc. [isim](https://pypi.org/project/isim/), and you might be seeing a pattern here, tries to make that as simple as possible. 

Many of you reading this will be familiar with the `xcrun simctl` command. If you are working in a system where Bash works for you, then you don't need to read any further. If you are a Python shop, then isim will be a life saver. It's essentially a wrapper around that command to make it as easy to use as possible, while being easy to use if you are already familiar with the command. 

For example, `xcrun simctl list runtimes` becomes `isim.Runtime.list_all()`. And in general, `xcrun simctl do_thing [DEVICE_ID] arg1 arg2` becomes: 

```python
device = isim.Device.from_identifier(DEVICE_ID)
device.do_thing(arg1, arg2)
```

If your CI is Python based and you aren't using isim, then either you are making life harder for yourself, or you have a fantastic solution of your own I'd love to know about!

## Localization

### 7. localizationkit

Localization is incredibly difficult. In the best case scenario, you write some strings, send them off to translators, get them back and ship them. But what if there was a mistake? What if you sent the string `Hello %@!` where you'd replace `%@` with the persons name, but your French translators send back `Bonjour!` with no token? Well, at runtime, your app is going to crash. Ok, sure, it's unlikely that this would happen, but what if you have 2000 strings in your app? Then it's 2000 times more likely to happen? What if you support 70 languages? Then it's 140,000 times more likely to happen! At that scale, mistakes happen. How do you catch them? With [localizationkit](https://pypi.org/project/localizationkit/). This tool is a suite of tests to ensure that your localized strings are the best that they can be. What sorts of things can it check for?

* Checking that all strings have comments
* Checking that the comments don't just match the value
* Check that tokens have position specifiers (e.g. `Hello %1$@, the weather is %2$@` instead of `Hello %@, the weather is %@`)
* Check that no invalid tokens are included (e.g. no accidental instances of `The stocks went up to 100 %*`)

This tool alone has saved us countless times from runtime crashes. 

I mentioned above, that `dotstrings` integrates with other tools. This is one example. `localizationkit` is platform agnostic. It takes in a string "collection" where each string consists of a key, value and comment. Combining the two to test is trivial:

```python
bundle = dotstrings.load_all_strings("/path/to/table.strings")
strings = [localizationkit.LocalizedString(string.key, string.value, string.comment, "en-GB") for string in bundle]

collection = localizationkit.LocalizedCollection(strings)
results = localizationkit.run_tests(config, collection) # `config` lets you set various parameters

failures = [result for result in results if not result.succeeded()]

assert len(failures) == 0, f"Encountered failures: {failures}"
```

### 8. LocalizedStringKit

I know, it's a super similar name to the previous entry, but I wasn't responsible for the naming scheme, just the code! Out of all of the examples I have here, this is the only one which is a derivative of some earlier work. This work was done by one (or more) of the engineers at Acompli and continues to this day, just in a significantly different form. 

[LocalizedStringKit](https://pypi.org/project/localizedstringkit/) is unique in this list as it's not only a Python program. It has a Swift/Objective-C counterpart too: <https://swiftpackageindex.com/microsoft/LocalizedStringKit> This tool makes it easier than ever for developers to localize their apps without even needing to think about it!

Normally, the flow to localized a string goes something like this:

1. Come up with some new string: `label.text = "Your account was successfully added!"`
2. Come up with some "key" for the string: "ACCOUNT_SUCCESSFULLY_ADDED"
3. Pray no one has used that key already for some similar string.
4. Add this entry into your English .strings file: `"ACCOUNT_SUCCESSFULLY_ADDED" = "Your account was successfully added!"
5. Open your PR.
6. Realise you forgot to add a comment. 
7. Add your comment and update your PR.
8. Merge your PR.
9. Find out that while no one was using your key before, they are now and you've got a conflict and weird things are happening. 

You get my point. 

With LocalizedStringKit, you do this:

1. Create your string and comment: `label.text = LocalizedString("Your account was successfully added", "Shown to the user in an alert when they've added an account to the app, letting them know everything was successful")`
2. Run `localizedstringkit --path /path/to/my/project/root --localized-string-kit-path /path/to/my/project/root/LocalizedStringKit` (which you are obviously going to provide a wrapper/alias for which is easy to remember)

That's it. You can add a check in your CI to ensure no one forgets to run the generation script either. 

It works by taking a hash of the English string as the key, which is therefore deterministic. Developers lives are significantly simpler and less error prone now. 

## System

### 9. keyper

Interacting with the system keychain from the command line can be a nightmare at best. We all have to do it to install certificates, secrets, etc. and it never gets any easier. So let's bypass the CLI entirely and use [keyper](https://pypi.org/project/keyper/)` in Python instead. 

Getting a password is as simple as `password = keyper.get_password(label="my_keychain_password")`

Installing a certificate is just 3 lines of code:

```python
with keyper.TemporaryKeychain() as keychain:
    certificate = keyper.Certificate("/path/to/cert", password="password")
    keychain.install_cert(certificate)
```

Of course, you can install to the system keychain, you just need to make sure it is unlocked first. 

For this tool, if you are handling certificates or passwords, there is simply no easier way to get them into the keychain.

## REST API Wrappers

Our first two here are Microsoft stack specific, so if you use something else, feel free to skip. 

### 10. appcenter

There's no point in dressing this one up. [appcenter](https://pypi.org/project/appcenter/) is a Python wrapper around the App Center APIs. There is an Open API verison, but we found that the code it generated was difficult to understand and use. `appcenter` was born from that. Here are some examples of how it works:

```python
# 1. Import the library
import appcenter

# 2. Create a new client
client = appcenter.AppCenterClient(access_token="abc123def456")

# 3. Check some error groups
start = datetime.datetime.now() - datetime.timedelta(days=10)
for group in client.crashes.get_error_groups(owner_name="owner", app_name="myapp", start_time=start):
    print(group.errorGroupId)
    
# 4. Get recent versions
for version in client.versions.all(owner_name="owner", app_name="myapp"):
    print(version)
    
# 5. Create a new release
client.versions.upload_and_release(
    owner_name="owner",
    app_name="myapp",
    version="0.1",
    build_number="123",
    binary_path="/path/to/some.ipa",
    group_id="12345678-abcd-9012-efgh-345678901234",
    release_notes="These are some release notes",
    branch_name="test_branch",
    commit_hash="1234567890123456789012345678901234567890",
    commit_message="This is a commit message"
)
```

What more is there to say? If you use AppCenter, this library will be a life saver. 

### 11. simple_ado

Just like above, there is a Python wrapper around the Azure DevOps (ADO) APIs, but it is difficult to understand and use, and makes reading code reviews significantly more complex as it is difficult to understand intended behavior. Enter [simple_ado](https://pypi.org/project/simple-ado/). The ADO APIs are expansive and `simple_ado` can't possibly cover them all (the clue is in the name: `simple`), but it covers the majority of what you would ever need as an iOS/macOS developer. You can manage builds, pull requests, work items, commits, teams, identities, security, plus a ton of other things. 

Taking hold of your CI is of the utmost importance for any team. If you use Azure DevOps, this is _the_ tool you want to use. Point me at a different CI and I'm going to create the same thing again for it. 

### 12. asconnect

There's something so satisfying about saving the best for last. There isn't an iOS developer out there who hasn't heard of [Fastlane](https://fastlane.tools/). If you want to automate your release process, Fastlane is _the_ tool to use. Unless you aren't Ruby devs... At which point where do you turn? A few years back, Apple announced they were opening up the App Store Connect APIs. The capabilities don't yet match what Fastlane is capable of (which uses web scraping if an API isn't available), but it covers the majority of cases that any developer would care about. 

With [asconnect](https://pypi.org/project/asconnect/), you can easily:

* Upload builds
* Create new TestFlight versions
* Set review information
* Submit for review
* Set new screenshots and information

Plus a bunch of other things. Outlook switched from Fastlane to asconnect almost 2 years ago and has never looked back. No more issues dealing with Fastlane not working because Apple changed a page layout. The APIs work. Every. Single. Time. 

As an example of how easy it is to use, let's look at uploading a build and creating a new app store submission:

```python
import asconnect

client = asconnect.Client(key_id="...", key_contents="...", issuer_id="...")

# Upload the build
client.build.upload(
  ipa_path="/path/to/the/app.ipa",
  platform=asconnect.Platform.ios,
)

# Wait for it to finish processing
build = client.build.wait_for_build_to_process("com.example.my_bundle_id", build_number)

# Create a new version
version = client.app.create_new_version(version="1.2.3", app_id=app.identifier)

# Set the build for that version
client.version.set_build(version_id=version.identifier, build_id=build.identifier)

# Submit for review
client.version.submit_for_review(version_id=version.identifier)
```

It's as simple as that. You will no longer have to have someone do these steps manually every week if you aren't already using a similar tool. And if you are using FastLane, while a phenomenal tool that asconnect can never hope to compete with, you won't have to worry about it breaking because Apple made some changes to a random web page. 
