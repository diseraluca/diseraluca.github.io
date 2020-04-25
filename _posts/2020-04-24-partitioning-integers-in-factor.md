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
Now, in the book, the first parsing method that is encountered is that of [Unger's Parsers](https://user.phil-fak.uni-duesseldorf.de/~kallmeyer/Parsing/unger.pdf) which requires all the k-partitions of the input string to execute ( which is actually the partitioning of a set, but we can build that from integer partitions ).

> From what I could gather it seems that, in regards to Unger's Parsers, we are actually interested in Integer Compositions rather than partitions. 
> The difference seems to be that order does not matter in partitions, such that $$ 2 + 1 $$ and $$ 1 + 2 $$ represent the same partition, while it matters in compositions.

To my surprise, it seems that there is no default way in factor to partition a sequence in k-partitions; or, at least, I was unable to find it neither by searching where I toughth it would be, namely, [math.combinatorics](https://docs.factorcode.org/content/vocab-math.combinatorics.html), [grouping](https://docs.factorcode.org/content/vocab-grouping.html) or [splitting](https://docs.factorcode.org/content/vocab-splitting.html); neither by a general search.

Thus, I decided to look a bit into the topic, which is mostly new to me ( I haven't really done much combinatorics and I just know some small bits here and there that I learned when needed ).

<!--godomalissimo-->

# What does it mean to partition an integer?

A partition of a positive integer $$ n $$ is a multiset of positive integers which sum is $$ n $$.
For example, $$ \{\, 6\, 1\, 1\, \} $$ is a partition of $$ 8 $$ since $$ 6 + 1 + 1 = 8 $$.

<table>
  <caption>Partitions for the positive integer 8</caption>
  <tr>
    <th>1-partitions</th>
    <th>2-partitions</th>
    <th>3-partitions</th>
    <th>4-partitions</th>
    <th>5-partitions</th>
    <th>6-partitions</th>
    <th>7-partitions</th>
    <th>8-parttions</th>
  </tr>
  <tr>
    <td>8</td>
    <td>7 + 1</td>
    <td>6 + 1 + 1</td>
    <td>5 + 1 + 1 + 1</td>
    <td>4 + 1 + 1 + 1 + 1</td>
    <td>3 + 1 + 1 + 1 + 1 + 1 </td>
    <td>2 + 1 + 1 + 1 + 1 + 1 + 1</td>
    <td>1 + 1 + 1 + 1 + 1 + 1 + 1 + 1</td>
  </tr>
  <tr>
    <td></td>
    <td>6 + 2</td>
    <td>5 + 2 + 1</td>
    <td>4 + 2 + 1 + 1</td>
    <td>3 + 2 + 1 + 1 + 1 + 1</td>
    <td>2 + 2 + 1 + 1 + 1 + 1</td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>5 + 3</td>
    <td>4 + 2 + 2</td>
    <td>3 + 2 + 2 + 1</td>
    <td>2 + 2 + 2 + 1 + 1</td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td>4 + 4</td>
    <td>4 + 3 + 1</td>
    <td>2 + 2 + 2 + 2</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td>3 + 3 + 2</td>
    <td>3 + 3 + 1 + 1</td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>

We call an element of a partition a part. Thus, by $$ k-partition $$ of $$ n $$ we will mean a partition of $$ n $$ that has $$ k $$ parts.

On a really basic level, this seems to be all there is to the definition of Integer Partitions.
> On a more involved level, there seems to be a lot of cool knowledge related to Integer Partitions.
> One such bit that I encountered is the [work](https://www.wikiwand.com/en/Ramanujan%27s_congruences) done by mathematician [Srinivasa Ramanujan](https://en.wikipedia.org/wiki/Srinivasa_Ramanujan)( On which, by the way, there is a somewhat engaging [movie](https://en.wikipedia.org/wiki/The_Man_Who_Knew_Infinity_(film)) ) on the partition function.

# How are those partitions generated ?

There seems to be quite a few different sources on how to generate partitions.
One of those sources seems to be from [Knuth's The Art Of Computer Programming](http://www.cs.utsa.edu/~wagner/knuth/) which reminds me that I should really add the collection to my bookshelf as it is quite the unmatched reference ( which helped me a lot when looking at Random Number Generators ).
> On a tangetially related note, the phrase "generating the partitions", while not necessarily related to the [generating function](https://en.wikipedia.org/wiki/Generating_function) term, reminds me that I really want to read [Wilf's Generatingfunctionology](https://www.amazon.com/generatingfunctionology-Third-Herbert-S-Wilf/dp/1568812795/ref=sr_1_1?crid=8MI0O0ZY545X&dchild=1&keywords=generatingfunctionology&qid=1587745869&sprefix=generatingfunctionology%2Caps%2C-1&sr=8-1) but am currently unable as I don't have the required mathematical knowledge, yet.

Of the methods I've found, I've decided to follow [this paper](http://www.nakano-lab.cs.gunma-u.ac.jp/Papers/e90-a_5_888.pdf) by [Katsuhisa Yamanaka](http://www.kono.cis.iwate-u.ac.jp/~yamanaka/),[Shin-ichiro Kawano](http://archive.today/2020.04.25-095832/https://www.researchgate.net/scientific-contributions/17765461_Shin-ichiro_KAWANO),[Yosuke Kikuchi](http://archive.today/2020.04.25-101245/https://www.researchgate.net/profile/Yosuke_Kikuchi) and [Shin-ichi Nakano](http://archive.today/2020.04.25-100254/https://www.researchgate.net/profile/Shin_Ichi_Nakano).

## Partitions as a binary tree

> This section is a summary of Section 2 of the aforementioned paper. Much of the terminology and structure of the text > comes from the paper and isn't an original creation of mine.

| ![Family tree for the positive number 8]({{ "/assets/images/familytreefornumbereight.png" | absolute_url }}) | 
|:--:| 
| *Family tree for the positive number 8* |

Let $$ S(n) $$ be the set of all partitions of the positive integer $$ n $$.

A partition $$ A \in S(n) $$ is a monotonically decreasing sequence of positive integers {% raw %} $$ A = a_1a_2 \ldots a_m $$ {% endraw %}, where $$ m \ge 1 $$ holds, such that $$ n = \sum_{i=1}^{m} a_i $$.

When $$ m = 1 $$ then $$ A = n $$ holds.
This partition is called the root partition.

Let $$ A $$ be a partition in $$ S(n) $$ that is not the root partition.
$$ P(A) $$, the *parent partition* of $$ A $$, is defined over two cases:

1. The last element of $$ A $$ is $$ 1 $$, i.e $$ a_m = 1 $$
   > {% raw %} $$ P(A) = (a_1+1)a_2 \ldots a_{m-1} $$ {% endraw %}
   > That is, $$ P(A) $$ is defined as incrementing the first and removing the last part of $$ A $$.
   > In this case, the parts of $$ P(A) $$ are one less than the parts of $$ A $$.
2. The last element of $$ A $$ is greater than $$ 1 $$, i.e $$ a_m > 1 $$
   > {% raw %} $$ P(A) = (a_1+1)a_2 \ldots (a_m-1) $$ {% endraw %}
   > That is, $$ P(A) $$ is defined as incrementing the first and decrementing the last part of $$ A $$.
   > In this case, the parts of $$ P(A) $$ are equal to the parts of $$ A $$.

We call $$ A $$ the *child partition* of $$ P(A) $$.

$$ A $$ has a single *parent partition* and $$ P(A) $$ has at most two *child partition*.
If $$ A \in S(n) $$ then $$ P(A) \in S(n) $$.

Thus, if $$ A \in S(n) $$, and $$ A $$ is not the root partition, repeatedly applying $ P $ builds a sequence $$ A,P(A),P(P(A)) \ldots $$ of partitions $$ \in S(n) $$ that will, in the end, reach the *root partition*.

The intersection of all such sequences is the *family tree* of $$ S(n) $$, $$ T_n $$, where a vertex represents a partition $$ \in S(n) $$ and an edge is formed by a pair $$ A $$ and $$ P(A) $$.
$$ T_n $$, thus, contains all partitions of $$ n $$.
