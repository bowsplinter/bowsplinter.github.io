---
layout: post
title: "Debugging in python using print statements"
---

Note: This is a subset of some resources I prepared as a Teaching Assistant for SUTD's Digital World Students

## Debugging
This is the most powerful skill in programming that is not being taught in class. Debugging is the art of understanding what your program is doing vs what you expect the program to do. And using print statements allows you to quickly and conveniently check your code.

Here are some guidelines to follow when you test your code:

1. Think of what you expect the variable you are going to be printing to be before you print it. This forces you to think about your program and its goals.

2. Treat your code with suspicion. Assume every line is wrong unless proven otherwise. This is a good way to start, by challenging any assumptions that you may have about the code. As you grow more comfortable with programming, you will be better able to make educated guesses about where your code can go wrong.

Looking at an (incorrect) function to return the sum of numbers from 0 to n:

```
# adds sum of 0 to n
def sum(n):
    total = 0;
    for i in range(n):
        total += i
        return total

print(5)
# prints
# 0 
``` 

We see that the result returned is `0`, which is obviously not the correct sum. Systematically, lets try to check if each line of code works as we expect. To do that we insert print statements to see if the value of the variables at a particular point in the program is what we expect them to be.

```
# adds sum of 0 to n
def sum(n):
    total = 0;
    print "total: " + str(total) 
    for i in range(n):
        total += i
        return total

print sum(5) 
# prints 
# total: 0
# 0
``` 
We expect total to be `0` after the first line, and indeed this is the answer we get. 

```
# adds sum of 0 to n
def sum(n):
    total = 0;
    for i in range(n):
        print "i: " + str(i) # expect 0,1,2,3,4,5  (we are calling sum(5))
        total += i
        return total

print sum(5) 
# prints 
# i: 0
# 0
``` 

This is unexpected behaviour, it only prints `i: 0`. It is up to you to figure out why your program is executing differently from expected. But we have effectively narrowed down our field of search. Since variables up till the for loop have expected values, it has to be after the for loop. In this case, it's the `return` which is inside the for loop. Let's fix that.

```
# adds sum of 0 to n
def sum(n):
    total = 0;
    for i in range(n):
        print "i: " + str(i) # expect 0,1,2,3,4,5 (we are calling sum(5))
        total += i
    return total

print sum(5) 
# prints 
# i: 0
# i: 1
# i: 2
# i: 3
# i: 4
# 10
``` 

Again this is unexpected as we expect the return value to be `15`, and the for loop to output `i: 5`. We can guess that the error has to be something to do with the values `i` takes. `i` is created by using `range(n)` and thus, it would be wise to check it.

```
print range(5) # expect: [0,1,2,3,4,5]
# prints
# [0,1,2,3,4]
```

Reading the [range](https://docs.python.org/2/library/functions.html#range) docs, the stop input is exclusive so it stops at 4. If we fix it, it now works as expected.

```
# adds sum of 0 to n
def sum(n):
    total = 0;
    for i in range(n+1):
        total += i
    return total

print sum(5) 
# prints 
# 15
``` 

This seems long-winded and a slow way of debugging your code, but when you are starting out it can be really helpful. As you grow more comfortable, you will naturally get a hang of figuring out which sections of code you need to test and will be able to debug with speed and efficiency.