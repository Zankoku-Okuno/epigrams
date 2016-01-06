# On C and C++

It is an unfortunate necessity that these two languages should be lumped together in the same location.
This state of affairs is wholly created by the concept of "zero-cost abstraction".
On the one hand, "abstraction" (or, Stroustrop's idea of it) makes the two languages radically different.
On the other, "zero-cost" ties the languages with unbreakable bonds.
The situation may well doom both languages, but for now I think the C standards bodies are smart enough to avoid that trap.

## The Deathtrap Languages

Any instance of undefined behaviour makes it impossible to reason about the program in which it appears.
The philosophy I've developed is to always treat undefined behaviour as an error, because at one possibility is simply to crash the program then and there (if you're lucky; the other option is to silently corrupt your data).
Both C and C++ have laundry lists of undefined behaviour which rears its head everywhere from the most basic operations to the most intricate corner-cases.

TODO dynamic memory allocation

TODO runtime arithmetic driving dynamic memory allocation

TODO routines that can error do not force checking

TODO any pointer could be null (related above by malloc)

TODO no array bounds/pointer arithmetic are checked 

TODO casting makes the type system nearly meaningless

TODO silent automatic type conversions

## Linkers and ABIs

C exposes a dumb linker; C++ is usually implemented with a dumb linker, and can also expose it with `extern "C"`.
The dumb linker is a necessity in systems programming, because systems programs need to be able to interface with every langauge, both present and future.
The C++ language adds types together with its identifiers to determine linkage, going through great pains to extract information from the types involved.
We will never have general-purpose linkers that can do this, so C++ must fundamentally use a dumb linker.

Languages choose their type system based on the guarantees the language is meant to make.
The choice of guarantees is essentially driven by taste.
The only type systems that can accommodate all tastes are dependent type systems.
Unfortunately, it is impossible to do reconstruct dependent types in general.
But if we choose a type system that isn't dependently-types, it won't suit all tastes, and thus all languages.
So: do we solve an impossible problem, or arbitrarily restrict the usable languages on a machine?
No.

C++ manages to achieve its linking through name-mangling;
the salient features of a type are serialized into simple text, and this is appended, along with some magic bytes, to the end of the programmer-chosen identifier.
The mangling algorithm must be carefully chosen so that each identifier-type combination repeatably produces a unique dumb symbol for the linker.
Every C++ compiler does this, but they do not all agree with one another.
The C++ standard left name-mangling to be implementation-defined, which means there is no C++ library that is binary-portable...
unless it falls back to the C FFI with `extern "C"`, which defeats the purpose.

C is portable, C++ is not, and that's the end of it.

TODO mode this paragraph to a different rant:
The thing that makes programming hard is that languages offer lots of possibilities, and even in the best languages, the overwhelming majority of possible programs is wrong.
The greatest benefit of a type system is that it rules out a vast number of these wrong programs.
Most type systems (those that are subsumed by System F with subtyping) are remarkably simple,
but dependent type systems are whole programming languages by themselves: they offer a massive universe of possibility.
How can we hope to improve the situation by adding more wrong options?

## C Types as Data Formats

TODO C syntax is used for data format specs, but the notions are always wrong;
do not use C to specify binary data (transmission) formats,
it fails to give a specific size, give endianness, and worst: gives the illusion that the data can be portably `memmap`ed into a C data type

## On Elegance

A language is not excellent because of the good features it has, but because of the bad features it lacks.
A language is excellent not when it has many features, but when it covers the same range of possibility with only a few.
It is clear that C++ adds many features, but it is my contention that these features do not add commensurate power;
C therefore excels C++.
Let's look at what C++ adds.

Disclaimer: in the following, I note several times that C++ has a some "complicated rules".
Taken individualy, the rules are not so difficult to learn, but together they add up.
A misunderstanding in even one of these rule sets causes the programmer to introduce an error, and therefore break the whole system.
In languages as bit-twiddly as C and C++, that means someone just stole your box.

Maybe you're a rockstar; maybe you think C++ isn't hard to learn?
I've mastered four diverse languages and have a good grasp on over a dozen more.
I've been learning C++ for five years now, but in my research for this, I'm running across basic concepts I'd never even heard of before.
I shudder to think of the danger an average programmer brings into a C++ project just because of the language.

### Templates
Some sugar for the preprocessor, plus a some intricate instantiation rules.
If a template is needed in C, simply create a file which substitutes in arguments in the form of preprocessor definitions.
To instantiate a template, `#define` the template's arguments, then `#include` the template file.
The C preprocessor has its well-known downsides, but if its problems prove too great, it is not difficult to substitute in a more suitable template language.
If you give all your template files the `.i` extension or similar (see below), you can inventory the amount of code generated with a simple `grep`.

In C++, knowledge of what is instantiated where is obscured by an algorithm buried in the standards documents
(yes, the section number is known, but standards are famously written more in legalese than computerese).
The algorithm itself is a fairly complex beast, 
The benefit is fewer files, though your text editor should be quite capable, and that arguments are automatically checked.
Oddly, the famously poor error messages from instantiated templates reveal their implementation as little more than preprocessing,
even though the techniques to check templates before instantiation were known at the time they were introduced.

