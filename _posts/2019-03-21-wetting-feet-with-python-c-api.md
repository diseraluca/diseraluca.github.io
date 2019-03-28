---
layout: post
title: Getting our feet wet with Python's C API
date: 2019-03-21
categories: introduction
tags: python C introduction tutorial
draft: true
---

#Approaching the shore

The Pythons'C API is something that I was always interested in.
Nonetheless, I never found an exercise for which it would have made sense to try my hand at it.
The toy exercise I'm currently working on, that expands on the last assignment of the work of genius that is
the [From Nand To Tetris Course](https://www.coursera.org/learn/build-a-computer), provided me with a decent option to go at it.

I will write about that exercise in a future post. For this one, instead, I'd like to provide a resource for some of the basics related to writing Python's extension modules.

I must say, to my disdain, that to actually start and write something with the C API was a little bit more difficult that I initially tought.
I could not find that many good resources on it and the best resource there is, the [Python/C API Reference Manual](https://docs.python.org/3/c-api/index.html) and the [Extending and Embedding the Python Interpreter Tutorial](https://docs.python.org/3/extending/index.html),
while good, are not QT level good regarding documentation.
Nonetheless with a bit of effort, combined with the [CPython source code](https://github.com/python/cpython), it is more than enough to start writing something.

To have some code reference, I will use the first thing I build in this last few days, a static size array of references that is type-checked at runtime ( similar to a C array of pointers in some way but with some notable differences ).

This is an example of what our extension module will let us do:

~~~python
>>> import lds_array
>>> a = lds_array.array(4, int, 3, 5, 6, 7)
>>> b = lds_array.array(3, str, "aaa", "nnn", "ffff")
>>> print(a)
[3, 5, 6, 7]
>>> print(a * 5)
[3, 5, 6, 7, 3, 5, 6, 7, 3, 5, 6, 7, 3, 5, 6, 7, 3, 5, 6, 7]
>>> print(b + lds_array.array(2, str, "abc", "bcs"))
[aaa, nnn, ffff, abc, bcs]
>>> for x in b:
...     print(x * 5)
...
aaaaaaaaaaaaaaa
nnnnnnnnnnnnnnn
ffffffffffffffffffff
>>> a[3]
7
>>> a[3] = 56
>>> a[3]
56
~~~

Before we start looking at the C API, I'd like to say a few things.
First, I don't particularly feel suitable to teach anyone anything about the C API.
I used it for only a few days ( as this was the time I allotted for exercising with it ), and probably am not even at the level that I can write something decent without the risk of leaking memory or riddling it with bugs.

For this reasons, I can't say that the code I will use in this post is a good example of the API usage nor that is correctly idiomatic in the way it is written.
So please take everything with a grain of salt.

Furthermore, we will only look at using C code in Python and not at calling Python from C.

The goal of this post is to try and streamline the very first step of approaching the C API, the step I did this last few days, in the hope of helping someone that is beginning this journey that has a few hurdles at the start.

With this behind us, let's go on.

# Some basics

Before starting with the code, I'd like to talk about some concepts and idioms that are present in the C API.
If you are like me, some of this will look too extraneous to remember without some code. In that case, I suggest to skim this part and return to it, if needed, while reading the rest of the post.

## Accessing the C API

Most of what we need to access the C API is stored in the header file *Python.h* that usually comes with your python installation.
There are a few "catches" with this header file that are of note.

1. It brings with it some standard headers, precisely:
    * stdio.h 
    * string.h
    * errno.h 
    * stdlib.h
    
  if the latter is not present *malloc*, *realloc* and *free* are defined by the *Python.h* header file.
2. This header file can define some pre-processor definitions that changes the way in which standard header files behave. As such, **it is important to** *#include* **it before any standard header**.

All *Python.h* user-visible names, be it of function, macros or structure, have the prefix *Py* or *PY* and it is strongly advised to never use this prefixes in our code to avoid any kind of confusion.

Another useful header is [structmember.h](https://github.com/python/cpython/blob/master/Include/structmember.h) which contains what we need to map our C structure representation of objects to Python's object attributes.
It brings *stddef.h* with it to provide *offsetof* that we need when we define our attributes.

## Reference Counting

Python manages the lifetime of its objects trough [Reference Counting](https://mortoray.com/2012/01/08/what-is-reference-counting/) ( boasting an optional cycle detector on top that we won't use in this exercise ).
While all the intricacies of this are hidden from a Python user, while working with the C API we have to manually manage references.
Failure to do so would cause memory leaks in the program.

While this may seem like a complicate matter ( and well it is, worse than managing C heap-allocated memory in my opinion ), the tool we use to accomplish this task are small and simple.
*Py_INCREF*, *Py_DECREF* and *Py_XDECREF* are the three macros that *Python.h* provides us to control the refcount of PyObjects.

Those three macros takes a *PyObject\** as their only argument.
As you may imagine, *Py_INCREF* increments the count by one while *Py_DECREF* decrements it by one.

Furthermore, if *Py_DECREF* decrements a refcount to 0, it frees the object memory, not directly but by calling the object destructor trough a function pointer which is stored in the *tp_deallocator* member of its *PyTypeObject*.
*Py_XDECREF* works exactly like *Py_DECREF* but, while the latter requires its argument to be non-NULL, correctly handles a NULL-argument.

We have yet another ( and last ) macro at our disposal in *Py_CLEAR*, which decrements a refcount to 0, frees the object and sets its argument to *NULL* ( while the other decrement macros don't ).
*Py_CLEAR* like *Py_XDECREF* handles *NULL* arguments ( by doing nothing ).

Now, while we have to manually manage the reference count of objects, it is not always the case to do so.
Python itself owns each and every one of its objects. What we are able to own, is a reference to one of the objects.
This reference is the one we are going to actually manage, deciding the object lifetime at the same time.

In Python's ownership model, we can identify three types of references, each one with its own management tactic.

Firstly, we have **new references**, for example the one returned by a PyObject-building function; e.g *Py_BuildValue* or *PyTuple_New*.
When we get hold of a *new reference*, as already said this usually happens when we construct a new object, we become its owner.
As the owner of a *new reference* it is our job to dispose of it, by *Py_DECREF*ing it to 0 or to, otherwise, pass it to someone who will do it for, essentially giving up our ownership in the process.

If we won't take care of our *new reference* this way, we will end with a memory leak.

For example:

~~~c
static PyObject* square(long n) {
    PyObject* num = NULL;
	PyObject* res = NULL;
	
	num = PyLong_FromLong(n); // Here we will get a new reference to a python int object we created from the original argument
	res = PyNumber_Multiply(num, num); // Again we get a new reference that we have to manage
	
	Py_DECREF(num); // As we don't need it anymore we do our job and decrease the reference, deleting it
	
	return res; // Instead of res ourselves, we pass the ownership to the caller, which will need to eventually dispose of it or pass it along, as a new reference
}
~~~ 


We have forgone any error checking here for the sake of the example readability but we can incur in a core dump as both *new reference*-returning function we call may return *NULL* on failure which is not supported by *Py_DECREF* ( we could use *Py_XDECREF*, but it would be wrong as we would have a dangling error which, as we will see later, should be propagated ).
It may be tempting to simply write something along the line of:

~~~c
static PyObject* square(long n) {	
	return PyNumber_Multiply(PyLong_FromLong(n), PyLong_FromLong(n));
~~~

But in this case, and every case where a *new reference*-returning function return value is used as a temporary object, we will leak both of the *PyLong_FromLong* created references as the *PyNumber_Multiply* function will only borrow them without taking on their ownership ( and as such, *Py_DECREF* them to zero ).

The second type we can encounter are **stolen references**.
*Stolen refernces* are usually encountered when compositing a reference we own with another object that manages it, e.g any kind of container.
When we deal with *stolen references* the obligation of managing the reference is left to the "thief".

For example:

~~~c
static PyObject* longToUnaryTuple(long n) {
	PyObject* res = NULL;
	
	res = PyTuple_New(1); // We build a new tuple and own its reference
	PyTuple_SetItem(res, 0, PyLong_FromLong(n)); // The PyLong_FromLong new reference gets stolen by SetItem and isn't our responsability anymore
	
	return res; // We pass ownership of the tuple reference to the caller
}
~~~

There isn't much more to stolen references. If we know which functions steal a reference we are good to go.
The one thing we have to be careful about is not mindlessly dabbling with a reference that was stolen:

~~~c
static PyObject* longToUnaryTuple(long n) {
	PyObject* res = NULL;
	PyObject* num = PyLong_FromLong(n);
	
	res = PyTuple_New(1); 
	PyTuple_SetItem(res, 0, num);
	
	Py_DECREF(num) // We don't really own the reference anymore and are potentially disrupting the tuple internals
	
	return res; 
}
~~~

We should not decrement a reference refcount that we don't own without first incrementing it ( that we do if we need it to be surely available for a certain time or scope ).
Usually we should have nothing to do with a reference that was stolen as it isn't our concern anymore in any way.

Getting to the last type we encounter **borrowed references**.
While they are pretty essential in their contract, we can use a *borrowed reference* but we aren't its owner, they have some tricky parts.
The fact is that we have a reference that is actually managed by someone else that we need to use ( contrary to *new references* that we own and *stolen references* which aren't ours anymore and we should not manage ).
They may, for example, be invalidated by code that dabbles with the original owner.

For example:

~~~c
static void buggyTupleAccess(PyObject* o) {
	if (PyTuple_Check(o)) {
		PyObject* last = PyTuple_GetItem(o, PyTuple_Size(o)-1); // Here we get a borrowed reference to the last item in the tuple

		PyTuple_SetItem(o, PyTuple_Size(o)-1, PyLong_FromLong(10000)); // Here we set a new item as the last element of the tuple
		
		// do something with last
	}
}
~~~  

The problem with this snippet is that when we modify the last element of the list, setting a new one, the list, that manages the reference we borrowed, may as well ( and actually will ) *Py_DECREF* it, potentially freeing the object in the process.
When we access last later in the code, we have no garantuee that it wasn't invalidated.

The trickiest part is that these type of bugs are not always obvious. Python can reuse memory addresses that were previosly freed, meaning that we won't necessarily get bogus values in the process.
Or it may be that sometimes last may have a refcount greater than 1, making it seem like the code is working and then randomly crash when this is not true anymore.

Sometimes we may work with special values, like the integers from -5 to 255 which Python always keeps in memory, that makes the code work.

For all this reasons, when using *borrowed references*, we should increment the refcount while in scope, and decrement it when we are done:

~~~c
static void buggyTupleAccess(PyObject* o) {
	if (PyTuple_Check(o)) {
		PyObject* last = PyTuple_GetItem(o, PyTuple_Size(o)-1); 		
		Py_INCREF(last); // increase when the scope we need it in starts
		
		PyTuple_SetItem(o, PyTuple_Size(o)-1, PyLong_FromLong(10000));
		
		// do something with last
		
		Py_DECREF(last) // Decrease the reference, potentially freeing it, as we have no need for it anymore
	}
}
~~~  

{ By the way another thing we can do here is to get the item trough *PySequence_GetItem*, from the abstract sequence protocol, which already calls *Py_INCREF* giving us a new reference we only have to decref ).

Simple enough but can get tricky at time. Furthermore, we have to make sure that the reference gets decreased in each code path.

## Exception handling

In the C API, exceptions are raised as a two-step process.

First we have to actually set an exception. The current exception details are stored globally, per thread. You can see the members that store the exception in [cpython/pystate.h](https://github.com/python/cpython/blob/c11183cdcff6af13c4339fdcce84ab63f7930ddc/Include/cpython/pystate.h#L103) under the thread state structure.
To do this we can use a [series of macros](https://docs.python.org/3/c-api/exceptions.html#raising-exceptions) defined in [cpython/error.c](https://github.com/python/cpython/blob/a24107b04c1277e3c1105f98aff5bfa3a98b33a0/Python/errors.c#L844).

The most important one is probably *PyObject\* PyErr_Format(PyObject\* exception, const char\* format, ...)*, which sets a given exception and a formatted message.
The C API provides the same [standard exception objects](https://docs.python.org/3/c-api/exceptions.html#standard-exceptions) that [Pyhon does](https://docs.python.org/3/library/exceptions.html#built-in-exceptions) as [global PyObject*s](https://github.com/python/cpython/blob/8f59ee01be3d83d5513a9a3f654a237d77d80d9a/Include/pyerrors.h#L77).

As a second step, we have to return an error indicator value from the current function.
For a pointer-returning function, we should return *NULL*. For int-returning functions we should return -1 ( and 0 on success ) { An exception to this is the *PyArg_* family of functions which return 1 for success and 0 for failure }.

For example:

~~~c
static PyObject* tuple_get(PyObject* o, Py_ssize_t index) {
	if (!PyTuple_CheckExact(o)) {
	   PyErr_Format(PyExc_TypeError, "Unsupported operand type for %s: '%s'", __FUNCTION__, Py_TYPE(o)->tp_name);
	   return NULL;
	}
	
	.....
} 
~~~

A failure to follow the two-step process will result in an error.
If we forget to set an exception we will get a SystemError:

~~~python
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
SystemError: error return without exception set
~~~

Setting an exception but fail to return a value that signals it, will give us an error at runtime.
Usually, we should check for most return values and propagate an exception when set ( Without overwriting it so that we know where and what initially occurred ).

Other than using the built-in exception, we can build custom module-level exceptions trough *PyErr_NewException* and *PyErr_NewExceptionWithDoc*.
For example:

~~~c
static PyObject* CustomException; // We need to declare a static PyObject* that points to our exception

...

PyMODINIT_FUNC
PyInit_custom()
{
    PyObject* module; // This is our module object. We skip how to build it in this example

	...
	
    CustomException = PyErr_NewException("custom.CustomException", NULL, NULL); // We create the exception object and point to it
    Py_INCREF(CustomException);
    PyModule_AddObject(module, "CustomException", CustomException); // We need to add the exception to the module's objects
    
	...
}
~~~

## PyObject and PyTypeObject

At the heart of all Python's object resides the [PyObject structure](https://github.com/python/cpython/blob/9bdd2de84c1af55fbc006d3f892313623bd0195c/Include/object.h#L109):
 
 ~~~c
 typedef struct _object {
    _PyObject_HEAD_EXTRA
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
} PyObject;
 ~~~
 
A PyObject, as you can see, simply contains the object reference count and a pointer to its type ( which we will look at in a moment ).
There are a few interesting thing to know about Python's object ( and a few more interesting things you can read in CPython source that we normally should not need to know ).
 
All object are allocated on the heap ( there are some exception regarding type objects ).
We always work with PyObject* and each Python's object can be cast to a PyObject*.

The most interesting part about the PyObject structure is the pointer to a [type](https://github.com/python/cpython/blob/bb86bf4c4eaa30b1f5192dab9f389ce0bb61114d/Include/cpython/object.h#L177) object, this is where all the action actually resides.
This structure contains all the [informations](https://docs.python.org/3/c-api/typeobj.html#type-objects) needed to know how the type actually operates.

We will look at some of the fields later while looking at the array implementation as it is easier to understand trough an example.

There are a few more things we could talk about but those were the most important concepts. Many things are just unclear without a more consistent example and as such we will explore them with the array example.

# Building an array

Now we finally get to do some practice.
As said before, this was the very first thing I build and is riddled with bad choices.
I've decided to use this as an example as I think it may better present some questions and errors that someone who has never touched the C API may encounter.
Furthermore, it gives me a lot of great chances to think and discuss how things could have been done differently.

## The code

~~~c
#include <Python.h>
#include <structmember.h>

static PyTypeObject arrayType;

typedef struct {
	PyObject_HEAD
	PyObject** data;
	Py_ssize_t size;
	PyTypeObject* storedType;
} array;


static PyMemberDef array_members[] = {
	{"size", T_PYSSIZET, offsetof(array, size), READONLY, "Describe how many elements are stored in the array"},
	{NULL}
};

//-- Array Utilities --//

#define ARR_AS_ARRAY(obj) ((array*)(obj))
#define ARR_EXACT_CHECK(obj) (Py_TYPE((obj)) == &arrayType)
#define ARR_CHECK(obj)  (PyObject_Check((obj), &arrayType))

#define ARR_SIZE(arr) ((ARR_AS_ARRAY(arr))->size)
#define ARR_STORED_TYPE(arr) (((ARR_AS_ARRAY(arr))->storedType))

#define ARR_CHECK_TYPE(arr, obj) (ARR_STORED_TYPE((arr)) == Py_TYPE((obj)))

#define ARR_GET(arr, i) (*(((ARR_AS_ARRAY(arr)->data)) + (i))) 
#define ARR_ASSIGN(arr, i, obj) ((ARR_GET((arr), (i))) = (obj))

static PyObject* newArray(PyTypeObject* type, PyTypeObject* storedType, Py_ssize_t size) {
	array* self;
	self = (array*)type->tp_alloc(type, 0);
	if (self == NULL) {
		return NULL;
	}

	self->data = (PyObject**)PyMem_Calloc(sizeof(PyObject*), size);
	if (self->data == NULL) {
		Py_DECREF(self);
		return NULL;
	}

	self->size = size;
	self->storedType = storedType;

	Py_INCREF(self->storedType);

	return (PyObject*)self;
}

// Array Utilities End //

//-- Array Type Methods --//

static void array_dealloc(array* self) {
	for (Py_ssize_t index = 0; index < self->size; ++index) {
		Py_XDECREF(ARR_GET(self, index));
	}

	PyMem_Free((void*)self->data);
	Py_DECREF(ARR_STORED_TYPE(self));
}

static PyObject* array_new(PyTypeObject* type, PyObject* args, PyObject* kwds) {
	if (PyTuple_Size(args) < 2) {
		PyErr_Format(PyExc_TypeError, "%s() takes at least 2 arguments (%i given)", __FUNCTION__, PyTuple_Size(args));
		return NULL;
	}

	Py_ssize_t size = PyLong_AsSsize_t(PyTuple_GET_ITEM(args, 0));
	PyObject* storedType = PyTuple_GET_ITEM(args, 1);

	if (PyErr_Occurred() || !PyType_Check(storedType)) {
		PyErr_Format(PyExc_TypeError, "Unsupported operand type(s) for %s: '%s' and '%s'", __FUNCTION__, Py_TYPE(PyTuple_GET_ITEM(args, 0))->tp_name, Py_TYPE(PyTuple_GET_ITEM(args, 1))->tp_name);
		return NULL;
	}

	if (size <= 0) {
		PyErr_Format(PyExc_ValueError, "The array size must be a positive integer greater than 0");
		return NULL;
	}

	if (size > (PY_SSIZE_T_MAX / sizeof(PyObject*))) {
		return PyErr_NoMemory();
	}


	return newArray(type, (PyTypeObject*)storedType, size);
}

static int array_init(array* self, PyObject* args, PyObject* kwds) {
	static const Py_ssize_t MIN_ARGUMENTS = 2;
	
	Py_ssize_t argsSize = PyTuple_Size(args);

	if (argsSize > (ARR_SIZE(self) + MIN_ARGUMENTS)) {
		PyErr_Format(PyExc_TypeError, "%s() takes at most %i arguments for an array of size %i (%i given)", __FUNCTION__, ARR_SIZE(self) + MIN_ARGUMENTS, ARR_SIZE(self), argsSize);
		return -1;
	}

	for (Py_ssize_t index = 2; index < argsSize; ++index) {
		PyObject* tmp = PyTuple_GET_ITEM(args, index);
		if (!ARR_CHECK_TYPE(self, tmp)) {
			PyErr_Format(PyExc_TypeError, "Unsupported operand type(s) for %s for array of type '%s': '%s'", __FUNCTION__, ARR_STORED_TYPE(self)->tp_name, Py_TYPE(tmp)->tp_name);
			return -1;
		}

		Py_INCREF(tmp);
		ARR_ASSIGN(self, index-MIN_ARGUMENTS, tmp);
	}

	return 0;
}

static PyObject* array_str(PyObject* o) {
	PyObject* openingCharacter = PyUnicode_FromString("[");
	PyObject* closeningCharacter = PyUnicode_FromString("]");
	PyObject* separator = PyUnicode_FromString(", ");

	PyObject* stringRepresentations = PyTuple_New(ARR_SIZE(o));
	for (Py_ssize_t index = 0; index < ARR_SIZE(o); ++index) {
		PyTuple_SET_ITEM(stringRepresentations, index, PyObject_Str(ARR_GET(o, index)));
	}

	PyObject* elementsString = PyUnicode_Join(separator, stringRepresentations);
	PyObject* openedString = PyUnicode_Concat(openingCharacter, elementsString);
	PyObject* completeString = PyUnicode_Concat(openedString, closeningCharacter);

	Py_DECREF(openingCharacter);
	Py_DECREF(closeningCharacter);
	Py_DECREF(separator);
	Py_DECREF(stringRepresentations);
	Py_DECREF(elementsString);
	Py_DECREF(openedString);

	return completeString;
}

// Array Type Methods End //

//-- Sequence Protocol --//

static Py_ssize_t array_sq_length(PyObject* o) {
	return ARR_SIZE(o);
}

static PyObject* array_sq_item(PyObject* o, Py_ssize_t i) {
	PyObject* item = NULL;

	if (i < 0 || i >= ARR_SIZE(o)) {
		PyErr_Format(PyExc_IndexError, "array index out of range");
		return NULL;
	}

	item = ARR_GET(o, i);
	if (item == NULL) {
		PyErr_Format(PyExc_IndexError, "Accessing uninitialized object at index %i", i);
		return NULL;
	}

	Py_INCREF(item);
	return item;
}

static int array_sq_ass_item(PyObject* o, Py_ssize_t i, PyObject* v) {
	if (!ARR_CHECK_TYPE(o, v)) {
		PyErr_Format(PyExc_TypeError, "Unsupported operand type(s) for %s for array of type '%s': '%s'", __FUNCTION__, ARR_STORED_TYPE(o)->tp_name, Py_TYPE(v)->tp_name);
		return -1;
	}

	if (i < 0 || i >= ARR_SIZE(o)) {
		PyErr_Format(PyExc_IndexError, "array index out of range");
		return -1;
	}

	Py_XDECREF(ARR_GET(o, i));
	ARR_ASSIGN(o, i, v);
	Py_INCREF(v);

	return 0;
}

static PyObject* array_sq_concat(PyObject* o1, PyObject* o2) {
	if (!ARR_EXACT_CHECK(o2)) {
		PyErr_Format(PyExc_TypeError, "can only concatenate array ( not '%s' ) to array", Py_TYPE(o2)->tp_name);
		return NULL;
	}

	if (ARR_STORED_TYPE(o1) != ARR_STORED_TYPE(o2)) {
		PyErr_Format(PyExc_TypeError, "can only concatenate array of type '%s' ( not '%s' ) to array of type '%s'", ARR_STORED_TYPE(o1)->tp_name, ARR_STORED_TYPE(o2)->tp_name, ARR_STORED_TYPE(o1)->tp_name);
		return NULL;
	}

	PyObject* arr = newArray(Py_TYPE(o1), ARR_STORED_TYPE(o1), ARR_SIZE(o1) + ARR_SIZE(o2));
	if (arr == NULL) {
		return NULL;
	}

	for (Py_ssize_t index = 0; index < ARR_SIZE(o1); ++index) {
		array_sq_ass_item(arr, index, ARR_GET(o1, index));
		if (PyErr_Occurred()) {
			Py_DECREF(arr);
			return NULL;
		}
	}

	for (Py_ssize_t index = 0; index < ARR_SIZE(o2); ++index) {
		array_sq_ass_item(arr, index + ARR_SIZE(o1), ARR_GET(o2, index));
		if (PyErr_Occurred()) {
			Py_DECREF(arr);
			return NULL;
		}
	}

	return arr;
}

static PyObject* array_sq_repeat(PyObject* o, Py_ssize_t count) {
	if (count <= 0) {
		PyErr_Format(PyExc_ValueError, "Can't multiply array by non-positive non-greater than 0 int");
	}

	PyObject* arr = newArray(Py_TYPE(o), ARR_STORED_TYPE(o), ARR_SIZE(o) * count);
	if (arr == NULL) {
		return NULL;
	}

	for (Py_ssize_t repetition = 0; repetition < count; ++repetition) {
		for (Py_ssize_t index = 0; index < ARR_SIZE(o); ++index) {
			array_sq_ass_item(arr, index + (ARR_SIZE(o) * repetition), ARR_GET(o, index));
			if (PyErr_Occurred()) {
				Py_DECREF(arr);
				return NULL;
			}
		}
	}

	return arr;
}

static PySequenceMethods array_sq_methods = {
	.sq_length = array_sq_length,
	.sq_item = array_sq_item,
	.sq_ass_item = array_sq_ass_item,
	.sq_concat = array_sq_concat,
	.sq_repeat = array_sq_repeat
};

// Sequence Protocol End //

//-- Array Structures and Type --//

static PyTypeObject arrayType = {
	PyVarObject_HEAD_INIT(NULL, 0)
	.tp_name = "lds_array.array",
	.tp_doc = "A c-like array structure",
	.tp_basicsize = sizeof(array),
	.tp_itemsize = 0,
	.tp_flags = Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE,
	.tp_new = array_new,
	.tp_init =  array_init,
	.tp_dealloc = array_dealloc,
	.tp_members = array_members,
	.tp_as_sequence = &array_sq_methods,
	.tp_str = array_str
};

// Array Structures and Type End //

//-- Module --//

static PyModuleDef arraymodule = {
	PyModuleDef_HEAD_INIT,
	.m_name = "lds_array",
	.m_doc = "C-like array module",
	.m_size = -1
};

PyMODINIT_FUNC
PyInit_lds_array() {
	PyObject* m;
	if (PyType_Ready(&arrayType) < 0) {
		return NULL;
	}

	m = PyModule_Create(&arraymodule);
	if (m == NULL) {
		return NULL;
	}

	Py_INCREF(&arrayType);
	PyModule_AddObject(m, "array", (PyObject*)&arrayType);
	return m;
}

// Module End //
~~~

It has a few things going on. Some things are may be a bit difficult to untangle at first glance if you've never used the C API but the example should be simple enough to be used as a kick-start.

# The module object and initialization

While it is the last thing in the array code, the simplest unit we can start from to untangle the code is [the module object](https://github.com/python/cpython/blob/a24107b04c1277e3c1105f98aff5bfa3a98b33a0/Objects/moduleobject.c).

~~~c
static PyModuleDef arraymodule = {
	PyModuleDef_HEAD_INIT,
	.m_name = "lds_array",
	.m_doc = "C-like array module",
	.m_size = -1
};

PyMODINIT_FUNC
PyInit_lds_array() {
	PyObject* m;
	if (PyType_Ready(&arrayType) < 0) {
		return NULL;
	}

	m = PyModule_Create(&arraymodule);
	if (m == NULL) {
		return NULL;
	}

	Py_INCREF(&arrayType);
	PyModule_AddObject(m, "array", (PyObject*)&arrayType);
	return m;
}
~~~

When we add an extension trough the C API we are adding a new module. To do this we have a two-stage process where we define the module and a function to initialize it.
[PyModuleDef](https://github.com/python/cpython/blob/aedc273fd90e31c7a20904568de3115f8957395b/Include/moduleobject.h#L75) is a structure that defines all the fields that a module object has.
We will see that there is a similar structure for each type of entity we can add.

While the module array module we provide is really basic, PyModuleDef has a few [members](https://docs.python.org/3/c-api/module.html#c.PyModuleDef) we can use:

| Type              | Member     | Use                                                    |
|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:| 
| PyModuleDef_Base  | m_base     | This is always initialized to [PyModuleDef_HEAD_INIT](https://github.com/python/cpython/blob/aedc273fd90e31c7a20904568de3115f8957395b/Include/moduleobject.h#L51) which adds some needed members                                                         |
| const char*       | m_name     | The name for the module                                                                                                                                                                                                                                  |
| const char*       | m_doc      | The docstring for the module. As for all docstrings members, we can use a PyDoc_STRVAR variable                                                                                                                                                          |
| Py_ssize_t        | m_size     | A module state may be kept in a module specific memory. m_size defines the size of that memory. If set to -1 the module has, instead, global state and does not support sub-interpreters. A non negative size is required for multi-phase initialization |
| PyMethodDef*      | m_methods  | Points to a PyMethodDef structure that defines all the module-level functions                                                                                                                                                                            |
| PyModuleDef_Slot* | m_slots    | Slot definitions for multi-phase initialization. Must be NULL when single-phase initialization is used                                                                                                                                                   |
| traverseproc      | m_traverse | The traversal function neede for GC traversal. May be NULL in case the GC is not needed                                                                                                                                                                  |
| inquiry           | m_clear    | The clear function for the GC. May be NULL in case the GC is not needed                                                                                                                                                                                  |
| freefunc          | m_free     | A function called to deallocate the module. May be NULL if it isn't needed                                                                                                                                                                               |  

While some members are pretty straighforward, like m_name and m_size, a few interesting things are introduced in this table.
The most important one for modules, as it is specific to them, is the difference between multi-phase and single-phase initialization, which we will look at in a bit.

Going top to bottom,  we have three things to talk about before initialization:

***Docstrings***:

Everyone who used Python will know what docstrings are. From a C API point of view, we can relate a docstring to a module, function or anything else thanks to the doc member we can find in many definition structures.
  
While we can simply pass a string literal or a C-string to it, the C API provides us with the PyDoc_STRVAR helper macro.
Interesting enough, this does not seem to be documented in the API reference, but [we can look at the code](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Include/pymacro.h#L71) to see that it simply declares a *static char[]* variable containing the string we pass to it.
  
~~~c
PyDoc_STRVAR(custom_doc, "A custom docstring");
/*
  
which expands to
  
static char custom_doc[] = "A custom docstring";
  
or to
  
static char custom_doc[] = "";
  
if WITH_DOC_STRING is not defined in pyconfig.h
  
*/
~~~

***The PyMethodDef structure***:

The [PyMethodDef structure](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Include/methodobject.h#L56) is used to define a python method.

It only has a few members:

| Type              | Member     | Use                                                    |
|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:| 
| const char*       | ml_name    | The name of the method |
| PyCFunction       | ml_meth    | Pointer the function implementation |
| int               | ml_flags   | Bits-flag indicating how Python should construct the call for this method. See below for more informations |
| const char*       | ml_doc     | Docstring for the method |

There are two interesting bits here, the PyCFunction type and ml_flags.
[PyCFunction](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Include/methodobject.h#L18) is a typedefed function pointer to a function that returns a PyObject* and accepts two PyObject* as parameters.
This is the basic signature for C Function that are callable by python.

The first PyObject*, usually called self, is actually the self object ( the same you would get in a Python instance method ). For a module-level function this is the module instance itself.
The second parameter, ususally called args, is a PyTuple object that contains all the arguments of the function. Again, you can see the parallelism with Python.

Extending on this is [PyCFunctionWithKeywords](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Include/methodobject.h#L20), which adds a third PyObject* paramater that, as you probably guessed, is a PyDictionary containing the named arguments passed to the function.
We will look at how to deal with both the args and kwargs arguments later.

The ml_meth field has to be a PyCFunction pointer or a pointer to another function that is castable, and casted, to it.
Python bases the way in which the arguments are passed to this function, when called, on the ml_flags parameter.
There are a few flags we can use:

| Flag              | Meaning                 |
|:-------------------------------------------:| 
| METH_VARARGS      | This is the standard calling convention. It passes a self and args argument and expect the method to be of the PyCFucntion type |
| METH_KEYWORDS     | This is for PyCFunctionWithKeywords types. It passes self and args as METH_VARARGS but adds a third dictionary argument on top of them |
| METH_NOARGS       | This is for function that expects no arguments. The ml_meth should still be of the PyCFunction type but the second parameter will always bbe passed aas NULL |
| METH_O            | This is a commodity flags for PyCFunctions that expects a single PyObject as argument. Instead of passing a tuple the second argument is passed directly as the object that would be the sole element of the tuple. |

Of all this flags only *METH_VARARGS* and *METH_KEYWORDS* can be combined togheter.
There are two more flags,called binding flags, that can be combined with any of the previously described flags but are exclusive between themselves.

| Flag              | Meaning                 |
|:-------------------------------------------:| 
| METH_CLASS        | The self parameter will be the type object of the instance instead of the instance itself. This is used to create class methods |
| METH_STATIC       | The self parameter will be NULL. Used to create static methods |

An example to better wrap your head around this ( with some spoilers on arguments parsing ):

~~~c
static PyObject* simplePow(PyObject* self, PyObject* args) { // PyCFunction
    PyObject* base = NULL;
    PyObject* exponent = NULL;
    PyObject* result = NULL;
    
    if (!PyArg_UnpackTuple(args, __FUNCTION__, 2, 2, &base, &exponent)) { // This is one of the ways to parse the tuple arguments
        return NULL;
    }
    
    Py_INCREF(base); Py_INCREF(exponent); // We got borrowed references from the tuple. We increment the refcount for good meassure.
    
    if (!PyNumber_CHECK(base) || !PyNumber_CHECK(exponent)) { // We check if the passed object implements the Number Protocol otherwise we raise an exception
       PyErr_Format(PyExc_TypeError, "Unsupported operand type(s) for %s: '%s' and '%s'", __FUNCTION__, Py_TYPE(base)->tp_name, Py_TYPE(exponent)->tp_name);
       
       Py_DECREF(base); Py_DECREF(exponent); // Remember to decrement the reference count in each path!
       return NULL; 
    }
    
    result = PyNumber(base, exponent, Py_None); // We should probably check if any error happened here but for this example we won't

    Py_DECREF(base); Py_DECREF(exponent); // We have to decrement the refcount of the borrowed references

    return result; // result is new reference that we are passing to the caller to handle
}

// Check https://pythonextensionpatterns.readthedocs.io/en/latest/canonical_function.html for another way we could structure this ( and any other ) function.
// Furthermore, you will see a good goto use in a C program ( This pattern is used troughout some of the CPython source code too )

static PyMethodDef custom_methods[] = {
   { "simplePow", simplePow, METH_VARARGS, "" }, // Our method entry
   { NULL, NULL, 0, NULL} // A sentinel to know when we are at the end
}

static struct PyModuleDef custom_module = {
   PyModuleDef_HEAD_INIT,
   "custom",
   NULL,
   -1,
   custom_methods
};

PyMODINIT_FUNC
PyInit_spam(void)
{
    return PyModule_Create(&custom_module);
}
~~~

## m_traverse, m_clear and GC Support

As said before, on top of the reference counting system Python ships with a an [optional-use garbage collector](https://rushter.com/blog/python-garbage-collector/) to deal, specifically, with cycles.
To quote the API reference:

> Python’s support for detecting and collecting garbage which involves circular references requires support from object types which are > “containers” for other objects which may also be containers. Types which do not store references to other objects, or which only store > references to atomic types (such as numbers or strings), do not need to provide any explicit support for garbage collection.

The array structure we are building should actually implement GC support. I must say that in the few days I studied the API I never really implemented GC support nor looked into it much.
While I can't give you any personal experience "insight", I can at least provide you with pointers to how such a support is added and where to look for informations on it.

Implementing GC support doesn't seem to require too much.
First we have to provide the traverse function, we will see later that the same GC support functions appear for object types too, which has a [traverseproc](https://github.com/python/cpython/blob/bb86bf4c4eaa30b1f5192dab9f389ce0bb61114d/Include/object.h#L158:15) signature.

If needed we have to provide a tp_clear function, which is of the [inquiry](https://github.com/python/cpython/blob/bb86bf4c4eaa30b1f5192dab9f389ce0bb61114d/Include/object.h#L148) type.
Going by the reference, the discriminator factor for needing a tp_clear function is the fact that the container we are garbage-collecting is mutable ( in which case we need one ) or not.
Nonetheless, it seems a bit tricky to provide one and I advise you to read the [reference entry](https://docs.python.org/3/c-api/typeobj.html#c.PyTypeObject.tp_traverse) on it if interested.

The other [things](https://docs.python.org/3/c-api/gcsupport.html#supporting-cyclic-garbage-collection) we have to ensure are that, quoting the reference:

1. The memory for the object must be allocated using PyObject_GC_New() or PyObject_GC_NewVar().
2. Once all the fields which may contain references to other containers are initialized, it must call PyObject_GC_Track() { which we will later untrack }.

Furthermore, to actually enable the GC for the object, we need to add the [Py_TPFLAGS_HAVE_GC](https://docs.python.org/3/c-api/typeobj.html#Py_TPFLAGS_HAVE_GC) flag to its type.

To start working with the GC the best resource seems to be the [GC-support tutorial]()https://docs.python.org/3/extending/newtypes_tutorial.html#supporting-cyclic-garbage-collection in the [Extending and Embedding the Python Interpreter](https://docs.python.org/3/extending/index.html) guide.

# Single-phase initialization and Multi-phase initiliazation

With those out of the way, the last thing we are interested in is the difference between the two types of module initialization.
I've found the [PEP 489](https://www.python.org/dev/peps/pep-0489/) to be a really cool read on this argument. The reference advises to read [PEP 3121](https://www.python.org/dev/peps/pep-3121/) for more details on the sense of the m_size field.

Like the names may imply, the two strategies differ in the number of steps taken to initialize a module and the way in which those steps are carried.  
With single-phase initialization, the module is created, populated and returned directly by the initiliazation function, uniting the creation and initialization process and generating a singleton module.
As we can read from [PEP 489](https://www.python.org/dev/peps/pep-0489/), this process is different from how Python Modules are built and is specific to C's extension modules.

Furthermore, as the initialization function receives no context, the initialization process lacks access to some informations and brings along some difficult to resolve problems, like the support for sub-interpreters.

On a practical note, this is what our array is doing and how single-phase initialization is supported in the C API:

~~~c
PyMODINIT_FUNC
PyInit_lds_array() {
	PyObject* m;
	if (PyType_Ready(&arrayType) < 0) {
		return NULL;
	}

	m = PyModule_Create(&arraymodule);
	if (m == NULL) {
		return NULL;
	}

	Py_INCREF(&arrayType);
	PyModule_AddObject(m, "array", (PyObject*)&arrayType);
	return m;
}
~~~

This is a way of initiliazing the module, where we create a module instance from its definition and the populate it trough the use of [supporting functions](https://docs.python.org/3/c-api/module.html#support-functions) like [PyModule_AddObject](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Python/modsupport.c#L615:1), that uses single-phase initialization.

The code is pretty self-explicative so I won't explain it.

Multi-phase initialization is a bit more interesting. Let's start by going practical.
To request Multi-phase initialization, the Init function should return a PyModuleDef instance pointer for which the m_slots field is correctly set and non-empty.

This should be done trough the use of the [PyModuleDef_Init](https://github.com/python/cpython/blob/a24107b04c1277e3c1105f98aff5bfa3a98b33a0/Objects/moduleobject.c#L44) function:

~~~c
PyMODINIT_FUNC
PyInit_lds_array(void)
{
    return PyModuleDef_Init(&arraymodule);
}
~~~

This will give our module def to Python wich will call the constructing functions provided trough the m_slots field.
The structure connect our m_slots field should be an array of [PyModuleDef_slots](https://github.com/python/cpython/blob/b232df9197a19e78d0e2a751e56e0e62547354ec/Include/moduleobject.h#L61) which has two fields:

| Type              | Member     | Use                                                                |
|:---------------------------------------------------------------------------------------------------:| 
| int               | slot       | Defines the type of this slot. See below                           |
| void*             | value      | The function for this slot whose meaning depends on the slot field |

The slot field has two possible types.

[Py_mod_create](https://github.com/python/cpython/blob/b232df9197a19e78d0e2a751e56e0e62547354ec/Include/moduleobject.h#L66) and [Py_mod_exec](https://github.com/python/cpython/blob/b232df9197a19e78d0e2a751e56e0e62547354ec/Include/moduleobject.h#L67).

The first points to function of signature:

~~~c
PyObject* f(PyObject *spec, PyModuleDef *def)
~~~

which should create the module object and return it or set an error if it is not possible. This is the equivalent step to *__new__* in the module initialization.
The second parameter will receive the *PyModuleDef* we return from the init function.
The spec parameter is a bit more interesting. There is a lot to understand about the ModuleSpec object and I will point you to [PEP 451](https://www.python.org/dev/peps/pep-0451/) for more informations on it.

If a creation slot is not provided, Python will create the module trough the [PyModule_New function](https://github.com/python/cpython/blob/a24107b04c1277e3c1105f98aff5bfa3a98b33a0/Objects/moduleobject.c#L114).
I could not really find an example of a creation slot in the source code. It seems idiomatic to leave this part to Python.

The second type points to a function with signature:

~~~c
int f(PyObject* module)
~~~

that should actually execute the module ( which is equivalent to actually evaluating the code in a module ). We should add classes, functions and so on to our module in this function.

More than one execution can be provided. In this latter case they are executed sequentially in the order they appear in the PyModuleDef_slot array.

If we wanted to rewrite our example code to support Multi-phase initialization, we could try something like the following:

~~~c
static int array_mod_exec(PyObject* module) {
    if (PyType_Ready(&arrayType) < 0) {
        return -1;
    }

    Py_INCREF(&arrayType);
    if (PyModule_AddObject(module, "array", (PyObject *) &arrayType) < 0) {
        // While I tought that we would need to decref &arrayType, every snippet of this code that I found in the CPython source doesn't
	// I'm still not sure why
        return -1;
    }
    
    return 0;
}

static PyModuleDef_Slot[] arrayslots { // Our slots array
    {Py_mod_exec, array_mod_exec}, // An exec function. We don't provide a create function and let Python do it for us
    {0, NULL} // a sentinel
}

static PyModuleDef arraymodule = {
	PyModuleDef_HEAD_INIT,
	.m_name = "lds_array",
	.m_doc = "C-like array module",
	.m_size = -1,
	.m_slots = arrayslots // We added our slots to the module def
};

PyMODINIT_FUNC
PyInit_lds_array() {
    return PyModuleDef_Init(&arraymodule); // We now return a PyModuleDef instance instead of the complete module
}
~~~

This is acutally pretty similar to what we had before, we mostly moved some things around.

If you've read [PEP 489](https://www.python.org/dev/peps/pep-0489/), there seems to be quite a few advantage to it. I think this should be the preferred initialization for our extension modules.

# The array object

Moving on we should look at our array.
First thing first, this is how we have defined the array:

~~~c
typedef struct {
	PyObject_HEAD
	PyObject** data;
	Py_ssize_t size;
	PyTypeObject* storedType;
} array;
~~~

It doesn't have many fields.
[PyObject_HEAD](https://github.com/python/cpython/blob/130893debfd97c70e3a89d9ba49892f53e6b9d79/Include/object.h#L86:9) is boilerplate code that defines the initial segment of a PyObject. We always need it at the head of our custom objects.

*data* Is where our references will be stored. We will allocate this memory in the __new__ method of our array.

*size* tells us how big the array is. This is non-changing value that is decided on construction.
[Py_ssize_t](https://github.com/python/cpython/blob/7a2368063f25746d4008a74aca0dc0b82f86ff7b/Include/pyport.h#L84) is a signed number type for which sizeof(size_t) == sizeof(Py_ssize_t) holds.

*storedType* is a pointer to a Python Type ( which is a PyObject too ) we will use to provide some type-checks of inserted elements.

One interesting thing is that, later, I've found out that Python provides a second type of PyObject specifically for objects that have a notion of length ( like containers ): the [PyVarObject](https://github.com/python/cpython/blob/9bdd2de84c1af55fbc006d3f892313623bd0195c/Include/object.h#L118) that adds a field to store the number of items in the object, *ob_size*.

I think we may have used it for the array object but I haven't tried it. 

To make an object a var object we have to use [PyObject_VAR_HEAD](https://github.com/python/cpython/blob/9bdd2de84c1af55fbc006d3f892313623bd0195c/Include/object.h#L101) instead of *PyObject_HEAD* and ensure that the last field of the struct is an array of length one where Python will malloc enough space for ob_size elements.
Lastly, we have to make some changes to the type of the object that we will see soon enough.
You can see an examle of this [here](https://github.com/python/cpython/blob/54ba556c6c7d8fd5504dc142c2e773890c55a774/Include/cpython/tupleobject.h#L9).

~~~c
static PyMemberDef array_members[] = {
	{"size", T_PYSSIZET, offsetof(array, size), READONLY, "Describe how many elements are stored in the array"},
	{NULL}
};
~~~

[PyMemberDef](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Include/structmember.h#L18) describes the attribute of a type as a C struct member.
It has [a few fields](https://docs.python.org/3/c-api/structures.html#c.PyMemberDef):

| Type            | Member    | Use                                                                                      |
|:----------------------------------------------------------------------------------------------------------------------:| 
| const char*     | name      | The name of the attribute                                                                |
| int             | type      | The type of the attribute. See below                                                     |
| Py_ssize_t      | offset    | The offset in bytes that the member is located in the C struct. We use offsetof for this |
| int             | flags     | Flags that defines the readability and writability of the attribute                      |
| const char*     | doc       | The docstring for the attribute                                                          |

Flags can either be 0 for Read-Write access or [READONLY](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Include/structmember.h#L60) going by the reference.
We can see from the source code that there exist a restricted version of both read and write. They are used in just a few places in the code but I could not find what they exactly represent or if we should care about them.

We have a few possible [types](https://github.com/python/cpython/blob/e42b705188271da108de42b55d9344642170aa2b/Include/structmember.h#L27):

| Macro | CType                    |
|:--------------------------------:|
| T_SHORT     | short              |
| T_INT       | int                |
| T_LONG      | long               |
| T_FLOAT     | float              |
| T_DOUBLE    | double             |
| T_STRING    | const char*        |
| T_OBJECT    | PyObject*          |
| T_OBJECT_EX | PyObject*          |
| T_CHAR      | char               |
| T_BYTE      | char               |
| T_UBYTE     | unsigned char      |
| T_UINT      | unsigned int       |
| T_USHORT    | unsigned short     |
| T_ULONG     | unsigned long      |
| T_BOOL      | char               |
| T_LONGLONG  | long long          |
| T_ULONGLOGN | unsigned long long |
| T_PYSSIZET  | Py_ssize_t         |

The difference between T_OBJECT and T_OBJECT_EX is that the first returns None if the member is NULL while the latter raises an AttributeError.
The reference advises us to use the EX type because it handles the del statement more correctly.

This NULL-terminated array of PyMemberDef structures is plugged into the array type to define the object attributes.
There is no real reason to give read access to the size attribute as we support the Python's len trough the sequence protocol. Nonetheless, I was trying out attributes.

Going forward we finally get to the core of the action of a PyObject: its type.

~~~c
static PyTypeObject arrayType = {
	PyVarObject_HEAD_INIT(NULL, 0)
	.tp_name = "lds_array.array",
	.tp_doc = "A c-like array structure",
	.tp_basicsize = sizeof(array),
	.tp_itemsize = 0,
	.tp_flags = Py_TPFLAGS_DEFAULT | Py_TPFLAGS_BASETYPE,
	.tp_new = array_new,
	.tp_init =  array_init,
	.tp_dealloc = array_dealloc,
	.tp_members = array_members,
	.tp_as_sequence = &array_sq_methods,
	.tp_str = array_str
};
~~~

Our array has a pretty bare type, but we have [a lot of possibilities to customize our types](https://docs.python.org/3/c-api/typeobj.html).

Instead of rewriting them here I advise you to look at the [reference page](https://docs.python.org/3/c-api/typeobj.html).
We will look specifically at out example.


