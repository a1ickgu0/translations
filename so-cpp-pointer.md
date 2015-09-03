
[原文地址](http://stackoverflow.com/questions/22146094/why-should-i-use-a-pointer-rather-than-the-object-itself)


Why should I use a pointer rather than the object itself?
===
为什么要用指针而不是直接使用对象？
=====

##问题描述


I'm coming from a Java background and have started working with objects in C++. But one thing that occurred to me is that people often use pointers to objects rather than the objects themselves, for example this declaration:

我刚从 Java 转到使用 C++ 进行面向对开发，我发现一个很让我非常困惑的问题：C++ 中经常出现使用对象指针而不是直接使用对象本身的代码，比如下面这个例子：


```cpp
Object *myObject = new Object;
```

rather than:

而不是使用：

```cpp
Object myObject;
```

Or instead of using a function, let's say testFunc(), like this:

要不就是调用对象的方法（比如 testFunc()）时不使用这种方式：

```cpp
myObject.testFunc();
```

we have to write:

而是得写成这样：


```cpp
myObject->testFunc();
```

But I can't figure out why should we do it this way. I would assume it has to do with efficiency and speed since we get direct access to the memory address. Am I right?

我不明白代码为什么要写成这种形式，我能想到的是指针方式是直接访问内存，这么写代码可以提高代码效率以及执行速度，是这样的么？


##最佳回复来自 Joseph Mansfield

It's very unfortunate that you see dynamic allocation so often. That just shows how many bad C++ programmers there are.

非常不幸，你在代码中遇到这么多的动态内存分配，但这个只能说明有现在有太多不合格的C++程序员。

In a sense, you have two questions bundled up into one. The first is when should we use dynamic allocation (using new)? The second is when should we use pointers?

这么说吧，你的两个问题本质上是同个问题。第一个问题是，应该什么时候使用动态分配（使用 new 方法）？第二问题是，应该什么时候该使用指针？

The important take-home message is that you should always use the appropriate tool for the job. In almost all situations, there is something more appropriate and safer than performing manual dynamic allocation and/or using raw pointers.

**最先要牢记的重点是，你应该根据实际需求选择合适的方法。** 一般来说，使用定义对象的方式比起使用手工动态分配（或new指针）的方式会更加合理以及安全。

###Dynamic allocation
###动态分配

In your question, you've demonstrated two ways of creating an object. The main difference is the storage duration of the object. When doing Object myObject; within a block, the object is created with automatic storage duration, which means it will be destroyed automatically when it goes out of scope. When you do new Object(), the object has dynamic storage duration, which means it stays alive until you explicitly delete it. You should only use dynamic storage duration when you need it. That is, you should always prefer creating objects with automatic storage duration when you can.

你的提问中，所列出的两种分配对象方式的主要区别在于对象的生存期。通过 ``Object myObject`` 方式定义对象，对象的生存期是在其作用域内自维护（automatic storage），这个意味着程序离开对象的作用域之后，对象将被自动销毁。当通过 ``new Object()`` 方式分配对象时，对象的生存期是动态的，这个意味着若不显式地 ``detete`` 对象，对象将一直存在。你应该只在必要的时候使用动态分配对象，换句话说，**只要有可能，你应该首选定义可自维护的对象。**


The main two situations in which you might require dynamic allocation:

这里是两个常见需要动态分配对象的情况：

1. You need the object to outlive the current scope - that specific object at that specific memory location, not a copy of it. If you're okay with copying/moving the object (most of the time you should be), you should prefer an automatic object.

2. You need to allocate a lot of memory, which may easily fill up the stack. It would be nice if we didn't have to concern ourselves with this (most of the time you shouldn't have to), as it's really outside the purview of C++, but unfortunately we have to deal with the reality of the systems we're developing for.

1. **分配不限制作用域的对象**，对象存储在其特定的内存中，而不是在内存中存储对象的拷贝。如果对象是可以拷贝/移动的，一般情况下你应该选择使用定义对象的方式。
2. **定义的对象会消耗大量内存**，这时可能会耗尽栈空间。如果我们永远不需要考虑这个问题那该多好（实际大部分情况下，我们真不需要考虑），因为这个本身已经超出 C++ 语言的范畴，但不幸的是，在我们实际的开发过程中却不得不去处理这个问题。

When you do absolutely require dynamic allocation, you should encapsulate it in a smart pointer or some other type that performs RAII (like the standard containers). Smart pointers provide ownership semantics of dynamically allocated objects. Take a look at std::unique_ptr and std::shared_ptr, for example. If you use them appropriately, you can almost entirely avoid performing your own memory management (see the Rule of Zero).

