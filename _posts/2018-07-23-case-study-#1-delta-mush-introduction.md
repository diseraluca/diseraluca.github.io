---
layout: post
title: Case Study 1 Delta Mush - Introduction
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

{% raw %}

$$
\begin{align*}
    x_i = \sum{j=1}^N w_jp_j
\end{align*}
$$

{% endraw %}

Where $$ N $$ is number of connected vertices to vertex $$i$$, $$ p_j $$ are the position in space of the connected vertices and $$x_i$$ is the resulting position of vertex $$i$$.
$$w_j$$ is a weight factor that is applied to the transformation. It can be constant or it can be calculated from a series of factors. One techniques is to use the lenght of the edge between vertex $$i$$ and vertex $$j$$ to find an inversely proportional weight.
In this Case Study we'll keep it simple(r) and use a basic smoothing where we find the average position of the connected vertexes and displace the vertex to it.

![An example of a smoothing algorithm. Left - Non Smoothed Mesh \ Right - Smoothed Mesh]({{ "/assets/DeltaMushIntroduction_CaseStudy_smoothExample.png" | absolute_url }})

Those smoothing algorithms usually lets the user decide a number of iterations to perform. Let $$N$$ be the number of iterations to perform, the 0 < $$n$$ < $$N$$ iteration is performed on the corresponding mesh that is smoothed $$n-1$$ times.
In simpler terms, the smoothing is additive with subsequent iterations performed on the already smoothed mesh and not on the original mesh.

Now, these kind of smoothing algorithms produce loss of volume and details. Delta Mush performs a second operation to resolve this issue.

## The delta in Delta Mush

How can we prevent this loss of volume and detail? Well, Delta Mush gives us a simple solution by using the smoothed deltas.
What are those?
A delta is non-other than a difference between two related values. In the Delta Mush case the delta is a vector in tangent space going from the final smoothed position of a vertex to the original position of the vertex in tangent space.
Those deltas will then be reapplied to the deformed mesh to regain the lost volume.
The idea here is that if the smoothing of the mesh ( in its original position - The bind pose ) produced a certain loss of volume the smoothing of the deformed mesh will produce a similar amount of loss. By reapplying the delta we are trying to get as near as possible to the original position of the vertex.

There is a problem tough. If the mesh is deformed, and we apply the deltas, we may find that we are displacing the vertexes in the wrong direction. That is because the displacement of the smoothing may have changed its orientation.
Here comes the aforementioned tangent space.

## Tangent Space

