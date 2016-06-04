---
layout: post
title:  Keyword Arguments
date:   2016-03-03 12:00:01 -0400
---

Today let’s take a quick look at how Keyword Arguments can help us avoid errors. 

Ruby methods are flexible, able to be defined with no arguments or any number of arguments.  For example, a #birthday_greeting method taking zero, one, and two arguments might look as follows:

```
def birthday_greeting_zero
  puts "Happy Birthday!"
end

def birthday_greeting_one(name)
  puts "Happy Birthday #{name}!"
end

def birthday_greeting_two(name,age)
  puts "Happy Birthday #{name} - you are #{age} years old!"
end
```
With #birthday_greeting_none and #birthday_greeting_one, the user calling the method either inserts zero or one argument. There is little room for error. In #birthday_greeting_two, the door for mistakes is now open! Let’s see what happens when the method is called with:

(i) the arguments in the proper order –

```
birthday_greeting_two("Ruby", 20)
  #=> Happy Birthday Ruby - you are 20 years old!
```

and (ii) the arguments in the improper order –

```
birthday_greeting_two(20, "Ruby")
  #=> Happy Birthday 20 - you are Ruby years old!
```

Keyword Arguments alleviate this problem by adding a colon “:” to the end of the arguments, transforming it into a key-value pair of a hash. The method body can now identify the proper value by reference to the key – regardless of the argument order.

Our #birthday_greeting_two can be refactored with Keyword Arguments, as shown below.  At run time, the method will produce the expected output regardless of the argument order.

```
def birthday_greeting_two(name:, age:)
  puts "Happy Birthday #{name} - you are #{age} years old!"
end
```

```
birthday_greeting_two(age: 20, name: "Ruby") #<= Order doesn't matter!
  #=> Happy Birthday Ruby - you are 20 years old!
```