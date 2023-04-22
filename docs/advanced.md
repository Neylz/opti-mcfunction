[← Back to the README](../README.md)

# Work in progress
### This file can contain errors or be incomplete.

### Table of contents
- [0. Introduction](#0-introduction)
- [1. Understanding the minecraft TPS and MSPT](#1-understanding-the-minecraft-tps-and-mspt)
  - [1.1. TPS](#11-tps)
  - [1.2. MSPT](#12-mspt)
- [2. Introduction to algorithmic complexity](#2-introduction-to-algorithmic-complexity)
  - [2.1. What is algorithmic complexity?](#21-what-is-algorithmic-complexity)
  - [2.2. In mcfunction](#22-in-mcfunction)
- [3. Binary search](#3-binary-search)
  - [3.1. What is binary search?](#31-what-is-binary-search)
  - [3.2. In mcfunction](#32-in-mcfunction)
- [References and sources](#references-and-sources)


# 0. Introduction

This document is a collection of more advanced methods and techniques that can be used to improve the performance of your datapacks. It is not meant to be a complete guide, but rather a collection of tips and tricks that can be used to improve your datapacks.

Unlike the [basics of optimization document](./basics.md), this document is not meant to be read from top to bottom. Each section is independent from the others, and you can read them in any order you want. That's why there is no indices to each section.

The goal is to gather more advanced methods, giving links and references to external sources. The methods can be mcfunction specific, or more general optimization methods of programming.

Also some thechnics here can only be described in a general way, as these technics are very situational and depend on the context.


[↑ Back to top](#)

# 1. Understanding the minecraft TPS and MSPT

A quick introduction to the TPS and MSPT has been done in the [basics of optimization document](./basics.md). But it is important to understand these concepts to be able to optimize your datapacks.

## 1.1. TPS

TPS refers to the number of game ticks that occur per second. In Minecraft, each game tick is a unit of time that measures 1/20th of a second, or 50 milliseconds. Ticks are used to measure the passage of time in the game and to update the game world.

The TPS rate is important because if it drops below 20 TPS, the game will start to feel laggy and unresponsive. This can happen if the game is overloaded with too many entities, such as mobs, items, and players, if two many functions are executed in the same time or if the game is generating new chunks of terrain.

To check the TPS rate in singleplayer, you can press `Alt + F3`. In additiion to the normal debug screen, you will see a two new graphs. The one on the right is displaying the TPS rate. In multiplayer, you can use mods like the [Carpet mod](https://github.com/gnembon/carpetmod) to check the TPS rate with the command `/log tps` and it will be displayed in the player list.

## 1.2. MSPT

MSPT stands for Milliseconds Per Tick. MSPT refers to the amount of time it takes for the game to process one tick. This metric is important because it gives an indication of how much time the server is spending on each tick, and whether it is taking too long to process.

A high MSPT value can indicate that the server is struggling to keep up with the demand placed on it, which can cause lag and other performance issues. On the other hand, a low MSPT value indicates that the server is processing ticks quickly and efficiently.

To check the MSPT value, you can use the command /debug start and then /debug stop to see the results in the server log. Ideally, the MSPT value should be below 50ms per tick. A value above 50ms per tick indicates that the server may be experiencing performance issues and should be optimized or upgraded if possible.

## 1.3. TPS and MSPT in datapacks

In summary, the MSPT is the time that the server takes to process one tick, so it is also dependent of the hardware configuration, and the TPS is the number of ticks that occur per second. The TPS is also dependent of the MSPT.

That's why we will focus on the MSPT to optimize our datapacks. If the MSPT is too high, the TPS will also be too low. But if the MSPT is low, the TPS can still be too low if the server is overloaded with too many entities. Having a TPS at 20 is great, but it is not enough to have a good experience. The MSPT should also be low. So it is important to allways keep an eye on the MSPT.



[↑ Back to top](#)

# 2. Introduction to algorithmic complexity

This section is a general introduction to algorithmic complexity. It is not meant to be a complete guide, but rather a collection of tips and tricks that can be used to improve your datapacks.

## 2.1. What is algorithmic complexity?

In general programming, there exists two types of complexity: time complexity and space complexity. The time complexity is the time it takes to execute a function, and the space complexity is the memory used by the function. In our case of optimizing minecraft datapacks, we will only talk about time complexity.

The time complexity of an algorithm is the amount of time it takes to run as a function of the size of the input. The time complexity is usually expressed in terms of the number of operations that the algorithm performs. The time complexity is usually expressed in terms of the number of operations that the algorithm performs.

For that we are defining the number of operations as `n`. Then, we use the big O notation to express the time complexity.

Big O notation is a way of measuring the efficiency of an algorithm or function in terms of how much time or space it requires to execute. It is used to describe the upper bound of the worst-case scenario of an algorithm's performance.

In other words, Big O notation gives us a way to compare the scalability and performance of algorithms in terms of how their running time or space requirements grow as the size of the input data increases.

The notation is written as `O(f(n))`, where `f(n)` represents a mathematical function that describes the upper bound of the algorithm's time or space complexity. For example, `O(1)` means that the algorithm's time or space complexity is constant, while `O(n)` means that the algorithm's time or space complexity grows linearly with the size of the input data.

**Some common examples of Big O notation include:**

* O(1): constant time complexity
* O(log n): logarithmic time complexity
* O(n): linear time complexity
* O(n log n): linearithmic time complexity
* O(n^2): quadratic time complexity
* O(2^n): exponential time complexity

By analyzing the Big O notation of an algorithm, we can determine its scalability and whether it will be efficient for larger inputs. We can also use it to optimize algorithms and make them more efficient by reducing their time or space complexity.

Here is a graphical representation of the Big O notation:

![Big O notation](https://live.staticflickr.com/65535/51957970066_dd5fd167f6_c.jpg)


To summarize, the big O notation is a way to measure the efficiency of an algorithm or function in terms of how much time or space it requires to execute. It is used to describe the upper bound of the worst-case scenario of an algorithm's performance.

## 2.2. In mcfunction

In minecraft, we will compute the time complexity with `n` as the number of commands executed for a part of our algorithm. For example, if we have a function that executes 10 commands, then `n` will be equal to 10.

Thus for this same algorithm, we will have a time complexity of `O(10)`, which is the same as `O(1)`.

For a recursive function which will call itself `n` times, we will have a time complexity of `O(n)`. Up to this point, the time complexity is the same as in general programming. But in mcfunction some loops are hidden, and we will have to take them into account. Indeed, the `execute` command can be used to execute a command multiple times. For example, the following function will execute the command `say hello` 10 times:
```hs
execute as @a run say hello
```
Yes, you read it right, the `execute as` is increasing our complexity by (in this case) the number of players. So we will have a time complexity of `O(n)`.

So if we have `execute as ... run function ...` and in the second function an other `execute as ... run ...`, exists we will have a time complexity of `O(n^2)`.

So it is mandatory to keep in mind that the `execute as ... run function ..` command ***gives its context to the executed function***. 

There is two major ways to avoid a double loop. The first one is to put a second `execute as ... run function ...` in the first function. The second one is to execute `as @s` in the subcommand. The first option gives a time complexity of `O(2n)`, which is the same as `O(n)` and the second option we will have a time complexity of `O(n)`.

Also, please note that the big O notation is not a precise measure of the time complexity. It is only an approximation giving a rough idea of the final time complexity. Especially in minecraft, as computing commands is more expensive than in a classical programming language, you can even see a difference between `n` and `10n` for exemple.

Big O notation should be used in order to structure your arlgorithm. It isn't because the complexity is `O(1)` because it has 1000 hardcoded commands that it is efficient. A `O(log n)` could be more optimized (see binary search 😉).



[↑ Back to top](#)


# 3. Binary search

## 3.1. What is binary search?

In general algorithmics, binary search is a popular algorithm used to search for a particular element in a sorted array or list. The algorithm works by repeatedly dividing the search interval in half until the target element is found or determined to not exist in the array.

Consider this following binary search tree:

![Binary search](https://courses.engr.illinois.edu/cs225/sp2023/assets/notes/bst/goodbst.png)

To find a value in the tree, we start at the root node. If the value we are looking for is less than the value of the root node, we search the left subtree. If the value we are looking for is greater than the value of the root node, we search the right subtree. If the value we are looking for is equal to the value of the root node, we have found the value.

Binary search has a time complexity of O(log n), which means that its performance increases logarithmically with the size of the input array. It is an efficient search algorithm for large sorted arrays.

## 3.2. In mcfunction

In mcfunction, the binary trees are generaly used with hardcoded data to answer to a scoreboard input. Generally, it avoids a function like this:
```hs
execute if score #input score matches 1 run function namespace:1
execute if score #input score matches 2 run function namespace:2
execute if score #input score matches 3 run function namespace:3
...
execute if score #input score matches x run function namespace:x
```

To do a binary tree, we will use a function for each node of the binary search tree. The function will have the following structure:
```hs
execute if score #input score matches x run function namespace:dound/<node>
execute if score #input score matches ..(x-1) run function namespace:node/<left>
execute if score #input score matches (x+1).. run function namespace:node/<right>
```
Where `x` is the value of the node, `namespace` is the namespace of the function and `node` is the name of the function.

This guide isn't about how to generate them with an external script; it would be to long to explain the logic for this format, so if you want to generate one without writing it by hand, you can use [this program made by CloudWolf](https://github.com/CloudWolfYT/CloudScripts/releases). Note that generating one with a custom script is more efficient because more adapted to what you need. 




[↑ Back to top](#)


# References and sources


[↑ Back to top](#)

[← Back to the README](../README.md)