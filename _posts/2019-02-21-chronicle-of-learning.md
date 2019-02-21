---
layout: post
title: The chronicle of self-learning - mov, Turing Machines and Compilers
date: 2019-02-21
categories: abstract
tags: compilers design turing-machines abstract ramblings
draft: true
---

### "And so for a time it looked as if all the adventures were coming to an end, but that was not to be"

Another adventure is coming to its end as I'm finally reaching the last few chapters of the [Crafting Interpreters book](http://craftinginterpreters.com/), from [Bob Nystrom](http://journal.stuffwithstuff.com/).
I already liked the author's [previous book](http://gameprogrammingpatterns.com/){ Which I have never completed tough }, but I found that this one was by far more interesting { Probably because languages and compilers are by far more compelling to me }.
While the first part was interesting but a bit boring at some points { It may be because some things I already knew }, the second half of the book is beautifully crafted, furthermore it is a pleasure writing in pure C after all these years { especially now that I'm full on the C++ library and am reading wall of text on its abstractions almost every day }.
It has an everlasting elegance and minimality that I didn't know I was missing.

Furthermore, I must praise the book as it is a kind of book I searched for a lot. I will someday find the strength and time to read the [dragon book](https://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools), but for starters, a more practical, scoped, introduction to compilers was something that I always needed.
I've tried many books at that, but either they skip some important information for a complete beginner ( like using generated scanners and parsers ) or they lack any kind of real detail.
[Crafting Interpreters](http://craftinginterpreters.com/) has hit that sweet spot for me that I searched for so long.

On the topic at hand, as I do with most books I complete, I have to devise some kind of exciting exercise for reinforcement learning.
As always, it should help me revise the topics of the books on a more personal note, it should be practical and it should contain some chance to deepen some topics.
While sometimes it can be difficult to find the concept of such an exercise, it is straightforward that is should be some kind of compiler { a bytecode + simple virtual machine one as in the second half of the book at that }.
But the problem is: Which language should I implement?

> BTW, for those who recognized the title reference, I have a small story.
> When I was little, and Narnia was one of the new cool things, with the upcoming movie, I decided to buy the big thousand-and-spare pages book with all the chronicles in it.
> Little me was so happy, less so when the same day { as it was my birthday } I received my second and third copies as birthday's gifts { ugh }.
> Well, in the end, this was one of the most boring, annoying, poorly-written books I have read. So much that, about 200 pages in, I felt like puking at the sole idea of opening the book again.
> I'm not sure where my three copies { Oh the Irony of having 3 copies of one of my most hated books and zero copies of some of my most loved ones } are, but I hope that they're burning in hell.
> Anyway, the idea of referencing that book came to me as I vividly felt a kind of uneasiness that brought me back to that time as I was laying out the details of this upcoming exercise.

### MOVing along

We don't need another useless, badly designed, horribly implemented toy language. This was the first thing I felt when I started designing this exercise. There are many existing languages that
are perfect for a first compiler.

I was epsecially leaning on, maybe, [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)), a [Z-machine](http://www.ifwiki.org/index.php/Z-machine) or the obvious [PL\0](https://en.wikipedia.org/wiki/PL/0).
While this was the sanest choice, I had the worst of luck by accidentally { \s } remembering a [beatiful paper](https://www.cl.cam.ac.uk/~sd601/papers/mov.pdf) that I have randomly read some weeks ago.

Stupid me, then, decided that "We don't need another useless, badly designed, horribly implemented toy language" - but no one will be hurt if some average joe goes at it!
And so comes *Tummys*
<!--godomalissimo-->

## They're comin' outta the goddamn walls!

So, what the hell is a *Tummys*?
Well, *Tummys* is my will-be { or so I hope } toy language that acts as a DSL for deterministic **Turing Machines**.
There are some design seeds that I chose for *Tummys*:

* { Almost } everything is a TM.
* The basic programming paradigm is based on TMs'composition, akin to functional composition.

While bare, they were enough to start designing the language.

### How are our TMs defined

For *Tummys* we are mostly using the classic intuition of a TM with a configuration, a read-write head and a tape.
A TM reads a symbol from the current tape's cell under the read-write head at each step and writes a symbol on that cell, moves either left or right and sets to a new state. These actions are chosen in accordance with the TM's configuration.
A TM stops its execution when it reaches a halt state in its configuration.
Our tapes are one-dimensional and infinite in both directions. Each cell of the tape has either a character from the input alphabet or the blank character written on it at each step.

Mathematically, we think of our TMs using a seven-tuple model I've seen in [The Computability, Complexity & Algorithms course on Udacity](https://classroom.udacity.com/courses/ud061).
Taking from Lesson 2: Turing Machine resource pdf of the course :

> Mathematically, a Turing machine consists of:
> 1. A finite set of states. (Everything used to specify a Turing machine is finite. That is
> important.)
> 2. An input alphabet of allowed input symbols.
> 3. A tape alphabet of symbols that the read/write head can use
> 4. It also includes a transition function from a (state, tape symbol) to a (state, tape
> symbol, direction) triple. This, of course, tells the machine what to do. For every,
> possible current state and symbol that could be read, we have the appropriate
> response: the new state to move to, the symbol to write to the tape (make this same
> as the read symbol to leave it alone), and the direction to move the head relative to
> the tape. Note that we can always move the head to the right, but if the head is
> currently over the first position on the tape, then we can’t actually move left. When
> the transition function says that the machine should be left, we have it stay in the
> Q
> same position by convention.
> 5. We also have a start state. The machine always starts in the first position on the tape
> and in this state.
> 6. Finally, we have an accept state,
> 7. and a reject state. When these are reached the machine halts its execution and
> displays the final state.

Tummys uses bi-directional infinite tapes so we don't have the constraint that we can't move left at the start of the input.
This is the first time that I've seen a separate input and tape alphabet, but we use this same convention in Tummys. Furthermore, the input alphabet cannot contain the blank symbol that instead must be contained in the tape alphabet.
This is useful as this means that we can't accept input strings that have blank symbols.

The TM in Tummys works like any classic TM you can think about given these constraints.


## A, b, c, d, e ...

> This section was pretty mangled. In the beginning, this post was about a dozen times larger. I iteratively simplified *Tummys* as I was adding far too much-unneded complexity.
> This is one of the things I cut from the most.
> At the start Tummy had a kind of type system based on alphabets. Furthermore, I was designing some things around another entity that modelled languages.
> In the end, most of this was scrapped off, so that alphabets are pretty bare.
> I decided to still keep a minimal iteration of them to use in the definition of TMs as I think they can provide a useful way for the compiler to spot some logical errors.

One of the basic building blocks of *Tummys* is **alphabets**. Alphabets are a collection of unique comma-separated characters ( between a pair of single quotes ) between a pair of braces, as follows:

~~~
['0', '1'] 
~~~

This is, for example, the binary alphabet.
Alphabet with repeating characters are collapsed to the set of their unique elements:

~~~
['0', '1', '1']
~~~

This is the same alphabet as above.

Given two alphabets $$A$$ and $$B$$ Tummys provides the following operation on them:

* $$ A \cap B $$ trough the binary operator '|'
* $$ A \cup B $$ trough the binary operator '+'
* $$ A \setminus B $$ trough the binary operator '-'

Alphabets can be assigned to an identifier as follows:

~~~
#BINARY ['0', '1']
~~~

The identifier must be a sequence of characters that are elements of **[A-Z]**.
After an alphabet is assigned to an identifier, the identifier acts as an alias for the alphabet and the two can be replaced by one another without changing the meaning of the program. 
Assigning to the same identifier more than once is an error.

## Creating and executing TMs

First of all, we have to have TMs to work with for *Tummys* to make sense. While *Tummys* will provide some default TMs, it gives us the chance to define our own.
This is how a TMs definition looks like in *Tummys*:

~~~
:: odd ['0', '1'] ['0', '1']+[' '] { 
    S ['0', '1'] -> (S, '1', R),
    S [' '] -> (1, ' ', L),
    1 ['0'] -> (REJ, '0', R),
    1 ['1'] -> (ACC, '1', L)
}
~~~

This is a Turing Machine on the alphabet **['0', '1']** ( Binary ) that accepts if the input string represents an odd number.
Breaking this down we have:

* The "::" define operator that opens the definition for a TM.
* The identifier that will be used to reference the TM { odd in this case }. An identifier starts with a lowercase letter and is composed of any number of characters.
* The input alphabet that the TM will accept, in this case ['0', '1'].
* The tape alphabet, defined as the union between two alphabets $$ A $$ and $$ B $$, where $$ \card {B} = 1 $$ and $$ x \in B $$ is the character representing the blank character on the tape {e.g the absence of a character on the tape }. It is an error to have the blank character appear in the input alphabet or to have a tape alphabet that isn't of the form $$ A + [a] $$.
* The transition table defined as a block starting with "{" and ending with "}" with a series of comma-separated transition declarations in it.

A transition declaration has the form:

~~~
state alphabet -> transition
~~~

Where:

* "state" is a state identifier of the form [a-zA-z0-9]
* "alphabet" is an alphabet $$ A $$ such that $$ A \subseteq B$$ where $$ B $$ is the tape alphabet of the TM
* The "->" transition operator
* A triplet of values composed as (new-state, write, direction) where "new-state" is the state identifier that refers to the state that the TM will transition to, "write" is a character $$c \in B$$, where $$ B $$ is the tape alphabet of the TM and direction is a symbol $$ s \in \set {R, L, ^} $$, defining the direction that the head should move. $$ R $$ move the head to the right while $$ L $$ move the head to the left.


The special move direction symbol '^', leaves the head where it currently is.
A transition declaration of the form:

~~~
N A -> (N', s, ^)
~~~

is equivalent to:

~~~
N A   -> (N'', s, >)
N'' A' -> (N', _, <)
~~~

where $$ A \subseteq A^'$$, $$ s \in A^' $$ and $$ A' $$ is the tape alphabet of the TM.

This is mostly equivalent to actually having the head be able to stay put in the first place.
But there is an edge case where the character to the right of the head is not a valid character in the alphabet, meaning that, as per Tummys' rules, the machine would halt immediately. If the head was effectively never moved that character may not have been encountered.
A way to partially resolve this is to have the head move to the opposite direction of the last made movement and then to the opposite of that.
This is so that we can be sure that we will not encounter an unknown character as we pass in a place where the head has already been.

Unfortunately, this too has an edge case where there was no prior movement to a stay-put transition. In that case, a direction must be chosen, at random or in a fixed way.
Tummys, for consistency reasons, will always expand to a right-left move.

A special write character _ is used to denote that no character has to be written, leaving the character that was already there.
For example the transiction declaration:

~~~
S ['0', '1'] -> (S, _, >)
~~~

is equivalent to the two transiction declaration:

~~~
S['0'] -> (S, '0', >),
S['1'] -> (S, '1', >)
~~~

This is true whenever the _ symbol appears in a transition declaration, independent of where it appears.
This means that a delegating transition to a parametrized Turing machine ( which we will see in a bit ) on which the _ is used as a parameter, such as the following:

~~~
N ['0', '1'] -> M<_> -> (...)
~~~

is expanded to:

~~~
N ['0'] -> M<'0'> -> (...),
N ['1'] -> M<'1'> -> (...),
~~~

S, REJ and ACC are special state identifier referring to the starting state, the reject state and the accepting state. 
At least a transition declaration from state S must be present in each TM declaration. The REJ and ACC state are implicitly defined.
While some transition declaration may be given for them, the TM will halt as soon as they will be reached making those declarations useless.

At least one transition to the REJ or ACC states must be present in any TM definition.

A state identifier is implicitly defined the first time it is encountered in a transition table for the given TM.
If a transiction declaration from $$A$$ on the alphabet $$X$$ comes after a transiction declaration from $$A$$ on the alphabet $$Y$$ and $$X\capY$$ is non-empty, the transiction from $$A$$ on $$\allz\elemX\capY$$ wil follow $$B$$ the transiction declaration $$A$$ on $$X$$.

When a character for which no state transition exists is met, the TM halts on the rejecting state immediately.

The second type of transition declaration is the delegating transition of the form:

~~~
state alphabet -> expression -> transition
~~~

Where "state", "alphabet" and "transiction" are defined as above. "expression" is a valid Tummy expression ( e.g given a sequence of TMs $$M1, M2, ... MN$$, ~~~ M1 M2 ... MN ~~~ ).
An image can help us understand this concept.

![delegating transiction]({{ "/assets/TUMMY_delegating_transiction.jpg" | absolute_url }})

Basically, we have this TM $$M$$ that is executing on the tape $$T$$. $$M$$ is currently at state $$n$$ when it encounters a delegating transition. The transition is delegated to $$M'$$, which is a TM
that writes two 5s to the tape and the halts. $$M'$$ is executed on the original tape $$T$$ with its head starting at the position where the head of $$M$$ was when it encountered the delegating transition.
After $$M'$$ halts the original TM continues its execution on tape $T$ with its head moved to where $$M'$$ left it.

This is a shortcut for the following situation.
Let's say we have the following Tummy's TM definition:

~~~
:: M' A A' {
    S A''' -> (1, a, >),
    1 A''' -> (2, a, >),
    2 A''' -> (ACC, a, >),
}
~~~

This is a TM that moves the tape to the right three times and writes three times the character $$a\elemA''$$.
Now let's say we are defining the TM $$M$$, that at some point in its execution has to do the same steps as $$M'$$:

~~~
:: M A A' {
    ...
    N A'' -> M' -> (N',...),
    ...
~~~

This is equivalent to expanding the transition table of $$M'$$ in $$M$$ at the point of the delegating transition as follows:

~~~
:: M A A' {
    ...
    N A''   -> (M'S, _, ^),
    M'S A''' -> (1, a, >)
    M'1 A''' -> (M'2, a, >),
    M'2 A''' -> (N'', a, >),
    N'' A' -> (N', ...),
    ...
~~~
    
We basically inline the TM that is the result of expression'. Each transition that would transition to $$ACC$$ or $$REJ$$ transitions to $$N''$ instead.
First, the machine transitions to the initial state of the delegated to TM.
We use an intermediary transition that leaves the tape and head unchanged instead of expanding the original delegating transition directly to the starting state transition declaration of the delegated to machine to avoid problems with multiple starting state declaration or multiple delegating declarations on different alphabets.
$$N''$$ is an intermediate state that transition that for every character $c\elemA'$$ will transition to $$N'$$ with the transition triple $$(N', ...).

Delegating Transition are a way of composing TMs from other TMs.
Given a TM $$M$$ with a tape alphabet $$A+[a]$$ that has one or more delegating transictions to other TMs $$M_1, M_2, ... M_n$$ with tape alphabets $$A_1+[a_1], A_2+[a_2], ... A_n+[a_n]$$,
the tape alphabet of $$M$$ is considered to be $$[A]+(A_1+[a_1])+(A_2+[a_2])+ ... +(A_n+[a_n]) + [a]$$ where $$[a]$$ is the blank symbol for $$M$$.

Lastly, Tummys supports a second type of TMs, parametrized TMs.
Parametrized TMs are inspired by C++ templates and C's macros and are a mean to generate a series of similar TMs that differ in some way.
For example, let's say we are making a program that requires two TMs: $$M$$ that substitutes the first blank character for the character '#' and $$M'$$ that substitutes the first blank character for the character '*'.
A way they can be defined is as follows:

~~~
:: M A A+['#']+[a] {
    S A -> (S, _, >),
    S [a]-> (ACC, '#', >)
}

:: M' A A+['*']+[a] {
    S B -> (S, _, >),
    S [b] -> (ACC, '*', >)
}
~~~

As you can see they differ only in what they append to the tape. Instead of replicating the code, we can parametrize a single TM as follows:

~~~

:: <c> append A A+[c]+[a] {
     S A -> (S, _, >),
    S [a]-> (A, c, >)
}

append can then be used in an expression as follows:

~~~
append<'#'>
~~~

The compiler will generate a new definition replacing all appearances of c with "'#'".
They are more C's macros than C++ templates apart from the syntax.

## Expressions

In Tummys there exist a single expression type, The conjunction of TMs that has the form:

~~~
M M1 M2 M3 M4 ... MN
~~~

Each expression, no matter where it appears, is firstly compacted and then compiled.
The process for which an expression is compacted is through the conjunction of TMs.

Given any two turing machines $$M=(Q, \sigma, \theta, \lambda, q_0, q_a, q_r)$$ and $$M^'=(Q^', \sigma^', \theta^', \lambda^', q_0^', q_a^', q_r^')$$, for which $$Q$$ and $$Q^'$$ are disjoint, their conjuction $$M\cupM^'$$ is the turing machine
$$M^\cup=(Q\cupQ^\cupQ^\cup, \sigma\cup\sigma^', \theta\cup\theta^', \lambda\cup\lambda^'\cup\lambda^\cup-\lambda_{ar}, q_0, q_a^', q_r^')$$, where $$Q^\cup$$ is the set of additional states that are needed for the conjuction
and $$\lambda^\cup$$ is a transiction table with the additional entries needed for the conjuction and $$lambda_{ar}$$ is a transiction table $$\subset\lambda$$ that contains only the transiction to the accepting or rejecting states.

Each transition to the rejecting or accepting state in $$\lambda$$ is substituted to a transition to a state $$q_\lambda^\cup$$ that start the process of rewinding the tape.

When we go from a state $$\elemQ$$ to a state $$\elemeQ^'$$ a series of additional steps are added to move the tape head to the leftmost character that is not blank symbol so that the execution of the states of the second TM start from the perceived start of the tape.
This means that the conjunction of a Turing machine $$M$$ that moves $$n$$ elements to the right without modifying the tape writes a blank symbol and then moves one element to the right, effectively moves the perceived start of the tape by $$n$$ cells to the right.

An expression of the form:

~~~
M M1 M2 M3 M4 ... MN
~~~


is actually equivalent to $$( ( ( ( (M_N\cup ...)\cupM_4)\cup M_3 )\cup M_2 )\cup M_1 )\cup M )$$

In a more practical way, we follow the following process.

Let's suppose that we have the following two TMs:

~~~
:: M [0, 1] [0, 1]+[' '] {
    S [0]   -> (ACC, '0', >)
    S [1]   -> (ACC, '0', >)
    S [' '] -> (ACC, '0', >)
}

:: M' [0, 1] [0, 1]+[' '] {
    S [0]   -> (1, '0', >)
    S [1]   -> (1, '1', >)
    S [' '] -> (1, ' ', >)
    1 [0]   -> (ACC, '1', >)
    1 [1]   -> (ACC, '1', >)
    1 [' '] -> (ACC, '1', >)
}
~~~

The expression:

~~~
M' M
~~~

is compacted to a new Turing machine $$M^{''}$$ by the following process.

First, we change the name of their states so that the two set of states are disjoint ( this is true for implicitly declared states and transitions like the REJ state).

~~~
:: M [0, 1] [0, 1]+[' '] {
    S [0]   -> (ACC, '0', >),
    S [1]   -> (ACC, '0', >),
    S [' '] -> (ACC, '0', >)
}

:: M' [0, 1] [0, 1]+[' '] {
    S' [0]   -> (1, '0', >),
    S' [1]   -> (1, '1', >),
    S' [' '] -> (1, ' ', >),
    1 [0]   -> (ACC', '1', >),
    1 [1]   -> (ACC', '1', >),
    1 [' '] -> (ACC', '1', >)
}
~~~

then start defining $$M^{''}$$ with the transiction table of $$M$$:

~~~
:: M [0, 1] [0, 1]+[' '] {
    S [0]   -> (ACC, '0', >),
    S [1]   -> (ACC, '0', >),
    S [' '] -> (ACC, '0', >)
}

:: M' [0, 1] [0, 1]+[' '] {
    S' [0]   -> (1, '0', >),
    S' [1]   -> (1, '1', >),
    S' [' '] -> (1, ' ', >),
    1 [0]   -> (ACC', '1', >),
    1 [1]   -> (ACC', '1', >),
    1 [' '] -> (ACC', '1', >)
}

:: M'' [0, 1] [0, 1]+[' '] {
    S [0]   -> (ACC, '0', >),
    S [1]   -> (ACC, '0', >),
    S [' '] -> (ACC, '0', >)
}
~~~

Then we expand $$M''$$ so that each transiction to $$ACC$$ or $$REJ$$ transitions to a new states that starts the rewind process and add the necessary transiction declarations to rewind the tape:

~~~
:: M [0, 1] [0, 1]+[' '] {
    S [0]   -> (ACC, '0', >),
    S [1]   -> (ACC, '0', >),
    S [' '] -> (ACC, '0', >)
}

:: M' [0, 1] [0, 1]+[' '] {
    S' [0]   -> (1, '0', >),
    S' [1]   -> (1, '1', >),
    S' [' '] -> (1, ' ', >),
    1 [0]   -> (ACC', '1', >),
    1 [1]   -> (ACC', '1', >),
    1 [' '] -> (ACC', '1', >)
}

:: M'' [0, 1] [0, 1]+[' '] {
    S [0]   -> (REWIND, '0', >),
    S [1]   -> (REWIND, '0', >),
    S [' '] -> (REWIND, '0', >),
    
    REWIND [0, 1, ' '] -> (REWIND2, _, <),
    REWIND2 [0, 1] -> (REWIND2, _, <),
    REWIND2 [' '] -> (ACC, _, >)
}
~~~

The rewind TM first moves once to the opposite direction of the last movement ( or stays put if the last movement was '^' or its equivalent expansion), then moves to the left until a blank character is encountered.
On the first blank character, the tape gets moved to the right to the last encountered non-blank characters and the rewind machine halts.

We then expands $$M''$$ alphabets with the alphabet from $$M^'$$:

~~~
:: M [0, 1] [0, 1]+[' '] {
    S [0]   -> (ACC, '0', >),
    S [1]   -> (ACC, '0', >),
    S [' '] -> (ACC, '0', >)
}

:: M' [0, 1] [0, 1]+[' '] {
    S' [0]   -> (1, '0', >),
    S' [1]   -> (1, '1', >),
    S' [' '] -> (1, ' ', >),
    1 [0]   -> (ACC', '1', >),
    1 [1]   -> (ACC', '1', >),
    1 [' '] -> (ACC', '1', >)
}

:: M'' [0, 1]+[0, 1] [0, 1]+[' ']+[0, 1]+[' '] {
    S [0]   -> (REWIND, '0', >),
    S [1]   -> (REWIND, '0', >),
    S [' '] -> (REWIND, '0', >)
    
    REWIND [0, 1, ' '] -> (REWIND2, _, <),
    REWIND2 [0, 1] -> (REWIND2, _, <),
    REWIND2 [' '] -> (ACC, _, >)
}
~~~

We add the states and transictions of $$M'$$ to $$M'$$ and connect the accepting state of the rewind machine to the starting state of $$M'$$:

~~~
:: M [0, 1] [0, 1]+[' '] {
    S [0]   -> (ACC, '0', >),
    S [1]   -> (ACC, '0', >),
    S [' '] -> (ACC, '0', >)
}

:: M' [0, 1] [0, 1]+[' '] {
    S' [0]   -> (1, '0', >),
    S' [1]   -> (1, '1', >),
    S' [' '] -> (1, ' ', >),
    1 [0]   -> (ACC', '1', >),
    1 [1]   -> (ACC', '1', >),
    1 [' '] -> (ACC', '1', >)
}

:: M'' [0, 1]+[0, 1] [0, 1]+[' ']+[0, 1]+[' '] {
    S [0]   -> (REWIND, '0', >),
    S [1]   -> (REWIND, '0', >),
    S [' '] -> (REWIND, '0', >),
    
    REWIND [0, 1, ' '] -> (REWIND2, _, <),
    REWIND2 [0, 1] -> (REWIND2, _, <),
    REWIND2 [' '] -> (S', _, >),
    
    S' [0]   -> (1, '0', >),
    S' [1]   -> (1, '1', >),
    S' [' '] -> (1, ' ', >),
    1 [0]   -> (ACC', '1', >),
    1 [1]   -> (ACC', '1', >),
    1 [' '] -> (ACC', '1', >)
}
~~~

And with this, we are done. $$ACC'$$ is the new accepting state. In this particular case, we have the TM that, on any valid input, writes '0' in the first cell, and '1' in the second cell then halts.

The unary expression of the form:

~~~
M
~~~

is compacted to $$M$$ itself.

A compacted expression is evaluated to the final tape state resulting from the application of the empty input on the resulting TM.
This is a remnant of a previous iteration of Tummys and is now unnecessary.

### Special Turing Machines

Tummys provide a series of inbuilt Turing Machines that enables special capabilities or streamlines some process.

## String Literals

In Tummys, a string literal is a collection of characters between two double quotes:

~~~
"Tummys"
"Hello World"
"Here I am"
~~~

String literals are themselves Turing Machines.
In fact, given any string literal $$S$$, we build a corresponding TM $$M$$ that writes something to its tape and halts independent of its input tape.

Any string literal of the form "$$c_1c_2c_3c_4...c_n$$" is equivalent to a turing machine of the form:

~~~

:: M A A+[a] {
    S A+[a] -> (1, c1, >),
    1 A+[a] -> (2, c2, >),
    2 A+[a] -> (3, c3, >),
    3 A+[a] -> (4, c4, >),
    
    ....
    
    N A+[a] -> (ACC, cn, >)
}


it follows that the unary expression:

~~~
M
~~~ 

is equivalent to the expression of the form:

~~~
S'
~~~

where $$M$$ is the Turing machine that writes $$S$$ to its tape and $$S'$$ is a string literal of the form "S".

String literals are the idiomatic way of building input tapes. An expression of the form:

~~~
M1 ... MN S'
~~~

where $$M1 ... MN$$ are Turing Machines and $$S^'$$ is a string literal of the form $$"S"$$, is equivalent to running the compacted Turing machine:

~~~
M'
~~~

compacted from the expression $$M1 ... MN$$ on the input tape which has written $$S$$ on its cells, starting at some cell $$c_i$$, with the head of $$M'$$ initially pointing at $$c_i$$.

The second idiomatic use of a string literal is to streamline input-state independent write operation on some tape $$T$$ through the use of a delegating transition:

~~~
...
N A+[a] -> "Hello World" -> (N', _, <)
...
~~~
 
is the same as adding a series of states that writes "Hello World" on the TMs tape starting at the cell that is currently pointed to by the head, ending on the cell containing the last written character, "d" in this case,
and transitioning to state $$N'$$.

## Side Effect TMs

# I/O TMs

A sequence of special Turing machine is provided for input/output purposes.

The $$I$$ TM is a special Turing machine that reads a line of input from the user and writes it to its tape halting afterwards.
The $$IC$$ TM is a special Turing machine that reads a single character of input from the user and writes it to its tape halting afterwards.

The $$O$$ TM is a special Turing machine that writes all the contents of its tape, starting at the leftmost non-blank character, to the output and then halts, leaving its tape and read-write head unchanged.

# State machine

The state machine $$S$$ is a special Turing machine that, when composed with another machine, writes 'A' to its tape if the machine it was composed with, when run, ends on the accepting state and 'R' otherwise.
As with everything in Tummys, non-halting Turing machines are considered undefined behaviour.

# Debugging and Inspecting machines

This is a category of machines that I'm still unsure about. They would provide a way to attach a debugger to a TM so that information about its execution can be shown to the user.
For example, the $$D$$ machine would run a given TM until the $$D$$ starting state is met, outputting information about the tape, state and followed transition at each step to the user.

Inspecting machines would be similar but would give information about a machine configuration, alphabet or particular transition.
For example, $$\delta$$ would print to the user the expanded transition table of a given machine.

I'm unsure about it as it may be better to provide such functionalities through the Tummys environment ( eg. its compiler or interpreter or other tools ) than directly as language constructs.

### Some examples

Here there are some examples of valid Tummys programs. I prepared a [Github gist] with some snippets of code that can be run on [turing-machine.io](http://turingmachine.io/) to graphically see the TMs in action.
For practical reasons, not every state is expanded completely ( for example a state that in Tummys would be expanded to states over the whole alphabet may be simplified on the blank character that is the character we are expecting to be there ).

# Is divisible by 10?

~~~
#DIGITS ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']

:: <A, A', a> toEnd A A'+[a] {
    S A' -> (S, _, R),
    S [a] -> (ACC, _, ^),
}

:: divisibleBy10 DIGITS DIGITS+[' '] {
    S DIGITS+[' '] -> toEnd<DIGITS, DIGITS, ' '> -> (1, _ ,L),
    1 [0] -> (ACC, _, R),
    1 DIGITS-[0] -> (REJ, _, R)
}

O S divisibleBy10 I
~~~

This is a valid Tummy program that takes as input a line from the user and writes A back if the input represents a number $$n\elemZ^*$$ such that $$n$$ is divisible by 10 or R otherwise.
We first define the #DIGITS alphabet that contains the digits character from which a positive natural number can be formed. While alphabet and languages were mostly removed from Tummys now, it is still important to use basic alphabets so that we naturally reject an incorrect input.
In fact, one of the rules of Tummys is that any unrecognized character that is met automatically halts the TM execution on the rejecting state.

toEnd is a convenience TM, that is designed to be delegated to, that moves a TM head to the right until the first blank character is met, on which it stops.

We then have divisibleBy10 which is the main TM of this program. It works on the DIGITS alphabet and accepts any input on the DIGITS alphabet that is divisible by ten. This is done by going to the end of the input and then checking if it is a '0'.
If the input has unknown characters the machine will reject as soon as that is encountered.

This is all wrapped in the OSI idiom ( $$O S M_1 ... M_N I$$ ) that is Tummys way of checking if a given composition of Turing machines accepts or reject a given input.

## Right shift

~~~
:: <A, A', a> toEnd A A'+[a] {
    S A' -> (S, _, R),
    S [a] -> (ACC, _, ^),
}

:: <A, A', a, c> write A A'+[a] {
    S A'+[a] -> (ACC, c, ^)
}

:: <A, A', a> moveRight A A'+[a] {
    S A'+[a] -> (ACC, _, >)
}
    
:: <A, A', a, e1, e2> compose A A'+[a] {
    S A'+[a] -> e1 -> (1, _, ^),
    1 A'+[a] -> e2 -> (ACC, _, ^)
}

:: <A, A', a, e, c> hold A A'+[a] {
    S A'+[a] -> e -> (ACC, c, ^)
}

:: <A, A', a> rightShift A A'+[a] {
    S A' -> toEnd<A, A', a> -> (1, _, L),
    1 A' -> hold<A, A', a, compose<A, A', a, write<A, A', a, a>, moveRight<A, A', a>>, _> -> (1, _, L),
    1 [a] -> (ACC, _, ^),
    2 [a] -> (1, a, L)
}
~~~

This is a bit more abstract and difficult to untangle. Here we see the general programming paradigm of parametrized TMs and the functional aspect of composition in Tummys.
You can look at an example of a right shift machine on the binary alphabet in the [Github gist]. I suggest you do so to better understand the idioms we are using here.

Let's look at this in small bits.
First, we have toEnd, we already know her ( I'm not sure why but I think of Turing machines as females ) from the previous exercise.

Then we have the first new bit, the write TM. This is a parametrized TM that is perfect to meet first as it is easy to grasp while providing an important insight into what parametrized TMs and delegating transition are designed for: Generalizing and reusing patterns.
Writing something to a cell is a part of the core of a TM. write does exactly that. 
With the way delegating transition are written in Tummys, we can use the transition after the delegation to move the head right or left. This effectively let us use write as a way to define a transition.
While this is not needed in a concrete case, encapsulating this behaviour in a TM will provide us with a way of composing expression that builds transitions.

This is the first important insight on how TMs as parametrizable, expandable, first-class citizens will let us provide a functional approach to the mutating, stateful world of Turing Machines.

Write on itself has no particular use. Let's move a little bit down.

We encounter moveRight. Again this is a parametrized TM that encapsulates one of the core behaviours of a Turing machine.
It does exactly what it says, move the head to the right.

Now meet compose. Compose sequentially executes two expressions. The most important difference between the composition of compose and the compacting of an expression is that there is no rewind in-between.
This means that with compose we are effectively creating two sequential states that do some operation without changing the state of the head in between.
This is the core pattern of transitioning between two states in a Turing machine.

With compose we can, for example, and as will see later, compose the two core behaviour of writing something to the tape and moving right into a single machine.
But why go to such length for something that is built into the language?

This, in fact, does not make sense without another piece of this program. hold.
hold is the first higher-level pattern that we are meeting in Tummys.

First, what pattern is hold?
hold is a pattern where we remember a value from a state to the next.

This does not make sense without an example. A small part of shifting a whole input one to the right is to blank a cell and rewrite its character to the right.
In a Turing machine this is doable by writing a series of states that transition in the same way for each character we want to remember.

![delegating transiction]({{ "/assets/TUMMY_hold.png" | absolute_url }})

By doing this we can "remember" the last met character and write it again.
hold is doing exactly that. It parametrizes an expression to delegate to and then writes the character that was passed to it in the next transition. By staying put we can model the direction and states for the end transition with the transition of hold-caller delegating transition.
Using hold with _ as the "c" parameter, we can expand a series of equal paths to the same state that varies in the character written in the last transition over an entire alphabet.

As this may not make too much sense we will look at the expansion that happens in the rightShift TM.
We'll see a concrete case with the following expression:

~~~
rightShift<['0', '1'], ['0', '1'], ' '>
~~~

First we concretize rightShift with the given parameters:

~~~
::  rightShift ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1'] -> toEnd<['0', '1'], ['0', '1'], ' '> -> (1, _, L),
    1 ['0', '1'] -> hold<['0', '1'], ['0', '1'], ' ', compose<['0', '1'], ['0', '1'], ' ', write<['0', '1'], ['0', '1'], ' ', ' '>, moveRight<['0', '1'], ['0', '1'], ' '>>, _> -> (1, _, L),
    1 [' '] -> (ACC, _, ^)
    2 [' '] -> (1, ' ', L)
}
~~~

We won't expand the toEnd delegation as that is easily understood.

As we know, in a transition declaration the first thing that gets expanded is the _ symbol.

~~~
::  rightShift ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1'] -> toEnd<['0', '1'], ['0', '1'], ' '> -> (1, _, L),
    
    1 ['0'] -> hold<['0', '1'], ['0', '1'], ' ', compose<['0', '1'], ['0', '1'], ' ', write<['0', '1'], ['0', '1'], ' ', ' '>, moveRight<['0', '1'], ['0', '1'], ' '>>, '0'> -> (1, _, L),
    1 ['1'] -> hold<['0', '1'], ['0', '1'], ' ', compose<['0', '1'], ['0', '1'], ' ', write<['0', '1'], ['0', '1'], ' ', ' '>, moveRight<['0', '1'], ['0', '1'], ' '>>, '1'> -> (1, _, L),
    
    1 [' '] -> (ACC, ' ', ^)
    
    2 [' '] -> (1, ' ', L)
}
~~~

We start expanding the parametrized TM that we delegate too. In the case of nested parametrized TMs we start with the outermost one.

~~~
:: hold ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1']+[' '] -> compose<['0', '1'], ['0', '1'], ' ', write<['0', '1'], ['0', '1'], ' ', ' '>, moveRight<['0', '1'], ['0', '1'], ' '>> -> (ACC, '0', ^)
}

:: hold ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1']+[' '] -> compose<['0', '1'], ['0', '1'], ' ', write<['0', '1'], ['0', '1'], ' ', ' '>, moveRight<['0', '1'], ['0', '1'], ' '>> -> (ACC, '1', ^)
}

::  rightShift ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1'] -> toEnd<['0', '1'], ['0', '1'], ' '> -> (1, _, L),
    
    1 ['0'] -> (holdS0, _, ^),
    
    holdS0 ['0', '1']+[' '] -> compose<['0', '1'], ['0', '1'], ' ', write<['0', '1'], ['0', '1'], ' ', ' '>, moveRight<['0', '1'], ['0', '1'], ' '>> -> (holdIntermediate0, '0', ^),
    holdIntermediate0 ['0', '1']+[' '] -> (1, _, L),
    
    1 ['1'] -> (holdS1, _, ^),
    
    holdS1 ['0', '1']+[' '] -> compose<['0', '1'], ['0', '1'], ' ', write<['0', '1'], ['0', '1'], ' ', ' '>, moveRight<['0', '1'], ['0', '1'], ' '>> -> (holdIntermediate1, '1', ^),
    holdIntermediate1 ['0', '1']+[' '] -> (1, _, L),

    1 [' '] -> (ACC, ' ', ^)
    
    2 [' '] -> (1, ' ', L)
}
~~~

I have added the two hold definition that were created to help in the visualization. Please refer back to the section on delegating declaration if the expansion is not clear.
We now expand the next outermost parametrized TM, compose.

~~~
:: compose ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1']+[' '] -> write<['0', '1'], ['0', '1'], ' ', ' '> -> (1, _, ^),
    1 ['0', '1']+[' '] -> moveRight<['0', '1'], ['0', '1'], ' '> -> (ACC, _, ^)
}

::  rightShift ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1'] -> toEnd<['0', '1'], ['0', '1'], ' '> -> (1, _, L),
    
    1 ['0'] -> (holdS0, _, ^),
    
    holdS0 ['0'] -> (composeS0, _, ^),
    
    composeS0  ['0', '1']+[' '] -> write<['0', '1'], ['0', '1'], ' ', ' '> -> (compose01, _, ^),
    compose01 ['0', '1']+[' '] -> moveRight<['0', '1'], ['0', '1'], ' '> -> (composeIntermediate0, _, ^)
    
    composeIntermediate0 ['0', '1']+[' '] -> (holdIntermediate0, _, ^)
    
    holdIntermediate0 ['0', '1']+[' '] -> (1, _, L),
    
    1 ['1'] -> (holdS1, _, ^),
    
    holdS1 ['1'] -> (composeS1, _, ^),
    
    composeS1  ['0', '1']+[' '] -> write<['0', '1'], ['0', '1'], ' ', ' '> -> (compose11, _, ^),
    compose11 ['0', '1']+[' '] -> moveRight<['0', '1'], ['0', '1'], ' '> -> (composeIntermediate1, _, ^)
    
    composeIntermediate1 ['0', '1']+[' '] -> (holdIntermediate1, _, ^)
    
    holdIntermediate1 ['0', '1']+[' '] -> (1, _, L),

    1 [' '] -> (ACC, ' ', ^)
    
    2 [' '] -> (1, ' ', L)
}
~~~

We do this a few times until all the delegting transiction are expanded:

~~~
:: write ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1']+[' '] -> (ACC, ' ', ^)
}

:: moveRight ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1']+[' '] -> (ACC, _, >)
}

::  rightShift ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1'] -> toEnd<['0', '1'], ['0', '1'], ' '> -> (1, _, L),
    
    1 ['0'] -> (holdS0, _, ^),
    
    holdS0 ['0'] -> (composeS0, _, ^),
    
    composeS0  ['0', '1']+[' '] -> (writeS0, _, ^),
    
    writeS0 ['0', '1']+[' '] -> (writeIntermediate0, ' ', ^),

    writeIntermediate0 ['0', '1']+[' '] -> (compose01, _, ^)
    
    compose01 ['0', '1']+[' '] -> (moveRightS0, _, ^)
    
    moveRightS0 ['0', '1']+[' '] -> (moveRightIntermediate0, _, >)
    
    moveRightIntermediate0 ['0', '1']+[' '] -> (composeIntermediate0, _, ^)
    
    composeIntermediate0 ['0', '1']+[' '] -> (holdIntermediate0, _, ^)
    
    holdIntermediate0 ['0', '1']+[' '] -> (1, _, L),
    
    1 ['1'] -> (holdS1, _, ^),
    
    holdS1 ['1'] -> (composeS1, _, ^),
    
    composeS1  ['0', '1']+[' '] -> (writeS1, _, ^),
    
    writeS1 ['0', '1']+[' '] -> (writeIntermediate1, ' ', ^),

    writeIntermediate1 ['0', '1']+[' '] -> (compose11, _, ^)
    
    compose11 ['0', '1']+[' '] -> (moveRightS1, _, ^)
    
    moveRightS1 ['0', '1']+[' '] -> (moveRightIntermediate1, _, >)
    
    moveRightIntermediate1 ['0', '1']+[' '] -> (composeIntermediate1, _, ^)
    
    composeIntermediate1 ['0', '1']+[' '] -> (holdIntermediate1, _, ^)
    
    holdIntermediate1 ['0', '1']+[' '] -> (1, _, L),

    1 [' '] -> (ACC, ' ', ^)
    
    2 [' '] -> (1, ' ', L)
}
~~~

We now expand the _ and ^ symbols to have our final TM:

~~~
::  rightShift ['0', '1'] ['0', '1']+[' '] {
    S ['0', '1'] -> toEnd<['0', '1'], ['0', '1'], ' '> -> (1, _, L),
    
    1 ['0'] -> (stay10, '0', >),
    stay10 ['0'] -> (holdS0, '0', <)
    stay10 ['1'] -> (holdS0, '1', <)
    stay10 [' '] -> (holdS0, ' ', <)
    
    holdS0 ['0'] -> (stayHoldS0, '0', >),
    stayHoldS0 ['0'] -> (composeS0, '0', <)
    stayHoldS0 ['1'] -> (composeS0, '1', <)
    stayHoldS0 [' '] -> (composeS0, ' ', <)
    
    composeS0  ['0'] -> (stayComposeS0, '0', >),
    composeS0  ['1'] -> (stayComposeS0, '1', >),
    composeS0  [' '] -> (stayComposeS0, ' ', >),
    
    stayComposeS0 ['0'] -> (writeS0, '0', <)
    stayComposeS0 ['1'] -> (writeS0, '1', <)
    stayComposeS0 [' '] -> (writeS0, ' ', <)
    
    writeS0 ['0', '1']+[' '] -> (stayWriteS0, ' ', >),
    
    stayWriteS0 ['0'] -> (writeIntermediate0, '0', <)
    stayWriteS0 ['1'] -> (writeIntermediate0, '1', <)
    stayWriteS0 [' '] -> (writeIntermediate0, ' ', <)
    
    writeIntermediate0 ['0'] -> (stayWriteIntermediate0, '0', >)
    writeIntermediate0 ['1'] -> (stayWriteIntermediate0, '1', >)
    writeIntermediate0 [' '] -> (stayWriteIntermediate0, ' ', >)
    
    stayWriteIntermediate0 ['0'] -> (compose01, '0', <)
    stayWriteIntermediate0 ['1'] -> (compose01, '1', <)
    stayWriteIntermediate0 [' '] -> (compose01, ' ', <)
    
    compose01 ['0'] -> (stayCompose01, '0', >)
    compose01 ['1'] -> (stayCompose01, '1', >)
    compose01 [' '] -> (stayCompose01, ' ', >)
    
    stayCompose01 ['0'] -> (moveRightS0, '0', <)
    stayCompose01 ['1'] -> (moveRightS0, '1', <)
    stayCompose01 [' '] -> (moveRightS0, ' ', <)
    
    moveRightS0 ['0'] -> (moveRightIntermediate0, '0', >)
    moveRightS0 ['1'] -> (moveRightIntermediate0, '1', >)
    moveRightS0 [' '] -> (moveRightIntermediate0, ' ', >)
    
    moveRightIntermediate0 ['0'] -> (stayMoveRightIntermediate0, '0', >)
    moveRightIntermediate0 ['1'] -> (stayMoveRightIntermediate0, '1', >)
    moveRightIntermediate0 [' '] -> (stayMoveRightIntermediate0, ' ', >)
    
    stayMoveRightIntermediate0 ['0'] -> (composeIntermediate0, '0', <)
    stayMoveRightIntermediate0 ['1'] -> (composeIntermediate0, '1', <)
    stayMoveRightIntermediate0 [' '] -> (composeIntermediate0, ' ', <)
    
    composeIntermediate0 ['0'] -> (stayComposeIntermediate0, '0', >)
    composeIntermediate0 ['1'] -> (stayComposeIntermediate0, '1', >)
    composeIntermediate0 [' '] -> (stayComposeIntermediate0, ' ', >)
    
    stayComposeIntermediate0 ['0'] -> (holdIntermediate0, '0', <)
    stayComposeIntermediate0 ['1'] -> (holdIntermediate0, '1', <)
    stayComposeIntermediate0 [' '] -> (holdIntermediate0, ' ', <)
    
    holdIntermediate0 ['0'] -> (1, '0', L),
    holdIntermediate0 ['1'] -> (1, '1', L),
    holdIntermediate0 [' '] -> (1, ' ', L),
    
    1 ['1'] -> (stay11, '1', <),
    
    stay11 ['0'] -> (holdS1, '0', >)
    stay11 ['1'] -> (holdS1, '1', >)
    stay11 [' '] -> (holdS1, ' ', >)
    
    holdS1 ['1'] -> (stayHoldS1, '1', >),
    
    stayHoldS1 ['0'] -> (holdS1, '0', <)
    stayHoldS1 ['1'] -> (holdS1, '1', <)
    stayHoldS1 [' '] -> (holdS1, ' ', <)
    
    composeS1  ['0'] -> (stayComposeS1, '0', >),
    composeS1  ['1'] -> (stayComposeS1, '1', >),
    composeS1  [' '] -> (stayComposeS1, ' ', >),
    
    stayComposeS1 ['0', '1']+[' '] -> (writeS1, '0', <)
    stayComposeS1 ['0', '1']+[' '] -> (writeS1, '1', <)
    stayComposeS1 ['0', '1']+[' '] -> (writeS1, ' ', <)
    
    writeS1 ['0', '1']+[' '] -> (stayWriteS1, ' ', >),
    
    stayWriteS1 ['0'] -> (writeIntermediate1, '0', <)
    stayWriteS1 ['1'] -> (writeIntermediate1, '1', <)
    stayWriteS1 [' '] -> (writeIntermediate1, ' ', <)

    writeIntermediate1 ['0'] -> (stayWriteIntermediate1, '0', >)
    writeIntermediate1 ['1'] -> (stayWriteIntermediate1, '1', >)
    writeIntermediate1 [' '] -> (stayWriteIntermediate1, ' ', >)
    
    stayWriteIntermediate1 ['0'] -> (compose11, '0', <)
    stayWriteIntermediate1 ['1'] -> (compose11, '1', <)
    stayWriteIntermediate1 [' '] -> (compose11, ' ', <)
    
    compose11 ['0'] -> (stayCompose11, '0', >)
    compose11 ['1'] -> (stayCompose11, '1', >)
    compose11 [' '] -> (stayCompose11, ' ', >)
    
    stayCompose11 ['0'] -> (moveRightS1, '0', <)
    stayCompose11 ['1'] -> (moveRightS1, '1', <)
    stayCompose11 [' '] -> (moveRightS1, ' ', <)
    
    moveRightS1 ['0'] -> (moveRightIntermediate1, '0', >)
    moveRightS1 ['1'] -> (moveRightIntermediate1, '1', >)
    moveRightS1 [' '] -> (moveRightIntermediate1, ' ', >)
    
    moveRightIntermediate1 ['0'] -> (stayMoveRightIntermediate1, '0', >)
    moveRightIntermediate1 ['1'] -> (stayMoveRightIntermediate1, '1', >)
    moveRightIntermediate1 [' '] -> (stayMoveRightIntermediate1, ' ', >)
    
    stayMoveRightIntermediate1 ['0'] -> (composeIntermediate1, '0', <)
    stayMoveRightIntermediate1 ['1'] -> (composeIntermediate1, '1', <)
    stayMoveRightIntermediate1 [' '] -> (composeIntermediate1, ' ', <)
    
    composeIntermediate1 ['0'] -> (stayComposeIntermediate1, '0', >)
    composeIntermediate1 ['1'] -> (stayComposeIntermediate1, '1', >)
    composeIntermediate1 [' '] -> (stayComposeIntermediate1, ' ', >)
    
    stayComposeIntermediate1 ['0'] -> (holdIntermediate1, '0', <)
    stayComposeIntermediate1 ['1'] -> (holdIntermediate1, '1', <)
    stayComposeIntermediate1 [' '] -> (holdIntermediate1, ' ', <)
    
    holdIntermediate1 ['0'] -> (1, '0', L),
    holdIntermediate1 ['1'] -> (1, '1', L),
    holdIntermediate1 [' '] -> (1, ' ', L),

    1 [' '] -> (stay1Blank, ' ', >)
    
    stay1Blank ['0'] -> (ACC, '0', <)
    stay1Blank ['1'] -> (ACC, '1', <)
    stay1Blank [' '] -> (ACC, ' ', <)
    
    2 [' '] -> (1, ' ', L)
}
~~~

And this is the final turing machine that we end up with. You can see it in action on [turing-machine.io]().

And if you're wondering, like I would, if this expansion was done by hand, then yes it was! And I would not inflict this on my greatest enemy! 

### MOV MOV MOV MOV MOV MOV MOV MOV

Going back to the start, I said that Tummys was chosen because of [this paper](https://www.cl.cam.ac.uk/~sd601/papers/mov.pdf).
For those that do not want to read it, the TLDR is that the x86 MOV instruction is Turing complete on its own ( and a single JMP instruction and four registers {or less for some programs}).

What this has to do with Tummys, is that Tummys was made because the real exercise I wanted to tackle was to build a bytecode VM witch basically supports only MOV, and then build a compiler for some language $$X$$ that compiled to that VM bytecode.
A language that worked with actual TMs seemed easier to translate than some other language with other constructs.

While this is what spawned this exercise, this is probably the thing that makes it the most inaccessible to me right now. 

This is all speculative though, it may happen that it is actually an easy implementation to do and parsing Tummys becomes the actually hard part who knows.

### "All shall be done, but it may be harder than you think."

This is what I consider a cool exercise to try. Nonetheless, I'm not sure if I will tackle it right away. Unfortunately, my current study routine is completely packed by rotating between 2 programming books, 1 math book { which is slowing me down the most }, 2 MOOCs and a practical exercise where I bang my head against the wall while implementing a Bitmapped Vector Trie in C++.
It is far more realistic that after completing the [Crafting Interpreters book](http://craftinginterpreters.com/) I will try my hand at something like the [PL\0](https://en.wikipedia.org/wiki/PL/0) at first.

So, why write all this?

Mostly because writing about this exercise was an exercise in itself ( and I needed to relax a little after some all-nighters and getting the damn Cold ). Basically, right now, a great focus of my studies is to
build a better Math and CS foundation and Turing Machines were themselves something that I looked at again recently in more depth than the last time I studied them.
It was really helpful brainstorming on this as an excuse to revise some TM concepts while at the same time looking at some of the learning I did on [Crafting Interpreters](http://craftinginterpreters.com/) from another angle.
How I would parse this, Is this as expressive as it can, How would I define a Turing Machine that does X and such were all interesting question to explore this way.

At the same time, it helped me a little with trying to use a more rigorous and formal thinking and writing process. Anyone with real experience and knowledge is surely able to point me to thousands of things that I failed at in this regard, but this small steps are helping me a lot with slowly filling the gaps of a completely self-taught education that sometimes weights too much { My greatest regret is probably that I was not able to follow a normal academic path }.

I hope this was at least a little bit interesting to someone else, I will probably try to write down a formal grammar for Tummys in my free-time-from-my-free-time for the time being while hoping to tackle this as soon as possible.
