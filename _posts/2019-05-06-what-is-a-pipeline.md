---
layout: post
title: What is a pipeline
date: 2019-05-06
categories: learning
tags: python pipeline abstract
draft: true
---

Lately I've been working on two projects to build a better resume as it seems many times I'm not even given a chance to show if I'm capable of doing a job I apply to.
One of those two projects aim is to build a mini-pipeline for Maya.

I've tried many different variations to build a better intuition of what a pipeline is. Starting from small experiments, explorative designs and more.
I've tried building a design, called *Pippyline*, that miserably failed. The design was not good enough and I got myself into one of my worst habit, forgoing the practice-first approach, losing myself
in unneded complexety and unnecessary details.
After taking a small break to concentrate on the other project, I've tried to produce a renewed design in *Pippyline 2.0*. While I was able to slim down on the complexeties, a plethora of design errors came
to the surface resulting in a massive failure.

I was pretty frustrated at this point and needed to take a break to recollect my thoughts.
For how unfortunate it is, the practice-first approach does not come naturally to me.
After forcibly working in this type of mindset I tend to accumulate stress and lose sight of what I should be doing, ironically.
Nonetheless, working with this kind of approach has given me the chance to learn a lot about the problem space I was trying to tackle which would have not been possible with my natural approach that lies on the other side of the scale.

Returning to my natural approach was what I needed to actually mmove forward on this project. While the practice-first approach was great, I concentrated on practice so much that I stopped examining and developing
the knowledge and intuitions I was getting from the practice.

While working on my other project, I took some time to think about pipelines. In the end I've forged a new design that I'd like to try, which I called **Microline**.

# What is a Pipeline?

Daunting. This is the best adjective to represent how I feel about this question which still haunts me to this day.
I already wanted to tackle this almost an year ago, but decided to defer it to a later time as I could not find enough information on what a pipeline is and had too much to learn at the time.
About a month ago I received a kind suggestion to try my hand at this project again after being rejected from one of my dream jobs for lack of experience ( which is understandable considering the position I was running for ).
This time I felt that I may be able to tackle the problem which had this lingering question at its base.

To my dismay, I must say I wasn't as prepared as I needed and found myself stuck and incapable of even exploring the problem space.
I had the luck to receive some kinds words and the trite-but-needed advice of "You learn by doing", which I've heard so many times and am still unable to easily do as it is the opposite of my natural approach to problem solving and learning, giving me the necessary confidence to actually try doing and learn about
the problem.

