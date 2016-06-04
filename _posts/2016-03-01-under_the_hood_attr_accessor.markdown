---
layout: post
title:  Under the Hood - attr_accessor
date:   2016-03-01 12:00:00 -0400
---

Welcome to a post where I look Under the Hood of Ruby! One of the beauties of Ruby is its elegance.  When you find yourself writing multiple or repetitive lines of code, Ruby often has a pre-existing method that encapsulates your intent.  In this post, my goal is to walk-through what is going on Under the Hood of helpful Ruby syntax.

Let’s put the spotlight on attr_accessor, which efficiently creates attribute setter and getter methods.

In object-oriented programming (OOP), it is common to create a Class and instances of that Class with attributes.  For example, if we had a Dog class, we might make multiple instances (e.g., Beethoven is a Dog, Snoopy is a Dog) each with its own attributes (e.g., Beethoven has a name, Snoopy has a name).  The Class attributes can be created in Ruby using “setter” and “getter” methods, as follows:

```
def name=(name)
  @name = name
end   #<= our setter, setting name to Snoopy
```
```
def name
  @name
end   #<= our getter, allowing access to name Snoopy
```

This is fine if your dog only has a name. But, that’s not realistic! In programming we mirror the real world, and our dogs should have way more attributes – breed, hair-color, gender, etc. This is where attr_accessor comes to the rescue.  With simple comma-separated syntax, it provides the setter and getter for the attributes, as follows:

```
attr_accessor :name, :breed, :hair-color, :gender 
#^=encapsulates setter and getter
```