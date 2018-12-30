---
layout: single
title: "Python Devs: Asking for Forgiveness Is Not Better than Asking for Permission"
date: 2018-12-30 12:53:00
tags: [python, deserialize, programming]
---

I recently started working on a Python wrapper to an API which returns JSON. There were a few existing implementations already, but none behaved quite like I wanted, so I set out to write my own. One of the big decisions you make when writing a wrapper around an API in Python is how do you return the data? Let's say you have an API that returns:

```
{
    "name": "Hodor",
    "age": 42
}
```

A naive approach might just do something like:

```python
def get_data():
    data = requests.get("https://example.com")
    return data.json()
```

Then when you want to access the data, you do:

```python
name = data['name']
age = data['age']
```

This works, it's simple and it's easy. It also conforms to one of the unwritten rules of Python: It's better to ask forgiveness than for permission. i.e. Hope that the values are there, and deal with it if not, rather than checking that they are there first. 

Python isn't the only language I write day to day. It's shared between it, Swift, Objective-C and C#. One of the features that both Swift and C# have is the ability to deserialize JSON directly into objects. That lets you define an object which looks like the response you expect and it will parse directly into it. Here's a Swift example:

```swift
struct Person: Decodable {
    let name: String
    let age: Int
}

guard let person = try? JSONDecoder().decode(Person.self, from: responseData) else {
    print("Error: Couldn't decode data into Person")
    return
}

print(person.name)
print(person.number)
```

This pulls out the data, type checks it and places it into a newly created object for you. This implementation lets you handle missing keys by using optional types, you can specify alternative identifier mappings if you want your property to be named differently than the API key, etc. You can be sure that whatever comes back from the API definitely conforms to `Person`. C# has a very similar ability using the [Json.NET](https://www.newtonsoft.com/json) library.

So, how did I want to return the data from my library? Well, I decided that if I could parse it into an object, perform the validation, type checking, etc. this would result in a safer library for users to consume. There would be less surprises in store, and it would be easier to use. So I started writing out my Python code to handle this:

```python
class Person:

    def __init__(self, name, age):
        self.name = name
        self.age = age

    @staticmethod
    def from_json(json_data):
        name = json_data.get('name')
        age = json_data.get('age')

        if name is None:
            raise Exception('No "name" was found in the data')

        if age is None:
            raise Exception('No "age" was found in the data')

        if not isinstance(name, str):
            raise Exception('"name" was not a string')

        if not isinstance(age, int):
            raise Exception('"age" was not an int')

        return Person(name, age)

person = Person.from_json(json_data)
```

Phew... that's a lot of boilerplate, but it gets the job done (lets not get into namedtuples, dataclasses, etc.). It checks that the values are there. It checks that they are the correct type. The end result is a nice, safe and clean type that the user can use without any surprises.

The problem is that I am explicitly checking, rather than letting the user handle it if it goes wrong, and this goes against the rule I mentioned above. However, clearly when writing an API wrapper, the burden of validating the responses is on the wrapper and not the end user. This clearly shows that the permission vs forgiveness rule is not right in all cases[^1]. This is just one case though, there are many others. 

So, Python devs, before you repeat the line about permission and forgiveness, pause for a moment and actually think about it. Is that actually what's best, or is it just something you've believed without knowing why? 

#### A better solution for the above

The code above is ridiculously verbose and it's just checking two properties. I was dealing with a lot more. The validation code got insane. That doesn't even include the ones where I wanted to do things like convert a Unix timestamp to a `datetime.datetime`, etc. Inspired by the Swift and C# solutions, I went and created [deserialize](https://github.com/dalemyers/deserialize)[^2] which takes advantage of type hints. By using this, the solution above can be condensed to:

```python
import deserialize

class Person:
    name: str
    age: int

person = deserialize.deserialize(Person, json_data)
```

That does all the same checks as above, but is obviously much easier to read and maintain. It also lets me continue working on the wrapper without feeling like my soul is being sucked out as I validate the 400th value. 


[^1]: Even the creator of Python thinks that it is a bad rule: <https://mail.python.org/pipermail/python-dev/2014-March/133118.html>

[^2]: I initially looked for an existing library that did the same thing, but couldn't find one. Sure enough though, as soon as I had something that did what I needed, I found what I was looking for in the first place: <https://git.iapc.utwente.nl/rkleef/serializer_utils> Don't just blindly go with my implementation. Have a look at both solutions and use the correct one for you. 
