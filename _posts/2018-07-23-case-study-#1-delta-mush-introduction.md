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

The Delta Mush algorithm is a smoothing algorithm that aims to reduce the time needed for production level skinning.
It was developed at Rhythm & Hues studios ( which unfortunaly has signed bankruptcy a few years ago ) and then became popular as a production level tool.
The algorithm itself isn't much difficult, which is a plus, but delivers great results and cuts production times.

The original paper ***Joe Mancewicz, Matt L. Derksen, and Cyrus A. Wilson, Delta mush: smoothing deformations while preserving detail*** can be found
at [ACM Digital Library](https://dl.acm.org/citation.cfm?id=2633376) and is a great read for those interested.

[Here](https://www.youtube.com/watch?v=EaCktzhxbTA) you can see an example of it in action ( and oh, boy is it wonderful ).

But how does it work? Well, it is actually simpler than it seems. The algorithm is divided in two main parts.

## The smoothing

The first part of the algorithm is the smoothing part ( not so surprising uh? ). The Delta Mush uses the simple Laplacian Smoothing.
Formally it is the defined as such:

$$
\begin{align*}
    \textsubscript{\bar{x}}
\end{align*}
$$
