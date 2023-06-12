# Control flow, continuations, and exceptions

This chapter describes continuations, which may represent execution tokens and exception handlers in TVM. Continuations are deeply involved with the control flow of a TVM program; in particular, subroutine calls and conditional and iterated execution are implemented in TVM using special primitives that accept one or more continuations as their arguments.

We conclude this chapter with a discussion of the problem of recursion and of families of mutually recursive functions, exacerbated by the fact that cyclic references are not allowed in TVM data structures (including TVM code).

## Continuations and subroutines

Recall (cf 1.1.3) that Continuation values represent "execution tokens" that can be executed later-for example, by EXECUTE=CALLX ("execute" or "call indirect") or JMPX ("jump indirect") primitives. As such, the continuations are responsible for the execution of the program, and are heavily used by control flow primitives, enabling subroutine calls, conditional expressions, loops, and so on.

### 4.1.1. Ordinary continuations. 

The most common kind of continuations are the ordinary continuations, containing the following data:

- A Slice code (cf. $$\mathbf{1 . 1 . 3}$$ and 3.2.2 , containing (the remainder of) the TVM code to be executed.
- A (possibly empty) Stack stack, containing the original contents of the stack for the code to be executed.
- A (possibly empty) list save of pairs (c $$\left.(i), v_{i}\right)$$ (also called "savelist"), containing the values of control registers to be restored before the execution of the code.
- A 16-bit integer value cp, selecting the TVM codepage used to interpret the TVM code from code.
- An optional non-negative integer nargs, indicating the number of arguments expected by the continuation. 4.1.2. Simple ordinary continuations. In most cases, the ordinary continuations are the simplest ones, having empty stack and save. They consist essentially of a reference code to (the remainder of) the code to be executed, and of the codepage $$c$$ to be used while decoding the instructions from this code.

### 4.1.3. Current continuation cc. 

The "current continuation" cc is an important part of the total state of TVM, representing the code being executed right now (cf. 1.1). In particular, what we call "the current stack" (or simply "the stack") when discussing all other primitives is in fact the stack of the current continuation. All other components of the total state of TVM may be also thought of as parts of the current continuation cc; however, they may be extracted from the current continuation and kept separately as part of the total state for performance reasons. This is why we describe the stack, the control registers, and the codepage as separate parts of the TVM state in 1.4

### 4.1.4. Normal work of TVM, or the main loop. 

TVM usually performs the following operations:

If the current continuation cc is an ordinary one, it decodes the first instruction from the Slice code, similarly to the way other cells are deserialized by TVM LD* primitives (cf. 3.2 and 3.2.11): it decodes the opcode first, and then the parameters of the instruction (e.g., 4-bit fields indicating "stack registers" involved for stack manipulation primitives, or constant values for "push constant" or "literal" primitives). The remainder of the Slice is then put into the code of the new cc, and the decoded operation is executed on the current stack. This entire process is repeated until there are no operations left in cc. code.

If the code is empty (i.e., contains no bits of data and no references), or if a (rarely needed) explicit subroutine return (RET) instruction is encountered, the current continuation is discarded, and the "return continuation" from control register c0 is loaded into cc instead (this process is discussed in more detail starting in 4.1.6.$${ }^{20}$$ Then the execution continues by parsing operations from the new current continuation.

### 4.1.5. Extraordinary continuations. 

In addition to the ordinary continuations considered so far (cf. 4.1.1), TVM includes some extraordinary contin-

$${ }^{20}$$ If there are no bits of data left in code, but there is still exactly one reference, an implicit JMP to the cell at that reference is performed instead of an implicit RET. uations, representing certain less common states. Examples of extraordinary continuations include:

- The continuation ec_quit with its parameter set to zero, which represents the end of the work of TVM. This continuation is the original value of c0 when TVM begins executing the code of a smart contract.
- The continuation ec_until, which contains references to two other continuations (ordinary or not) representing the body of the loop being executed and the code to be executed after the loop.

Execution of an extraordinary continuation by TVM depends on its specific class, and differs from the operations for ordinary continuations described in 4.1.4

### 4.1.6. Switching to another continuation: JMP and RET. 

The process of switching to another continuation $$c$$ may be performed by such instructions as JMPX (which takes $$c$$ from the stack) or RET (which uses c0 as $$c$$ ). This process is slightly more complex than simply setting the value of cc to $$c$$ : before doing this, either all values or the top $$n$$ values in the current stack are moved to the stack of the continuation $$c$$, and only then is the remainder of the current stack discarded.

If all values need to be moved (the most common case), and if the continuation $$c$$ has an empty stack (also the most common case; notice that extraordinary continuations are assumed to have an empty stack), then the new stack of $$c$$ equals the stack of the current continuation, so we can simply transfer the current stack in its entirety to $$c$$. (If we keep the current stack as a separate part of the total state of TVM, we have to do nothing at all.)

Determining the number $$n$$ of arguments passed to the next continuation $$c$$. By default, $$n$$ equals the depth of the current stack. However, if $$c$$ has an explicit value of nargs (number of arguments to be provided), then $$n$$ is computed as $$n^{\prime}$$, equal to $$c$$.nargs minus the current depth of $$c$$ 's stack.Furthermore, there are special forms of JMPX and RET that provide an explicit value $$n^{\prime \prime}$$, the number of parameters from the current stack to be passed to continuation $$c$$. If $$n^{\prime \prime}$$ is provided, it must be less than or equal to

$${ }^{21}$$ Technically, TVM may simply invoke a virtual method run() of the continuation currently in cc. the depth of the current stack, or else a stack underflow exception occurs. If both $$n^{\prime}$$ and $$n^{\prime \prime}$$ are provided, we must have $$n^{\prime} \leq n^{\prime \prime}$$, in which case $$n=n^{\prime}$$ is used. If $$n^{\prime \prime}$$ is provided and $$n^{\prime}$$ is not, then $$n=n^{\prime \prime}$$ is used.

One could also imagine that the default value of $$n^{\prime \prime}$$ equals the depth of the original stack, and that $$n^{\prime \prime}$$ values are always removed from the top of the original stack even if only $$n^{\prime}$$ of them are actually moved to the stack of the next continuation $$c$$. Even though the remainder of the current stack is discarded afterwards, this description will become useful later.

### 4.1.8. Restoring control registers from the new continuation $$c$$. 

After the new stack is computed, the values of control registers present in c.save are restored accordingly, and the current codepage $$\mathrm{cp}$$ is also set to c.cp. Only then does TVM set cc equal to the new $$c$$ and begin its execution. $${ }^{22}$$

### 4.1.9. Subroutine calls: CALLX or EXECUTE primitives. 

The execution of continuations as subroutines is slightly more complicated than switching to continuations.

Consider the CALLX or EXECUTE primitive, which takes a continuation $$c$$ from the (current) stack and executes it as a subroutine.

Apart from doing the stack manipulations described in $$\mathbf{4 . 1 . 6}$$ and $$\mathbf{4 . 1 . 7}$$ and setting the new control registers and codepage as described in $$\mathbf{4 . 1 . 8}$$ these primitives perform several additional steps:

1. After the top $$n^{\prime \prime}$$ values are removed from the current stack (cf. $$\mathbf{4 . 1 . 7}$$ ), the (usually empty) remainder is not discarded, but instead is stored in the (old) current continuation cc.
2. The old value of the special register co is saved into the (previously empty) savelist cc. save.
3. The continuation cc thus modified is not discarded, but instead is set as the new c0, which performs the role of "next continuation" or "return continuation" for the subroutine being called.
4. After that, the switching to $$c$$ continues as before. In particular, some control registers are restored from c.save, potentially overwriting the value of c0 set in the previous step. (Therefore, a good optimization would be to check that $$c 0$$ is present in $$c$$.save from the very beginning, and skip the three previous steps as useless in this case.)

$${ }^{22}$$ The already used savelist cc. save of the new cc is emptied before the execution starts. In this way, the called subroutine can return control to the caller by switching the current continuation to the return continuation saved in c0. Nested subroutine calls work correctly because the previous value of c0 ends up saved into the new c0's control register savelist c0. save, from which it is restored later.

### 4.1.10. Determining the number of arguments passed to and/or return values accepted from a subroutine. 

Similarly to JMPX and RET, CALLX also has special (rarely used) forms, which allow us to explicitly specify the number $$n^{\prime \prime}$$ of arguments passed from the current stack to the called subroutine (by default, $$n^{\prime \prime}$$ equals the depth of the current stack, i.e., it is passed in its entirety). Furthermore, a second number $$n^{\prime \prime \prime}$$ can be specified, used to set nargs of the modified cc continuation before storing it into the new c0; the new nargs equals the depth of the old stack minus $$n^{\prime \prime}$$ plus $$n^{\prime \prime \prime}$$. This means that the caller is willing to pass exactly $$n^{\prime \prime}$$ arguments to the called subroutine, and is willing to accept exactly $$n^{\prime \prime \prime}$$ results in their stead.

Such forms of CALLX and RET are mostly intended for library functions that accept functional arguments and want to invoke them safely. Another application is related to the "virtualization support" of TVM, which enables TVM code to run other TVM code inside a "virtual TVM machine". Such virtualization techniques might be useful for implementing sophisticated payment channels in the TVM Blockchain (cf. [1, 5]).

### 4.1.11. CALLCC: call with current continuation. Notice that TVM supports a form of the "call with current continuation" primitive. 

Namely, primitive CALLCC is similar to CALLX or JMPX in that it takes a continuation $$c$$ from the stack and switches to it; however, CALLCC does not discard the previous current continuation $$c^{\prime}$$ (as JMPX does) and does not write $$c^{\prime}$$ to c0 (as CALLX does), but rather pushes $$c^{\prime}$$ into the (new) stack as an extra argument to $$c$$. The primitive JMPXDATA does a similar thing, but pushes only the (remainder of the) code of the previous current continuation as a Slice.

## Control flow primitives: conditional and iterated execution

### 4.2.1. Conditional execution: IF, IFNOT, IFELSE. 

An important modification of EXECUTE (or CALLX) consists in its conditional forms. For example, IF accepts an integer $$x$$ and a continuation $$c$$, and executes $$c$$ (in the same way as EXECUTE would do it) only if $$x$$ is non-zero; otherwise both values are simply discarded from the stack. Similarly, IFNOT accepts $$x$$ and $$c$$, but executes $$c$$ only if $$x=0$$. Finally, IFELSE accepts $$x, c$$, and $$c^{\prime}$$, removes these values from the stack, and executes $$c$$ if $$x \neq 0$$ or $$c^{\prime}$$ if $$x=0$$.

### 4.2.2. Iterated execution and loops. 

More sophisticated modifications of EXECUTE include:

- REPEAT - Takes an integer $$n$$ and a continuation $$c$$, and executes $$c n$$ times, 23
- WHILE - Takes $$c^{\prime}$$ and $$c^{\prime \prime}$$, executes $$c^{\prime}$$, and then takes the top value $$x$$ from the stack. If $$x$$ is non-zero, it executes $$c^{\prime \prime}$$ and then begins a new loop by executing $$c^{\prime}$$ again; if $$x$$ is zero, it stops.
- UNTIL - Takes $$c$$, executes it, and then takes the top integer $$x$$ from the stack. If $$x$$ is zero, a new iteration begins; if $$x$$ is non-zero, the previously executed code is resumed.

### 4.2.3. Constant, or literal, continuations. 

We see that we can create arbitrarily complex conditional expressions and loops in the TVM code, provided we have a means to push constant continuations into the stack. In fact, TVM includes special versions of "literal" or "constant" primitives that cut the next $$n$$ bytes or bits from the remainder of the current code cc.code into a cell slice, and then push it into the stack not as a Slice (as a PUSHSLICE does) but as a simple ordinary Continuation (which has only code and $$\mathrm{cp}$$ ).

The simplest of these primitives is PUSHCONT, which has an immediate argument $$n$$ describing the number of subsequent bytes (in a byte-oriented version of TVM) or bits to be converted into a simple continuation. Another primitive is PUSHREFCONT, which removes the first cell reference from the current continuation cc.code, converts the cell referred to into a cell slice, and finally converts the cell slice into a simple continuation.

### 4.2.4. Constant continuations combined with conditional or iterated execution primitives. 

Because constant continuations are very often used as arguments to conditional or iterated execution primitives, combined

$${ }^{23}$$ The implementation of REPEAT involves an extraordinary continuation that remembers the remaining number of iterations, the body of the loop $$c$$, and the return continuation $$c^{\prime}$$. (The latter term represents the remainder of the body of the function that invoked REPEAT, which would be normally stored in c0 of the new cc.) versions of these primitives (e.g., IFCONT or UNTILREFCONT) may be defined in a future revision of TVM, which combine a PUSHCONT or PUSHREFCONT with another primitive. If one inspects the resulting code, IFCONT looks very much like the more customary "conditional-branch-forward" instruction.

## Operations with continuations

### 4.3.1. Continuations are opaque. 

Notice that all continuations are opaque, at least in the current version of TVM, meaning that there is no way to modify a continuation or inspect its internal data. Almost the only use of a continuation is to supply it to a control flow primitive.

While there are some arguments in favor of including support for nonopaque continuations in TVM (along with opaque continuations, which are required for virtualization), the current revision offers no such support.

### 4.3.2. Allowed operations with continuations. 

However, some operations with opaque continuations are still possible, mostly because they are equivalent to operations of the kind "create a new continuation, which will do something special, and then invoke the original continuation". Allowed operations with continuations include:

- Push one or several values into the stack of a continuation $$c$$ (thus creating a partial application of a function, or a closure).
- Set the saved value of a control register $$c(i)$$ inside the savelist $$c$$.save of a continuation $$c$$. If there is already a value for the control register in question, this operation silently does nothing.

### 4.3.3. Example: operations with control registers. 

TVM has some primitives to set and inspect the values of control registers. The most important of them are PUSH $$c(i)$$ (pushes the current value of $$c(i)$$ into the stack) and POP $$\mathrm{c}(i)$$ (sets the value of $$\mathrm{c}(i)$$ from the stack, if the supplied value is of the correct type). However, there is also a modified version of the latter instruction, called POPSAVE $$\mathrm{c}(i)$$, which saves the old value of $$\mathrm{c}(i)($$ for $$i>0)$$ into the continuation at c0 as described in $$\mathbf{4 . 3 . 2}$$ before setting the new value.

### 4.3.4. Example: setting the number of arguments to a function in its code. 

The primitive LEAVEARGS $$n$$ demonstrates another application of continuations in an operation: it leaves only the top $$n$$ values of the current stack, and moves the remainder to the stack of the continuation in c0. This primitive enables a called function to "return" unneeded arguments to its caller's stack, which is useful in some situations (e.g., those related to exception handling).

### 4.3.5. Boolean circuits. 

A continuation $$c$$ may be thought of as a piece of code with two optional exit points kept in the savelist of $$c$$ : the principal exit point given by $$c . c 0:=c$$.save $$(c 0)$$, and the auxiliary exit point given by $$c . c 1:=c . \operatorname{save}(c 1)$$. If executed, a continuation performs whatever action it was created for, and then (usually) transfers control to the principal exit point, or, on some occasions, to the auxiliary exit point. We sometimes say that a continuation $$c$$ with both exit points $$c . c 0$$ and $$c . c 1$$ defined is a two-exit continuation, or a boolean circuit, especially if the choice of the exit point depends on some internally-checked condition.

### 4.3.6. Composition of continuations. 

One can compose two continuations $$c$$ and $$c^{\prime}$$ simply by setting $$c . c 0$$ or $$c . c 1$$ to $$c^{\prime}$$. This creates a new continuation denoted by $$c \circ_{0} c^{\prime}$$ or $$c \circ_{1} c^{\prime}$$, which differs from $$c$$ in its savelist. (Recall that if the savelist of $$c$$ already has an entry corresponding to the control register in question, such an operation silently does nothing as explained in 4.3 .2 .

By composing continuations, one can build chains or other graphs, possibly with loops, representing the control flow. In fact, the resulting graph resembles a flow chart, with the boolean circuits corresponding to the "condition nodes" (containing code that will transfer control either to c0 or to c1 depending on some condition), and the one-exit continuations corresponding to the "action nodes".

### 4.3.7. Basic continuation composition primitives. 

Two basic primitives for composing continuations are COMPOS (also known as SETCONT c0 and BOOLAND) and COMPOSALT (also known as SETCONT c1 and BOOLOR), which take $$c$$ and $$c^{\prime}$$ from the stack, set $$c . c 0$$ or $$c . c 1$$ to $$c^{\prime}$$, and return the resulting continuation $$c^{\prime \prime}=c \circ_{0} c^{\prime}$$ or $$c \circ_{1} c^{\prime}$$. All other continuation composition operations can be expressed in terms of these two primitives.

### 4.3.8. Advanced continuation composition primitives. 

However, TVM can compose continuations not only taken from stack, but also taken from c0 or $$c 1$$, or from the current continuation cc; likewise, the result may be pushed into the stack, stored into either c0 or c1, or used as the new current continuation (i.e., the control may be transferred to it). Furthermore, TVM can define conditional composition primitives, performing some of the above actions only if an integer value taken from the stack is non-zero.

For instance, EXECUTE can be described as cc $$\leftarrow c \circ_{0} c c$$, with continuation $$c$$ taken from the original stack. Similarly, JMPX is cc $$\leftarrow c$$, and RET (also known as RETTRUE in a boolean circuit context) is cc $$\leftarrow$$ c0. Other interesting primitives include THENRET $$\left(c^{\prime} \leftarrow c \circ_{0} c 0\right)$$ and ATEXIT $$\left(c 0 \leftarrow c \circ_{0} c 0\right)$$.

Finally, some "experimental" primitives also involve $$c 1$$ and $$\circ_{1}$$. For example:

- RETALT or RETFALSE does $$c c \leftarrow c 1$$.
- Conditional versions of RET and RETALT may also be useful: RETBOOL takes an integer $$x$$ from the stack, and performs RETTRUE if $$x \neq 0$$, RETFALSE otherwise.
- INVERT does c0 $$\leftrightarrow \mathrm{c} 1$$; if the two continuations in c0 and c1 represent the two branches we should select depending on some boolean expression, INVERT negates this expression on the outer level.
- INVERTCONT does $$c . c 0 \leftrightarrow c . c 1$$ to a continuation $$c$$ taken from the stack.
- Variants of ATEXIT include ATEXITALT $$\left(c 1 \leftarrow c \circ_{1} c 1\right)$$ and SETEXITALT $$\left(c 1 \leftarrow\left(c \circ_{0} \mathrm{c} 0\right) \circ_{1} c 1\right)$$.
- BOOLEVAL takes a continuation $$c$$ from the stack and does $$c c \leftarrow\left(\left(c \circ_{0}\right.\right.$$ $$\left.(\mathrm{PUSH}-1)) \circ_{1}(\mathrm{PUSHO})\right) \circ_{0} \mathrm{cc}$$. If $$c$$ represents a boolean circuit, the net effect is to evaluate it and push either -1 or 0 into the stack before continuing.


## Continuations as objects

### 4.4.1. Representing objects using continuations. 

Object-oriented programming in Smalltalk (or Objective C) style may be implemented with the aid of continuations. For this, an object is represented by a special continuation $$o$$. If it has any data fields, they can be kept in the stack of $$o$$, making o a partial application (i.e., a continuation with a non-empty stack).

When somebody wants to invoke a method $$m$$ of $$o$$ with arguments $$x_{1}, x_{2}$$, $$\ldots, x_{n}$$, she pushes the arguments into the stack, then pushes a magic number corresponding to the method $$m$$, and then executes o passing $$n+1$$ arguments (cf. 4.1.10). Then $$o$$ uses the top-of-stack integer $$m$$ to select the branch with the required method, and executes it. If o needs to modify its state, it simply computes a new continuation $$o^{\prime}$$ of the same sort (perhaps with the same code as $$o$$, but with a different initial stack). The new continuation $$o^{\prime}$$ is returned to the caller along with whatever other return values need to be returned.

### 4.4.2. Serializable objects. 

Another way of representing Smalltalk-style objects as continuations, or even as trees of cells, consists in using the JMPREFDATA primitive (a variant of JMPXDATA, cf. $$\mathbf{4 . 1 . 1 1}$$ ), which takes the first cell reference from the code of the current continuation, transforms the cell referred to into a simple ordinary continuation, and transfers control to it, first pushing the remainder of the current continuation as a Slice into the stack. In this way, an object might be represented by a cell $$\tilde{o}$$ that contains JMPREFDATA at the beginning of its data, and the actual code of the object in the first reference (one might say that the first reference of cell $$\tilde{o}$$ is the class of object $$\tilde{o}$$ ). Remaining data and references of this cell will be used for storing the fields of the object.

Such objects have the advantage of being trees of cells, and not just continuations, meaning that they can be stored into the persistent storage of a TVM smart contract.

### 4.4.3. Unique continuations and capabilities. 

It might make sense (in a future revision of TVM) to mark some continuations as unique, meaning that they cannot be copied, even in a delayed manner, by increasing their reference counter to a value greater than one. If an opaque continuation is unique, it essentially becomes a capability, which can either be used by its owner exactly once or be transferred to somebody else.

For example, imagine a continuation that represents the output stream to a printer (this is an example of a continuation used as an object, cf. 4.4.1). When invoked with one integer argument $$n$$, this continuation outputs the character with code $$n$$ to the printer, and returns a new continuation of the same kind reflecting the new state of the stream. Obviously, copying such a continuation and using the two copies in parallel would lead to some unintended side effects; marking it as unique would prohibit such adverse usage.

## Exception handling

TVM's exception handling is quite simple and consists in a transfer of control to the continuation kept in control register c2. 4.5.1. Two arguments of the exception handler: exception parameter and exception number. Every exception is characterized by two arguments: the exception number (an Integer) and the exception parameter (any value, most often a zero Integer). Exception numbers 0-31 are reserved for TVM, while all other exception numbers are available for user-defined exceptions.

### 4.5.2. Primitives for throwing an exception. 

There are several special primitives used for throwing an exception. The most general of them, THROWANY, takes two arguments, $$v$$ and $$0 \leq n<2^{16}$$, from the stack, and throws the exception with number $$n$$ and value $$v$$. There are variants of this primitive that assume $$v$$ to be a zero integer, store $$n$$ as a literal value, and/or are conditional on an integer value taken from the stack. User-defined exceptions may use arbitrary values as $$v$$ (e.g., trees of cells) if needed.

### 4.5.3. Exceptions generated by TVM. 

Of course, some exceptions are generated by normal primitives. For example, an arithmetic overflow exception is generated whenever the result of an arithmetic operation does not fit into a signed 257-bit integer. In such cases, the arguments of the exception, $$v$$ and $$n$$, are determined by TVM itself.

### 4.5.4. Exception handling. 

The exception handling itself consists in a control transfer to the exception handler-i.e., the continuation specified in control register c2, with $$v$$ and $$n$$ supplied as the two arguments to this continuation, as if a JMP to c2 had been requested with $$n^{\prime \prime}=2$$ arguments (cf. 4.1.7 and 4.1.6). As a consequence, $$v$$ and $$n$$ end up in the top of the stack of the exception handler. The remainder of the old stack is discarded.

Notice that if the continuation in c2 has a value for c2 in its savelist, it will be used to set up the new value of c2 before executing the exception handler. In particular, if the exception handler invokes THROWANY, it will rethrow the original exception with the restored value of c2. This trick enables the exception handler to handle only some exceptions, and pass the rest to an outer exception handler.

### 4.5.5. Default exception handler. 

When an instance of TVM is created, c2 contains a reference to the "default exception handler continuation", which is an ec_fatal extraordinary continuation (cf. 4.1.5). Its execution leads to the termination of the execution of TVM, with the arguments $$v$$ and $$n$$ of the exception returned to the outside caller. In the context of the TVM Blockchain, $$n$$ will be stored as a part of the transaction's result. 4.5.6. TRY primitive. A TRY primitive can be used to implement $$\mathrm{C}++-$$ like exception handling. This primitive accepts two continuations, $$c$$ and $$c^{\prime}$$. It stores the old value of c2 into the savelist of $$c^{\prime}$$, sets c2 to $$c^{\prime}$$, and executes $$c$$ just as EXECUTE would, but additionally saving the old value of c2 into the savelist of the new c0 as well. Usually a version of the TRY primitive with an explicit number of arguments $$n^{\prime \prime}$$ passed to the continuation $$c$$ is used.

The net result is roughly equivalent to $$\mathrm{C}++$$ 's try $$\{c\} \operatorname{catch}(\ldots)$$ $$\left\{c^{\prime}\right\}$$ operator.

### 4.5.7. List of predefined exceptions. 

Predefined exceptions of TVM correspond to exception numbers $$n$$ in the range $$0-31$$. They include:

- Normal termination $$(n=0)$$ - Should never be generated, but it is useful for some tricks.
- Alternative termination $$(n=1)$$ - Again, should never be generated.
- Stack underflow $$(n=2)$$ - Not enough arguments in the stack for a primitive.
- Stack overflow $$(n=3)$$ - More values have been stored on a stack than allowed by this version of TVM.
- Integer overflow $$(n=4)$$ - Integer does not fit into $$-2^{256} \leq x<2^{256}$$, or a division by zero has occurred.
- Range check error $$(n=5)$$ - Integer out of expected range.
- Invalid opcode $$(n=6)$$ - Instruction or its immediate arguments cannot be decoded.
- Type check error $$(n=7)-$$ An argument to a primitive is of incorrect value type.
- Cell overflow $$(n=8)$$ - Error in one of the serialization primitives.
- Cell underflow $$(n=9)$$ - Deserialization error.
- Dictionary error $$(n=10)$$ - Error while deserializing a dictionary object.
- Unknown error $$(n=11)$$ - Unknown error, may be thrown by user programs. - Fatal error $$(n=12)$$ - Thrown by TVM in situations deemed impossible.
- Out of gas $$(n=13)$$ - Thrown by TVM when the remaining gas $$\left(g_{r}\right)$$ becomes negative. This exception usually cannot be caught and leads to an immediate termination of TVM.

Most of these exceptions have no parameter (i.e., use a zero integer instead). The order in which these exceptions are checked is outlined below in $$\mathbf{4 . 5 . 8}$$.

### 4.5.8. Order of stack underflow, type check, and range check exceptions. 

All TVM primitives first check whether the stack contains the required number of arguments, generating a stack underflow exception if this is not the case. Only then are the type tags of the arguments and their ranges (e.g., if a primitive expects an argument not only to be an Integer, but also to be in the range from 0 to 256) checked, starting from the value in the top of the stack (the last argument) and proceeding deeper into the stack. If an argument's type is incorrect, a type-checking exception is generated; if the type is correct, but the value does not fall into the expected range, a range check exception is generated.

Some primitives accept a variable number of arguments, depending on the values of some small fixed subset of arguments located near the top of the stack. In this case, the above procedure is first run for all arguments from this small subset. Then it is repeated for the remaining arguments, once their number and types have been determined from the arguments already processed.

## Functions, recursion, and dictionaries

### 4.6.1. The problem of recursion. 

The conditional and iterated execution primitives described in $$\mathbf{4 . 2}$$ along with the unconditional branch, call, and return primitives described in $$\mathbf{4 . 1}$$ enable one to implement more or less arbitrary code with nested loops and conditional expressions, with one notable exception: one can only create new constant continuations from parts of the current continuation. (In particular, one cannot invoke a subroutine from itself in this way.) Therefore, the code being executed-i.e., the current continuation-gradually becomes smaller and smaller. $${ }^{24}$$

$${ }^{24}$$ An important point here is that the tree of cells representing a TVM program cannot have cyclic references, so using CALLREF along with a reference to a cell higher up the tree 

### 4.6.2. $$Y$$-combinator solution: pass a continuation as an argument to itself. 

One way of dealing with the problem of recursion is by passing a copy of the continuation representing the body of a recursive function as an extra argument to itself. Consider, for example, the following code for a factorial function:

$$\begin{array}{ll}71 & \text { PUSHINT 1 } \\ \text { 9C } & \text { PUSHCONT }\{ \\ 22 & \text { PUSH s2 } \\ 72 & \text { PUSHINT } 2 \\ \text { B9 } & \text { LESS } \\ \text { DC } & \text { IFRET } \\ 59 & \text { ROTREV } \\ 21 & \text { PUSH s1 } \\ \text { A8 } & \text { MUL } \\ 01 & \text { SWAP } \\ \text { A5 } & \text { DEC } \\ \text { 02 } & \text { XCHG s2 } \\ 20 & \text { DUP } \\ \text { D9 } & \text { JMPX } \\ & \text { \} } \\ 20 & \text { DUP } \\ \text { D8 } & \text { EXECUTE } \\ 30 & \text { DROP } \\ 31 & \text { NIP }\end{array}$$

This roughly corresponds to defining an auxiliary function body with three arguments $$n, x$$, and $$f$$, such that $$\operatorname{body}(n, x, f)$$ equals $$x$$ if $$n<2$$ and $$f(n-$$ $$1, n x, f)$$ otherwise, then invoking $$\operatorname{bod} y(n, 1, \operatorname{body})$$ to compute the factorial of $$n$$. The recursion is then implemented with the aid of the DUP; EXECUTE construction, or DUP; JMPX in the case of tail recursion. This trick is equivalent to applying $$Y$$-combinator to a function body.

### 4.6.3. A variant of $$Y$$-combinator solution. 

Another way of recursively computing the factorial, more closely following the classical recursive definition

$$$$
\operatorname{fact}(n):= \begin{cases}1 & \text { if } n<2 \\ n \cdot \operatorname{fact}(n-1) & \text { otherwise }\end{cases}
$$$$

would not work. is as follows:

$$\begin{array}{ll}\text { 9D } & \text { PUSHCONT }\{ \\ 21 & \text { OVER } \\ \text { C102 } & \text { LESSINT } 2 \\ 92 & \text { PUSHCONT }\{ \\ 5 B & \text { 2DROP } \\ 71 & \text { PUSHINT } 1 \\ & \text { F } \\ \text { E0 } & \text { IFJMP } \\ 21 & \text { OVER } \\ \text { A5 } & \text { DEC } \\ 01 & \text { SWAP } \\ 20 & \text { DUP } \\ \text { D8 } & \text { EXECUTE } \\ \text { A8 } & \text { MUL } \\ & \text { 3 } \\ 20 & \text { DUP } \\ \text { D9 } & \text { JMPX }\end{array}$$

This definition of the factorial function is two bytes shorter than the previous one, but it uses general recursion instead of tail recursion, so it cannot be easily transformed into a loop.

### 4.6.4. Comparison: non-recursive definition of the factorial function. 

Incidentally, a non-recursive definition of the factorial with the aid of a REPEAT loop is also possible, and it is much shorter than both recursive definitions:

$$\begin{array}{ll}71 & \text { PUSHINT } 1 \\ 01 & \text { SWAP } \\ 20 & \text { DUP } \\ 94 & \text { PUSHCONT }\{ \\ 66 & \text { TUCK } \\ \text { A8 } & \text { MUL } \\ 01 & \text { SWAP } \\ \text { A5 } & \text { DEC } \\ & \text { \} } \\ \text { E4 } & \text { REPEAT } \\ 30 & \text { DROP }\end{array}$$

### 4.6.5. Several mutually recursive functions. 

If one has a collection $$f_{1}, \ldots, f_{n}$$ of mutually recursive functions, one can use the same trick by passing the whole collection of continuations $$\left\{f_{i}\right\}$$ in the stack as an extra $$n$$ arguments to each of these functions. However, as $$n$$ grows, this becomes more and more cumbersome, since one has to reorder these extra arguments in the stack to work with the "true" arguments, and then push their copies into the top of the stack before any recursive call.

### 4.6.6. Combining several functions into one tuple. 

One might also combine a collection of continuations representing functions $$f_{1}, \ldots, f_{n}$$ into a "tuple" $$\mathbf{f}:=\left(f_{1}, \ldots, f_{n}\right)$$, and pass this tuple as one stack element $$\mathbf{f}$$. For instance, when $$n \leq 4$$, each function can be represented by a cell $$\tilde{f}_{i}$$ (along with the tree of cells rooted in this cell), and the tuple may be represented by a cell $$\tilde{\mathbf{f}}$$, which has references to its component cells $$\tilde{f}_{i}$$. However, this would lead to the necessity of "unpacking" the needed component from this tuple before each recursive call.

### 4.6.7. Combining several functions into a selector function. 

Another approach is to combine several functions $$f_{1}, \ldots, f_{n}$$ into one "selector function" $$f$$, which takes an extra argument $$i, 1 \leq i \leq n$$, from the top of the stack, and invokes the appropriate function $$f_{i}$$. Stack machines such as TVM are well-suited to this approach, because they do not require the functions $$f_{i}$$ to have the same number and types of arguments. Using this approach, one would need to pass only one extra argument, $$f$$, to each of these functions, and push into the stack an extra argument $$i$$ before each recursive call to $$f$$ to select the correct function to be called.

### 4.6.8. Using a dedicated register to keep the selector function. 

However, even if we use one of the two previous approaches to combine all functions into one extra argument, passing this argument to all mutually recursive functions is still quite cumbersome and requires a lot of additional stack manipulation operations. Because this argument changes very rarely, one might use a dedicated register to keep it and transparently pass it to all functions called. This is the approach used by TVM by default.

### 4.6.9. Special register c3 for the selector function. 

In fact, TVM uses a dedicated register c3 to keep the continuation representing the current or global "selector function", which can be used to invoke any of a family of mutually recursive functions. Special primitives CALL $$n n$$ or CALLDICT $$n n$$ (cf. A.8.7) are equivalent to PUSHINT $$n n$$; PUSH c3; EXECUTE, and similarly JMP $$n n$$ or JMPDICT $$n n$$ are equivalent to PUSHINT $$n n$$; PUSH c3; JMPX. In this way a TVM program, which ultimately is a large collection of mutually recursive functions, may initialize c3 with the correct selector function representing the family of all the functions in the program, and then use CALL $$n n$$ to invoke any of these functions by its index (sometimes also called the selector of a function).

### 4.6.10. Initialization of c3. 

A TVM program might initialize c3 by means of a POP c3 instruction. However, because this usually is the very first action undertaken by a program (e.g., a smart contract), TVM makes some provisions for the automatic initialization of c3. Namely, c3 is initialized by the code (the initial value of cc) of the program itself, and an extra zero (or, in some cases, some other predefined number $$s$$ ) is pushed into the stack before the program's execution. This is approximately equivalent to invoking JMPDICT 0 (or JMPDICT $$s$$ ) at the very beginning of a program-i.e., the function with index zero is effectively the main() function for the program.

### 4.6.11. Creating selector functions and switch statements. 

TVM makes special provisions for simple and concise implementation of selector functions (which usually constitute the top level of a TVM program) or, more generally, arbitrary switch or case statements (which are also useful in TVM programs). The most important primitives included for this purpose are IFBITJMP, IFNBITJMP, IFBIT JMPREF, and IFNBITJMPREF (cf. A.8.2). They effectively enable one to combine subroutines, kept either in separate cells or as subslices of certain cells, into a binary decision tree with decisions made according to the indicated bits of the integer passed in the top of the stack.

Another instruction, useful for the implementation of sum-product types, is PLDUZ (cf. A.7.2). This instruction preloads the first several bits of a Slice into an Integer, which can later be inspected by IFBITJMP and other similar instructions.

### 4.6.12. Alternative: using a hashmap to select the correct function. 

Yet another alternative is to use a Hashmap (cf. 3.3) to hold the "collection" or "dictionary" of the code of all functions in a program, and use the hashmap lookup primitives (cf. A.10 to select the code of the required function, which can then be BLESSed into a continuation (cf. A.8.5 and executed. Special combined "lookup, bless, and execute" primitives, such as DICTIGETJMP and DICTIGETEXEC, are also available (cf. A.10.11). This approach may be more efficient for larger programs and switch statements.