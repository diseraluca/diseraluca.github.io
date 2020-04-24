---
layout: post
title: Partitioning integers in factor
date: 2020-04-24
hidden: true
---

# Why Integer Partitioning?

The last few months have been pretty stressfull between university, searching for a challenging job ( which is absolutely the most stressfull one ), personal projects and, obviously, the whole COVID-19 situtation.
I thus was looking for a breather, something that would distract myself but that I would be able to take less seriously than a personal project ( which, in the end, will stress me as I fight my unreachable perfection problem ).

I've decided to read [Grune/Jacob's Parsing Techniques](https://dickgrune.com/Books/PTAPG_2nd_Edition/).
Now, in the book, the first parsing method that is encountered is that of [Unger's Parsers](https://user.phil-fak.uni-duesseldorf.de/~kallmeyer/Parsing/unger.pdf) which requires all the k-partitions of the input string to execute.

> I'm not completely sure on this as I found divergent convetions, but we are probably looking for the union of all the premutations of each k-partition for an Unger Parser, since the term Integer Partitions implies unordered entities such that $$ 3 + 1 $$ is the same partition as $$ 1 + 3 $$.
I've seen this type called Integer Combinations but I cannot find any definitive proof on the correct terminology.

To my surprise, it seems that there is no default way in factor to partition a sequence in k-partitions; or, at least, I was unable to find it neither by searching where I toughth it would be, namely, [math.combinatorics](https://docs.factorcode.org/content/vocab-math.combinatorics.html), [grouping](https://docs.factorcode.org/content/vocab-grouping.html) or [splitting](https://docs.factorcode.org/content/vocab-splitting.html); neither by a general search.

Thus, I decided to look a bit into the topic, which is mostly new to me ( I haven't really done much combinatorics and I just know some small bits here and there that I learned when needed ).

<!--godomalissimo-->

# What does it mean to partition an integer?

A partition of a positive integer $$ n $$ is a multiset of positive integers which sum is $$ n $$.
For example, $$ { 6 1 1 } $$ is a partition of $$ 8 $$ since $$ 6 + 1 + 1 = 8 $$.

|--------------+--------------+--------------+---------------+-------------------+-----------------------+---------------------------+-------------------------------|
| 1-Partitions | 2-partitions | 3-partitions | 4-Partitions  | 5-partitions      | 6-partitions          | 7-partitions              | 8-partitions                  |
|:------------:|:------------:|:------------:|:-------------:|:-----------------:|:---------------------:|:-------------------------:|:-----------------------------:|
|      8       |    7 + 1     |  6 + 1 + 1   | 5 + 1 + 1 + 1 | 4 + 1 + 1 + 1 + 1 | 3 + 1 + 1 + 1 + 1 + 1 | 2 + 1 + 1 + 1 + 1 + 1 + 1 | 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 |    
|--------------+--------------+--------------+---------------+-------------------+-----------------------+---------------------------+-------------------------------|
|              |    6 + 2     |  5 + 2 + 1   | 4 + 2 + 1 + 1 | 3 + 2 + 1 + 1 + 1 | 2 + 2 + 1 + 1 + 1 + 1 |                           |                               |
|--------------+--------------+--------------+---------------+-------------------+-----------------------+---------------------------+-------------------------------|
|              |    5 + 3     |  4 + 2 + 2   | 3 + 2 + 2 + 1 | 2 + 2 + 2 + 1 + 1 |                       |                           |                               |
|--------------+--------------+--------------+---------------+-------------------+-----------------------+---------------------------+-------------------------------|
|              |    4 + 4     |  4 + 3 + 1   | 2 + 2 + 2 + 2 |                   |                       |                           |                               |
|--------------+--------------+--------------+---------------+-------------------+-----------------------+---------------------------+-------------------------------|
|              |              |  3 + 3 + 2   | 3 + 3 + 1 + 1 |                   |                       |                           |                               |
|====================================================================================================================================================================|
|                                      `Partitions for the positive integer 8`                                                                                         |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
