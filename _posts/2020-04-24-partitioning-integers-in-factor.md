---
layout: post
title: Partitioning integers in factor
date: 2020-04-24
hidden: true
---

# Why Integer Partitioning?

The last few months have been pretty stressful between university, searching for an interesting job ( which is absolutely the most stressful thing ), personal projects and, obviously, the whole COVID-19 situtation.
I thus was looking for a breather, something that would distract myself and that I would be able to take less seriously than a personal project ( which, in the end, will stress me as I fight my unreachable perfection problem ).

I've decided to read [Grune/Jacob's Parsing Techniques](https://dickgrune.com/Books/PTAPG_2nd_Edition/).
Now, in the book, the first parsing method that is encountered is that of [Unger's Parsers](https://user.phil-fak.uni-duesseldorf.de/~kallmeyer/Parsing/unger.pdf) which requires all the k-compositions of the input string to execute.

To my surprise, it seems that there is no default way in factor to partition a sequence in k-partitions; or, at least, I was unable to find it neither by searching where I toughth it would be, namely, [math.combinatorics](https://docs.factorcode.org/content/vocab-math.combinatorics.html), [grouping](https://docs.factorcode.org/content/vocab-grouping.html) or [splitting](https://docs.factorcode.org/content/vocab-splitting.html); neither by a general search.

EDIT: I've asked on the factor's mailng list and indeed there is no default way. It seems that will now be added tough.

Thus, I decided to look a bit into the topic, which is mostly new to me ( I haven't really done much combinatorics and I just know some small bits here and there that I learned when needed ).

Since we can build set-partitions/compositions from integer-partitions/compositions, I've decided to look into, specifically, integer-patitioning ( compositions can then be built from the partitions of the same number ). 

<!--godomalissimo-->

# What does it mean to partition an integer?

A partition of a positive integer $$ n $$ is a multiset of positive integers which sum is $$ n $$.
For example, $$ \{\!\{\, 6\, 1\, 1\, \}\!\} $$ is a partition of $$ 8 $$ since $$ 6 + 1 + 1 = 8 $$.

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

> This section is a summary of Sections [2,6] of the aforementioned paper. Much of the terminology and structure of the text > comes from the paper and isn't an original creation of mine.

The paper models the partitions of a positive integer $$ n $$ as a binary tree ( called its *family tree* ).

| ![Family tree for the positive number 8]({{ "/assets/images/familytreefornumbereight.png" | absolute_url }}) | 
|:--:| 
| *Family tree for the positive number 8* |

Let $$ S(n) $$ be the set of all partitions of the positive integer $$ n $$.

From the paper, we define a partition as follows:

A partition $$ A \in S(n) $$ is a monotonically decreasing sequence of positive integers {% raw %} $$ A = a_1a_2 \ldots a_m $$ {% endraw %}, where $$ m \ge 1 $$ holds, such that $$ n = \sum_{i=1}^{m} a_i $$.

Each $$ a_i $$ is called a part of $$ A $$.

Thus, for example:

* $$ \{\!\{\, 1 \, 1 \, 1 \, 1 \, \}\!\} $$ is a partition of four with four parts.
* $$ \{\!\{\, 2 \, 2 \, 1 \, \}\!\} $$ is a partition of five with three parts.
* $$ \{\!\{ \, 3 \, 2 \, 1 \}\!\} $$ is a partition of six with three parts.
* $$ \{\!\{ \, 4 \, 5 \, 2 \, 1 \}\!\} $$ cannot be a partition since $$ a_1 < a_2 $$.

The root of the binary tree is identified into the *root partition*, which is the partition $$ A \in S(n) $$ that has a single part.
The *root partition* is always a multiset with $$ n $$ as its sole element.

There is then a single direct child to the root partition that has the form $$ (n-1)1 $$.

Given a partition $$ A \in S(n) $$ that has at least two parts, there are two ways in which we can generate a new partition that is still $$ \in S(n) $$ ( thus each partition has at most two child and we can envision the partitions as a binary tree ).
We call them $$ A[m] $$ and $$ A[m+1] $$.

1. $$ A[m]$$ ( In code, I call this a preserving child )[This terminology is not present in the paper]
   > We can generate $$ A[m] $$ from a partition $$ A = a_1a_2 \ldots a_m $$ by subtracting one from $$ a_1 $$ and adding one to $$ a_m $$.
   > Thus, $$ A[m] $$ has the form $$ (a_1 - 1)a_2 \ldots (a_m + 1) $$.
2. $$ A[m+1] $$ ( In code, I call this an expanding child )[This terminology is not present in the paper]
   > We can generate $$ A[m+1] $$ from a partition $$ A = a_1a_2 \ldots a_m $$ by subtracting one from $$ a_1 $$ and adding a one at the end.
   > Thus, $$ A[m+1] $$ has form $$ (a_1 - 1)a_2 \ldots a_ma_{m+1} $$ where $$ a_{m+1} = 1 $$.

Now, we can see that not every partition generates both or any valid partition $$ A[m] $$ or $$ A[m+1] $$.
For example, $$ \{\!\{ \,8  \,1 \,1\, \}\!\} $$ produces a preserving child $$ \{\!\{\,7\, 1\, 2\, \}\!\} $$ which is not a partition as $$ a_2 < a_3 $$.

Each partition has either zero, one or two child. The paper identifies three cases:

1. $$ a_1 = a_2 $$
   * We have no child partition $$ a_1 < a_2 $$ holds in both $$ A[m] $$ and $$ A[m+1] $$.
2. $$ a_1 > a_2 $$ and $$ a_{m-1} = a_m $$
   * $$ A[m] $$ is not a partitition as $$ a_{m-1} < a_m $$ holds in it.
   * $$ A[m+1] $$ is a partition an thus we have one child.
3. $$ a_1 > a_2 $$ and $$ a_{m-1} > a_m $$. We consider two subcases:
   1. $$ m = 2 $$ and $$ a_{m-1} - {a_m} = 1 $$
      * $$ A[m] $$ is not a partition as $$ a_{m-1} < a_m $$ holds in it.
      * $$ A[m+1] $$ is a partition and thus we have one child.
   2. Otherwise
      * Both $$ A[m] $$ and $$ A[m+1] $$ are partitions and those we have two children.
      
Thus, the $$ S(n) $$ can be built by starting from the *root partition*, finding its direct child and then recursively generating the tree from here.

Let's compute an example, we will find the partitions of $$ 4 $$.

1. The *root partition* of $$ 4 $$ is $$ \{\!\{ \,4\, \}\!\} $$.
2. The child of the root partition is $$ \{\!\{ \,3\, 1\, \}\!\} $$.
3. $$ \{\!\{ \,3\, 1\, \}\!\} $$ is of case 3.2, thus we have two children.
   1. $$ \{\!\{ \,2\, 2\, \}\!\} $$ is the $$ A[m] $$ child. It is a case 1 partition and thus has no child, making it a leaf of the tree.
   2. $$ \{\!\{ \,2\, 1\, 1\, \}\!\} $$ is the $$ A[m+1] $$ child. It is a case 2 partition and thus has an $$ A[m+1] $$ child.
      1. $$ \{\!\{\, 1\, 1\, 1\, 1\, \}\!\} $$ is the $$ A[m+1] $$ chld. It is a case 1 partition and thus has no child, making it a leaf of the tree.

| ![Family tree for the positive number 4]({{ "/assets/images/familytreefornumberfour.png" | absolute_url }}) | 
|:--:| 
| *Family tree for the positive number 4* |

Thus, the partitions of $$ 4 $$ are $$ \{\!\{ \,4\, \}\!\} $$, $$ \{\!\{ \,3\, 1\, \}\!\} $$, $$ \{\!\{ \,2\, 2\, \}\!\} $$, $$ \{\!\{ \,2 \,1 \,1\, \}\!\} $$, $$ \{\!\{ \,1 \,1 \,1 \,1\, \}\!\} $$.

      
#### Intermezzo:

TODO: Complete and add the experiments done in idris.
TODO: Complete and add a personal latex interpretation of the paper where some proof are tried.

