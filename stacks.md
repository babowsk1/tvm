# Stacks

This chapter contains a general discussion and comparison of register and stack machines, expanded further in Appendix $$\mathbf{C}$$, and describes the two main classes of stack manipulation primitives employed by TVM: the basic and the compound stack manipulation primitives. An informal explanation of their sufficiency for all stack reordering required for correctly invoking other primitives and user-defined functions is also provided. Finally, the problem of efficiently implementing TVM stack manipulation primitives is discussed in [2.3](#efficiency-of-stack-manipulation-primitives)

## Stack calling conventions

A stack machine, such as TVM, uses the stack-and especially the values near the top of the stack - to pass arguments to called functions and primitives (such as built-in arithmetic operations) and receive their results. This section discusses the TVM stack calling conventions, introduces some notation, and compares TVM stack calling conventions with those of certain register machines.

### 2.1.1. Notation for "stack registers". 
 
Recall that a stack machine, as opposed to a more conventional register machine, lacks general-purpose registers. However, one can treat the values near the top of the stack as a kind of "stack registers".

We denote by s0 or $$s(0)$$ the value at the top of the stack, by s1 or $$s(1)$$ the value immediately under it, and so on. The total number of values in the stack is called its depth. If the depth of the stack is $$n$$, then $$s(0), s(1), \ldots$$, $$\mathrm{s}(n-1)$$ are well-defined, while $$\mathrm{s}(n)$$ and all subsequent $$\mathrm{s}(i)$$ with $$i>n$$ are not. Any attempt to use $$\mathrm{s}(i)$$ with $$i \geq n$$ should produce a stack underflow exception.

A compiler, or a human programmer in "TVM code", would use these "stack registers" to hold all declared variables and intermediate values, similarly to the way general-purpose registers are used on a register machine.

### 2.1.2. Pushing and popping values. 
 
When a value $$x$$ is pushed into a stack of depth $$n$$, it becomes the new s0; at the same time, the old s0 becomes the new s1, the old s1-the new s2, and so on. The depth of the resulting stack is $$n+1$$. Similarly, when a value $$x$$ is popped from a stack of depth $$n \geq 1$$, it is the old value of s0 (i.e., the old value at the top of the stack). After this, it is removed from the stack, and the old s 1 becomes the new s0 (the new value at the top of the stack), the old s2 becomes the new s1, and so on. The depth of the resulting stack is $$n-1$$.

If originally $$n=0$$, then the stack is empty, and a value cannot be popped from it. If a primitive attempts to pop a value from an empty stack, a stack underflow exception occurs.

### 2.1.3. Notation for hypothetical general-purpose registers. 
 
In order to compare stack machines with sufficiently general register machines, we will denote the general-purpose registers of a register machine by $$r 0, r 1$$, and so on, or by $$r(0), r(1), \ldots, r(n-1)$$, where $$n$$ is the total number of registers. When we need a specific value of $$n$$, we will use $$n=16$$, corresponding to the very popular x86-64 architecture.

### 2.1.4. The top-of-stack register s0 vs. the accumulator register $$r 0$$. 
 
Some register machine architectures require one of the arguments for most arithmetic and logical operations to reside in a special register called the accumulator. In our comparison, we will assume that the accumulator is the general-purpose register r0; otherwise we could simply renumber the registers. In this respect, the accumulator is somewhat similar to the top-ofstack "register" s0 of a stack machine, because virtually all operations of a stack machine both use s0 as one of their arguments and return their result as $$\mathrm{s} 0$$.

### 2.1.5. Register calling conventions. 
 
When compiled for a register machine, high-level language functions usually receive their arguments in certain registers in a predefined order. If there are too many arguments, these functions take the remainder from the stack (yes, a register machine usually has a stack, too!). Some register calling conventions pass no arguments in registers at all, however, and only use the stack (for example, the original calling conventions used in implementations of Pascal and $$\mathrm{C}$$, although modern implementations of $$\mathrm{C}$$ use some registers as well).

For simplicity, we will assume that up to $$m \leq n$$ function arguments are passed in registers, and that these registers are $$r 0, r 1, \ldots, r(m-1)$$, in that order (if some other registers are used, we can simply renumber them) $$\cdot{ }^{6}$$

$${ }^{6}$$Our inclusion of ro here creates a minor conflict with our assumption that the accumulator register, if present, is also r0; for simplicity, we will resolve this problem by assuming that the first argument to a function is passed in the accumulator destination register $$r(k)$$. This form is common for most RISC processors, and for the XMM and AVX SIMD instruction sets in the x86-64 architecture.

### 2.1.6. Order of function arguments. 

If a function or primitive requires $$m$$ arguments $$x_{1}, \ldots, x_{m}$$, they are pushed by the caller into the stack in the same order, starting from $$x_{1}$$. Therefore, when the function or primitive is invoked, its first argument $$x_{1}$$ is in $$s(m-1)$$, its second argument $$x_{2}$$ is in $$\mathbf{s}(m-2)$$, and so on. The last argument $$x_{m}$$ is in s0 (i.e., at the top of the stack). It is the called function or primitive's responsibility to remove its arguments from the stack.

In this respect the TVM stack calling conventions-obeyed, at least, by TMV primitives-match those of Pascal and Forth, and are the opposite of those of $$\mathrm{C}$$ (in which the arguments are pushed into the stack in the reverse order, and are removed by the caller after it regains control, not the callee).

Of course, an implementation of a high-level language for TVM might choose some other calling conventions for its functions, different from the default ones. This might be useful for certain functions-for instance, if the total number of arguments depends on the value of the first argument, as happens for "variadic functions" such as scanf and printf. In such cases, the first one or several arguments are better passed near the top of the stack, not somewhere at some unknown location deep in the stack.

### 2.1.7. Arguments to arithmetic primitives on register machines. 
 
On a stack machine, built-in arithmetic primitives (such as ADD or DIVMOD) follow the same calling conventions as user-defined functions. In this respect, user-defined functions (for example, a function computing the square root of a number) might be considered as "extensions" or "custom upgrades" of the stack machine. This is one of the clearest advantages of stack machines (and of stack programming languages such as Forth) compared to register machines.

In contrast, arithmetic instructions (built-in operations) on register machines usually get their parameters from general-purpose registers encoded in the full opcode. A binary operation, such as SUB, thus requires two arguments, $$r(i)$$ and $$r(j)$$, with $$i$$ and $$j$$ specified by the instruction. A register $$r(k)$$ for storing the result also must be specified. Arithmetic operations can take several possible forms, depending on whether $$i, j$$, and $$k$$ are allowed to take arbitrary values:

- Three-address form - Allows the programmer to arbitrarily choose not only the two source registers $$r(i)$$ and $$r(j)$$, but also a separate
- Two-address form - Uses one of the two operand registers (usually $$r(i))$$ to store the result of an operation, so that $$k=i$$ is never indicated explicitly. Only $$i$$ and $$j$$ are encoded inside the instruction. This is the most common form of arithmetic operations on register machines, and is quite popular on microprocessors (including the x86 family).
- One-address form - Always takes one of the arguments from the accumulator $$r 0$$, and stores the result in $$r 0$$ as well; then $$i=k=0$$, and only $$j$$ needs to be specified by the instruction. This form is used by some simpler microprocessors (such as Intel 8080).

Note that this flexibility is available only for built-in operations, but not for user-defined functions. In this respect, register machines are not as easily "upgradable" as stack machines. 7

### 2.1.8. Return values of functions. 

In stack machines such as TVM, when a function or primitive needs to return a result value, it simply pushes it into the stack (from which all arguments to the function have already been removed). Therefore, the caller will be able to access the result value through the top-of-stack "register" s0.

This is in complete accordance with Forth calling conventions, but differs slightly from Pascal and $$\mathrm{C}$$ calling conventions, where the accumulator register $$r 0$$ is normally used for the return value.

### 2.1.9. Returning several values. 
 
Some functions might want to return several values $$y_{1}, \ldots, y_{k}$$, with $$k$$ not necessarily equal to one. In these cases, the $$k$$ return values are pushed into the stack in their natural order, starting from $$y_{1}$$.

For example, the "divide with remainder" primitive DIVMOD needs to return two values, the quotient $$q$$ and the remainder $$r$$. Therefore, DIVMOD pushes $$q$$ and $$r$$ into the stack, in that order, so that the quotient is available thereafter at s1 and the remainder at s0. The net effect of DIVMOD is to divide the original value of s1 by the original value of s0, and return the quotient in s1 and the remainder in s0. In this particular case the depth of the stack and the values of all other "stack registers" remain unchanged, because DIVMOD takes two arguments and returns two results. In general, the values of other "stack registers" that lie in the stack below the arguments passed and the values returned are shifted according to the change of the depth of the stack.

In principle, some primitives and user-defined functions might return a variable number of result values. In this respect, the remarks above about variadic functions (cf. [2.1.6](#2.1.6.-order-of-function-arguments.) apply: the total number of result values and their types should be determined by the values near the top of the stack. (For example, one might push the return values $$y_{1}, \ldots, y_{k}$$, and then push their total number $$k$$ as an integer. The caller would then determine the total number of returned values by inspecting s0.)

In this respect TVM, again, faithfully observes Forth calling conventions.

$${ }^{7}$$For instance, if one writes a function for extracting square roots, this function will always accept its argument and return its result in the same registers, in contrast with a hypothetical built-in square root instruction, which could allow the programmer to arbitrarily choose the source and destination registers. Therefore, a user-defined function is tremendously less flexible than a built-in instruction on a register machine.


### 2.1.10. Stack notation. 
 
When a stack of depth $$n$$ contains values $$z_{1}, \ldots$$, $$z_{n}$$, in that order, with $$z_{1}$$ the deepest element and $$z_{n}$$ the top of the stack, the contents of the stack are often represented by a list $$z_{1} z_{2} \ldots z_{n}$$, in that order. When a primitive transforms the original stack state $$S^{\prime}$$ into a new state $$S^{\prime \prime}$$, this is often written as $$S^{\prime}-S^{\prime \prime}$$; this is the so-called stack notation. For example, the action of the division primitive DIV can be described by $$S$$ $$x y-S\lfloor x / y\rfloor$$, where $$S$$ is any list of values. This is usually abbreviated as $$x$$ $$y-\lfloor x / y\rfloor$$, tacitly assuming that all other values deeper in the stack remain intact.

Alternatively, one can describe DIV as a primitive that runs on a stack $$S^{\prime}$$ of depth $$n \geq 2$$, divides s1 by s0, and returns the floor-rounded quotient as s0 of the new stack $$S^{\prime \prime}$$ of depth $$n-1$$. The new value of $$\mathrm{s}(i)$$ equals the old value of $$\mathrm{s}(i+1)$$ for $$1 \leq i<n-1$$. These descriptions are equivalent, but saying that DIV transforms $$x y$$ into $$\lfloor x / y\rfloor$$, or $$\ldots x y$$ into..$$\lfloor x / y\rfloor$$, is more concise.

The stack notation is extensively used throughout Appendix $$\mathbf{A}$$, where all currently defined TVM primitives are listed.

## Explicitly defining the number of arguments to a function.

 Stack machines usually pass the current stack in its entirety to the invoked primitive or function. That primitive or function accesses only the several values near the top of the stack that represent its arguments, and pushes the return values in their place, by convention leaving all deeper values intact. Then the resulting stack, again in its entirety, is returned to the caller.Most TVM primitives behave in this way, and we expect most user-defined functions to be implemented under such conventions. However, TVM provides mechanisms to specify how many arguments must be passed to a called function (cf. [4.1.10](control-flow-continuations-and-exceptions#4.1.10.-determining-the-number-of-arguments-passed-to-and-or-return-values-accepted-from-a-subroutin)). When these mechanisms are employed, the specified number of values are moved from the caller's stack into the (usually initially empty) stack of the called function, while deeper values remain in the caller's stack and are inaccessible to the callee. The caller can also specify how many return values it expects from the called function.

Such argument-checking mechanisms might be useful, for example, for a library function that calls user-provided functions passed as arguments to it.

## Stack manipulation primitives

A stack machine, such as TVM, employs a lot of stack manipulation primitives to rearrange arguments to other primitives and user-defined functions, so that they become located near the top of the stack in correct order. This section discusses which stack manipulation primitives are necessary and sufficient for achieving this goal, and which of them are used by TVM. Some examples of code using these primitives can be found in Appendix $$\mathbf{C}$$.

### 2.2.1. Basic stack manipulation primitives. 
 
The most important stack manipulation primitives used by TVM are the following:

- Top-of-stack exchange operation: XCHG s0, $$\mathrm{s}(i)$$ or XCHG $$\mathrm{s}(i)$$ - Exchanges values of $$\mathbf{s} 0$$ and $$\mathbf{s}(i)$$. When $$i=1$$, operation XCHG $$\mathbf{s} 1$$ is traditionally denoted by SWAP. When $$i=0$$, this is a NOP (an operation that does nothing, at least if the stack is non-empty).
- Arbitrary exchange operation: XCHG $$\mathrm{s}(i), \mathrm{s}(j)$$ - Exchanges values of $$\mathbf{s}(i)$$ and $$\mathbf{s}(j)$$. Notice that this operation is not strictly necessary, because it can be simulated by three top-of-stack exchanges: XCHG $$\mathrm{s}(i)$$; XCHG $$\mathrm{s}(j)$$; XCHG $$\mathrm{s}(i)$$. However, it is useful to have arbitrary exchanges as primitives, because they are required quite often.
- Push operation: PUSH $$\mathrm{s}(i)$$ - Pushes a copy of the (old) value of $$\mathbf{s}(i)$$ into the stack. Traditionally, PUSH s0 is also denoted by DUP (it duplicates the value at the top of the stack), and PUSH s1 by OVER. - Pop operation: POP $$s(i)$$ - Removes the top-of-stack value and puts it into the (new) $$\mathbf{s}(i-1)$$, or the old $$\mathbf{s}(i)$$. Traditionally, POP $$\mathbf{s} 0$$ is also denoted by DROP (it simply drops the top-of-stack value), and POP s1 by NIP.

Some other "unsystematic" stack manipulation operations might be also defined (e.g., ROT, with stack notation $$a b c-b c a$$. While such operations are defined in stack languages like Forth (where DUP, DROP, OVER, NIP and SWAP are also present), they are not strictly necessary because the basic stack manipulation primitives listed above suffice to rearrange stack registers to allow any arithmetic primitives and user-defined functions to be invoked correctly.

### 2.2.2. Basic stack manipulation primitives suffice. 
 
A compiler or a human TVM-code programmer might use the basic stack primitives as follows.

Suppose that the function or primitive to be invoked is to be passed, say, three arguments $$x, y$$, and $$z$$, currently located in stack registers $$\mathbf{s}(i), \mathrm{s}(j)$$, and $$\mathbf{s}(k)$$. In this circumstance, the compiler (or programmer) might issue operation PUSH $$\mathrm{s}(i)$$ (if a copy of $$x$$ is needed after the call to this primitive) or XCHG $$\mathbf{s}(i)$$ (if it will not be needed afterwards) to put the first argument $$x$$ into the top of the stack. Then, the compiler (or programmer) could use either PUSH $$\mathrm{s}\left(j^{\prime}\right)$$ or XCHG $$\mathrm{s}\left(j^{\prime}\right)$$, where $$j^{\prime}=j$$ or $$j+1$$, to put $$y$$ into the new top of the stack. $${ }^{8}$$

Proceeding in this manner, we see that we can put the original values of $$x, y$$, and $$z$$-or their copies, if needed-into locations $$\mathrm{s} 2, \mathrm{~s} 1$$, and $$\mathrm{s} 0$$, using a sequence of push and exchange operations (cf. 2.2.4 and 2.2.5 for a more detailed explanation). In order to generate this sequence, the compiler will need to know only the three values $$i, j$$ and $$k$$, describing the old locations of variables or temporary values in question, and some flags describing whether each value will be needed thereafter or is needed only for this primitive or function call. The locations of other variables and temporary values will be affected in the process, but a compiler (or a human programmer) can easily track their new locations.

Similarly, if the results returned from a function need to be discarded or moved to other stack registers, a suitable sequence of exchange and pop operations will do the job. In the typical case of one return value in s0, this is achieved either by an XCHG $$\mathrm{s}(i)$$ or a POP $$\mathrm{s}(i)$$ (in most cases, a DROP) operation $$9^{9}$$

Rearranging the result value or values before returning from a function is essentially the same problem as arranging arguments for a function call, and is achieved similarly.

$${ }^{8}$$Of course, if the second option is used, this will destroy the original arrangement of $$x$$ in the top of the stack. In this case, one should either issue a SWAP before XCHG $$\mathbf{s}\left(j^{\prime}\right)$$, or replace the previous operation XCHG $$\mathbf{s}(i)$$ with XCHG $$\mathbf{s 1}, \mathbf{s}(i)$$, so that $$x$$ is exchanged with s1 from the beginning. 

### 2.2.3. Compound stack manipulation primitives. 
 
In order to improve the density of the TVM code and simplify development of compilers, compound stack manipulation primitives may be defined, each combining up to four exchange and push or exchange and pop basic primitives. Such compound stack operations might include, for example:

- XCHG2 $$\mathrm{s}(i), \mathrm{s}(j)$$ - Equivalent to XCHG $$\mathrm{s} 1, \mathrm{~s}(i) ; \mathrm{XCHG} \mathrm{s}(j)$$.
- PUSH2 $$\mathbf{s}(i), \mathrm{s}(j)$$ - Equivalent to PUSH $$\mathrm{s}(i) ;$$ PUSH $$\mathbf{s}(j+1)$$.
- XCPU $$\mathbf{s}(i), \mathrm{s}(j)$$ - Equivalent to XCHG $$\mathrm{s}(i)$$; PUSH $$\mathrm{s}(j)$$.
- PUXC $$\mathrm{s}(i), \mathrm{s}(j)$$ - Equivalent to PUSH $$\mathrm{s}(i)$$; SWAP; XCHG $$\mathrm{s}(j+1)$$. When $$j \neq i$$ and $$j \neq 0$$, it is also equivalent to XCHG $$\mathrm{s}(j)$$; PUSH $$\mathrm{s}(i)$$; SWAP.
- XCHG3 $$\mathrm{s}(i), \mathrm{s}(j), \mathrm{s}(k)$$ - Equivalent to XCHG s2, $$\mathrm{s}(i)$$; XCHG s1, $$\mathrm{s}(j)$$; $$\mathrm{XCHG} \mathrm{s}(k)$$.
- PUSH3 $$\mathbf{s}(i), \mathbf{s}(j), \mathbf{s}(k)$$ - Equivalent to PUSH $$\mathbf{s}(i)$$; PUSH $$\mathbf{s}(j+1)$$; PUSH $$\mathrm{s}(k+2)$$.

Of course, such operations make sense only if they admit a more compact encoding than the equivalent sequence of basic operations. For example, if all top-of-stack exchanges, XCHG s1,s(i) exchanges, and push and pop operations admit one-byte encodings, the only compound stack operations suggested above that might merit inclusion in the set of stack manipulation primitives are PUXC, XCHG3, and PUSH3.

These compound stack operations essentially augment other primitives (instructions) in the code with the "true" locations of their operands, somewhat similarly to what happens with two-address or three-address register machine code. However, instead of encoding these locations inside the opcode of the arithmetic or another instruction, as is customary for register machines, we indicate these locations in a preceding compound stack manipulation operation. As already described in [2.1.7](#2.1.7.-arguments-to-arithmetic-primitives-on-register-machines.), the advantage of such an approach is that user-defined functions (or rarely used specific primitives added in a future version of TVM) can benefit from it as well (cf. [C.3](code-density-of-stack-and-register-machines#c.3-sample-non-leaf-function) for a more detailed discussion with examples).

$${ }^{9}$$ Notice that the most common XCHG $$s(i)$$ operation is not really required here if we do not insist on keeping the same temporary value or variable always in the same stack location, but rather keep track of its subsequent locations. We will move it to some other location while preparing the arguments to the next primitive or function call.

### 2.2.4. Mnemonics of compound stack operations. 
 
The mnemonics of compound stack operations, some examples of which have been provided in 2.2 .3 , are created as follows.

The $$\gamma \geq 2$$ formal arguments $$\mathbf{s}\left(i_{1}\right), \ldots, \mathrm{s}\left(i_{\gamma}\right)$$ to such an operation $$O$$ represent the values in the original stack that will end up in $$s(\gamma-1), \ldots$$, s0 after the execution of this compound operation, at least if all $$i_{\nu}, 1 \leq$$ $$\nu \leq \gamma$$, are distinct and at least $$\gamma$$. The mnemonic itself of the operation $$O$$ is a sequence of $$\gamma$$ two-letter strings $$\mathrm{PU}$$ and $$\mathrm{XC}$$, with PU meaning that the corresponding argument is to be PUshed (i.e., a copy is to be created), and XC meaning that the value is to be eXChanged (i.e., no other copy of the original value is created). Sequences of several PU or XC strings may be abbreviated to one PU or XC followed by the number of copies. (For instance, we write PUXC2PU instead of PUXCXCPU.)

As an exception, if a mnemonic would consist of only PU or only XC strings, so that the compound operation is equivalent to a sequence of $$m$$ PUSHes or eXCHanGes, the notation PUSHm or XCHGm is used instead of PUm or XCm.

### 2.2.5. Semantics of compound stack operations. 
 
Each compound $$\gamma$$ ary operation $$O \mathbf{s}\left(i_{1}\right), \ldots, \mathrm{s}\left(i_{\gamma}\right)$$ is translated into an equivalent sequence of basic stack operations by induction in $$\gamma$$ as follows:

- As a base of induction, if $$\gamma=0$$, the only nullary compound stack operation corresponds to an empty sequence of basic stack operations.
- Equivalently, we might begin the induction from $$\gamma=1$$. Then PU $$\mathrm{s}(i)$$ corresponds to the sequence consisting of one basic operation PUSH $$\mathrm{s}(i)$$, and $$\mathrm{XC} \mathrm{s}(i)$$ corresponds to the one-element sequence consisting of XCHG $$\mathrm{s}(i)$$. - For $$\gamma \geq 1$$ (or for $$\gamma \geq 2$$, if we use $$\gamma=1$$ as induction base), there are two subcases:

1. $$O \mathbf{s}\left(i_{1}\right), \ldots, \mathrm{s}\left(i_{\gamma}\right)$$, with $$O=\mathrm{xCO}^{\prime}$$, where $$O^{\prime}$$ is a compound operation of arity $$\gamma-1$$ (i.e., the mnemonic of $$O^{\prime}$$ consists of $$\gamma-1$$ strings $$\mathrm{XC}$$ and PU). Let $$\alpha$$ be the total quantity of PUshes in $$O$$, and $$\beta$$ be that of eXChanges, so that $$\alpha+\beta=\gamma$$. Then the original operation is translated into XCHG $$\mathrm{s}(\beta-1), \mathrm{s}\left(i_{1}\right)$$, followed by the translation of $$O^{\prime} \mathrm{s}\left(i_{2}\right), \ldots, \mathrm{s}\left(i_{\gamma}\right)$$, defined by the induction hypothesis.
2. $$O \mathbf{s}\left(i_{1}\right), \ldots, \mathbf{s}\left(i_{\gamma}\right)$$, with $$O=\mathrm{PU}^{\prime}$$, where $$O^{\prime}$$ is a compound operation of arity $$\gamma-1$$. Then the original operation is translated into PUSH $$\mathrm{s}\left(i_{1}\right)$$; XCHG $$\mathrm{s}(\beta)$$, followed by the translation of $$O^{\prime} \mathrm{s}\left(i_{2}+1\right), \ldots, \mathrm{s}\left(i_{\gamma}+1\right)$$, defined by the induction hypothesis. $${ }^{10}$$

### 2.2.6. Stack manipulation instructions are polymorphic. 
 
Notice that the stack manipulation instructions are almost the only "polymorphic" primitives in TVM-i.e., they work with values of arbitrary types (including the value types that will appear only in future revisions of TVM). For example, SWAP always interchanges the two top values of the stack, even if one of them is an integer and the other is a cell. Almost all other instructions, especially the data processing instructions (including arithmetic instructions), require each of their arguments to be of some fixed type (possibly different for different arguments).

## Efficiency of stack manipulation primitives

Stack manipulation primitives employed by a stack machine, such as TVM, have to be implemented very efficiently, because they constitute more than half of all the instructions used in a typical program. In fact, TVM performs all these instructions in a (small) constant time, regardless of the values involved (even if they represent very large integers or very large trees of cells).

### 2.3.1. Implementation of stack manipulation primitives: using references for operations instead of objects. 
 
The efficiency of TVM's implementation of stack manipulation primitives results from the fact that a typical TVM implementation keeps in the stack not the value objects themselves, but only the references (pointers) to such objects. Therefore, a SWAP instruction only needs to interchange the references at s0 and s1, not the actual objects they refer to.

$${ }^{10}$$ An alternative, arguably better, translation of $$\mathrm{PU} O^{\prime} \mathrm{s}\left(i_{1}\right), \ldots, \mathbf{s}\left(i_{\gamma}\right)$$ consists of the translation of $$O^{\prime} \mathbf{s}\left(i_{2}\right), \ldots, \mathbf{s}\left(i_{\gamma}\right)$$, followed by PUSH $$\mathbf{s}\left(i_{1}+\alpha-1\right) ;$$ XCHG $$\mathbf{s}(\gamma-1)$$.

### 2.3.2. Efficient implementation of DUP and PUSH instructions using copy-on-write. 
 
Furthermore, a DUP (or, more generally, PUSH $$\mathbf{s}(i)$$ ) instruction, which appears to make a copy of a potentially large object, also works in small constant time, because it uses a copy-on-write technique of delayed copying: it copies only the reference instead of the object itself, but increases the "reference counter" inside the object, thus sharing the object between the two references. If an attempt to modify an object with a reference counter greater than one is detected, a separate copy of the object in question is made first (incurring a certain "non-uniqueness penalty" or "copying penalty" for the data manipulation instruction that triggered the creation of a new copy).

### 2.3.3. Garbage collecting and reference counting.
 
When the reference counter of a TVM object becomes zero (for example, because the last reference to such an object has been consumed by a DROP operation or an arithmetic instruction), it is immediately freed. Because cyclic references are impossible in TVM data structures, this method of reference counting provides a fast and convenient way of freeing unused objects, replacing slow and unpredictable garbage collectors.

### 2.3.4. Transparency of the implementation: Stack values are "values", not "references". 
 
Regardless of the implementation details just discussed, all stack values are really "values", not "references", from the perspective of the TVM programmer, similarly to the values of all types in functional programming languages. Any attempt to modify an existing object referred to from any other objects or stack locations will result in a transparent replacement of this object by its perfect copy before the modification is actually performed.

In other words, the programmer should always act as if the objects themselves were directly manipulated by stack, arithmetic, and other data transformation primitives, and treat the previous discussion only as an explanation of the high efficiency of the stack manipulation primitives.

### 2.3.5. Absence of circular references. O
  
One might attempt to create a circular reference between two cells, $$A$$ and $$B$$, as follows: first create $$A$$ and write some data into it; then create $$B$$ and write some data into it, along with a reference to previously constructed cell $$A$$; finally, add a reference to $$B$$ into $$A$$. While it may seem that after this sequence of operations we obtain a cell $$A$$, which refers to $$B$$, which in turn refers to $$A$$, this is not the case. In fact, we obtain a new cell $$A^{\prime}$$, which contains a copy of the data originally stored into cell $$A$$ along with a reference to cell $$B$$, which contains a reference to (the original) cell $$A$$.

In this way the transparent copy-on-write mechanism and the "everything is a value" paradigm enable us to create new cells using only previously constructed cells, thus forbidding the appearance of circular references. This property also applies to all other data structures: for instance, the absence of circular references enables TVM to use reference counting to immediately free unused memory instead of relying on garbage collectors. Similarly, this property is crucial for storing data in the TVM Blockchain.