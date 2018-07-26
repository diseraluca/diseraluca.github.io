---
layout: experimentation
title: Experimentation 2 - A dive into the dotnet API
date: 2018-07-26
categories: experimentation
tags: c# maya plugin maya-api dotnet-api
---

## Why C# ?

For a possible-work I'm studying **Unity** and, obviously, **C#**. Now, I'm still a total beginner with C# ( *C# in a nutshell* is so much more boring than I tought that I'm having difficulties reading it ) but what better way to get accustomed
to a new language than to get your hands dirty and write something you know how to write and like to write ?
For this experimentation I've chosen to rewrite the [SingleBlendMesh V1.0](https://diseraluca.github.io/blog/2018/07/20/cool-story-bro-1) ( The completely unoptimized and ugly one ).
It is a pretty simple code but exposes some of maya functionality. Furthermore it uses almost only *Maya API* call so it is a good testing ground for speed ( being a deformer is a plus ).

Now before diving in let's introduce the *.Net API*

<!--godomalissimo-->
## The .Net Maya API

The *.Net API* was introduced with *Maya 2013 extension 2* and uses *Microsoft .Net technology*.
The API is generated from the *C++ API* and we, mostly, have equivalent interfaces between the two API.
There are some notable differences, tough:

1. **C#** properties instead of getter and setters ( for example *MFnAttribute set/isReadable()* ).
2. *MMessage* derived classes ( callbacks ) use **C#** event system. Notably, a system that deregisters callbacks automatically
   when the plugin is unloaded was added for the *.Net API*
3. *MStatus* is non-existant in the *.Net API*. API methods that used *MStatus* return *void* or *bool*. In case of errors exceptions are raised.
4. Iterators implements the *C#* interface *IEnumerable<T>*.
5. Maya Collections implement the *C#* *IEnumerable* and *IList* interfaces. Furthermore, they provide constructors that accepts *C#* arrays
   and methods that return them.
6. **OpenMayaFx** has not been ported to the *.Net API*.
7. A new class, *MDockStation* was added to enable the use of **WPF** windows in the Maya UI.

There are some other differences in how the code is written since it uses *C#* specific features but those are the main one reported by the Maya Documentation.

## Setting up Visual Studio for .Net API development

It isn't too difficult to setup VS for the API. I'm using Visual Studio 2017 Community, other version of the software may vary the needed steps.

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

```c#
using Autodesk.Maya.OpenMaya;
using Autodesk.Maya.OpenMayaAnim;

[assembly: MPxNodeClass(typeof(SingleMeshBlend_cs.SingleBlendMesh), "SingleBlendMesh", 0x0d12309, NodeType = MPxNode.NodeType.kDeformerNode)]

namespace SingleMeshBlend_cs
{
    class SingleBlendMesh : MPxGeometryFilter, IMPxNode
    {
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
```

#### Autodesk namespaces

As you can see from the first few lines, the API modules resides in the Autodesk.Maya namespace. We import them with the using directive.

~~~c#
using Autodesk.Maya.OpenMaya;
using Autodesk.Maya.OpenMayaAnim;
~~~

#### C# attributes and the .Net API

~~~c#
[assembly: MPxNodeClass(typeof(SingleMeshBlend_cs.SingleBlendMesh), "SingleBlendMesh", 0x0d12309, NodeType = MPxNode.NodeType.kDeformerNode)]
~~~

This may be the line that confused you the most if you're not accustomed to *C#*. What does it mean?
Well, what you see here are **C#'s attributes**. They are a way to add metadata information that can be retrieved trought the *reflection* system.
In this particular case we are attaching data to initialize to register the node on the assembly of the build. This code replaces what you would usually put in the initializePlugin with MFnPlugin::registerNode.

You can see other examples of **C#'s attributes** later in the code. For example to declare an attribute of the node:

~~~c#
[MPxNodeNumeric("blw", "blendWeight", MFnNumericData.Type.kDouble, Min = new[] { 0.0 }, Max = new[] { 1.0 }, Keyable = true)]
public static MObject blendWeight = null;
~~~

#### The initialize method

In *C#* plugins we don't necessarily need an initialize method. Most nodes attributes can be declared using **C#'s attributes** with *[MPxNodeNumeric/Enum/...]* and with *[MPxNodeAffectby]* and *[MPxNodeAffects]*.
In this particular case we had to declare it because it doens't seem to be an **C#'s attribute** for *typed attributes*.
Another thing to note is that the initialize too is marked by an attribute, *[MPxNodeInitializer]*:

~~~c#
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
~~~