### Methods

Functions.

### Virtual Methods

Function pointers.

### Implementation Inheritance

Composition. Because is-a-is-a-has-a.

Okay, I guess you also need some boilerplate to delegate the call off to a member.

### Interface Inheritace

Wait, no: C++ doesn't add this.
But C does have it, because an interface is a vtable, which is just a structfull of function pointers.

In fact, the structfull-of-function-pointers approach allows you to use interfaces and implementations independently.
That is, it's just easy to add interfaces to implementation as the usual implementations to interfaces.

### Namespaces

This is a good addition for the API, but it breaks the ABI (see below).
Unfortunately, many style guides discourage `using` a namespace, so we see "name-mangling" everywhere anyway,
recognized by the copious sprinkling of `std::`, `boost::`, and so forth.

On the other hand, C just uses a dumb linker, so it doesn't matter what characters appear in identifiers.
If we had just added colon to the list of alaowable characters in identifiers, we would be very near what C++ namespaces actually do in practive.
And it wouldn't have messed with the ABI.

### Lambdas

Honestly, this is a weak spot of C; there's no getting around it.
Nevertheless, taking the language as a whole, I'm not sure it really matters.

Sure, lambda is the foundational concept in computer science, the fundamental force of nature, but C is about building a wheel.
It's only when you deviate from the use cases of C --- operating systems, drivers, runtime systems, microcontrollers --- that you need lambda.
When you get so high-level you need lambda, it may be best to switch to a language that is just high-level, plain and simple: ML, Haskell, even Lisp, if you choose the right compiler.
For the most part, I think C does just fine without lambdas, but to see that, let's tease apart all the use cases.

Perhaps you don't want to pollute the namespace with a one-off helper function.
Lambdas can appear right where they're used, and you don't have to name them.
Lambda-lifting is a small price to pay, even if you need to manually pass captured variables.

Perhaps you don't know what function you want to use until run-time.
Function pointers.

Perhaps you want to create a family of functions, each parameterized differently.
How many of these families do you need in a low-level program?
Just make a struct for each family that carries the parameters.
If you also need to vary the algorithm, but leave the parameter schema the same, bundle the parameters with a function pointer.

Perhaps you really want lots of functions all of which need to close over diverse sets of local variables.
Use a Lisp. or a ML. Or Haskell. Or anything that isn't a systems programming language.
Here's the thing: if a variable is closed over, when should it be deallocated?
In C/C++ you will have to do this manually (or, litter your code with `my_choice_of_smart_ptr<type_and_do_not_forget_the_closing>`).
In a high-level language, we can use garbage collection, so ot's not a problem to begin with.
Although, Rust manages to be a systems programming language with reasonable lambdas --- it has an ownership/lifetime system --- which should help push C++ further away.

### References

Actually, this is one of the C++ features I truly envy.
In C and C++ you have to be careful with pointer ownership.
In C++ however, we have references, and references never have ownership of their data.

If I had references in C, I would use pointers only when the referencee is owned by that pointer, and references wherever the referencee is not owned.
That would fix a ton of memory management mistakes even before they happen.
I'm honestly surprised no one is even talking about backporting references into C, what's the committee thinking, I wonder?
That said, we have Rust now, so if I really need careful memory management, I'd can go there without bothering C++.

### Abstraction

Stroustrop has said, on-camera, that C doesn't have facilities for abstraction.
I question his knowledge of C.
I'm not even sure he read K&R.
Be that as it may, the attempt to achieve "zero-cost" abstractions really is impressive, and I can't slight C++ for it.
Zero-cost abstractions are essentially understood to mean that C++ attempts be be "as fast as C",
which strikes me as an odd use of the word "zero-cost", since both Fortran and assembly are faster still.

In C, the sad fact of the matter is that abstraction means indirection, and indirection is costly.
Now, you can sometimes combat the indirection using whole-program analysis, essentially inlining lots of function calls and combining the dereferences.
To be fair, C++ has done a wonderful job of this, so we should really not be surprised when C++ compilers are slow.

## File Extensions

C files use `.h` for headers and `.c` for implementation files.
Sometimes there are C files with `.i`, `.inl`, or something similar, which are are files meant for inclusion in both headers and implementation files, such as files of inline functions.
These are non-standard however.

C++ use a variety of extensions, usually involving the addition of `xx`, `pp`, or `++` to the corresponding C file concept.
There's also the `.hh`/`.cc` pair.
For simplicity we'll use `xx` here, so C++ header files will be `.hxx` and C++ implemetation files will be `.cxx`.
The choice of these should be the same within a project, but it doesn't matter much.
Admittedly, when I see CPP, I think C PreProcessor, but no one ever adds `.cpp` for a file to be run through CPP.
The thing to avoid is distinguishing C++ files with capital `.H` and `.C` extensions, as several file systems are case-insensitive.

Many C++ projects decide to use `.h` as the extension for its header files.
This causes confusion for automated tools that attempt to identify the file.
C++ projects can use extensions for their files that disambiguate them from C files.
C projects have no ability to disambiguate, since they only have `.h` available.
Therefore, do not use the `.h` extension for C++ files, or you will annoy the people that use tools, which is everyone.
