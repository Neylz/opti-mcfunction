[← Back to the README](../README.md)

### Table of contents

- [0. Preface](#0-preface)
- [1. Introduction](#1-introduction)
    * [What should we optimize ?](#what-should-we-optimize-)
    * [How optimize by yourself ?](#how-optimize-by-yourself-)
- [2. The mcfunction programming language](#2-the-mcfunction-programming-language)
    * [2.1. Selectors](#21-selectors)
        + [2.1.1. Which target selector choosing ?](#211-which-target-selector-choosing-)
        + [2.1.2. Selectors' arguments efficiency](#212-selectors-arguments-efficiency)
        + [2.1.3. In what order to put the arguments?](#213-in-what-order-to-put-the-arguments)
    * [2.2. `execute if` selection](#22-execute-if-selection)
    * [2.3. `as @e[scores={}]` or `as @e if score @s`](#23-as-escores-or-as-e-if-score-s)
    * [2.4. Tagging instead of testing NBT](#24-tagging-instead-of-testing-nbt)
    * [2.5. Predicates](#25-predicates)
- [3. Datapack Structure](#3-datapack-structure)
    * [3.1. Running functions only when needed](#31-running-functions-only-when-needed)
        + [3.1.1. Tick functions](#311-tick-functions)
        + [3.1.2. Schedule functions](#312-schedule-functions)
    * [3.2. Advancements and Events](#32-advancements-and-events)
    * [3.3. Functions and Structure](#33-functions-and-structure)
- [4. Summary and Important Points](#4-summary-and-important-points)
- [5. References and sources](#5-references-and-sources)





# 0. Preface


Here is the big question: How can we optimize our code in datapacks? This crucial question has the potential to transform the outcome of datapacks causing the game to crash into improved performance content.


I realized the problem of the low documentation of optimizing datapacks when someone came into the Minecraft Commands discord with a datapack that was crashing the game because of its low optimization.


So here I will talk only about basic optimization.


This gist has the goal of being updated in the future with your feedback to be improved. So, feel free to discuss it on MCC discord!


# 1. Introduction


There are different levels of optimization, but like I said above, **I will not talk here about maths optimization**.
I will divide this gist into two principal sections :
* The mcfunction language
* Datapack Structure


### What should we optimize ?

Optimization is important because it allows the creation of larger programs without negatively impacting the user experience. The goal is not to strive for perfection, as this can hinder productivity. Instead, The goal is to be realistic about the circumstances and choose the best option. Different levels of optimization provide multiple options to choose from.

In this gist, you will find some key points that will provide you with more options.

### How optimize by yourself ?

This gist provides an overview of how the mcfunction language works, but sometimes these guidelines may not be sufficient and you may need to conduct your own tests. There are several tools available for this purpose. One such tool is Alt + F3, which displays the evolution of your TPS (ticks per second) and MSPT (milliseconds per tick).

MSPT is the most important value, as it represents the time it takes the game to calculate a single tick. A lower MSPT value is better. It is more important than TPS, as TPS is capped at 20 while MSPT is not.

You can also use F3 + L in single player mode to profile the performance of your code. This will allow you to analyze the results with the misode's [report inspector](https://misode.github.io/report/). On server, use the command `/profile` but the results will be not as complete as in singleplayer.



# 2. The mcfunction programming language


In this section, you can find optimization using a language approach. In other terms, there are some structures to absolutely avoid in your code.



## 2.1. Selectors


The selectors are essential in mcfunction. There are many reasons and justifications for why they are so important, but did you know that they can have a performance impact depending on how you use them?


### 2.1.1. Which target selector choosing ?


`@e` is the generic selector, it selects all entities. To this selector you can add target selectors (I will not handle all of them; you can find them all [here on the wiki](https://minecraft.fandom.com/wiki/Target_selectors)). So to recap, `@e[type=minecraft:player]` will select only players.

We can find properties of `@e` in other selectors, but they still have their own properties:

>- `@a`, equivalent of `@e[type=minecraft:player]` but is faster and can points to dead players where `@e` can't
>- `@p`, equivalent of `@a[sort=nearest,limit=1]` or `@e[type=minecraft:player,sort=nearest,limit=1]` but faster
>- `@r`, equivalent of `@a[sort=random,limit=1]` or `@e[type=minecraft:player,sort=random,limit=1]`
>- `@s` is the exception - even if simmilar to @p - it only detects the source. If the source isn't an entity (like a command block or the console), no entity is selected.



### 2.1.2. Selectors' arguments efficiency


Let's go back to optimization; the first important point is to select an entity **only when you need it**. Let me develop:


Selectors can have different impacts on performance depending on their nature, so it's really important to choose them according to the circumstances. Here is a list of selectors' arguments organized by categories and their impact in order to refine our selection and select only the pertinent entities.



| Category | Selectors Arguments | Direct Performance Impact (compared to `@e`) | Comment | Global Impact description when used |
|--|--|--|--|--|
| Selection | `limit` | negligeable | / | improves performances |
| Selection | `sort` | low | `sort=arbitrary` is implicit in almost all commands | improves performances |
| Entity type | `type` | negligeable | Should be used each time you can sepcify it (*cf. **2.1.3.***) | improves performances |
| Position | `x`, `y`, `z`, `dx`, `dy`, `dz`, `distance` | negligeable | Should be used on large sets of entities | improves a bit performances |
| Scoreboard values | `tag`, `team`, `scores` | low - medium | To the most efficient to the less one, they are good selectors agrument to refine your selection (*cf. **2.1.3.***), you should use them when possible | improves performances |
| Predicate | `predicate` | depends of what's inside | preferable to `NBT/Player Data` but to avoid | decreases performances |
| NBT/Player Data | `advancement`, `name`, `nbt` | strong | to avoid (use a string parser, which is very heavy) | decreases performances |


The last column indicates if it is worth to use or not, the argument to refine the selection. IE: Does the performance cost of this argument preferable to the one involved in executing the following command?

So, in the list above, you have the majority of arguments available, sorted from the slightest ones to the heaviest ones. `Selection`, `Position` and `Rotation` categories don't have a significant impact.


`Entity type` should be — as far as possible — always present. It has been shown that the game always checks the mob type with the selectors [[link to MCC's discord]](https://discord.com/channels/154777837382008833/154777837382008833/985503145239142461).


`Scoreboard values` should be — once again as far as possible — used to restrain the number of entities selected by the selector. The performance impact is low, and even if it is not null, it is still better than executing the rest of the command for entities that we don't want.


If there was only one rule to respect, limit your usage of the arguments in the `NBT/Player Data` category. It will strongly negatively impact your performances. Using `Predicate` is better than NBT but is still code heavy, so use these categories sparingly.

`sort` decreases performances if set to anything other than default `arbitrary`


### 2.1.3. In what order to put the arguments?


So I said that the main point was to limit the damages when using NBTs. But how? The selectors have an order. For example: `@e[type=minecraft:bee, nbt={HasNectar:1b}]` is a way more optimized than `@e[nbt={HasNectar:1b}, type=minecraft:bee]`. Because in the first case, the game will check the NBT value of only the bees; whereas in the second case, the game is checking for all entities if they have nectar, and only after that will check if the entity is a bee. So what's the order?


`Selection`, `Position` and `Rotation` categories have no order with a consequent impact.


As mentioned in the board of ***2.1.2.***, `type` should be used whenever possible. Same thing for `tag`.


With these details, we can now determine the best order:
- `type`
- `tag`
- `scores`
- `level`
- `gamemode`
- `name`
- `advancements`
- `predicate`
- `nbt`

> `type` is always checked first regardless of order when an exact entity (not with entity tags). Additionally, this check uses an efficient class type check which is more performant than any other check. *(cf. **5.7.**)*

## 2.2. `execute if` selection
The execute commands permits to verify conditions with the `if` argument [[ wiki ](https://minecraft.fandom.com/wiki/Commands/execute)].


In addition to the entity selectors, you can — and it is recommanded — to use the `if` argument. There is a variant of the `if` argument which is `unless` equivalent to "if not".


In 1.19.3, you can test with it:
| condition type | Description |
|--|--|
| biome | Test a biome |
| block | Test one block |
| blocks | Test an area of blocks |
| data | Test the existence of an NBT |
| entity | Test the existence of at least one entity with the specified tag |
| predicate | Test if the predicate is true |
| score | Test if the score matches a specified integer or can test the relation between two scorers |




## 2.3. `as @e[scores={}]` or `as @e if score @s`



These two commands below have the same effect (they will give the same context to the following command).
```hs
execute if score @s <objective> matches <int> run ...
```
```hs
execute as @e[scores=<int>] run ...
```
So which one of those is the most efficient?


According to the analysis posted a few months ago on the MCC's reddit by u/Wooden_chest (*cf. reference 6*), the performance delta between the two cases is too small to be considered. But sometimes, the `@e[scores={}]` syntax is preferable because it can avoid checking next arguments.

_eg:_
```hs
execute as @e[type=item,scores={objective=10..},nbt={custom:1b}] run ...
```

Here, like explained in 2.1.3., the order of arguments is important. So using the `if score` syntax will slow down our code.

To summarize, `as @e[scores={}]` is more efficient than `as @e if score @s` but in reality the optimization is so minimal compared to the rest that you will never see the impact.


## 2.4. Tagging instead of testing NBT


In some cases, it is possible to not check the nbt for each tick and instead use tags. It is really case-specific and must be adjusted in each situation.


So let's do an example:


We want to create some particles on some custom items which have the custom NBT tag `{customItem:1b}`.


Then let's create our tick function (we should use the loop function instead, but let's keep this example simple. For loop functions, *cf **3.1.***).


```hs
# # # # # # # # # # # # # # # # # # # # # # #
# foo:tick                                  #
#                                           #
# function executed each tick               #
# # # # # # # # # # # # # # # # # # # # # # #


# check the items that have NEVER been checked before, unless they have the custom tag; they are tagged with foo_ignore in order to not be checked anymore
execute as @e[type=item,tag=!foo_ignore,tag=!foo_custom,nbt=!{Item:{tag:{customItem:1b}}}] run tag @s add foo_ignore


# Then all items not yet tagged with foo_ignore are in all case items having the customItem tag
# So we can just tag them
tag @e[type=item,tag=!foo_ignore] add foo_custom


# Now we are able to select the entity with a tag and not with a NBT
execute as @e[type=item,tag=foo_custom] at @s run particle ...
```


So the goal of this method is to limit the usage of the selector argument `nbt` and instead use tags. This technique should be used in conjunction with the one described in section ***3.1.1.***.


## 2.5. Predicates

The impact performance indicated in the section ***2.1.2.*** is a generalization which must be explained. The content of prerdicates is as variable as its performance impact. In fact, using the `nbt` field in the predicate is less efficient than using `nbt=` selector argument.

So if a nbt check can be replaced by a predicate using other fields than `nbt` in order to replace a `nbt` argument can mokre efficient in most of cases.

Eg: using a predicate in order to check the item in the mainhand is preferable than using the nbt `SelectedItem`.


# 3. Datapack Structure


Datapacks are not only mcfunction files; their structure is important, and there are other features to datapacks that permit you some optimization without using functions.


## 3.1. Running functions only when needed


### 3.1.1. Tick functions
These functions are running every tick, so be careful when putting commands in them. The majority of lags from datapacks come from (too many) useless functions put in these files.


### 3.1.2. Schedule functions
Schedule functions are amazing!
They can permit you to run slower loops than tick functions.


In fact, a lot of functions don't require to be executed each tick and can be satisfied with being run only once every two ticks. Using this method, you can reduce your lag almost by two. Of course, you can use slower functions to reduce lag even further.


#### How to use schedule loops ?
```hs
# # # # # # # # # # # # # # # # # # # # # # #
# foo:load                                  #
#                                           #
# function executed when the pack is loaded #
# # # # # # # # # # # # # # # # # # # # # # #


schedule function foo:loop2t 2t
```
```hs
# # # # # # # # # # # # # # # # # # # # # # #
# foo:loop2t                                #
#                                           #
# loop function running every two ticks     #
# # # # # # # # # # # # # # # # # # # # # # #


# execute your commmands here



# schedule the next itteration
schedule function foo:loop2t 2t
```
The concept is pretty simple; we are scheduling our function a first time at the load of our datapack, and then the function will schedule itself when executed.



## 3.2. Advancements and Events
Another way to replace commands running from `minecraft/tags/tick.json` is to use the Minecraft advancements. We can consider them an equivalent of events in other programming languages. Indeed, advancements are allowing the use of functions as rewards.


So we can use this functionality to use built-in conditions for advancements.


You can find the full list of triggers on [this page](https://minecraft.fandom.com/wiki/Advancement/JSON_format#List_of_triggers) of the wiki.

⚠️ Using the `minecraft:tick` crtiteria **will not** optimize your code at all.


#### An example to illustrate:
Let's detect when a player kills a zombie and make him say make him say "Task completed!".



Instead of using a scoreboard with the `minecraft.killed:minecraft.zombie` criteria and checking if the score changed, we are going to use advancements.


First, let's setup our advancement:
`foo/advancement/triggerkillszombie.json`
```json
{
    "criteria": {
        "requirement": {
            "trigger": "minecraft:player_killed_entity",
                "conditions": {
                    "entity": {
                    "type": "minecraft:zombie"
                }
            }
        }
    },
    "rewards": {
        "function": "foo:zombiekilled"
    }
}
```
You can notice the presence in the reward of the function `foo:zombiekilled`. So let's create this function:
```hs
# # # # # # # # # # # # # # # # # # # # # # # # # # #
# foo:zombiekilled                                  #
#                                                   #
# function executed when a player kills a zombie    #
# # # # # # # # # # # # # # # # # # # # # # # # # # #


# execute your commmands here
say Task completed!


# revoke the advancement
advancement revoke @s only foo:triggerkillszombie
```
NB: When the function is executed, the function is executed in the context of the player who made the advancement. That means that you can use `@s` in order to select the player.


As we are revoking the advancements after executing our commands, we are "reactivating" the trigger of the advancement.



## 3.3. Functions and Structure
Let's talk about the functions and structure. Even if it isn't recommended to put all your functions in the same folder, we will not talk about that here beyond this recommendation because it has no impact on the performance of your code. We are going to talk in this part about the appeal of function in other functions. In fact, you can significantly improve the performance of your data pack.


You can use as below a function as a procedure, in order to execute multiple times the same group of commands.
```hs
function foo:myfunction
```


Furthermore, the command `/function` keeps the context. So that means that instead of this example:
```hs
# exemple 1
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command1
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command2
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command3
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command4
```


You can use this version, where you will divide your function into two parts:


```hs
# # # # # # # # # # # # # # # # # # # # # # # # # # #
# foo:exemple2                                      #
# # # # # # # # # # # # # # # # # # # # # # # # # # #


execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run function foo:exemple2-procedure
```
```hs
# # # # # # # # # # # # # # # # # # # # # # # # # # #
# foo:exemple2-procedure                            #
# # # # # # # # # # # # # # # # # # # # # # # # # # #


command1
command2
command3
command4
```


The goal is to check the condition imposed by the selector only once instead of four times. As mcfunction is an imperative programming language, the order of the functions will still be the same, and the result will be exactly the same between examples 1 and 2.

It is recommended to use this structure in the cases where there is a lot of subcommands, or the selectors/conditions is complex.
If the condition is simple and the number of subcommands is pretty low, the cost of calling a function will be higher than checking two times the condition.


# 4. Summary and Important Points


- The arguments of selectors are ordered.
- Modifying data with "/data" necessitates a high level of performance.
- Instead of using the same selector multiple times, use a function.
- Use the tick function when it is absolutely necessary.
- Use advancements as events or triggers.
- If possible, use tags checks instead of nbt checks in tick/loop functions.
- Don't forget to run by yourself profiling which will permit you to find what is the most unefficient part of your code.
- Focus in a first time on the laggiest parts of your code. Start by optimizing `as @e if score @s` is unefficient.



# 5. References and sources
All references and credits for anyone who participated directly or indirectly in this gist can be found here.



### Sources & References
1. [@Dominexis](https://github.com/Dominexis) selectors efficiency analysis [[ GSheet ](https://docs.google.com/spreadsheets/d/1Z0XVvyfzVSGstmpLSMKnwlxwYg8N2ZFl3Xmh0ZV0yZU/edit#gid=0)] [[ Original post ](https://discord.com/channels/154777837382008833/154777837382008833/1031977637620498462)]
2. MCP-Reborn [[ Github Repo ](https://github.com/Hexeption/MCP-Reborn)]
3. Minecraft Commands' Discord server [[ link ](https://discord.gg/QAFXFtZ)]
4. [@Misode](https://github.com/misode) McMeta repo [[ Github Repo ](https://github.com/misode/mcmeta)]
5. Minecraft Wiki [[ link ](https://minecraft.fandom.com/wiki/Minecraft_Wiki)]
6. [u/Wooden_chest](https://www.reddit.com/user/Wooden_chest/) performance tests [[ link ](https://www.reddit.com/r/MinecraftCommands/comments/w4vjs3/whenever_i_create_datapacks_i_sometimes_do/)]
7. [@capitalists#1171](https://discordapp.com/users/217271293668622344) `type` argument is allways checked (message on MinecraftCommands discord) [ link to message ](https://discord.com/channels/154777837382008833/154777837382008833/985503145239142461)


### Special thanks
- [@Dominexis](https://github.com/Dominexis) for his precious tips and advised indications
- [@Misode](https://github.com/misode) for checking the code about `name` selector argument efficiency
- [@capitalists#1171](https://discordapp.com/users/217271293668622344) for checking the code about `type` argument

#### Edits:
- [@BluePsychoRanger](https://github.com/BluePsychoRanger) edit about `sort` argument, code rectification
- [@CosmicAxolotl](https://github.com/CosmicAxolotl) precision about @a and dead players
- [@ICY105](https://github.com/ICY105) specification about `type` argument usage



[← Back to the README](../README.md)
