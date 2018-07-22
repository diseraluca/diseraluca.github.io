---
layout: experimentation
title: Experimentation 1 - CHECK_MSTATUS* macros
date: 2018-07-22
categories: experimentation
tags: c++ maya plugin optimization
---

## Have you ever wondered how checking an MStatus impact your code ?

Well, I recently did. I actually use the *CHECK_MSTATUS\** macros a lot in the first drafts of my plugins. 
While it usually won't matter in an MPxNode::initialize, for example, I was pretty curious about the performance of these macros.
Before diving in the experimentation let's introduce them.

## The MStatus class...

As you may know the Maya API provides an MStatus class ( this is actually true for the C++ API only as both the Python and .NET API provides different error checking mechanism ) that is, mostly, ubiquitous in the API methods returns or paramaters.
The MStatus class is used to perform error checking on a function execution and can be checked to control the program execution.
An example may be the following:


~~~ c++
MStatus uninitializePlugin(MObject obj) {
  MStatus status{};
  MFnPlugin plugin{obj};

  status = plugin.deregisterNode(FictionalNode::typeId);
  if ( status != MStatus::kSuccess ) {
    status.perror("Error deregistering FictionalNode.");
    return status;
  }
  
  return MStatus::kSuccess;
}
~~~

>> Sidenote: 
>>>There exists a Macro that evaluates to true if the MStatus is not MStatus::kSuccess. The if-statement could be rewritten as such:

>>> ~~~ c++
>>> if ( MFAIL(status) ) {
>>>     status.perror("Error deregistering FictionalNode.");
>>>     return status;
>>> }
>>> ~~~

>>> I never found myself using it but it's a question of style.

As most ( maybe all? I'm not actually sure about this ) methods that do not need to return a value directly deregisterNode returns an MStatus.
Most ( here again it may be all ) API methods that need to return a value provides an optional out_parameter to provide MStatus checking.

## ...and CHECK_MSTATUS* macros

Checking an MStatus is a tedious and repetitive process. As such, the Maya API provides us with three Macros to check an MStatus and print an error.
They're the following:

~~~ c++

#define CHECK_MSTATUS	( _status )

#define CHECK_MSTATUS_AND_RETURN	( _status, _retVal )

#define CHECK_MSTATUS_AND_RETURN_IT	( _status ) 	
~~~

All of them prints an error if the MStatus is not Mstatus::kSuccess. Furthermore CHECK_MSTATUS_AND_RETURN and CHECK_MSTATUS_AND_RETURN_IT returns _retVal and _status respectively in case of a not MStatus::kSucces _status.
Here you can see an example of their uses:

~~~ c++
MStatus UselessyExpensiveDeformerWithCheck::initialize()
{
	MStatus status{};

	MFnNumericAttribute nAttr;

	animateMe = nAttr.create("animateMe", "anm", MFnNumericData::kDouble, 0.0, &status);
	CHECK_MSTATUS_AND_RETURN_IT(status);
	CHECK_MSTATUS(nAttr.setKeyable(true));
	CHECK_MSTATUS(addAttribute(animateMe));

	CHECK_MSTATUS(attributeAffects(animateMe, outputGeom));

	return MStatus::kSuccess;
}
~~~

One notable thing about the CHECK_MSTATUS_AND_RETURN_IT is its definition:

~~~ c++
#define CHECK_MSTATUS_AND_RETURN_IT(_status)			\
CHECK_MSTATUS_AND_RETURN((_status), (_status))
~~~

Here you should be careful about what you pass to it since _status will be expanded two times and that could be a cause of side-effects ( it usually be a concern with how this macro is used but it's good to know ).
This is all you need to know about MStatus and the CHECK_MSTATUS* macros.

## Finally let's experiment a little!

For this experimentation I've prepared some simple deformers which present the following main loop:

~~~ c++
	for (unsigned int vertexIndex{ 0 }; vertexIndex < vertexCount; vertexIndex++, iterator.next()) {
		MPoint point = iterator.position(MSpace::kObject, &status);

		point.z = std::pow(std::pow(point.x, 8), std::floor(point.y / 8)) * 5;
		vertexPositions[vertexIndex] = point;
	}
~~~

There are equivalent versions that checks the MStatus from iterator.position.
All in all it's just a useless calculation used as an excuse for the experimentation.

The plugin was developed with Maya 2017 update 4 using Maya 2017 update 4 C++ API on Visual studio community 2017.
The tests were run on Maya 2017 update 4 and profiled with std::chrono using a 1 sample per 100 calls rate.

One test scene was prepared for each deformer (1 without status checking, 2 with status checking).
The test scene consists of 1 polyShpere with 633602 vertices with the deformer attached.
The deformer had an output-affecting value animated linearly from 0.0 to 1.0 on a 2000 frame timeline at 24fps.
The profiling was done on multiple playback of this animation.

The test rig was the following:
CPU: Intel Core i7-3930k @ 3.20GHz
RAM: 16GB ddr4
GPU: Geforce GTX 680
On a Windows 7 home premium 64 bit OS

## Some result to crunch

![Deformer Sample Chart]({{ "/assets/CHECK_MSTATUS_Experimentation_DeformerSamples_Chart.png" | absolute_url }})
![Deformer Sample Average Chart]({{ "/assets/CHECK_MSTATUS_Experimentation_DeformerAverage_Chart.png" | absolute_url }})

As we can see here there is a (depending on context) slight change in performance.
An interesting thing to note, that appeared in all the tests, is that the deformer which uses CHECK_MSTATUS_AND_RETURN actually seems to perform a bit better than the version which uses CHECK_MSTATUS. I didn't expect it.
Both of them, though, are trashed by the version without checking. With almost a full second of difference.

In the introduction of this post I said that in an MPxNode::initialize the checks performance may be ignore. This actually made me wonder about that. As such I prepared and run another experiment. This time I profiled 
