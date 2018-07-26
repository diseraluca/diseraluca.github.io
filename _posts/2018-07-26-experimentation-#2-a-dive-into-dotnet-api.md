---
layout: experimentation
title: Experimentation 2 - A dive into the dotnet API
date: 2018-07-26
categories: experimentation
tags: c# maya plugin maya-api dotnet-api
---

## Why C# ?

For a possible-work I'm studying **Unity** and, obviously, **C#**. Now, I'm still a total beginner with C# ( *C# in a nutshell* is so much more boring than I tought that I'm having difficulties reading it ) but what better way to get accustomated
to a new language than to get your hands dirty and write something you know how to write and like to write ?
For this experimentation I've chosen to rewrite the [SingleBlendMesh V1.0](https://diseraluca.github.io/blog/2018/07/20/cool-story-bro-1) ( The completely unoptimized and ugly one ).
It is a pretty simple code but exposes some of maya functionality. Furthermore it uses almost only *Maya API* call so it is a good testing ground for speed ( being a deformer is a plus ).

Now before diving in let's introduce the *.Net API*

<!--godomalissimo-->
## The .Net Maya API

The *.Net API* was introduced with *Maya 2013 extension 2* and uses *Microsoft .Net technology*.
The API is generated from the *C++ API* we, mostly, have equivalent interfaces between the two API.
There are some notable differences, tough:

1. **C#** properties instead of getter and setters ( for example *MFnAttribute set/isReadable()* ).
2. *MMessage* derived classes ( callbacks ) use **C#** event system. Notably, a system that deregisters callbacks automatically
   when the plugin is unloaded was added for the *.Net API*
3. *MStatus* is non-existant in the *.Net API*. API methods that used *MStatus* return *void* or *bool*. In case of errors exceptions are raised.
4. Iterators implements the *C#* interface *IEnumerable<T>*.
5. Maya Collections implement the *C#* *IEnumerable* and *IList* interfaces. Furthermore, they provide construsctors that accepts *C#* arrays
   and methods that return them.
6. **OpenMayaFx** has not been ported to the *.Net API*.
7. A new class, *MDockStation* was added to enable the use of **WPF** windows in the Maya UI.

There are some other differences in how the code is written since it uses *C#* specific features but those are the main one reported by the Maya Documentation.

## Setting up Visual Studio for .Net API development

It isn't to difficult to setup VS for the API. I'm using Visual Studio 2017 Community, other version of the software may vary the needed steps.

* Create a new *C#* project ( File->New->Project ) by choosing the **Class Library (.Net Framework)** Template. Here it is a good idea to set the target framework
   to the one used by the API code examples. I'm on *Maya 2017 update 4* and the *.net framework* is **4.5.1**.
   ![VS1]({{ "/assets/DOTNET_Experimentation_visualStudio_1.png" | absolute_url }})
   
* Add a reference ( In the solution explorer right-click references -> add reference ) to the openmayacs.dll that comes with your installation of maya.
   You can find it in %MAYA_LOCATION%/bin/. Coming with it you should find openmayacs.xml, the code documentation that will enable Intellisense etc.
   It will be loaded automatically with the reference.
   
* After loading the reference we should set ( click on the reference and you can see its properties ) *Copy Local* property of the openmayacs reference to False.
   This means that the openmayacs.dll reference won't be copied to the assembly directory. Failure to do so may create problems when the plugin will be used by maya.
   ![VS2]({{ "/assets/DOTNET_Experimentation_visualStudio_2.png" | absolute_url }})
   
* In the project property, under build events -> post-build event command line, add:

~~~
if not exist "$(SolutionDir)assemblies" mkdir "$(SolutionDir)assemblies"
copy "$(TargetPath)" "$(SolutionDir)assemblies\$(TargetName).nll.dll"
~~~

> This, after a build is complete, will create a directory called *Assemblies*, copy the builded assembly into it and change its extension to **.nll.dll** ( the extension needed by *C#* maya plugin ).

![VS3]({{ "/assets/DOTNET_Experimentation_visualStudio_3.png" | absolute_url }})
* That's all! We're ready to develop some plugins!

## The SingleMeshBlend_cs deformer

First of all I will show you the whole code. Then I will concentrate on those things I found interesting. Please remember that this is the first time I'm writing *C#* code ( apart from some trivial **Unity** script ). Many things are probably not the best, or even correct, way of writing *C#* code. The same may be valid for the *.Net API* side of things. Bear that in mind while reading this post.
Let's, for real this time, dive into the code:

~~~ c#
using Autodesk.Maya.OpenMaya;
using Autodesk.Maya.OpenMayaAnim;

[assembly: MPxNodeClass(typeof(SingleMeshBlend_cs.SingleBlendMesh), "SingleBlendMesh", 0x0d12309, NodeType = MPxNode.NodeType.kDeformerNode)]

namespace SingleMeshBlend_cs
{
    class SingleBlendMesh : MPxGeometryFilter, IMPxNode
    {
        public int cID = MProfiler.addCategory("SingleBlendMesh");

        public static MObject blendMesh = null;

        [MPxNodeNumeric("blw", "blendWeight", MFnNumericData.Type.kDouble, Min = new[] { 0.0 }, Max = new[] { 1.0 }, Keyable = true)]
        public static MObject blendWeight = null;

        [MPxNodeInitializer()]
        public static bool initialize()
        {
            MFnTypedAttribute tAttr = new MFnTypedAttribute();

            blendMesh = tAttr.create("blendMesh", "blm", MFnData.Type.kMesh );
            addAttribute(blendMesh);

            attributeAffects(blendMesh, outputGeom);
            attributeAffects(blendWeight, outputGeom);

            return true;
        }

        public override void deform(MDataBlock block, MItGeometry iter, MMatrix mat, uint multiIndex)
        {
            MProfilingScope p = new MProfilingScope(cID, MProfiler.ProfilingColor.kColorD_L1, "deform", "SingleBlendMeshDeformer", thisMObject());

            MPlug blendMeshPlug = new MPlug(thisMObject(), blendMesh);
            if (!blendMeshPlug.isConnected)
            {
                MGlobal.displayWarning(this.name() + ": blendMesh not connected. Please connect a mesh.");
                return;
            }
         
            float envelopeValue = block.inputValue(envelope).asFloat;
            MObject blendMeshValue = block.inputValue(blendMesh).asMesh;
            double blendWeightValue = block.inputValue(blendWeight).asDouble;

            MFnMesh blendMeshFn = new MFnMesh(blendMeshValue);

            for (iter.reset(); !iter.isDone; iter.next())
            {
                MPoint currentPosition = iter.position();

                MPoint targetPosition = new MPoint();
                blendMeshFn.getPoint(iter.index, targetPosition);

                
                MVector delta = (new MVector(targetPosition) - new MVector(currentPosition)) * blendWeightValue * envelopeValue;
                MPoint newPosition = new MPoint(delta + (new MVector(currentPosition)));

                iter.setPosition(newPosition);
            }
        }
    }
}
~~~

