---
layout: post
title: Case Study 1: Delta Mush - Introduction
date: 2018-07-22
categories: case-study
tags: c++ maya plugin optimization case-study
---

## Dulta mas... Delta mua... Delta uh??

This is the introduction to the first Case Study series. In this first Case Study we are going to build a basic Delta Mush deformer and
study some optimization technique on it.
But what is a Delta Mush?

The Delta Mush algorithm is a smoothing algorithm that aims to reduce the time needed for production level skinning by cleaning up undesirable deformations.
It was developed at Rhythm & Hues studios ( which unfortunaly has signed bankruptcy a few years ago ) and then became popular as a production level tool.
The algorithm itself isn't much difficult, which is a plus, but delivers great results and cuts production times.

The original paper ***Joe Mancewicz, Matt L. Derksen, and Cyrus A. Wilson, Delta mush: smoothing deformations while preserving detail*** can be found
at [ACM Digital Library](https://dl.acm.org/citation.cfm?id=2633376) and is a great read for those interested.

[Here](https://www.youtube.com/watch?v=EaCktzhxbTA) you can see an example of it in action ( and oh, boy is it wonderful ).

But how does it work? Well, it is actually simpler than it seems. The algorithm is divided in two main parts.

## The smoothing

The first part of the algorithm is the smoothing part ( not so surprising uh? ). The Delta Mush usually uses weighted Laplacian Smoothing.
Formally it is the defined per-vertex as such:

$$
\begin{align*}
    x_i = \sum{j=1}^N w_jp_j
\end{align*}
$$

Where $$ N $$ is number of connected vertices to vertex $$i$$, $$ p_j $$ are the position in space of the connected vertices and $$x_i$$ is the resulting position of vertex $$i$$.
$$w_j$$ is a weight factor that is applied to the transformation. It can be constant or it can be calculated from a series of factors. One techniques is to use the lenght of the edge between vertex $$i$$ and vertex $$j$$ to find an inversely proportional weight.
In this Case Study we'll keep it simple(r) and use a basic smoothing where we find the average position of the connected vertexes and displace the vertex to it.

Now these kind of smoothing algorithms produce loss of volume and details. Delta Mush performs a second operation to resolve this issue.

##
