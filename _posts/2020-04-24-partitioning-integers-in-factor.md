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

To my surprise, it seems that there is no default way in factor to divide a sequence into k-compositions; or, at least, I was unable to find it neither by searching where I toughth it would be, namely, [math.combinatorics](https://docs.factorcode.org/content/vocab-math.combinatorics.html), [grouping](https://docs.factorcode.org/content/vocab-grouping.html) or [splitting](https://docs.factorcode.org/content/vocab-splitting.html); neither by a general search.

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

> This section is a summary of Sections [2,6] of the aforementioned paper. Much of the terminology and structure of the text comes from the paper and isn't an original creation of mine.

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
For example, $$ \{\!\{ \,8  \,1 \,1\, \}\!\} $$ produces a preserving child $$ \{\!\{\,7\, 1\, 2\, \}\!\} $$ which is not a partition as $$ a_2 < a_3 $$ ( While not being a partition in our case, that is a composition of $$ 10 $$ with $$ 3 $$ parts ).

Each partition has either zero, one or two child. The paper identifies three cases:

1. $$ a_1 = a_2 $$
   * We have no child partition as $$ a_1 < a_2 $$ holds in both $$ A[m] $$ and $$ A[m+1] $$.
2. $$ a_1 > a_2 $$ and $$ a_{m-1} = a_m $$
   * $$ A[m] $$ is not a partitition as $$ a_{m-1} < a_m $$ holds in it.
   * $$ A[m+1] $$ is a partition an thus we have one child.
3. $$ a_1 > a_2 $$ and $$ a_{m-1} > a_m $$. We consider two subcases:
   1. $$ m = 2 $$ and $$ a_{m-1} - {a_m} = 1 $$
      * $$ A[m] $$ is not a partition as $$ a_{m-1} < a_m $$ holds in it.
      * $$ A[m+1] $$ is a partition and thus we have one child.
   2. Otherwise
      * Both $$ A[m] $$ and $$ A[m+1] $$ are partitions and thus we have two children.
      
Thus, $$ S(n) $$ can be built by starting from the *root partition*, finding its direct child and then recursively generating the tree from there.

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

###### Partitions with at most $$ k $$ parts

If we want to only build the partitions of $$ n $$ that have at most $$ k $$ parts, we will call this set $$ S_{\le k}(n) $$, we are able to do so by adding a restriction to the generation algorithm:

* If the currently inspected partition has an $$ A[m+1] $$ child and $$ m = k $$, we do not search the $$ A[m+1] $$ branch.
      
For example, the partitions of $$ 4 $$ with at most $$ 2 $$ parts can be generated as folllows:

1. The *root partition* of $$ 4 $$ is $$ \{\!\{ \,4\, \}\!\} $$.
2. The child of the root partition is $$ \{\!\{ \,3\, 1\, \}\!\} $$.
3. $$ \{\!\{ \,3\, 1\, \}\!\} $$ is of case 3.2, thus we have two children, but since $$ m = k $$ we will not search the $$ A[m+1] $$ branch
   1. $$ \{\!\{ \,2\, 2\, \}\!\} $$ is the $$ A[m] $$ child. It is a case 1 partition and thus has no child, making it a leaf of the tree.

###### Partitions with exactly $$ k $$ parts

We will call the set of partitions of $$ n $$ with exactly $$ k $$ parts, where $$ n \ge k $$ holds, $$ S_{=k}(n) $$.

If $$ k = n $$ there is a single, trivial partition of $$ k $$ parts, i.e a multiset of $$ n $$ ones.

In the case where $$ n > k $$, the paper shows that there is bijection between $$ S_{\le k}(n - k) $$ and $$ S_{=k}(n) $$.

We can go from a partition $$ A \in S_{=k}(n) $$ to a partition $$ B \in S_{\le k}(n - k) $$, by subtracting one from each part of $$ A $$.

$$ A $$ has exactly $$ k $$ parts thus $$ B $$ will have at most $$ k $$ parts.
Since we are removing one from each part, we will subtract $$ k $$ from the sum of the partition, which is $$ n $$, and thus the new partition will have a sum of $$ n - k $$.

To go from a partition $$ B \in S_{\le k}(n - k) $$, with $$ b \le k $$ parts, to a partition $$ A \in S_{=k}(n), we will add one to each part of $$ B $$ and then append $$ k - b $$ ones to the resulting partition.

Since we are adding one to each part of $$ B $$, the new partition has a sum of $$ n - k + b $$.
We then add $$ k - b $$ ones and thus have a sum of $$ n - k + b + k - b = n $$.

Furthermore, we will have exactly $$ b + k - b = k $$ parts.

Let's look at an example, $$ S_{\le 4}(8 - 4) $$ has the following elements, i.e all partitions of $$ 4 $$:

* $$ \{\!\{ \,4\, \}\!\} $$ 
* $$ \{\!\{ \,3\, 1\, \}\!\} $$ 
* $$ \{\!\{ \,2\, 2\, \}\!\} $$ 
* $$ \{\!\{ \,2 \,1 \,1\, \}\!\} $$ 
* $$ \{\!\{ \,1 \,1 \,1 \,1\, \}\!\} $$

We can thus find the partitions of $$ S_{=4}(8) $$ as follows:

1. $$ \{\!\{ \,4\, \}\!\} $$ has $$ 1 $$ part.
   1. $$ \{\!\{ \,5\, \}\!\} $$ is the result of adding one to each of its parts.
   2. $$ \{\!\{ \,5\, 1\,1\,1\, \}\!\} is the result of appending $$ 4 - 1 = 3 $$ ones to the new partition.
2. $$ \{\!\{ \,3\, 1\, \}\!\} $$ has $$ 2 $$ parts.
   1. $$ \{\!\{ \,4\, 2\, \}\!\} $$ is the result of adding one to each of its parts.
   2. $$ \{\!\{ \,4\, 2\,1\,1\, \}\!\} $$ is the result of appending $$ 4 - 2 = 2 $$ ones to the new partition.
3. $$ \{\!\{ \,2\, 2\, \}\!\} $$ has $$ 2 $$ parts.
   1. $$ \{\!\{ \,3\, 3\, \}\!\} $$ is the result of adding one to each of its parts.
   2. $$ \{\!\{ \,3\, 3\,1\,1\, \}\!\} $$ is the result of appending $$ 4 - 2 = 2 $$ ones to the new partition.
4. $$ \{\!\{ \,2 \,1 \,1\, \}\!\} $$ has $$ 3 $$ parts.
   1. $$ \{\!\{ \,3 \,2 \,2\, \}\!\} $$ is the result of adding one to each of its parts.
   2. $$ \{\!\{ \,3 \,2 \,2\,1\, \}\!\} $$ is the result of appending $$ 4 - 3 = 1 $$ ones to the new partition.
5. $$ \{\!\{ \,1 \,1 \,1 \,1\, \}\!\} $$ has $$ 4 $$ parts.
   1. $$ \{\!\{ \,2 \,2 \,2 \,2\, \}\!\} $$ is the result of adding one to each of its parts.
   2. $$ \{\!\{ \,2 \,2 \,2 \,2\, \}\!\} $$ is the result of appending $$ 4 - 4 = 0 $$ ones to the new partition.

Thus the $$ 4 $$-partitions of $$ 8 $$ are:

* $$ \{\!\{ \,5\, 1\,1\,1\, \}\!\} $$
* $$ \{\!\{ \,4\, 2\,1\,1\, \}\!\} $$
* $$ \{\!\{ \,3\, 3\,1\,1\, \}\!\} $$
* $$ \{\!\{ \,3 \,2 \,2\,1\, \}\!\} $$
* $$ \{\!\{ \,2 \,2 \,2 \,2\, \}\!\} $$

The paper has some more interesting sections about generating restricted partitions but I won't cover them as they are not needed for the task at hand. 

## Intermezzo:

TODO: Complete and add the experiments done in idris.

# Implementing it in Factor

## $$ S_{\le k}(n) $$ generation as the atomic procedure

Generating $$ S(n) $$ from a procedure that generates $$ S_{\le k}(n) $$ is as easy as generating $$ S_{\le n}(n) $$.
Further, from the paper we know that we can generate $$ S_{=k}(n) $$ from $$ S_{\le k}(n - k) $$.

If we consider the case where we have a procedure that generates $$ S(n) $$, we could easily generate $$ S_{\le k}(n) $$ and $$ S_{=k}(n) $$ by filtering the set.
This case would be far from ideal, tough, as we would need to do a lot of unneded work ( and $$ S(n) $$ becomes big fast enough ).

If we, instead, are able to generate $$ S_{=k}(n) $$ we could generate all partitions by $$ n $$ applications and
the same is true for generating $$ S_{\le k}(n) $$.
Again, this would require quite a bit of repeated or unneded work.

As such, I've started from a word that generates $$ S_{\le k}(n) $$ and built the other words from there.

I feel that this is a good middle ground. While we still incur in some overhead in not generating $$ S_{=k}(n) $$ directly, it is negligible for our use case and simplifies the implementation.

Now, you will actually see that I completely disregarded performance here, as it was not the point of the exercise.
The same is true for correctness and elegance. So talking about finding a good middle ground may feel a bit contradictory.
> You may wonder, then, what did I not disregard. Uhm... probably nothing. The time I allotted for this exercise was small and thus I simply produced an "it works!" solution to better understand the domain.

### Generating $$ S{\le k}(n) $$

The core of the generating procedure is the following, based on the code from the paper:

~~~factor
:: <=k-children ( partition k -- )
    partition ,
    partition has-an-expanding-child? [
        partition length k < [ partition expanding-child k <=k-children ] when
        partition has-a-preserving-child? [ partition preserving-child k <=k-children ] when
    ] when ;
~~~

This part is simple enough; if we consider the left child to be $$ A[m+1] $$ this basically describe a pre-order depth-first-search of our family tree.

we first collect the current partition. From *,* you can infer that the word is actually called trough [make](https://docs.factorcode.org/content/word-make%2Cmake.html).

This wasn't necessary and is a refuse from an initial implementation that was more contrived. Nonetheless, I think it might streamline the code a bit.

*has-an-expanding-child* is defined as follows:

~~~factor
: has-an-expanding-child? ( partition -- ? )
    first2 > ;
~~~

This check basically ensures that we are either in case 2 or 3 from the paper.

If you remember ( or check the first part of this article again ), if $$ a_1 = a_2 $$ we have no child.
In all other cases, we have an $$ A[m+1] $$ child.

Since we are generating $$ S_{\le k}(n) $$, instead of directly recurring we have one more check to ensure that we actually want to generate that $$ A[m+1] $$ child that we know to be existing.
If we do, i.e we are not working on a partition that has $$ k $$-parts, we recur and start descending the child-branch.

When we start to get back again, we check if we have an $$ A[m] $$ and descend that branch.

*has-a-preserving-child* is defined as follows:

~~~factor
: has-a-preserving-child? ( partition -- ? )
    [ last2 [ > ] [ - 1 > ] 2bi ]
    [ length 2 > ]
    bi or and ;
~~~

We check if $$ a_{m-1} > a_m $$, thus putting us in the third case.
Then we check that we are not in the first subcase such that we are in the general second subcase, which is the only case with an $$ A[m] $$ child.

The utility words *expanding-child* and *preserving-child* are defined as follows and are basically a one-to-one implementation of the generating procedure defined in the paper:

~~~factor
: expanding-child ( partition -- seq )
    unclip 1 - prefix 1 suffix ;

: preserving-child ( partition -- seq )
    unclip 1 - prefix unclip-last 1 + suffix ;
~~~

*<=k-children* is called from another word that provides the starting partition, i.e the child of the root partition, and calls *make*:

~~~factor
: root-child ( n -- partition )
    1 - 1 2array ;

: (<=k-partitions) ( n k -- partitions )
    [ root-child ] dip [ <=k-children ] { } make ;
~~~

As a last indirection, the actual public entry point, from which everything starts, is the following word:

~~~factor
: <=k-partitions ( n k -- partitions )
    [ [ [ 0 > ] both? ] [ drop 1array 1array ] [ { } ] smart-if* ]
    [
        [ [ 1 > ] bi@ and ]
        [ (<=k-partitions) append ]
        smart-when*
    ] 2bi ;
~~~

> This is one of the ugliest parts of the code in my opinion and one of the first parts I would surely refactor.

Basically, we generate a empty-sequence in the case where the request is malformed, i.e $$ n $$ or $$ k $$ are not positive naturals.
Otherwise, we generate a sequence containing the root partition and then dispatch to *(<=k-partitions)* and adding its result to our sequence.

This is all for generating $$ S{\le k}(n) $$.

A small example of it in action:

~~~factor
IN: scratchpad 8 4 <=k-partitions .
{
    { 8 }
    { 7 1 }
    { 6 1 1 }
    { 5 1 1 1 }
    { 6 2 }
    { 5 2 1 }
    { 4 2 1 1 }
    { 4 2 2 }
    { 3 2 2 1 }
    { 2 2 2 2 }
    { 5 3 }
    { 4 3 1 }
    { 3 3 1 1 }
    { 3 3 2 }
    { 4 4 }
}
IN: scratchpad 4 2 <=k-partitions .
{ { 4 } { 3 1 } { 2 2 } }
~~~

### Generating $$ S(n) $$

The word we implemented is already able to generate $$ S(n) $$ efficiently and providing a word for it is as trivial as follows:

~~~factor
: partitions ( n -- partitions )
    dup <=k-partitions ;
~~~

~~~factor
IN: scratchpad 12 partitions .
{
    { 12 }
    { 11 1 }
    { 10 1 1 }
    { 9 1 1 1 }
    { 8 1 1 1 1 }
    { 7 1 1 1 1 1 }
    { 6 1 1 1 1 1 1 }
    { 5 1 1 1 1 1 1 1 }
    { 4 1 1 1 1 1 1 1 1 }
    { 3 1 1 1 1 1 1 1 1 1 }
    { 2 1 1 1 1 1 1 1 1 1 1 }
    { 1 1 1 1 1 1 1 1 1 1 1 1 }
    { 10 2 }
    { 9 2 1 }
    { 8 2 1 1 }
    { 7 2 1 1 1 }
    { 6 2 1 1 1 1 }
    { 5 2 1 1 1 1 1 }
    { 4 2 1 1 1 1 1 1 }
    { 3 2 1 1 1 1 1 1 1 }
    { 2 2 1 1 1 1 1 1 1 1 }
    { 8 2 2 }
    { 7 2 2 1 }
    { 6 2 2 1 1 }
    { 5 2 2 1 1 1 }
    { 4 2 2 1 1 1 1 }
    { 3 2 2 1 1 1 1 1 }
    { 2 2 2 1 1 1 1 1 1 }
    { 6 2 2 2 }
    { 5 2 2 2 1 }
    { 4 2 2 2 1 1 }
    { 3 2 2 2 1 1 1 }
    { 2 2 2 2 1 1 1 1 }
    { 4 2 2 2 2 }
    { 3 2 2 2 2 1 }
    { 2 2 2 2 2 1 1 }
    { 2 2 2 2 2 2 }
    { 9 3 }
    { 8 3 1 }
    { 7 3 1 1 }
    { 6 3 1 1 1 }
    { 5 3 1 1 1 1 }
    { 4 3 1 1 1 1 1 }
    { 3 3 1 1 1 1 1 1 }
    { 7 3 2 }
    { 6 3 2 1 }
    { 5 3 2 1 1 }
    { 4 3 2 1 1 1 }
    { 3 3 2 1 1 1 1 }
    { 5 3 2 2 }
    { 4 3 2 2 1 }
    { 3 3 2 2 1 1 }
    { 3 3 2 2 2 }
    { 6 3 3 }
    { 5 3 3 1 }
    { 4 3 3 1 1 }
    { 3 3 3 1 1 1 }
    { 4 3 3 2 }
    { 3 3 3 2 1 }
    { 3 3 3 3 }
    { 8 4 }
    { 7 4 1 }
    { 6 4 1 1 }
    { 5 4 1 1 1 }
    { 4 4 1 1 1 1 }
    { 6 4 2 }
    { 5 4 2 1 }
    { 4 4 2 1 1 }
    { 4 4 2 2 }
    { 5 4 3 }
    { 4 4 3 1 }
    { 4 4 4 }
    { 7 5 }
    { 6 5 1 }
    { 5 5 1 1 }
    { 5 5 2 }
    { 6 6 }
}
IN: scratchpad 5 partitions .
{
    { 5 }
    { 4 1 }
    { 3 1 1 }
    { 2 1 1 1 }
    { 1 1 1 1 1 }
    { 3 2 }
    { 2 2 1 }
}
~~~

### Generating $$ S_{=k}(n) $$

### Intermezzo: A small test suite

## But didn't we need compositions?

## Uhm...okay that was easy. But didn't we need to partition a sequence?
