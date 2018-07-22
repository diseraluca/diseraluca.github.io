---
layout: post
title: Cool Story Bro - SingleBlendMesh and my interest for code optimization
date: 2018-07-20
categories:: cool-story-bro
tags: c++ maya plugin optimization deformer
---

In this post I'd like to talk about the hows of how I got interested in code optimization.

It all started somewhat randomly about a week ago....
<!--godomalissimo-->
# The SingleBlendMesh deformer

Just out of school I was looking for some project to develop my skills. Being interested in working in the VFX/Film industry the more reasonable
choice was to continue deepening my knowledge of Maya API.
Deformers are one of the main work a C++ developer has to do in a production environment when working with Maya so the choice of what to concentrate on was simple.
There were many deformers I could choose from : Smoothing deformers, Blendshape deformers, esoteric or funny deformers and so on.
Being interested in morphing algorithm, but wanting to keep it simple as a start, a BlendShape deformer was an obvious choice.
It presented a lot of interesting challenges, from deepening my array attributes knowledge to Node-architecture to confront some of the challenges that may arise like changing defined blendshapes or offering a good attribute interface with Maya templates.

A few hours of work and the base to work on was done. A simple BlendShape deformer that accepted only one mesh.
Looking to improve it I was getting ready to make it a multiple BlendShape deformer...

## ...But then I got lost...

One lingering interest I had for a while was working with Maya Profiler. I always loved profilers. They can give you an insight on your code workings and provide you with data to back up your development timeline choices.
I could not let such an opportunity pass and decided to use this project to start working with Maya Profiler.
Oh boy what I had done. From there it was a descent to the hell of computation time obsessions.

## The first profiling and a few optimization passes

Working with Maya Profiler is really easy. There isn't much you have to do to set it up and it offers you a decent profiling interface while compromising a bit about accuracy and versatility.
Half-an-hour of work and I had my hands on some numbers to crunch. Here you can get a look at some of them:

| Test Scene | Node Evaluation Total | Node Evaluation Average | Deform Evaluation Total | Deform Evaluation Average | Deform Evaluation Min | Deform Evaluation Max :|
|:-----------|:---------------------:|:-----------------------:|:-----------------------:|:-------------------------:|:---------------------:|-----------------------:|
| 4-sphere   | 14000ms               | 90ms                    | 9000ms                  | ~70ms                     | ~70ms                 | ~91ms                  |
|----
| 1-sphere   | 7580ms                | 57.3ms                  | 5894ms                  | 49.17ms                   | 48.2ms                | 51ms                   |


This was ugly to me. I wasn't dissatisfied with the deformer code, for how simple it was there was still a beauty to it. But looking at this I felt a ache in my heart.
It just seemed so FUCKING SLOW!

## I could never let it pass

It was a detour sure, but a didactical one at that. I started a first optimization pass. A bit of data caching and restructuring of code.
Not much work, the whole of this story actually spans not more than a few days of work, but pretty effective. With my heart full of hope I decided to do a new profiling.

    "Yet what was 'fore my eyes
     But if not the brightest light."
     
The following data was spread before me:

| Test Scene | Node Evaluation Total | Node Evaluation Average | Deform Evaluation Total | Deform Evaluation Average | Deform Evaluation Min | Deform Evaluation Max :|
|:-----------|:---------------------:|:-----------------------:|:-----------------------:|:-------------------------:|:---------------------:|-----------------------:|
| 4-sphere   | 15000ms               | 100ms                   | 10000ms                 | ~80ms                     | ~80ms                 | ~94ms                  |
|----
| 1-sphere   | 6526ms                | 47.56ms                 | 4751ms                  | 39.60ms                   | 30ms                  | 42ms                   |

It wasn't much, and I actually somehow slowed the 4-sphere test scene. But my heart was filled with joy. Like a baby moving its first step into a world full of wonder I was intoxicated by a sense of discovery.
I could never stop here. What was initially a detour was now the main objective of my work.
I dipped my toes into whatever optimization source I could find. In the end I got to multithread my work. I read many theoretical book about multithreading but never tried it practically.
It worked, beautifully. It was able to cut almost 50% of the computation times.
This filled me with pride.

## A friendly hand

I decided to make a post about this work on [facebook](https://www.facebook.com/luca.disera.1/posts/491041721333675). To share my joy.
To my surprise someone commented. I could not believe it when I saw that it was Marco Giordano. I admire him a lot. That one of my idol would respond to me was a great surprise.
To further my joy he offered a friendly hand. He asked me for the code so he would review it for me.
I got the opportunity as soon as possible.
It was incredible. He could teach me so much in so little. He showed me some detail that I could never see before and my curiosity was out of this world.
It was a friendly hand indeed, but one that brought me to the pit of obsession.

![SingleBlendMeshDeformer]({{ "/assets/SingleBlendMesh.PNG" | absolute_url }})

Everything I was learning filled me with joy. Every millisecond I could tear down furthered my sense of pride.
When I dipped into AVX I was completely captured.
I wanted to work in the High-performance computing and code-optimizations world.

## And so the story ends...

This was the story of why I'm studying this type of programming and the reason this blog exists. It was short but it holds a great meaning to myself.
The SingleBlendMesh deformer would be a perfect fit for a Case-Study and I would have all the code ready.
I decided to not use it as such since I'm starting a new project which will contain all of the things I learned on this project but gives me more space to experiment and learn.

Well this was all, I hope I could get you engaged and that I could let you know a bit about how I'm made.

    "He who fights with monsters should be careful lest he thereby become a monster.
     And if thou gaze long into an abyss, the abyss will also gaze into thee."
