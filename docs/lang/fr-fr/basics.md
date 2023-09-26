[← Back to the README](../../../README.md)

#### 🌐 Langue
[[EN]](../../basics.md) **[FR]**

### Table of contents

- [0. Preface](#0-preface)
- [1. Introduction](#1-introduction)
    * [Que devons-nous optimiser ?](#que-devons-nous-optimiser-)
    * [Comment optimiser par soi-même ?](#comment-optimiser-par-soi-même-)
- [2. Le langage de programmation mcfunction](#2-le-langage-de-programmation-mcfunction)
    * [2.1. Sélecteurs](#21-sélecteurs)
        + [2.1.1. Quel sélecteur choisir ?](#211-quel-sélecteur-choisir-)
        + [2.1.2. Efficacité des arguments des sélecteurs](#212-efficacité-des-arguments-des-sélecteurs)
        + [2.1.3. Dans quel ordre placer les arguments ?](#213-dans-quel-ordre-placer-les-arguments-)
    * [2.2. Séléction avec `execute if`](#22-séléction-avec-execute-if)
    * [2.3. `as @e[scores={}]` or `as @e if score @s`](#23-as-escores-ou-as-e-if-score-s)
    * [2.4. Utiliser des tags plutôt que tester un NBT](#24-utiliser-des-tags-plutôt-que-tester-un-nbt)
    * [2.5. Predicates](#25-predicates)
- [3. Structure du Datapack](#3-structure-du-datapack)
    * [3.1. Executer des fonctions seulement quand il y a besoin](#31-executer-des-fonctions-seulement-quand-il-y-a-besoin)
        + [3.1.1. Fonctions Tick](#311-fonctions-tick)
        + [3.1.2. Fonctions Schedule](#312-fonctions-schedule)
    * [3.2. Advancements et Events](#32-advancements-et-events)
    * [3.3. Fonctions et Structure](#33-fonctions-et-structure)
- [4. Récapitulatif et points importants](#4-récapitulatif-et-points-importants)
- [5. Sources et Références](#5-sources-et-références)





# 0. Preface


Là est la grande question: Comment pouvons nous optimiser notre code dans les datapacks ? C'est une question cruciale qui à le potentiel de pouvoir améliorer des datapacks qui faut crasher le jeu à du contenu très performant.


J'ai réalsisé le problème de la faible documentation de l'optimisation des datapacks quand quelqu'un est venu sur le serveur discord Minecraft Commands avec un datapack qui faisait crash son jeu à cause de la mauvaise optimisation.


Donc ici je ne parlerais que des bases de l'optimisation.


Ce repo github à la but d'êtrre mis a jour dans le futur et d'être amélioré. Donc sentez vous libre de rejoindre le discord de MCCmmds pour en discuter!



# 1. Introduction


Il existe différents niveaux d'optimisation, mais comme je l'ai dit précédemment, **je ne parlerai pas ici d'optimisation mathématique**. Je vais diviser ce résumé en deux sections principales :

* Le langage mcfunction
* La structure du datapack


### Que devons-nous optimiser ?

L'optimisation est importante car elle permet la création de programmes plus importants sans affecter négativement l'expérience utilisateur. L'objectif n'est pas de viser la perfection, car cela peut entraver la productivité. Au lieu de cela, l'objectif est d'être réaliste concernant les objectifs et de choisir la meilleure option. Différents niveaux d'optimisation offrent plusieurs options à choisir.

Dans ce document, vous trouverez quelques points clés qui vous fourniront plus d'options.

### Comment optimiser par soi-même ?

Ce résumé donne un aperçu du fonctionnement du langage mcfunction, mais parfois ces directives peuvent ne pas être suffisantes et vous pourriez avoir besoin de mener vos propres tests. Il existe plusieurs outils disponibles à cette fin. Notament Alt + F3, qui affiche l'évolution de votre TPS (ticks par seconde) et MSPT (millisecondes par tick).

La valeur MSPT est la plus importante, car il représente le temps nécessaire au jeu pour calculer un seul tick. Un MSPT bas est le meilleur. Il est plus importante que le TPS, car le TPS est limité à 20 alors que le MSPT n'a pas de limite finie.

Vous pouvez également utiliser F3 + L en mode solo pour profiler les performances de votre code. Cela vous permettra d'analyser les résultats avec le [report inspector](https://misode.github.io/report/) fait par misode. Sur le serveur, utilisez la commande /profile, mais les résultats ne seront pas aussi complets qu'en mode solo.


# 2. Le langage de programmation mcfunction


Dans cette section, vous pouvez trouver l'optimisation en utilisant une approche basée sur le langage. En d'autres termes, il y a des structures à éviter absolument dans votre code.


## 2.1. Sélecteurs


Les sélecteurs sont essentiels dans mcfunction. Il y a de nombreuses raisons et justifications pour lesquelles ils sont si importants, mais saviez-vous qu'ils peuvent avoir un impact sur les performances en fonction de la manière dont vous les utilisez ?


### 2.1.1. Quel sélecteur choisir ?


`@e` est le sélecteur générique, il sélectionne toutes les entités. À ce sélecteur, vous pouvez ajouter des sélecteurs de cible (vous pouvez les trouver tous [ici sur le wiki [en]](https://minecraft.wiki/w/Target_selectors)). Donc, pour récapituler, @e[type=minecraft:player] sélectionnera uniquement les joueurs.


On peut trouver des propriétés de @e dans d'autres sélecteurs, mais ils ont toujours leurs propres propriétés :

>- `@a`, un équivalent de `@e[type=minecraft:player]` mais est plus rapide et peut séléctionner les joueurs morts, ce que `@e` ne peut pas faire
>- `@p`, un équivalent de `@a[sort=nearest,limit=1]` ou `@e[type=minecraft:player,sort=nearest,limit=1]` mais plus rapide
>- `@r`, un équivalent de `@a[sort=random,limit=1]` ou `@e[type=minecraft:player,sort=random,limit=1]`
>- `@s` est l'exception - bien que similaire à `@p` - elle détecte uniquement la source. Si la source n'est pas une entité (comme un bloc de commande ou la console), aucune entité n'est sélectionnée.



### 2.1.2. Efficacité des arguments des sélecteurs


Revenons à l'optimisation ; le premier point important est de sélectionner une entité **uniquement lorsque vous en avez besoin**. Laissez-moi développer :


Les sélecteurs peuvent avoir différents impacts sur les performances en fonction de leurs arguments, il est donc très important de choisir ces derniers en fonction des circonstances. Voici une liste des arguments des sélecteurs organisés par catégories et leur impact afin d'affiner notre sélection et de ne sélectionner que les entités pertinentes.



| Catégorie | Arguments de Sélecteur | Impact sur les performances direct (comparé à `@e`) | Commentaire | Description d'impact global quand utilisé |
|--|--|--|--|--|
| Sélection | `limit` | négligeable | / | améliore les performances |
| Sélection | `sort` | faible | `sort=arbitrary` est implicite dans presque toutes les commandes | améliore les performances |
| Type d'entité | `type` | négligeable | Doit être utilisé chaque fois que vous pouvez le spécifier (*cf. **2.1.3.***) | améliore les performances |
| Position | `x`, `y`, `z`, `dx`, `dy`, `dz`, `distance` | négligeable | Devrait être utilisé sur de grands ensembles d'entités | améliore légèrement les performances |
| Valeurs de scoreboard | `tag`, `team`, `scores` | faible - moyen | De la plus efficace à la moins efficace, ce sont de bons arguments de sélecteur pour affiner votre sélection (*cf. **2.1.3.***), vous devriez les utiliser dès c'est possible | améliore les performances |
| Prédicat | `predicate` | dépend des conditions du prédicat | meilleur que `NBT/Player Data` sauf si utilisation de la condition `NBT` | dépends de son contenu |
| NBT/Player Data | `advancement`, `name`, `nbt` | forte | à éviter (utilise un analyseur de chaînes de caractères, ce qui est plutôt louf) | déteriore les performances |


La dernière colonne indique s'il vaut la peine d'utiliser ou non l'argument pour affiner la sélection. Par exemple : Le coût de performance de cet argument est-il préférable à celui impliqué dans l'exécution de la commande suivante ?


Ainsi, dans la liste ci-dessus, vous avez la majorité des arguments disponibles, classés des plus légers aux plus lourds. Les catégories `Sélection`, `Position` et `Rotation` n'ont pas d'impact significatif.


`Type d'entité` devrait toujours être présent, autant que possible. Il a été démontré que le jeu check toujours le type du mob que l'argument soit spécifé ou non [[lien vers le discord de MCCmmds]](https://discord.com/channels/154777837382008833/154777837382008833/985503145239142461).


`Valeurs de scoreboard` devrait être — encore une fois — utilisé le plus possbile pour limiter le nombre d'entitées sélectionnées par notre sélecteur.


Si il n'y avais qu'une seule règle à respecter, limitez votre usage des arguments de la catégorie `NBT/Player Data`. Leur utilisation impacte grandement les performances. 

`sort` diminue les performances si défini sur autre chose que `arbitrary`.


### 2.1.3. Dans quel ordre placer les arguments ?


Donc je disais que le point le plus important était de limiter les dommages quand on utilisais les NBTs. Mais comment ? Les sélecteurs ont un ordre. Par exemple: `@e[type=minecraft:bee, nbt={HasNectar:1b}]` est bien plus optimisé que `@e[nbt={HasNectar:1b}, type=minecraft:bee]`. C'est parce que dans le premier cas, le jeu ne check le NBT que des abeilles et ignorera les autres entitées; alors que dans le second cas, le jeu regardera le NBT de toutes les entitées de la map, puis sélectionnera parmis elles uniquement les abeilles.

Mais alors quel est l'ordre?

L'ordre des arguments des catégories `Sélection`, `Position` et `Rotation` n'a pas d'impact significatif.

Comme mentionné dans le tableau de la section ***2.1.2.***, `type` devrait être utilisé dès que possible. Pareil pour `tag`.


Avec ces détails, on peut déterminer d'un ordre à respecter:
- `type`
- `tag`
- `scores`
- `level`
- `gamemode`
- `name`
- `advancements`
- `predicate`
- `nbt`

> `type` est toujours à check en premier, quel que soit l'ordre. De plus, cet argument est plus efficace que n'importe quel autre check. *(cf. **5.7.**)*

## 2.2. Séléction avec `execute if`

La commande /execute permets de vérifier des conditions avec l'argument `if` [[ wiki ](https://minecraft.wiki/w/Commands/execute)].

En plus du sélécteur d'entitées, vous pouvez — et il est recommandé — d'utiliser l'agument `if`. Il y a une variante `unless` qui est son contraire, un "if not".


En 1.19.3, vous pouvez faire des conditions avec:
| condition type | Description |
|--|--|
| biome | Teste un biome |
| block | Teste un bloc |
| blocks | Teste une zone de blocs |
| data | Teste l'existence d'un NBT |
| entity | Teste l'existence d'au minimum une entitée correspondant au sélécteur |
| predicate | Teste si les condtions d'un prédicate est vrai |
| score | Teste si le score est égal un un entier spécifié ou teste une relation entre deux scores |




## 2.3. `as @e[scores={}]` ou `as @e if score @s`



Ces deux commandes ci-dessous ont le même effet (elles vont transmettre le même contexte à la fonction suivante).
```hs
execute if score @s <objective> matches <int> run ...
```
```hs
execute as @e[scores=<int>] run ...
```
Alors laquelle des deux est la plus efficace ?


En se référent à l'analyse postée il y a quelques mois sur le reddit de MCCmmds par u/Wooden_chest (*cf. reference 6*), le delta de performances entre les deux cas est trop petite pour être considérée. Mais parfois la syntaxe `@e[scores={}]` est préférable car cela peut permettre d'éviter de check les arguments suivants.

_exemple:_
```hs
execute as @e[type=item,scores={objective=10..},nbt={custom:1b}] run ...
```

Ici, comme expliqué dans la section 2.1.3., l'ordre des arguments est important. Donc utiliser la syntaxe `if score` va ralenir notre code.

Pour résumer, `as @e[scores={}]` est plus efficace que `as @e if score @s` mais en réalitée, l'optimisation est si minimale ccomparée au reste que vous n'en verrez jamais l'impacte (excépté le cas mentionné ci dessus).


## 2.4. Utiliser des tags plutôt que tester un NBT


Dans certains cas, il est possible de ne pas check un NBT pour chaque tick et d'utiliser un tag. Ce sont des cas vraiment spécifiques et doivent être ajustés en fonction des besoins dans chaque situation.

Voyons un exemple:

Nous souhaitons créer des particules sur certains items custom qui on un NBT custom `{customItem:1b}`.

Créons notre tick function, (nous devrions idéalement utiliser une fonction loop plutôt, mais pour garder cet exemple simple, on garde ici une fonction tick). Pour les fonctions loop, *cf **3.1.***.

```hs
# # # # # # # # # # # # # # # # # # # # # # #
# foo:tick                                  #
#                                           #
# function executed each tick               #
# # # # # # # # # # # # # # # # # # # # # # #


# check les items qui n'ont JAMAIS été check. Si ils n'ont pas de tag custom, ils sont tagués avec foo_ignore pour ne plus jamais être check.
execute as @e[type=item,tag=!foo_ignore,tag=!foo_custom,nbt=!{Item:{tag:{customItem:1b}}}] run tag @s add foo_ignore


# Alors, tous les items pas encore taggués avec foo_ignore sont tous des items ayant le tag custom recherché.
# Donc on peut les taguer avec foo_custom
tag @e[type=item,tag=!foo_ignore] add foo_custom


# Désormais, nous somme capables de séléctionner les entités avec un tag et plus avec un NBT
execute as @e[type=item,tag=foo_custom] at @s run particle ...
```

Le but de cette méthode est de limiter l'usage de l'argument de sélecteur `nbt` à 1 itération par item et d'utiliser à la place des tags. Cette technique devrait être utilisée en lien avec celle décrite dans la section ***3.1.1.***.


## 2.5. Predicates


L'impact de performance indiquée dans la section ***2.1.2.*** est une généralisation qui doit être expliquée. Le contenu des prédicats est aussi variable que leur impact sur les performances. En fait, l'utilisation du champ `nbt` dans le prédicat est moins efficace que l'utilisation de l'argument sélectif `nbt=`.

Ainsi, si une vérification nbt peut être remplacée par un prédicat, l'utilisation d'autres champs que `nbt` pour remplacer un argument `nbt` peut s'avérer plus efficace dans la plupart des cas.

Par exemple, l'utilisation d'un prédicat pour vérifier l'item dans la main principale est préférable à l'utilisation de l'argument nbt `SelectedItem`.


# 3. Structure du Datapack

Les datapacks ne sont pas seulement des fichiers mcfunction; leur structure est importante, et les datapacks présentent d'autres caractéristiques qui permettent une certaine optimisation sans utiliser de fonctions.


## 3.1. Executer des fonctions seulement quand il y a besoin


### 3.1.1. Fonctions Tick

Ces fonctions sont exécutées à chaque tick, il faut donc faire attention lorsque l'on met des commandes dans ces fichiers. La majorité des lags dus aux datapacks proviennent de (trop) nombreuses fonctions inutiles placées dans ces fichiers.

### 3.1.2. Fonctions Schedule

Les fonctions schedule sont extraordinaires !
Elles peuvent vous permettre d'exécuter des boucles plus lentes que les fonctions tick. Même si les règles applicables pour les fonctions tick s'appliquent aussi.


En fait, de nombreuses fonctions n'ont pas besoin d'être exécutées à chaque tick et peuvent se contenter d'être exécutées une fois tous les deux ticks. En utilisant cette méthode, vous pouvez diviser votre lag presque par deux. Bien entendu, vous pouvez utiliser des fonctions encore plus lentes pour réduire encore davantage le lag.


#### Comment planifier des boucles ?
```hs
# # # # # # # # # # # # # # # # # # # # # # #
# foo:load                                  #
#                                           #
# fonction éxcutée quand le pack est chargé #
# # # # # # # # # # # # # # # # # # # # # # #


schedule function foo:loop2t 2t
```
```hs
# # # # # # # # # # # # # # # # # # # # # # #
# foo:loop2t                                #
#                                           #
# fonction loop éxécutée tous les 2 ticks   #
# # # # # # # # # # # # # # # # # # # # # # #


# éxecutez vos commandes ici



# schedule la prochaine itération
schedule function foo:loop2t 2t
```
Le concept est plutôt simple; nous planifions (schedule) une fonction la première fois au début de notre datapack, puis la fonction se replanifiera elle-même quand éxécutée.



## 3.2. Advancements et Events
Une autre façon de remplacer les commandes tournant depuis `minecraft/tags/tick.json` est d'utiliser les advancements de Minecraft. On peut les considérer comme un équivalent des events dans d'autres languages de programmation. En effet, les advancements nous permettent d'éxécuter des fonctions lors de leurs activations.

Nous pouvons donc utiliser cette fonctionalitée pour les conditions existantes des advancements au lieu de check une condition tous les ticks.


Vous pouvez trouver une liste complète des différents critères sur [cette page](https://minecraft.wiki/w/Advancement/JSON_format#List_of_triggers) du wiki.

⚠️ Utiliser le critère `minecraft:tick` **n'optimisera pas** votre code du tout.


#### Exemple:
Détectons lorsqu'un joueur tue un zombie et faisons-lui dire "Tâche accomplie!".

Au lieu d'utiliser un scoreboard avec le critère `minecraft.killed:minecraft.zombie` et de vérifier si le score a changé, nous allons utiliser un advancement.


Premièrement, mettons en place notre advancement:
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
Vous pouvez remarquer la présence en récompense de la fonctions `foo:zombiekilled`. Donc créons cette fonction:
```hs
# # # # # # # # # # # # # # # # # # # # # # # # # # #
# foo:zombiekilled                                  #
#                                                   #
# fonction éxécutée quand un joueur tue unn zombie  #
# # # # # # # # # # # # # # # # # # # # # # # # # # #


# éxécutez vos commandes ici
say Tâche accomplie!


# révoquer l'advancement
advancement revoke @s only foo:triggerkillszombie
```
NB: Quand la fonction est éxécutée, celle-ci est éxécutée dans le context du joueur qui a accompli l'advncement. Cela veut dire que vous pouvez utiliser `@s` pour séléctionner le joueur.

Comme nous révoquons l'advancement après avoir éxécuté nos commandes, nous "réarmons" la détéction de l'advancement.




## 3.3. Fonctions et Structure
Parlons des fonctions et de la structure du datapack. Même s'il n'est pas recommandé de mettre toutes ses fonctions dans le même dossier, nous n'en parlerons pas ici au-delà de cette recommandation car cela n'a pas d'impact sur la performance de votre code. Nous allons parler dans cette partie de l'appel de la fonction dans d'autres fonctions. En effet, vous pouvez améliorer de manière significative les performances de votre datapack. On parle alors de "wrap".


Vous pouvez utiliser la fonction ci-dessous comme une procédure, de façon à éxécuter plusieurs fois le même groupe de commandes.
```hs
function foo:myfunction
```


De plus, la commande `/function` conserve le contexte. Cela signifie donc qu'au lieu de cet exemple :
```hs
# exemple 1
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command1
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command2
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command3
execute as @e[type=item,tag=Tag1,tag=Tag2,nbt={custom:1}] run command4
```


Vous pouvez utiliser cette version, dans laquelle vous diviserez votre fonction en deux parties :
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


L'objectif est de vérifier la condition imposée par le sélecteur une seule fois au lieu de quatre. Comme le mcfunction est un langage de programmation impératif, l'ordre des fonctions sera toujours le même, et le résultat sera exactement le même entre les exemples 1 et 2.

Il est recommandé d'utiliser cette structure dans les cas où il y a beaucoup de sous-commandes, ou lorsque les sélecteurs/conditions sont complexes.
Si la condition est simple et que le nombre de sous-commandes est assez faible, le coût d'appel d'une fonction sera plus élevé que la vérification de deux fois la condition.


# 4. Récapitulatif et points importants


- Les arguments de sélécteurs ont un ordre
- Modifier les données avec `/data` impacte beaucoup les performances.
- Au lieu d'utiliseer plusieurs fois le même sélécteur, appeler une fonction qui éxécutera les commandes.
- Utiliser les fonctions "tick" seulement quand nécéssaire.
- Utiliser les advancements comme des events.
- Si possible, utiliser des séléctions par tags plutôt que des séléctionner avec des nbt à chaque tick.
- Ne pas oublier de faire ces propres tests pour déterminer le plus optimal en fonction de la situation.
- Se focus dans un premier temps sur les parties les plus lourdes de votre code. Commencer par optimiser `as @e if score @s` est inefficace.



# 5. Sources et Références
Toutes les références et les crédits pour toute personne ayant participé directement ou indirectement à ce projet peuvent être trouvés ici.


### Sources & References
1. [@Dominexis](https://github.com/Dominexis) selectors efficiency analysis [[ GSheet ](https://docs.google.com/spreadsheets/d/1Z0XVvyfzVSGstmpLSMKnwlxwYg8N2ZFl3Xmh0ZV0yZU/edit#gid=0)] [[ Original post ](https://discord.com/channels/154777837382008833/154777837382008833/1031977637620498462)]
2. MCP-Reborn [[ Github Repo ](https://github.com/Hexeption/MCP-Reborn)]
3. Minecraft Commands' Discord server [[ link ](https://discord.gg/QAFXFtZ)]
4. [@Misode](https://github.com/misode) McMeta repo [[ Github Repo ](https://github.com/misode/mcmeta)]
5. Minecraft Wiki [[ link ](https://minecraft.wiki/w/Minecraft_Wiki)]
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



[← Back to the README](../../../README.md)