The rest of the initialize method behaves like it would in the *C++ API*.
Something to note here is that there exist a *[MPxNodeAffects(str, str)]* attribute that goes right before the class definition. I've tried using it for blendWeight but I couldn't get it to work.
I may have done something wrong or it may not work if a initialize method is used ( I tend to the former since the *[MPxNodeNumeric]* numeric attribute has worked anyway ).
This is why there is an *attributeAffects* for *blendWeight* too.

#### The IMPxNode interface

~~~c#
class SingleBlendMesh : MPxGeometryFilter, IMPxNode
~~~

*C#* supports single-inerithance but unlimited interfaces. The *IMPxNode* interface provides the needed abstract methods for a custom node ( compute ) and is needed to create it. Similar interfaces exist for other type of plugins like *IMPxCommand*.

One interesting thing to note is that the *C++* version of this deformer supports per-vertex weights. Here I had to use *MPxGeometryFilter* instead of *MPxDeformerNode* as the base class ( the difference between the two is that the latter supports per-vertex weights while the former doesn't. *MPxGeometryFilter* is actually the base class of *MPxDeformerNode* ) because *MPxDeformerNode* seems to not be supported correctly.
At least on my implementation of the *.Net API*, *MPxDeformerNode* isn't complete and does not provide a deform method to override nor does it implement *IMPxNode*.

I may have done something horribly wrong but I could not get it to work.

#### C# properties examples

~~~c#
if (!blendMeshPlug.isConnected)
~~~

~~~c#
float envelopeValue = block.inputValue(envelope).asFloat;
~~~

Those are examples of point 1 of the changes list:

1. **C#** properties instead of getter and setters ( for example *MFnAttribute set/isReadable()* ).

If you understand python **@property** decorator you probably already understand how they works.
Otherwise, they are wrappers that provide public access to a class field. When you are reading the value you are actually calling a *get()* method in the background and a *set()* method is called when assigning to it.
Properties can be read-only, write-only or read-and-write.

#### Implicit casting

~~~c#
MVector delta = (new MVector(targetPosition) - new MVector(currentPosition)) * blendWeightValue * envelopeValue;
~~~

In *C#* implicit casting is possible only when a conversion operator in marked as **implicit**. In this case I had to create some new vectors to make the computation work because no implicit casting could be done (actually there doesn't seem to exist a conversion from MPoint to MVector at all) ( in *C++* there is no need for explicit casting in this code for example ).
I'm sure there is a better way of writing this line, but it currently escapes me.

## Some numbers to go with

Now, I used the exact same testing environment from the *C++* version.
The results were.... Even worse than I expected.
For the 4-sphere scene I couldn't even complete one profiling. It was so absurdly slow that I waited hours and still could not get some result. I just felt disgusted.
For the 1-sphere version the result are horrible. You can see yourself:

| Language   | Node Evaluation Total | Node Evaluation Average | Deform Evaluation Total | Deform Evaluation Average | Deform Evaluation Min | Deform Evaluation Max :|
|:-----------|:---------------------:|:-----------------------:|:-----------------------:|:-------------------------:|:---------------------:|-----------------------:|
| C#         | ~366000ms             | ~3050                   | ~366000ms               | ~3620ms                   | ~3000ms               | ~6000ms                |
|----
| C++        | 7580ms                | 57.3ms                  | 5894ms                  | 49.17ms                   | 48.2ms                | 51ms                   |

Well, the data talks. The *C#* version came out a whoppin' ~***52*** times slower than the *C++* version. 
I expected it to be slow but this is too much. It's just WOW.

## Conclusion

Now, this wasn't the best case to test the *.Net API*, you would never use it for performance heavy code, the same as *Python*.
Regarding the prototyping potential, it surely as hell removes a lot of the boilerplate code ( even more than *Python* ) and that may help writing code faster ( but not faster code by far as we've seen \***chuckles stupidly**\* ).
Unfortunately I find it to be, currently, too cumbersome. The documentation is pretty non-existant and not many people seems to use it so it would be difficult getting an helping hand.
Furthermore, *Python* is practically the lingua-franca of the VFX/Film industry and would provide more growing possibilities while giving the same capacity of *C#* regarding Maya API.
Now, I'm a total beginner in *C#*, there may be some language features that makes *C#* the best tool for some production code but I can't see it at the moment and I'm not sure it would be worth the hassle.
I'm obviously biased since I'm disliking the language ( not as much as *Python* tough ) but I know that it has its uses in other fields, I just think the *.Net API* isn't really worth exploring more right now.

Well, it was a fun experiment and it actually made me accustomed to the basic language features but I don't think I'll work with *C#* on maya again for a long time.