当你确实需要动态分配对象时，应该将对象封装在一个智能指针（smart pointer）或其他提供[RAII](http://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)机制的类型中（类似标准的 container）。智能指针提供动态对象的所有权语义（ownership），具体可以看一下[``std::unique_ptr``](http://en.cppreference.com/w/cpp/memory/unique_ptr) 和 [``std::shared_ptr``](http://en.cppreference.com/w/cpp/memory/shared_ptr)  这两个例子。如果你使用得当，基本上可以避免自己管理内存（具参见 [Rule of Zero](http://flamingdangerzone.com/cxx11/rule-of-zero/)）。

###Pointers
###指针

However, there are other more general uses for raw pointers beyond dynamic allocation, but most have alternatives that you should prefer. As before, always prefer the alternatives unless you really need pointers.

当然，不使用动态分配而采取原始指针（raw pointer）的用法也很常见，但是大多数情况下动态分配可以取代指针，因此一般情况应该首选动态分配的方法，除非你遇到不得不用指针的情况。


1. You need reference semantics. Sometimes you want to pass an object using a pointer (regardless of how it was allocated) because you want the function to which you're passing it to have access that that specific object (not a copy of it). However, in most situations, you should prefer reference types to pointers, because this is specifically what they're designed for. Note this is not necessarily about extending the lifetime of the object beyond the current scope, as in situation 1 above. As before, if you're okay with passing a copy of the object, you don't need reference semantics.

1. **使用引用语义（reference semantics）的情况**。有时你可能需要通过传递对象的指针（不管对象是如何分配的）以便你可以在函数中去访问/修改这个对象的数据（而不是它的一份拷贝），但是在大多数情况下，你应该优先考虑使用引用方式，而不是指针，因为引用就是被设计出来实现这个需求的。注意，采用这种方式，对象生存期依旧在其作用域内自维护。当然，如果通过传递对象拷贝可以满足要求的情况下是不需要使用引用语义。

2. You need polymorphism. You can only call functions polymorphically (that is, according to the dynamic type of an object) through a pointer or reference to the object. If that's the behaviour you need, then you need to use pointers or references. Again, references should be preferred.

2. **使用多态的情况**。通过传递对象的指针或引用调用多态函数（根据入参类型不同，会调用不同处理函数）。如果你的设计就是可以传递指针或传递引用，显然，应该优先考虑使用传递引用的方式。

3. You want to represent that an object is optional by allowing a nullptr to be passed when the object is being omitted. If it's an argument, you should prefer to use default arguments or function overloads. Otherwise, you should prefer use a type that encapsulates this behaviour, such as boost::optional (or perhaps soon, std::optional - Edit std::optional is voted out of the current C++14 draft n3797).

3. **对于入参对象可选的情况**，常见的通过传递``空指针``表示忽略入参。如果只有一个参数的情况，应该优先考虑使用缺省参数或是对函数进行重载。要不然，你应该优先考虑使用一种可封装此行为的类型，比如 ``boost::optional`` （或者``std::optional``，已经在 C++ 14 草案 n3797 14 中发布  ）。

4. You want to decouple compilation units to improve compilation time. The useful property of a pointer is that you only require a forward declaration of the pointed-to type (to actually use the object, you'll need a definition). This allows you to decouple parts of your compilation process, which may significantly improve compilation time. See the Pimpl idiom.

4. **通过解耦编译类型依赖减少编译时间的情况**。使用指针的一个好处在于可以用于前向声名（forward declaration）指向特定类型（如果使用对象类型，则需要定义对象），这种方式可以减少参与编译的文件，从而显著地提高编译效率，具体可以看 [Pimpl idiom](http://en.wikipedia.org/wiki/Opaque_pointer) 用法。

5. You need to interface with a C library or a C-style library. At this point, you're forced to use raw pointers. The best thing you can do is make sure you only let your raw pointers loose at the last possible moment. You can get a raw pointer from a smart pointer, for example, by using its get member function. If a library performs some allocation for you which it expects you to deallocate via a handle, you can often wrap the handle up in a smart pointer with a custom deleter that will deallocate the object appropriately.

5. **与C库或C风格的库交互的情况**。此时只能够使用指针，这种情况下，你要确保的是指针使用只限定在必要的代码段中。指针可以通过智能指针的转换得到，比如使用智能指针的``get``成员函数。如果C库操作分配的内存需要你在代码中维护并显式地释放时，可以将指针封装在智能指针中，通过实现 ``deleter`` 从而可以有效的地释放对象。


-----
