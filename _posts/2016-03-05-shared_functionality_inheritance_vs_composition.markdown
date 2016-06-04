---
layout: post
title:  Shared Functionality - Inheritance vs Composition
date:   2016-03-05 12:00:00 -0400
---

As we build out programs in Ruby with multiple classes, we learn that shared functionality between classes can be streamlined through Inheritance and Composition (i.e., Modules/Mixins). They effectively accomplish the same task by allowing us to write code in one place and use it in another place, staying true to the DRY (Don’t Repeat Yourself!) principle. 

*But when should we use one vs the other?*

**“is-a” vs “has-a”/”uses-a”**

Although the safest answer is “it depends” (one can never pretend to know everything!) we can try to translate the relationship into a logical english sentence using “is-a” or “has-a”/”uses-a” to guide our use of Inheritance and Composition, respectively.

Let’s use a simple example using two classes (Class Vehicle and Class Car) and one method (#repair – which, if called, would send an instance for regular maintenance, because why not).

*Inheritance = “is-a”.* Testing our relationship between classes, what sounds more logical: (a) Car “is-a” Vehicle; or (b) Car “has-a”/”uses-a” Vehicle?  The “is-a” relationship appears more logical and we should use the subclass and superclass relationship of Inheritance.

*Composition = “has-a”/”uses-a”.* Testing our relationship between classes and method, what sounds more logical: (a) Car/Vehicle “is-a” #repair; or (b) Car/Vehicle “has-a” #repair?  The “has-a” relationship works better in this situation and we should use the modules/mixins relationship of Composition.