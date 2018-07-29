---
layout: post
title: Case Study 1 Delta Mush - Introduction
date: 2018-07-22
categories: case-study
tags: c++ maya plugin optimization case-study
---

## Introduction

Welcome the to the first part of the Delta Mush case study. We will dive into a first working, but completely and horribly unoptimized, version of the deformer.

<!--godomalissimo-->
Before starting there is a small disclaimer I'd like to write. It is important so bear the following in mind while reading or studying the code.
As a first draft the code is not optimized in any way. Actually I've written it in a worse way than I would ( or should ) so as to be able to show some concepts in the following case study parts. Some choice are there to make the code more expressive so that you can more easily understand what we are doing.
This has actually made the code a bit ugly for my taste. I'm not gonna refactor it for now ( even tough it needs it ) because we are going to have to restructure a lot of code and rethink how we store and pass around data and data structures when we will talk about data caching.
There is some code replication, there are useless comments, some  comments that should be there aren't and so on...
We are gonna change it slowly when we have a more representative structure.
So please bear with it.

#### An important change

So, last time I said that we would use a method for calculating and applying the deltas that would need the mesh to have correctly unwrapped UVs.
While I was writing the first version of the code I learned a new method that is pretty functional, easy to write, and interesting to experiment optimizations on.
This new method doesn't require UVs and it is better than the other non-UVs method I knew of.
So this code uses this new method.

So, how does this method work?
It has similarities with the method I explained last time. The difference is in how we build the tangent space representation mostly.

![deltas]({{ "/assets/DeltaMushPart1_CaseStudy_deltas.png" | absolute_url }})

So, as you can see from the image **(1)**, we will use the neighbour vertexes pairs to build two vectors that are relative to the vertex we are calculating for.
Those will be the base of our tanget space. We will do a cross product to find an orthogonal axis.
Since we could have a triangle that isn't a right triangle to ensure the orghonality of all the axis we are going to do a second cross product to replace one of the two neighbour vectors **(2)**.
This will be done on every triangle we can build from the neighbours **(3)**. We will have more than one delta with this process.
In the end we average the deltas we've find to have the the final delta we need **(4)**.

Now, calculating more than one delta is obviously slower. We could use only one triangle and use a single delta but we would have a less precise outcome.
The algorithm to use depends on what you purposes are. For performance reason we could even implement both of them and let the user choose which one to use.

As you can see it is a simple method that will translate easily into code.