Looking back at [one of my facebook post](https://www.facebook.com/luca.disera.1/posts/651924141912098) from the start of this project, I've felt like declaring that a pipeline is:

> a collection of automation processes that deliver and organize a project which is structured based on a given specification. 

While this definition shares some of the things that I currently consider part of a pipeline I feel it lacks in its understanding of what is at the core of a pipeline.

The first time I encountered the concept of a pipeline in the CG industry was at the start of my journey at Rainbow Academy's [3Ddp course](https://www.rainbowacademy.it/corsi/master-animazione-3d-visual-effects-videogames/) back in 2017.
To build an intuition of what are the different phases and pieces, usually, required to deliver a digital product, I was faced with a picture that looked something like the following:

![production pipeline]({{ "/assets/PIPELINE_ANDY_PRODUCTION_PIPELINE.jpg" | absolute_url }})

> The picture should be by [Andy Bean](https://www.lynda.com/Andy-Beane/356907-1.html)

It may be the case that many artists were introduced to the pipeline this way.
Now, the pipeline we are talking about here is obviously not this one, but, nonetheless, they are deeply connected.

![pipeline data storage]({{ "/assets/PIPELINE_PRODUCTION_PIPELINE.png" | absolute_url }})

In the model I'm using a similar concept to the one in the image that is referred to as a ***Production Pipeline***.
For simplicity's sakes, I'm treating it as a black box that takes as input a series of concrete assets { the budget, the actual artists and their contract, the workstations, the idea and so on } and produces an output, which we will later recognize as a completed project.
For the Microline project I've actually added some more specifics to the Production Pipeline as an excuse to bind the problem space to a manageable one.

In the model, I assign a few property to a production pipeline that we are interested in: The workers and a ***concrete pipeline***.

![pipeline data storage]({{ "/assets/PIPELINE_PRODUCTION_PIPELINE_PROPERTIES.png" | absolute_url }})

The workers are an abstraction over the user of a pipeline.
The concrete pipeline is a generator of pipeline instances { for reasons that I hope will be clear later }, which are the actual pipelines the users will interface with.

We were saying that the Production Pipeline and the Pipeline we are talking about are deeply connected.
Getting back to the introductory Production Pipeline image we can see that the central idea of the pipeline is that a sequential collection of phases are executed to produce the final project, for how inaccurate that actually is.
We can see that what ties all of the phases is a connection to the previous one; that is, that ***there is data flowing from one phase to another***.

Movie productions, game productions or, more generally, most digital productions { not unlike software development in many cases }, looked from an eagle-view, are just a way to produce data in a specific way.
They may have hundreds of peoples working in parallel producing terabytes of data. Even more important is the fact that, unlike what is showed in the picture, the process is not sequential.
Data flows back and forth, retakes, remodelling, updating a rig... Most of the times the process is not one dimensional and occurs in parallel.
While I'm not able to provide any realistic data myself { but I'm sure that such data can be found on the web }, the sheer size of data and people that a production manages requires a plethora of specific, robust procedures.

The need to actually meet a deadline makes it so this process must be manageable no matter the size and that as much of the work as possible is either automated or trivialized.
This is the reason we actually deploy a pipeline. Intuitively I feel that this gives us a pretty reasonable view over what a pipeline is:

> ***The collection of structures that makes the production pipeline manageable***

Now, this may seem like an obvious thing to some and an erroneus statement to others but I couldn't see this at the start and it is the intuition I've built my understanding of a pipeline on.

As we have said, a production pipeline is a process designed to produce a specific data output.
This puts data flow needed to correctly construct the necessary data trough the different phases at the center of the production pipeline.
More importantly, this tells us that the ***core responsability*** that a pipeline needs to support the production pipeline is ***the management of the data flow***.

Ironically, this was actually disclosed to me at the start of this project.
Worse yet, it was said to me more than once and each time I was unable to understand how much weigth was carried by those words, considering this responsability just one of the many axiomatically accepted responsabilities of a pipeline.

This new knowledge permits us to declare the first, and most important, requirement for a pipeline:

* A pipeline should be able to administer the data produced by the workers of a production pipeline. This means that it should store, deploy, version and synchronize the data generated by them.

Or in a kind of Agile's user stories way:

* As a user, I'm able to store data trough a pipeline so that it is accessible to the other users.
* As a user, I'm able to store a piece of data in multiple versions so that a log of the data body at different time is available.
* As a user, I'm able to update previously stored data trough a pipeline so that the most recent version is always available to other users.
* As a user, I'm able to retrieve data trough a pipeline so that I can modify or use it.

And so on and so on. I'm not that good with writing down these types of requirements but I'm sure many more could be written.

Leaving that aside, to tackle this requirement, the pipeline has to be able to store data somewhere and retrieve it back.

![pipeline data storage]({{ "/assets/PIPELINE_PRODUCTION_PIPELINE_STORAGE.png" | absolute_url }})

The data storage is an abstract place where data from the Production Pipeline can be stored. Usually, it may be a server in the production.
In this model, the data storage lives outside the pipeline, into the production pipeline, and communicates with it trough a Connector.

A connector knows how a data storage works and can store, organize and retrieve data to and from it.
A production pipeline may have more than one data storage and each one may provide a different communication interface.

In a previous model, the one for Pippyline, I had placed the equivalent of the data storage inside the pipeline, which was modeled quite differently, owned by it, and placed less importance on its role.
I now feel that this was a great error. A pipeline should not necessarily depend on a data storage ( even if, as we will see later, the data storage is still a part of how pipeline instances are defined in this model ).
Furthermore, the data storage may change for uknown reasons and this should not necessarily invalidate an entire pipeline ( again, in this model, this is actually not completely true ). 

The pipeline instance, which owns the connector, you see in the model, is the actual "pipeline" the user will interact with.
Such an instance should be generated by the concrete pipeline. This may seem somewhat contrived, especially since we should actually be trying to define a pipeline.

Looking back to the Production Pipeline, we know it gets some input and gives back a completed project.
The Production Pipeline knows everything about the structures that are at the base of a production and uses this knowledge to architect the way in which the production flows.
A pipeline instead, is a part of a Production Pipeline that is used to manage the flow of data.

For each Production Pipeline there are a number of different possible pipelines and viceversa, as long as the two are compatible.

Up to now, we treated the Production Pipeline as a function that maps to a single project. I find this to be false.
The way in which the Production Pipeline is modeled, as a manager of productions, makes it a massively parallel process whose management is able to produce a series of projects at the same time.
To fulfill this parallelism, the Production Pipeline may employ one or more "pipeline" for different projects at the same time.

The Concrete Pipeline was then modeled for this purpose. The idea is that a Production Pipeline produces a specific "pipeline" ( e.g a way to manage the data flow ), on the fly or preemptively, for a project.
A Concrete Pipeline defines the domain over the pipelines for a Production Pipeline.
In this particular model, the Concrete Pipeline is defined by a series of Modules and some Procedures that are injected in a pipeline instance and its project.

Talking about projects, they are an entity that acts as a container of managed data from a production data flow.
This is the entity with which a pipeline instance is actually connected. 
