---
layout: post
title: A cursory look at LLVM and Clang
---

Today, I spent my day looking at [LLVM](http://llvm.org/) and [Clang](http://clang.llvm.org/). It's more like a cursory look rather than an in-depth investigation. The first thing that you want to do is to install LLVM tools. You could checkout the subversion repositories and build them from source. Be prepared to wait for an hour or so and have about 6 gigabytes of free space. Or, you could just invoke:

{% highlight shell %}
    $ brew llvm
{% endhighlight %}

It took only a few minutes or so because it's a bottled package (binary). After that, we could write our first program:

{% highlight shell %}
    $ cat test.m
    #import <Foundation/Foundation.h>

    int main(int argc, char *argv[]) {
        NSLog(@"Hello World");
        return 0;
    }

    $ clang -framework Foundation test.m -o test

    $ ./test
    2013-08-12 20:12:30.409 test[6490:707] Hello World
{% endhighlight %}

Actually, we don't need to install LLVM to perform the steps above because Xcode comes with `clang`. But, we do need some LLVM tools if we want to play around with the LLVM intermediate representation ("LLVM IR").

Compiling a program is actually a multi step process that involves preprocessing, parsing, optimization, code generation, assembly, and linking. In addition to that, a compiler is normally implemented in a modular way; such that the part that handles processing the programming language code (front end) is independent from the part that handles generating the machine code (back end). This allows the compiler to handle multiple languages for multiple target machines. [Some](http://www.infeig.unige.ch/support/cpil/lect/basics/compilers/) [pictures](http://en.wikipedia.org/wiki/Compiler) that show a compiler architecture. If you have some existential questions about compilers, the following [article](http://oopweb.com/Compilers/Documents/Compilers/Volume/cha03s.htm) describes bootstrapping a compiler so that it can self-compile itself.

In LLVM, the middle layers, where the front end and the back end meet, are represented by the LLVM IR, a [well specified](http://llvm.org/docs/LangRef.html) code representation. The LLVM IR is defined in three isomorphic forms: a human readable textual format, an in-memory data structure, and an efficient on-disk binary format (bitcode). Using the above source file, we can see the intermediate representation:

{% highlight shell %}
    $ clang test.m -S -emit-llvm -o test.ll
    $ cat test.ll
    ; ModuleID = 'test.m'
    target datalayout = "e-p:64:64:64-i1:8:8-i8:8:8-i16:16:16-i32:32:32-i64:64:64-f32:32:32-f64:64:64-v64:64:64-v128:128:128-a0:0:64-s0:64:64-f80:128:128-n8:16:32:64-S128"
    target triple = "x86_64-apple-macosx10.8.0"

    %0 = type opaque
    %struct.NSConstantString = type { i32*, i32, i8*, i64 }

    @__CFConstantStringClassReference = external global [0 x i32]
    @.str = linker_private unnamed_addr constant [12 x i8] c"Hello World\00", align 1
    @_unnamed_cfstring_ = private constant %struct.NSConstantString { i32* getelementptr inbounds ([0 x i32]* @__CFConstantStringClassReference, i32 0, i32 0), i32 1992, i8* getelementptr inbounds ([12 x i8]* @.str, i32 0, i32 0), i64 11 }, section "__DATA,__cfstring"

    define i32 @main(i32 %argc, i8** %argv) uwtable ssp {
      %1 = alloca i32, align 4
      %2 = alloca i32, align 4
      %3 = alloca i8**, align 8
      store i32 0, i32* %1
      store i32 %argc, i32* %2, align 4
      store i8** %argv, i8*** %3, align 8
      call void (%0*, ...)* @NSLog(%0* bitcast (%struct.NSConstantString* @_unnamed_cfstring_ to %0*))
      ret i32 0
    }

    declare void @NSLog(%0*, ...)

    !llvm.module.flags = !{!0, !1, !2, !3}

    !0 = metadata !{i32 1, metadata !"Objective-C Version", i32 2}
    !1 = metadata !{i32 1, metadata !"Objective-C Image Info Version", i32 0}
    !2 = metadata !{i32 1, metadata !"Objective-C Image Info Section", metadata !"__DATA, __objc_imageinfo, regular, no_dead_strip"}
    !3 = metadata !{i32 4, metadata !"Objective-C Garbage Collection", i32 0}
{% endhighlight %}

The file `test.ll` contains the human readable LLVM IR which can be converted into the bitcode format:

{% highlight shell %}
    $ llvm-as test.ll
    $ file test.bc
    test.bc: LLVM bit-code object x86_64
    $ od -x -N 4 test.bc
    0000000      c0de    0b17
    0000004
{% endhighlight %}

Interesting trivia: the magic number for bitcode files is `0x0b17c0de`. The tool `llvm-dis` disassembles the bitcode into the human readable IR. And the tool `lli` directly executes programs from LLVM bitcode.

{% highlight shell %}
    $ rm test.ll
    $ llvm-dis test.bc # You'll get the test.ll file back
    $ lli -load=/System/Library/Frameworks/Foundation.framework/Versions/Current/Foundation test.bc
    2013-08-12 22:23:50.181 lli[9516:707] Hello World
{% endhighlight %}

Well, that's it. A cursory look at LLVM and Clang. More to come. Oh by the way, a highly recommended [article](http://www.aosabook.org/en/llvm.html) on LLVM by [Chris Lattner](http://www.nondot.org/sabre/) himself can be found in The Architecture of Open Source Applications [book](http://aosabook.org/en/index.html).
