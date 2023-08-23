# A Instructions and opcodes

This appendix lists all instructions available in the (experimental) codepage zero of TVM, as explained in 5.3 .

We list the instructions in lexicographical opcode order. However, the opcode space is distributed in such way as to make all instructions in each category (e.g., arithmetic primitives) have neighboring opcodes. So we first list a number of stack manipulation primitives, then constant primitives, arithmetic primitives, comparison primitives, cell primitives, continuation primitives, dictionary primitives, and finally application-specific primitives.

We use hexadecimal notation (cf. $$\mathbf{1 . 0}$$ ) for bitstrings. Stack registers $$\mathbf{s}(i)$$ usually have $$0 \leq i \leq 15$$, and $$i$$ is encoded in a 4-bit field (or, on a few rare occasions, in an 8-bit field). Other immediate parameters are usually 4-bit, 8-bit, or variable length.

The stack notation described in 2.1.10 is extensively used throughout this appendix.

## A.1 Gas prices

The gas price for most primitives equals the basic gas price, computed as $$P_{b}:=10+b+5 r$$, where $$b$$ is the instruction length in bits and $$r$$ is the number of cell references included in the instruction. When the gas price of an instruction differs from this basic price, it is indicated in parentheses after its mnemonics, either as $$(x)$$, meaning that the total gas price equals $$x$$, or as $$(+x)$$, meaning $$P_{b}+x$$. Apart from integer constants, the following expressions may appear:

* $$C_{r}$$ - The total price of "reading" cells (i.e., transforming cell references into cell slices). Currently equal to 100 or 25 gas units per cell depending on whether it is the first time a cell with this hash is being "read" during the current run of the VM or not.
* $$L$$ - The total price of loading cells. Depends on the loading action required.
* $$B_{w}$$ - The total price of creating new _Builders_. Currently equal to 0 gas units per builder.
* $$C_{w}$$ - The total price of creating new _Cells_ from _Builders_. Currently equal to 500 gas units per cell. By default, the gas price of an instruction equals $$P:=P_{b}+C_{r}+L+B_{w}+C_{w}$$.

## A.2 Stack manipulation primitives

This section includes both the basic (cf. [$$\mathbf{2.2.1}$$](stacks.md#2.2.1.-basic-stack-manipulation-primitives.) and the compound (cf. [$$\mathbf{2.2.3}$$](stacks.md#2.2.3.-compound-stack-manipulation-primitives.)) stack manipulation primitives, as well as some "unsystematic" ones. Some compound stack manipulation primitives, such as XCPU or XCHG2, turn out to have the same length as an equivalent sequence of simpler operations. We have included these primitives regardless, so that they can easily be allocated shorter opcodes in a future revision of TVM-or removed for good.

Some stack manipulation instructions have two mnemonics: one Forthstyle (e.g., -ROT), the other conforming to the usual rules for identifiers (e.g., ROTREV). Whenever a stack manipulation primitive (e.g., PICK) accepts an integer parameter $$n$$ from the stack, it must be within the range $$0 \ldots 255$$; otherwise a range check exception happens before any further checks.

### A.2.1. Basic stack manipulation primitives.

* $$00~-~NOP,~\text{does~nothing.}$$
* $$01~-~XCHG~{s}(1),~\text{also~known~as~SWAP.}$$
* $$0i~-~XCHG~{s}(i)~\text{or}~XCHG~{s}(0),{s}(i),~\text{interchanges~the~top~of~the~stack~with}~{s}(i),~1 ≤ i ≤ 15.$$
* $$10ij~-~XCHG~{s}(i),{s}(j),~1 ≤ i < j ≤ 15,~\text{interchanges}~{s}(i)~\text{with}~{s}(j).$$
* $$11ii~-~XCHG~{s}(0),{s}(ii),~\text{with}~0 ≤ ii ≤ 255.$$
* $$1i~-~XCHG~{s}(1),{s}(i),~2 ≤ i ≤ 15.$$
* $$2i~-~PUSH~{s}(i),~0 ≤ i ≤ 15,~\text{pushes~a~copy~of~the~old}~{s}(i)~\text{into~the~stack.}$$
* $$20~-~PUSH~{s}(0),~\text{also~known~as~DUP.}$$
* $$21~-~PUSH~{s}(1),~\text{also~known~as~OVER.}$$
* $$3i~-~POP~{s}(i),~0 ≤ i ≤ 15,~\text{pops~the~old~top-of-stack~value~into~the~old}~{s}(i).$$
* $$30~-~POP~{s}(0),~\text{also~known~as~DROP,~discards~the~top-of-stack~value.}$$
* $$31~-~\text{POP}~{s}(1),~\text{also~known~as~NIP.}$$

### A.2.2. Compound stack manipulation primitives.

Parameters $$i, j$$, and $$k$$ of the following primitives all are 4-bit integers in the range $$0 \ldots 15$$.

* $$4ijk~-~XCHG3~{s}(i), {s}(j), {s}(k)$$, equivalent to $$XCHG~{s}(2), {s}(i);~XCHG~{s}(1), {s}(j);~XCHG~{s}(0), {s}(k),~\text{with}~0 ≤ i, j, k ≤ 15$$.
* $$50ij~-~XCHG2~{s}(i),~{s}(j)$$, equivalent to $$XCHG~{s}(1),~{s}(i);~XCHG~{s}(j)$$.
* $$51ij~-~XCPU~{s}(i),~{s}(j)$$, equivalent to $$XCHG~{s}(i);~PUSH~{s}(j)$$.
* $$52ij~-~PUXC~{s}(i),~{s}(j-1)$$, equivalent to $$PUSH~{s}(i);~SWAP;~XCHG~{s}(j)$$.
* $$53ij~-~PUSH2~{s}(i),~{s}(j)$$, equivalent to $$PUSH~{s}(i);~PUSH~{s}(j+1)$$.
* $$540ijk~-~XCHG3~{s}(i),~{s}(j),~{s}(k)$$ (long form).
* $$541ijk~-~XC2PU~{s}(i),~{s}(j),~{s}(k)$$, equivalent to $$XCHG2~{s}(i),~{s}(j);~PUSH~{s}(k)$$.
* $$542ijk~-~XCPUXC~{s}(i),~{s}(j),~{s}(k-1)$$, equivalent to $$XCHG~{s}(1),~{s}(i);~PUXC~{s}(j),~{s}(k-1)$$.
* $$543ijk~-~XCPU2~{s}(i),~{s}(j),~{s}(k)$$, equivalent to $$XCHG~{s}(i);~PUSH2~{s}(j),~{s}(k)$$.
* $$544ijk~-~PUXC2~{s}(i),~{s}(j-1),~{s}(k-1)$$, equivalent to $$PUSH~{s}(i);~XCHG~{s}(2);~XCHG2~{s}(j),~{s}(k)$$.
* $$545ijk~-~PUXCPU~{s}(i),~{s}(j-1),~{s}(k-1)$$, equivalent to $$PUXC~{s}(i),~{s}(j-1);~PUSH~{s}(k)$$.
* $$546ijk~-~PU2XC~{s}(i),~{s}(j-1),~{s}(k-2)$$, equivalent to $$PUSH~{s}(i);~SWAP;~PUXC~{s}(j),~{s}(k-1)$$.
* $$547ijk~-~PUSH3~{s}(i),~{s}(j),~{s}(k)$$, equivalent to $$PUSH~{s}(i);~PUSH2~{s}(j+1),~{s}(k+1)$$.
* $$54C_~-~unused$$.

### A.2.3. Exotic stack manipulation primitives.

* $$55ij~-~\text{BLKSWAP}~i+1,j +1$$, permutes two blocks $${s}(j +i+1). . . {s}(j +1)$$ and $${s}(j). . . {s}(0)$$, for $$0 ≤ i, j ≤ 15$$. Equivalent to $$\text{REVERSE}~i + 1,j + 1;~\text{REVERSE}~j + 1,0;~\text{REVERSE}~i + j + 2,0$$.
* $$5513~-~\text{ROT2}~\text{or}~2ROT~(a~b~c~d~e~f~–~c~d~e~f~a~b)$$, rotates the three topmost pairs of stack entries.
* $$550i~-~\text{ROLL}~i + 1$$, rotates the top $$i + 1$$ stack entries. Equivalent to $$\text{BLKSWAP}~1,i + 1$$.
* $$55i0~-~\text{ROLLREV}~i+1~\text{or}~-ROLL~i+1$$, rotates the top $$i+1$$ stack entries in the other direction. Equivalent to $$\text{BLKSWAP}~i + 1,1$$.
* $$56ii~-~\text{PUSH}~{s}(ii)~\text{for}~0 ≤ ii ≤ 255.$$
* $$57ii~-~\text{POP}~{s}(ii)~\text{for}~0 ≤ ii ≤ 255.$$
* $$58~-~\text{ROT}~(a~b~c~–~b~c~a)$$, equivalent to $$\text{BLKSWAP}~1,2~\text{or~to}~\text{XCHG2}~s2,s1$$.
* $$59~-~\text{ROTREV}~\text{or}~-ROT~(a~b~c~–~c~a~b)$$, equivalent to $$\text{BLKSWAP}~2,1~\text{or~to}~\text{XCHG2}~s2,s2$$.
* $$5A~-~\text{SWAP2}~\text{or}~2SWAP~(a~b~c~d~–~c~d~a~b)$$, equivalent to $$\text{BLKSWAP}~2,2~\text{or~to}~\text{XCHG2}~s3,s2$$.
* $$5B~-~\text{DROP2~or~2DROP}~(a~b~–~),~\text{equivalent~to}~\text{DROP;}~\text{DROP.}$$
* $$5C~-~\text{DUP2~or~2DUP}~(a~b~–~a~b~a~b),~\text{equivalent~to}~\text{PUSH2}~{s}(1),{s}(0).$$
* $$5D~-~\text{OVER2~or~2OVER}~(a~b~c~d~–~a~b~c~d~a~b),~\text{equivalent~to}~\text{PUSH2}~{s}(3),{s}(2).$$
* $$5Eij~-~\text{REVERSE}~i+2,j,~\text{reverses~the~order~of}~{s}(j+i+1)~\ldots~{s}(j)~\text{for}~0 ≤ i, j ≤ 15;~\text{equivalent~to~a~sequence~of}~\frac{b}{2}~\text{XCHGs.}$$
* $$5F0i~-~\text{BLKDROP}~i,~\text{equivalent~to}~\text{DROP}~\text{performed}~i~\text{times.}$$
* $$5Fij~-~\text{BLKPUSH}~i,j$$, equivalent to $$\text{PUSH}~{s}(j)~\text{performed}~i~\text{times},~1 ≤ i ≤ 15,~0 ≤ j ≤ 15$$.
* $$60~-~\text{PICK~or~PUSHX,~pops~integer}~i~\text{from~the~stack,~then~performs}~\text{PUSH}~{s}(i).$$
* $$61~-~\text{ROLLX,~pops~integer}~i~\text{from~the~stack,~then~performs}~\text{BLKSWAP}~1,i.$$

## A.3 Tuple, List, and Null primitives

Tuples are ordered collections consisting of at most 255 TVM stack values of arbitrary types (not necessarily the same). Tuple primitives create, modify, and unpack Tuples; they manipulate values of arbitrary types in the process, similarly to the stack primitives. We do not recommend using Tuples of more than 15 elements.

When a Tuple $$$t$$$ contains elements $$x_{1}, \ldots, x_{n}$$ (in that order), we write $$$t=\left(x_{1}, \ldots, x_{n}\right)$$; number $$n \geq 0$$ is the *length* of *Tuple* $$$t$$$. It is also denoted by $$|t|$$. Tuples of length two are called pairs, and Tuples of length three are triples.

Lisp-style lists are represented with the aid of pairs, i.e., tuples consisting of exactly two elements. An empty list is represented by a Null value, and a non-empty list is represented by pair $$(h, t)$$, where $$h$$ is the first element of the list, and $$$t$$$ is its tail.

### A.3.1. Null primitives.

The following primitives work with (the only) value $$\perp$$ of type *Null*, useful for representing empty lists, empty branches of binary trees, and absence of values in *Maybe* $$X$$ types. An empty *Tuple* created by $$NIL$$ could have been used for the same purpose; however, *Null* is more efficient and costs less gas.

* $$6D~-~\text{NULL}~\text{or}~\text{PUSHNULL}~(–~\bot)$$, pushes the only value of type *Null*.
* $$6E~-~\text{ISNULL}~(x~–~?)$$, checks whether x is a *Null*, and returns $$−1$$ or $$0$$ accordingly.


### A.3.2. Tuple primitives.

* $$6F0n~-~\text{TUPLE}~n~(x1~.~.~.~xn~–~t)$$, creates a new *Tuple* $$t = (x1, . . . , xn)$$ containing $$n$$ values $$x1, . . . , xn$$, where $$0 ≤ n ≤ 15$$.
* $$6F00~-~\text{NIL}~( – t)$$, pushes the only *Tuple* $$t = ()$$ of length zero.
* $$6F01~-~\text{SINGLE}~(x~–~t)$$, creates a singleton $$t := (x)$$, i.e., a *Tuple* of length one.
* $$6F02~-~\text{PAIR}~\text{or}~\text{CONS}~(x~y~–~t)$$, creates pair $$t := (x, y)$$.
* $$6F03~-~\text{TRIPLE}~(x~y~z~–~t)$$, creates triple $$t := (x, y, z)$$.
* $$6F1k~-~\text{INDEX}~k~(t~–~x)$$, returns the $$k$$-th element of a *Tuple* $$t$$, where $$0 ≤ k ≤ 15$$. In other words, returns $$x_k+1$$ if $$t = (x1, . . . , xn)$$. If $$k ≥ n$$, throws a range check exception.
* $$6F10~-~\text{FIRST}~\text{or}~\text{CAR}~(t~–~x)$$, returns the first element of a *Tuple*.
* $$6F11~-~\text{SECOND}~\text{or}~\text{CDR}~(t~–~y)$$, returns the second element of a *Tuple*.
* $$6F12~-~\text{THIRD}~(t~–~z)$$, returns the third element of a *Tuple*.
* $$6F2n~-~\text{UNTUPLE}~n~(t~–~x_1~.~.~.~x_n)$$, unpacks a *Tuple* $$t = (x_1, . . . ,x_n)$$ of length equal to $$0 ≤ n ≤ 15$$. If $$t$$ is not a *Tuple*, or if $$|t| \neq n$$, a type check exception is *thrown*.
* $$6F21~-~\text{UNSINGLE}~(t~–~x)$$, unpacks a singleton $$t = (x)$$.
* $$6F22~-~\text{UNPAIR or UNCONS}~(t~–~x~y)$$, unpacks a pair $$t = (x, y)$$.
* $$6F23~-~\text{UNTRIPLE}~(t~–~x~y~z)$$, unpacks a triple $$t = (x, y, z)$$.
* $$6F3k~-~\text{UNPACKFIRST}~k~(t~–~x_1~.~.~.~x_k)$$, unpacks the first $$0 \leq k \leq 15$$ elements of a Tuple $$t$$. If $$|t| < k$$, throws a type check exception.
* $$6F30~-~\text{CHKTUPLE}~(t~–~)$$, checks whether $$t$$ is a *Tuple*.
* $$6F4n~-~\text{EXPLODE}~n~(t~–~x_1~.~.~.~x_m~m)$$, unpacks a *Tuple* $$t = (x_1, . . . , x_m)$$ and returns its length $$m$$, but only if $$m \leq n \leq 15$$. Otherwise throws a type check exception.
* $$6F5k~-~\text{SETINDEX}~k~(t~x~–~t')$$, computes *Tuple* $$t'$$ that differs from $$t$$ only at position $$t'[k+1]$$, which is set to $$x$$. In other words, $$|t'| = |t|$$, $$t'[i] = t[i]$$ for $$i \neq k + 1$$, and $$t'[k+1] = x$$, for given $$0 \leq k \leq 15$$. If $$k \geq |t|$$, throws a range check exception.
$$6F50~-~\text{SETFIRST}~(t~x~–~t')$$, sets the first component of *Tuple* $$t$$ to $$x$$ and returns the resulting *Tuple* $$t'$$.
* $$6F51~-~\text{SETSECOND}~(t~x~–~t')$$, sets the second component of *Tuple* $$t$$ to $$x$$ and returns the resulting *Tuple* $$t'$$.
* $$6F52~-~\text{SETTHIRD}~(t~x~–~t')$$, sets the third component of *Tuple* $$t$$ to $$x$$ and returns the resulting *Tuple* $$t'$$.
* $$6F6k~-~\text{INDEXQ}~k~(t~–~x)$$, returns the $$k$$-th element of a *Tuple* $$t$$, where $$0 \leq k \leq 15$$. In other words, returns $$t[k+1]$$ if $$t = (x_1, . . . , x_n)$$. If $$k \geq n$$, or if $$t$$ is *Null*, returns a *Null* instead of $$x$$.
* $$6F7k~-~\text{SETINDEXQ}~k~(t~x~–~t')$$, sets the $$k$$-th component of *Tuple* $$t$$ to $$x$$, where $$0 \leq k < 16$$, and returns the resulting *Tuple* $$t'$$. If $$|t| \leq k$$, first extends the original *Tuple* to length $$k+1$$ by setting all new components to *Null*. If the original value of $$t$$ is *Null*, treats it as an empty *Tuple*. If $$t$$ is not *Null* or *Tuple*, throws an exception. If $$x$$ is *Null* and either $$|t| \leq k$$ or $$t$$ is *Null*, then always returns $$t' = t$$ (and does not consume tuple creation gas).
* $$6F80~-~\text{TUPLEVAR}~(x_1~\dots~x_n~n~-~t)$$, creates a new *Tuple* $$t$$ of length $$n$$ similarly to $$TUPLE$$, but with $$0 \leq n \leq 255$$ taken from the stack.
* $$6F81~-~\text{INDEXVAR}~(t~k~-~x)$$, similar to $$INDEX~k$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* $$6F82~-~\text{UNTUPLEVAR}~(t~n~-~x_1~\dots~x_n)$$, similar to $$UNTUPLE~n$$, but with $$0 \leq n \leq 255$$ taken from the stack.
* $$6F83~-~\text{UNPACKFIRSTVAR}~(t~n~-~x_1~\dots~x_n)$$, similar to $$UNPACKFIRST~n$$, but with $$0 \leq n \leq 255$$ taken from the stack.
* $$6F84~-~\text{EXPLODEVAR}~(t~n~-~x_1~\dots~x_m~m)$$, similar to $$EXPLODE~n$$, but with $$0 \leq n \leq 255$$ taken from the stack.
* $$6F85~-~\text{SETINDEXVAR}~(t~x~k~-~t')$$, similar to $$SETINDEX~k$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* $$6F86~-~\text{INDEXVARQ}~(t~k~-~x)$$, similar to $$INDEXQ~n$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* $$6F87~-~\text{SETINDEXVARQ}~(t~x~k~-~t')$$, similar to $$SETINDEXQ~k$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* $$6F88~-~\text{TLEN}~(t~-~n)$$, returns the length of a *Tuple*.
* $$6F89~-~\text{QTLEN}~(t~-~n~\text{ or }~-1)$$, similar to $$TLEN$$, but returns $$-1$$ if $$t$$ is not a *Tuple*.
* $$6F8A~-~\text{ISTUPLE}~(t~-~?)$$, returns $$-1$$ or $$0$$ depending on whether $$t$$ is a *Tuple*.
* $$6F8B~-~\text{LAST}~(t~-~x)$$, returns the last element $$t|t|$$ of a non-empty *Tuple* $$t$$.
* $$6F8C~-~\text{TPUSH}~\text{or}~\text{COMMA}~(t~x~-~t')$$, appends a value $$x$$ to a *Tuple* $$t = (x_1, \dots , x_n)$$, but only if the resulting *Tuple* $$t' = (x_1, \dots , x_n, x)$$ is of length at most 255. Otherwise throws a type check exception.
* $$6F8D~-~\text{TPOP}~(t~-~t'~x)$$, detaches the last element $$x = x_n$$ from a non-empty *Tuple* $$t = (x_1, \dots, x_n)$$, and returns both the resulting *Tuple* $$t' = (x_1, \dots, x_{n-1})$$ and the original last element $$x$$.
* $$6FA0~-~\text{NULLSWAPIF}~(x~-~x~\text{or}~\bot~x)$$, pushes a *Null* under the topmost *Integer* $$x$$, but only if $$x~\neq~0$$.
* $$6FA1~-~\text{NULLSWAPIFNOT}~(x~-~x~\text{or}~\bot~x)$$, pushes a *Null* under the topmost *Integer* $$x$$, but only if $$x~=~0$$. May be used for stack alignment after quiet primitives such as $$PLDUXQ$$.
* $$6FA2~-~\text{NULLROTRIF}~(x~y~-~x~y~\text{or}~\bot~x~y)$$, pushes a *Null* under the second stack entry from the top, but only if the topmost *Integer* $$y$$ is non-zero.
* $$6FA3~-~\text{NULLROTRIFNOT}~(x~y~-~x~y~\text{or}~\bot~x~y)$$, pushes a *Null* under the second stack entry from the top, but only if the topmost *Integer* $$y$$ is zero. May be used for stack alignment after quiet primitives such as $$LDUXQ$$.
* $$6FA4~-~\text{NULLSWAPIF2}~(x~-~x~\text{or}~\bot~\bot~x)$$, pushes two *Nulls* under the topmost *Integer* $$x$$, but only if $$x~\neq~0$$. Equivalent to $$NULLSWAPIF;~NULLSWAPIF$$.
* $$6FA5~-~\text{NULLSWAPIFNOT2}~(x~-~x~\text{or}~\bot~\bot~x)$$, pushes two *Nulls* under the topmost *Integer* $$x$$, but only if $$x~=~0$$. Equivalent to $$NULLSWAPIFNOT;~NULLSWAPIFNOT$$.
* $$6FA6~-~\text{NULLROTRIF2}~(x~y~-~x~y~\text{or}~\bot~\bot~x~y)$$, pushes two *Nulls* under the second stack entry from the top, but only if the topmost *Integer* $$y$$ is non-zero. Equivalent to $$NULLROTRIF;~NULLROTRIF$$.
* $$6FA7~-~\text{NULLROTRIFNOT2}~(x~y~-~x~y~\text{or}~\bot~\bot~x~y)$$, pushes two *Nulls* under the second stack entry from the top, but only if the topmost *Integer* $$y$$ is zero. Equivalent to $$NULLROTRIFNOT;~NULLROTRIFNOT$$.
* $$6FB_{ij}~-~\text{INDEX2}~i,j~(t~-~x)$$, recovers $$x~=~(t_{i+1})_{j+1}$$ for $$0~\leq~i,~j~\leq~3$$. Equivalent to $$INDEX~i;~INDEX~j$$.
* $$6FB4~-~\text{CADR}~(t~-~x)$$, recovers $$x~=~(t_2)_1$$.
* $$6FB5~-~\text{CDDR}~(t~-~x)$$, recovers $$x=(t_2)_2$$.
* $$6FE_{ijk}~-~\text{INDEX3}~i,j,k~(t~-~x)$$, recovers $$x=(t_{i+1})_{j+1,k+1}$$ for $$0\leq~i,~j,~k\leq~3$$. Equivalent to $$INDEX2~i,j;~INDEX~k$$.
* $$6FD4~-~\text{CADDR}~(t~-~x)$$, recovers $$x=(t_2)_{2_1}$$.
* $$6FD5~-~\text{CDDDR}~(t~-~x)$$, recovers $$x=(t_2)_{2_2}$$.
* $$7i$$ — $$\text{PUSHINT}~x$$ with $$−5 \leq x \leq 10$$, pushes *integer* $$x$$ into the stack; here $$i$$ equals four lower-order bits of $$x$$ (i.e., $$i = x \mod 16$$).
* $$70$$ — $$\text{ZERO}$$, $$\text{FALSE}$$, or $$\text{PUSHINT 0}$$, pushes a zero.
* $$71$$ — $$\text{ONE}$$ or $$\text{PUSHINT 1}$$.
* $$72$$ — $$\text{TWO}$$ or $$\text{PUSHINT 2}$$.
* $$7A$$ — $$\text{TEN}$$ or $$\text{PUSHINT 10}$$.
* $$7F$$ — $$\text{TRUE}$$ or $$\text{PUSHINT -1}$$.
* $$80xx$$ — $$\text{PUSHINT}~xx$$ with $$-128 \leq xx \leq 127$$.
* $$81xxxx$$ — $$\text{PUSHINT}~xxxx$$ with $$−2^{15} \leq xxxx < 2^{15}$$, a signed 16-bit big-endian integer.
* $$81FC18$$ — $$\text{PUSHINT −1000}$$.
* $$82lxxx$$ — $$\text{PUSHINT}~xxx$$, where 5-bit $$0 \leq l \leq 30$$ determines the length $$n = 8l + 19$$ of signed big-endian integer xxx. The total length of this instruction is $$l + 4$$ bytes or $$n + 13 = 8l + 32$$ bits.
* $$821005F5E100$$ — $$\text{PUSHINT 108}$$.
* $$83xx$$ — $$\text{PUSHPOW2}~(xx + 1)$$, (quietly) pushes $$2^{xx+1}$$ for $$0 \leq xx \leq 255$$.
* $$83FF$$ — $$\text{PUSHNAN}$$, pushes $$\text{NaN}$$.
* $$84xx$$ — $$\text{PUSHPOW2DEC}~(xx + 1)$$, pushes $$2^{xx+1} − 1$$ for $$0 \leq xx \leq 255$$.
* $$85xx$$ — $$\text{PUSHNEGPOW2}~(xx + 1)$$, pushes $$−2^{xx+1}$$ for $$0 \leq xx \leq 255$$.
* $$86$$, $$87$$ — reserved for *integer* constants.


## A.4 Constant, or literal primitives

The following primitives push into the stack one literal (or unnamed constant) of some type and range, stored as a part (an immediate argument) of the instruction. Therefore, if the immediate argument is absent or too short, an "invalid or too short opcode" exception (code 6) is thrown.

### A.4.1. Integer and boolean constants.

* $$7 i$$ - PUSHINT $$x$$ with $$-5 \leq x \leq 10$$, pushes integer $$x$$ into the stack; here $$i$$ equals four lower-order bits of $$x$$ (i.e., $$i=x \bmod 16$$ ).
* 70 - ZERO, FALSE, or PUSHINT 0, pushes a zero.
* 71 - ONE or PUSHINT 1.
* 72 - TWO or PUSHINT 2.
* 7A - TEN or PUSHINT 10.
* $$7 \mathrm{~F}$$ - TRUE or PUSHINT -1.
* $$80 x x-$$ PUSHINT $$x x$$ with $$-128 \leq x x \leq 127$$.
* $$81 x x x x-$$ PUSHINT $$x x x x$$ with $$-2^{15} \leq x x x x<2^{15}$$ a signed 16-bit big-endian integer.
* 81FC18 - PUSHINT - 1000.
* 82lxxx-PUSHINT $$x x x$$, where 5 -bit $$0 \leq l \leq 30$$ determines the length $$n=8 l+19$$ of signed big-endian integer $$x x x$$. The total length of this instruction is $$l+4$$ bytes or $$n+13=8 l+32$$ bits.
* 821005F5E100 - PUSHINT $$10^{8}$$. - $$83 x x-$$ PUSHPOW2 $$x x+1$$, (quietly) pushes $$2^{x x+1}$$ for $$0 \leq x x \leq 255$$
* 83FF - PUSHNAN, pushes a NaN.
* $$84 x x$$ - PUSHPOW2DEC $$x x+1$$, pushes $$2^{x x+1}-1$$ for $$0 \leq x x \leq 255$$.
* $$85 x x$$ - PUSHNEGPOW2 $$x x+1$$, pushes $$-2^{x x+1}$$ for $$0 \leq x x \leq 255$$.
* 86,87 - reserved for integer constants.

### A.4.2. Constant slices, continuations, cells, and references.

Most of the instructions listed below push literal slices, continuations, cells, and cell references, stored as immediate arguments to the instruction. Therefore, if the immediate argument is absent or too short, an "invalid or too short opcode" exception (code 6) is thrown.

* $$88$$ — $$\text{PUSHREF}$$, pushes the first reference of cc.code into the stack as
a *Cell* (and removes this reference from the current continuation).
* $$89$$ — $$\text{PUSHREFSLICE}$$, similar to $$\text{PUSHREF}$$, but converts the cell into a
*Slice*.
* $$8A$$ — $$\text{PUSHREFCONT}$$, similar to $$\text{PUSHREFSLICE}$$, but makes a simple ordinary *Continuation* out of the cell.
* $$8Bxsss$$ — $$\text{PUSHSLICE}~sss$$, pushes the (prefix) subslice of cc.code consisting of its first $$8x + 4$$ bits and no references (i.e., essentially a bitstring), where $$0 \leq x \leq 15$$. A completion tag is assumed, meaning that
all trailing zeroes and the last binary one (if present) are removed from
this bitstring. If the original bitstring consists only of zeroes, an empty
slice will be pushed.
* $$8B08$$ — $$\text{PUSHSLICE}~x8_$$, pushes an empty slice (bitstring ‘’).
* $$8B04$$ — $$\text{PUSHSLICE}~x4_$$, pushes bitstring $$‘0’$$.
* $$8B0C$$ — $$\text{PUSHSLICE}~xC_$$, pushes bitstring $$‘1’$$.
* $$8Crxxssss$$ — $$\text{PUSHSLICE}~ssss$$, pushes the (prefix) subslice of cc.code
consisting of its first $$1 \leq r + 1 \leq 4$$ references and up to first $$8xx + 1$$
bits of data, with $$0 \leq xx \leq 31$$. A completion tag is also assumed.
* $$8C01$$ is equivalent to $$\text{PUSHREFSLICE}$$.
* $$8Drxxsssss$$ — $$\text{PUSHSLICE}~sssss$$, pushes the subslice of cc.code consisting of $$0 \leq r \leq 4$$ references and up to $$8xx + 6$$ bits of data, with
$$0 \leq xx \leq 127$$. A completion tag is assumed.
* $$8DE_$$ — unused (reserved).
* $$8F_rxxcccc$$ — $$\text{PUSHCONT}~cccc$$, where cccc is the simple ordinary continuation made from the first $$0 \leq r \leq 3$$ references and the first
$$0 \leq xx \leq 127$$ bytes of cc.code.
* $$9xccc$$ — $$\text{PUSHCONT}~ccc$$, pushes an x-byte continuation for $$0 \leq x \leq 15$$.


## A.5 Arithmetic primitives

### A.5.1. Addition, subtraction, multiplication.

* $$A0$$ — $$\text{ADD}~(x~y~\to~x + y)$$, adds together two integers.
* $$A1$$ — $$\text{SUB}~(x~y~\to~x - y)$$.
* $$A2$$ — $$\text{SUBR}~(x~y~\to~y - x)$$, equivalent to $$\text{SWAP}; \text{SUB}$$.
* $$A3$$ — $$\text{NEGATE}~(x~\to~−x)$$, equivalent to $$\text{MULCONST}~−1$$ or to $$\text{ZERO}; \text{SUBR}$$. Notice that it triggers an integer overflow exception if $$ x = -2^{256} $$.
* $$A4$$ — $$\text{INC}~(x~\to~x + 1)$$, equivalent to $$\text{ADDCONST}~1$$.
* $$A5$$ — $$\text{DEC}~(x~\to~x - 1)$$, equivalent to $$\text{ADDCONST}~−1$$.
* $$A6cc$$ — $$\text{ADDCONST}~cc~(x~\to~x + cc)$$, $$−128 \leq cc \leq 127$$.
* $$A7cc$$ — $$\text{MULCONST}~cc~(x~\to~x \cdot cc)$$, $$−128 \leq cc \leq 127$$.
* $$A8$$ — $$\text{MUL}~(x~y~\to~xy)$$.

### A.5.2. Division.

The general encoding of a $$\text{DIV}$$, $$\text{DIVMOD}$$, or $$\text{MOD}$$ operation is $$A9mscdf$$, with
an optional pre-multiplication and an optional replacement of the division or
multiplication by a shift. Variable one- or two-bit fields $$m$$, $$s$$, $$c$$, $$d$$, and $$f$$ are
as follows:

* $$0~\leq~m~\leq~1$$ — Indicates whether there is pre-multiplication ($$\text{MULDIV}$$ operation and its variants), possibly replaced by a left shift.
* $$0~\leq~s~\leq~2$$ — Indicates whether either the multiplication or the division have been replaced by shifts: $$s~=~0$$—no replacement, $$s~=~1$$—division replaced by a right shift, $$s~=~2$$—multiplication replaced by a left shift (possible only for $$m~=~1$$).
* $$0~\leq~c~\leq~1$$ — Indicates whether there is a constant one-byte argument $$tt$$ for the shift operator (if $$s~\neq~0$$). For $$s~=~0$$, $$c~=~0$$. If $$c~=~1$$, then $$0~\leq~tt~\leq~255$$, and the shift is performed by $$tt~+~1$$ bits. If $$s~\neq~0$$ and $$c~=~0$$, then the shift amount is provided to the instruction as a top-of-stack $$\text{Integer}$$ in range $$0~\dots~256$$.
* $$1~\leq~d~\leq~3$$ — Indicates which results of division are required: 1—only the quotient, 2—only the remainder, 3—both.
* $$0~\leq~f~\leq~2$$ — Rounding mode: 0—floor, 1—nearest integer, 2—ceiling [$$\mathbf{1.5.6}$$](README.md/#1.5.6.-division-and-rounding.).

Examples:

* $$A904$$ — $$\text{DIV}~(x~y~~q~:=~b\frac{x}{y}c)$$.
* $$A905$$ — $$\text{DIVR}~(x~y~~q_0~:=~b\frac{x}{y}~+~\frac{1}{2}c)$$.
* $$A906$$ — $$\text{DIVC}~(x~y~~q_{00}~:=~d\frac{x}{y}e)$$.
* $$A908$$ — $$\text{MOD}~(x~y~~r)$$, where $$q~:=~b\frac{x}{y}c$$, $$r~:=~x~\text{mod}~y~:=~x~−~yq$$.
* $$A90C$$ — $$\text{DIVMOD}~(x~y~~q~r)$$, where $$q~:=~b\frac{x}{y}c$$, $$r~:=~x~−~yq$$.
* $$A90D$$ — $$\text{DIVMODR}~(x~y~~q_0~r_0)$$, where $$q_0~:=~b\frac{x}{y}~+~\frac{1}{2}c$$, $$r_0~:=~x~−~yq_0$$.
* $$A90E$$ — $$\text{DIVMODC}~(x~y~~q_{00}~r_{00})$$, where $$q_{00}~:=~d\frac{x}{y}e$$, $$r_{00}~:=~x~−~yq_{00}$$.
* $$A924$$ — same as $$\text{RSHIFT}$$: $$(x~y~~b\frac{x}{y}c)$$ for $$0~\leq~y~\leq~256$$.
* $$A934tt$$ — same as $$\text{RSHIFT}~tt~+~1$$: $$(x~~b\frac{x}{y}c)$$.
* $$A938tt$$ — $$\text{MODPOW2}~tt~+~1$$: $$(x~~x~\text{mod}~2^{tt+1})$$.
* $$A985$$ — $$\text{MULDIVR}~(x~y~z~~q_0)$$, where $$q_0~=~b\frac{xy}{z}~+~\frac{1}{2}c$$.
* $$A98C$$ — $$\text{MULDIVMOD}~(x~y~z~~q~r)$$, where $$q~:=~b\frac{x~\cdot~y}{z}c$$, $$r~:=~x~\cdot~y~\text{mod}~z$$ (same as $$*/\text{MOD}$$ in Forth).
* $$A9A4$$ — $$\text{MULRSHIFT}~(x~y~z~~bxy~\cdot~2^{-z}c)$$ for $$0~\leq~z~\leq~256$$.
* $$A9A5$$ — $$\text{MULRSHIFTR}~(x~y~z~~bxy~\cdot~2^{-z}~+~\frac{1}{2}c)$$ for $$0~\leq~z~\leq~256$$.
* $$A9B4tt$$ — $$\text{MULRSHIFT}~tt~+~1~(x~y~~bxy~\cdot~2^{-tt-1}c)$$.
* $$A9B5tt$$ — $$\text{MULRSHIFTR}~tt~+~1~(x~y~~bxy~\cdot~2^{-tt-1}~+~\frac{1}{2}c)$$.
* $$A9C4$$ — $$\text{LSHIFTDIV}~(x~y~z~~b2^zx/yc)$$ for $$0~\leq~z~\leq~256$$.
* $$A9C5$$ — $$\text{LSHIFTDIVR}~(x~y~z~~b2^zx/y~+~\frac{1}{2}c)$$ for $$0~\leq~z~\leq~256$$.
* $$A9D4tt$$ — $$\text{LSHIFTDIV}~tt~+~1~(x~y~~b2^{tt+1}x/yc)$$.
* $$A9D5tt$$ — $$\text{LSHIFTDIVR}~tt~+~1~(x~y~~b2^{tt+1}x/y~+~\frac{1}{2}c)$$.

The most useful of these operations are $$\text{DIV}$$, $$\text{DIVMOD}$$, $$\text{MOD}$$, $$\text{DIVR}$$, $$\text{DIVC}$$, 
$$\text{MODPOW2}~t$$, and $$\text{RSHIFTR}~t$$ (for integer arithmetic); and $$\text{MULDIVMOD}$$, $$\text{MULDIV}$$, 
$$\text{MULDIVR}$$, $$\text{LSHIFTDIVR}~t$$, and $$\text{MULRSHIFTR}~t$$ (for fixed-point arithmetic).


### A.5.3. Shifts, logical operations.

* $$\text{AAcc}$$ — $$\text{LSHIFT}~cc + 1$$ ($$x~~x \cdot 2^{cc+1}$$), $$0 \leq cc \leq 255$$.
* $$\text{AA00}$$ — $$\text{LSHIFT}~1$$, equivalent to $$\text{MULCONST}~2$$ or to Forth’s $$2^*$$.
* $$\text{ABcc}$$ — $$\text{RSHIFT}~cc + 1$$ ($$x~~\left\lfloor x \cdot 2^{-cc-1} \right\rfloor$$), $$0 \leq cc \leq 255$$.
* $$\text{AC}$$ — $$\text{LSHIFT}$$ ($$x~y~~x \cdot 2^y$$), $$0 \leq y \leq 1023$$.
* $$\text{AD}$$ — $$\text{RSHIFT}$$ ($$x~y~~\left\lfloor x \cdot 2^{-y} \right\rfloor$$), $$0 \leq y \leq 1023$$.
* $$\text{AE}$$ — $$\text{POW2}$$ ($$y~~2^y$$), $$0 \leq y \leq 1023$$, equivalent to $$\text{ONE}; \text{SWAP}; \text{LSHIFT}$$.
* $$\text{AF}$$ — reserved.
* $$\text{B0}$$ — $$\text{AND}$$ ($$x~y~~x\&y$$), bitwise “and” of two signed integers $$x$$ and $$y$$, sign-extended to infinity.
* $$\text{B1}$$ — $$\text{OR}$$ ($$x~y~~x \vee y$$), bitwise “or” of two integers.
* $$\text{B2}$$ — $$\text{XOR}$$ ($$x~y~~x \oplus y$$), bitwise “xor” of two integers.
* $$\text{B3}$$ — $$\text{NOT}$$ ($$x~~x \oplus -1 = -1 - x$$), bitwise “not” of an integer.
* $$\text{B4cc}$$ — $$\text{FITS}~cc + 1$$ ($$x~~x$$), checks whether $$x$$ is a $$cc + 1$$-bit signed integer for $$0 \leq cc \leq 255$$ (i.e., whether $$-2^{cc} \leq x < 2^{cc}$$). If not, either triggers an integer overflow exception or replaces $$x$$ with a NaN (quiet version).
* $$\text{B400}$$ — $$\text{FITS}~1$$ or $$\text{CHKBOOL}$$ ($$x~~x$$), checks whether $$x$$ is a “boolean value” (i.e., either 0 or -1).
* $$\text{B5cc}$$ — $$\text{UFITS}~cc + 1$$ ($$x~~x$$), checks whether $$x$$ is a $$cc + 1$$-bit unsigned integer for $$0 \leq cc \leq 255$$ (i.e., whether $$0 \leq x < 2^{cc+1}$$).
* $$\text{B500}$$ — $$\text{UFITS}~1$$ or $$\text{CHKBIT}$$, checks whether $$x$$ is a binary digit (i.e., zero or one).
* $$\text{B600}$$ — $$\text{FITSX}$$ ($$x~c~~x$$), checks whether $$x$$ is a $$c$$-bit signed integer for $$0 \leq c \leq 1023$$.
* $$\text{B601}$$ — $$\text{UFITSX}$$ ($$x~c~~x$$), checks whether $$x$$ is a $$c$$-bit unsigned integer for $$0 \leq c \leq 1023$$.
* $$\text{B602}$$ — $$\text{BITSIZE}$$ ($$x~~c$$), computes smallest $$c \geq 0$$ such that $$x$$ fits into a $$c$$-bit signed integer ($$-2^{c-1} \leq c < 2^{c-1}$$).
* $$\text{B603}$$ — $$\text{UBITSIZE}$$ ($$x~~c$$), computes smallest $$c \geq 0$$ such that $$x$$ fits into a $$c$$-bit unsigned integer ($$0 \leq x < 2^c$$), or throws a range check exception.
* $$\text{B608}$$ — $$\text{MIN}$$ ($$x~y~~x~\text{or}~y$$), computes the minimum of two integers $$x$$ and $$y$$.
* $$\text{B609}$$ — $$\text{MAX}$$ ($$x~y~~x~\text{or}~y$$), computes the maximum of two integers $$x$$ and $$y$$.
* $$\text{B60A}$$ — $$\text{MINMAX}$$ or $$\text{INTSORT2}$$ ($$x~y~~x~y~\text{or}~y~x$$), sorts two integers. Quiet version of this operation returns two NaNs if any of the arguments are NaNs.
* $$\text{B60B}$$ — $$\text{ABS}$$ ($$x~~|x|$$), computes the absolute value of an integer $$x$$.


## A.6 Comparison primitives

### A.6.1. Integer comparison.

All integer comparison primitives return integer -1 ("true") or 0 ("false") to indicate the result of the comparison. We do not define their "boolean circuit" counterparts, which would transfer control to $$c 0$$ or $$c 1$$ depending on the result of the comparison. If needed, such instructions can be simulated with the aid of RETBOOL.

Quiet versions of integer comparison primitives are also available, encoded with the aid of the QUIET prefix (B7). If any of the integers being compared are NaNs, the result of a quiet comparison will also be a $$\mathrm{NaN}$$ ("undefined"), instead of a -1 ("yes") or 0 ("no"), thus effectively supporting ternary logic.

* $$\text{B8}$$ — $$\text{SGN}$$ ($$x~~\text{sgn}(x)$$), computes the sign of an integer $$x$$: −1 if $$x < 0$$, 0 if $$x = 0$$, 1 if $$x > 0$$.
* $$\text{B9}$$ — $$\text{LESS}$$ ($$x~y~~x < y$$), returns −1 if $$x < y$$, 0 otherwise.
* $$\text{BA}$$ — $$\text{EQUAL}$$ ($$x~y~~x = y$$), returns −1 if $$x = y$$, 0 otherwise.
* $$\text{BB}$$ — $$\text{LEQ}$$ ($$x~y~~x \leq y$$).
* $$\text{BC}$$ — $$\text{GREATER}$$ ($$x~y~~x > y$$).
* $$\text{BD}$$ — $$\text{NEQ}$$ ($$x~y~~x \neq y$$), equivalent to EQUAL; NOT.
* $$\text{BE}$$ — $$\text{GEQ}$$ ($$x~y~~x \geq y$$), equivalent to LESS; NOT.
* $$\text{BF}$$ — $$\text{CMP}$$ ($$x~y~~\text{sgn}(x - y)$$), computes the sign of $$x - y$$: −1 if $$x < y$$, 0 if $$x = y$$, 1 if $$x > y$$. No integer overflow can occur here unless $$x$$ or $$y$$ is a NaN.
* $$\text{C0yy}$$ — $$\text{EQINT}~yy$$ ($$x~~x = yy$$) for $$-2^7 \leq yy < 2^7$$.
* $$\text{C000}$$ — $$\text{ISZERO}$$, checks whether an integer is zero. Corresponds to Forth’s 0=.
* $$\text{C1yy}$$ — $$\text{LESSINT}~yy$$ ($$x~~x < yy$$) for $$-2^7 \leq yy < 2^7$$.
* $$\text{C100}$$ — $$\text{ISNEG}$$, checks whether an integer is negative. Corresponds to Forth’s 0<.
* $$\text{C101}$$ — $$\text{ISNPOS}$$, checks whether an integer is non-positive.
* $$\text{C2yy}$$ — $$\text{GTINT}~yy$$ ($$x~~x > yy$$) for $$-2^7 \leq yy < 2^7$$.
* $$\text{C200}$$ — $$\text{ISPOS}$$, checks whether an integer is positive. Corresponds to Forth’s 0>.
* $$\text{C2FF}$$ — $$\text{ISNNEG}$$, checks whether an integer is non-negative.
* $$\text{C3yy}$$ — $$\text{NEQINT}~yy$$ ($$x~~x \neq yy$$) for $$-2^7 \leq yy < 2^7$$.
* $$\text{C4}$$ — $$\text{ISNAN}$$ ($$x~~x = \text{NaN}$$), checks whether $$x$$ is a NaN.
* $$\text{C5}$$ — $$\text{CHKNAN}$$ ($$x~~x$$), throws an arithmetic overflow exception if $$x$$ is a NaN.
* $$\text{C6}$$ — reserved for integer comparison.

### A.6.2. Other comparison.

Most of these "other comparison" primitives actually compare the data portions of Slices as bitstrings.

* $$\text{C700}$$ — $$\text{SEMPTY}$$ ($$s~~s = \emptyset$$), checks whether a Slice $$s$$ is empty (i.e., contains no bits of data and no cell references).
* $$\text{C701}$$ — $$\text{SDEMPTY}$$ ($$s~~s \approx \emptyset$$), checks whether Slice $$s$$ has no bits of data.
* $$\text{C702}$$ — $$\text{SREMPTY}$$ ($$s~~r(s) = 0$$), checks whether Slice $$s$$ has no references.
* $$\text{C703}$$ — $$\text{SDFIRST}$$ ($$s~~s_0 = 1$$), checks whether the first bit of Slice $$s$$ is a one.
* $$\text{C704}$$ — $$\text{SDLEXCMP}$$ ($$s~s_0~~c$$), compares the data of $$s$$ lexicographically with the data of $$s_0$$, returning −1, 0, or 1 depending on the result.
* $$\text{C705}$$ — $$\text{SDEQ}$$ ($$s~s_0~~s \approx s_0$$), checks whether the data parts of $$s$$ and $$s_0$$ coincide, equivalent to $$\text{SDLEXCMP}$$; $$\text{ISZERO}$$.
* $$\text{C708}$$ — $$\text{SDPFX}$$ ($$s~s_0~~?$$), checks whether $$s$$ is a prefix of $$s_0$$.
* $$\text{C709}$$ — $$\text{SDPFXREV}$$ ($$s~s_0~~?$$), checks whether $$s_0$$ is a prefix of $$s$$, equivalent to $$\text{SWAP}$$; $$\text{SDPFX}$$.
* $$\text{C70A}$$ — $$\text{SDPPFX}$$ ($$s~s_0~~?$$), checks whether $$s$$ is a proper prefix of $$s_0$$ (i.e., a prefix distinct from $$s_0$$).
* $$\text{C70B}$$ — $$\text{SDPPFXREV}$$ ($$s~s_0~~?$$), checks whether $$s_0$$ is a proper prefix of $$s$$.
* $$\text{C70C}$$ — $$\text{SDSFX}$$ ($$s~s_0~~?$$), checks whether $$s$$ is a suffix of $$s_0$$.
* $$\text{C70D}$$ — $$\text{SDSFXREV}$$ ($$s~s_0~~?$$), checks whether $$s_0$$ is a suffix of $$s$$.
* $$\text{C70E}$$ — $$\text{SDPSFX}$$ ($$s~s_0~~?$$), checks whether $$s$$ is a proper suffix of $$s_0$$.
* $$\text{C70F}$$ — $$\text{SDPSFXREV}$$ ($$s~s_0~~?$$), checks whether $$s_0$$ is a proper suffix of $$s$$.
* $$\text{C710}$$ — $$\text{SDCNTLEAD0}$$ ($$s~~n$$), returns the number of leading zeroes in $$s$$.
* $$\text{C711}$$ — $$\text{SDCNTLEAD1}$$ ($$s~~n$$), returns the number of leading ones in $$s$$.
* $$\text{C712}$$ — $$\text{SDCNTTRAIL0}$$ ($$s~~n$$), returns the number of trailing zeroes in $$s$$.
* $$\text{C713}$$ — $$\text{SDCNTTRAIL1}$$ ($$s~~n$$), returns the number of trailing ones in $$s$$.


## A.7. Cell primitives

The cell primitives are mostly either *cell serialization primitives*, which work
with *Builders*, or *cell deserialization primitives*, which work with *Slices*.

### A.7.1. Cell serialization primitives.

All these primitives first check whether there is enough space in the Builder, and only then check the range of the value being serialized.

* $$\text{C8}$$ — $$\text{NEWC}$$ ($$\rightarrow~b$$), creates a new empty Builder.
* $$\text{C9}$$ — $$\text{ENDC}$$ ($$b~\rightarrow~c$$), converts a Builder into an ordinary Cell.
* $$\text{CA}_{\text{cc}}$$ — $$\text{STI}~\text{cc} + 1$$ ($$x~b~\rightarrow~b_0$$), stores a signed cc + 1-bit integer $$x$$ into Builder $$b$$ for \(0 \leq \text{cc} \leq 255$$, throws a range check exception if $$x$$ does not fit into cc + 1 bits.
* $$\text{CB}_{\text{cc}}$$ — $$\text{STU}~\text{cc} + 1$$ ($$x~b~\rightarrow~b_0$$), stores an unsigned cc + 1-bit integer $$x$$ into Builder $$b$$. In all other respects it is similar to STI.
* $$\text{CC}$$ — $$\text{STREF}$$ ($$c~b~\rightarrow~b_0$$), stores a reference to Cell $$c$$ into Builder $$b$$.
* $$\text{CD}$$ — $$\text{STBREFR}$$ or $$\text{ENDCST}$$ ($$b~b_{00}~\rightarrow~b$$), equivalent to ENDC; SWAP; STREF.
* $$\text{CE}$$ — $$\text{STSLICE}$$ ($$s~b~\rightarrow~b_0$$), stores Slice $$s$$ into Builder $$b$$.
* $$\text{CF00}$$ — $$\text{STIX}$$ ($$x~b~l~\rightarrow~b_0$$), stores a signed $$l$$-bit integer $$x$$ into $$b$$ for $$0 \leq l \leq 257$$.
* $$\text{CF01}$$ — $$\text{STUX}$$ ($$x~b~l~\rightarrow~b_0$$), stores an unsigned $$l$$-bit integer $$x$$ into $$b$$ for $$0 \leq l \leq 256$$.
* $$\text{CF02}$$ — $$\text{STIXR}$$ ($$b~x~l~\rightarrow~b_0$$), similar to STIX, but with arguments in a different order.
* $$\text{CF03}$$ — $$\text{STUXR}$$ ($$b~x~l~\rightarrow~b_0$$), similar to STUX, but with arguments in a different order.
* $$\text{CF04}$$ — $$\text{STIXQ}$$ ($$x~b~l~\rightarrow~x~b~f \text{ or } b_0~0$$), a quiet version of STIX. Description provided in the list.
* $$\text{CF05}$$ — $$\text{STUXQ}$$ ($$x~b~l~\rightarrow~b_0~f$$).
* $$\text{CF06}$$ — $$\text{STIXRQ}$$ ($$b~x~l~\rightarrow~b~x~f \text{ or } b_0~0$$).
* $$\text{CF07}$$ — $$\text{STUXRQ}$$ ($$b~x~l~\rightarrow~b~x~f \text{ or } b_0~0$$).
* $$\text{CF08}_{\text{cc}}$$ — a longer version of STI $$\text{cc} + 1$$.
* $$\text{CF09}_{\text{cc}}$$ — a longer version of STU $$\text{cc} + 1$$.
* $$\text{CF0A}_{\text{cc}}$$ — $$\text{STIR}~\text{cc} + 1$$ ($$b~x~\rightarrow~b_0$$), equivalent to SWAP; STI $$\text{cc} + 1$$.
* $$\text{CF0B}_{\text{cc}}$$ — $$\text{STUR}~\text{cc} + 1$$ ($$b~x~\rightarrow~b_0$$), equivalent to SWAP; STU $$\text{cc} + 1$$.
* $$\text{CF0C}_{\text{cc}}$$ — $$\text{STIQ}~\text{cc} + 1$$ ($$x~b~\rightarrow~x~b~f \text{ or } b_0~0$$).
* $$\text{CF0D}_{\text{cc}}$$ — $$\text{STUQ}~\text{cc} + 1$$ ($$x~b~\rightarrow~x~b~f \text{ or } b_0~0$$).
* $$\text{CF0E}_{\text{cc}}$$ — $$\text{STIRQ}~\text{cc} + 1$$ ($$b~x~\rightarrow~b~x~f \text{ or } b_0~0$$).
* $$\text{CF0F}_{\text{cc}}$$ — $$\text{STURQ}~\text{cc} + 1$$ ($$b~x~\rightarrow~b~x~f \text{ or } b_0~0$$).
* $$\text{CF10}$$ — a longer version of STREF ($$c~b~\rightarrow~b_0$$).
* $$\text{CF11}$$ — $$\text{STBREF}$$ ($$b_0~b~\rightarrow~b_{00}$$), equivalent to SWAP; STBREFREV.
* $$\text{CF12}$$ — a longer version of STSLICE ($$s~b~\rightarrow~b_0$$).
* $$\text{CF13}$$ — STB ($$b_0~b~\rightarrow~b_{00}$$), appends all data from Builder $$b_0$$ to Builder $$b$$.
* $$\text{CF14}$$ — STREFR ($$b~c~\rightarrow~b_0$$).
* $$\text{CF15}$$ — STBREFR ($$b~b_0~\rightarrow~b_{00}$$), a longer encoding of STBREFR.
* $$\text{CF16}$$ — STSLICER ($$b~s~\rightarrow~b_0$$).
* $$\text{CF17}$$ — STBR ($$b~b_0~\rightarrow~b_{00}$$), concatenates two Builders, equivalent to SWAP; STB.
* $$\text{CF18}$$ — STREFQ ($$c~b~\rightarrow~c~b~{-1} \text{ or } b_0~0$$).
* $$\text{CF19}$$ — STBREFQ ($$b_0~b~\rightarrow~b_0~b~{-1} \text{ or } b_{00}~0$$).
* $$\text{CF1A}$$ — STSLICEQ ($$s~b~\rightarrow~s~b~{-1} \text{ or } b_0~0$$).
* $$\text{CF1B}$$ — STBQ ($$b_0~b~\rightarrow~b_0~b~{-1} \text{ or } b_{00}~0$$).
* $$\text{CF1C}$$ — STREFRQ ($$b~c~\rightarrow~b~c~{-1} \text{ or } b_0~0$$).
* $$\text{CF1D}$$ — STBREFRQ ($$b~b_0~\rightarrow~b~b_0~{-1} \text{ or } b_{00}~0$$).
* $$\text{CF1E}$$ — STSLICERQ ($$b~s~\rightarrow~b~s~{-1} \text{ or } b_{00}~0$$).
* $$\text{CF1F}$$ — STBRQ ($$b~b_0~\rightarrow~b~b_0~{-1} \text{ or } b_{00}~0$$).
* $$\text{CF20}$$ — STREFCONST, equivalent to PUSHREF; STREFR.
* $$\text{CF21}$$ — STREF2CONST, equivalent to STREFCONST; STREFCONST.
* $$\text{CF23}$$ — ENDXC ($$b~x~\rightarrow~c$$), description provided in the list.
* $$\text{CF28}$$ — STILE4 ($$x~b~\rightarrow~b_0$$), stores a little-endian signed 32-bit integer.
* $$\text{CF29}$$ — STULE4 ($$x~b~\rightarrow~b_0$$), stores a little-endian unsigned 32-bit integer.
* $$\text{CF2A}$$ — STILE8 ($$x~b~\rightarrow~b_0$$), stores a little-endian signed 64-bit integer.
* $$\text{CF2B}$$ — STULE8 ($$x~b~\rightarrow~b_0$$), stores a little-endian unsigned 64-bit integer.
* $$\text{CF30}$$ — BDEPTH ($$b~\rightarrow~x$$), description provided in the list.
* $$\text{CF31}$$ — BBITS ($$b~\rightarrow~x$$), returns the number of data bits already stored in Builder $$b$$.
* $$\text{CF32}$$ — BREFS ($$b~\rightarrow~y$$), returns the number of cell references already stored in $$b$$.
* $$\text{CF33}$$ — BBITREFS ($$b~\rightarrow~x~y$$), returns the numbers of both data bits and cell references in $$b$$.
* $$\text{CF35}$$ — BREMBITS ($$b~\rightarrow~x_0$$), returns the number of data bits that can still be stored in $$b$$.
* $$\text{CF36}$$ — BREMREFS ($$b~\rightarrow~y_0$$).
* $$\text{CF37}$$ — BREMBITREFS ($$b~\rightarrow~x_0~y_0$$).
* $$\text{CF38}_{\text{cc}}$$ — BCHKBITS $$\text{cc} + 1$$ ($$b~\rightarrow~$$), checks whether $$\text{cc} + 1$$.
* $$\text{CF39}$$ — $$\text{BCHKBITS}$$ ($$b~x~\rightarrow~$$), checks if $$x$$ bits can be stored into $$b$$, where $$0 \leq x \leq 1023$$. An exception is thrown if there isn't enough space in $$b$$ for $$x$$ bits or if $$x$$ is out of range $$0 \leq x \leq 1023$$.
* $$\text{CF3A}$$ — $$\text{BCHKREFS}$$ ($$b~y~\rightarrow~$$), verifies if $$y$$ references can be stored in $$b$$, where $$0 \leq y \leq 7$$.
* $$\text{CF3B}$$ — $$\text{BCHKBITREFS}$$ ($$b~x~y~\rightarrow~$$), confirms if $$x$$ bits and $$y$$ references can fit into $$b$$, where $$0 \leq x \leq 1023$$ and $$0 \leq y \leq 7$$.
* $$\text{CF3Ccc}$$ — $$\text{BCHKBITSQ}~cc + 1$$ ($$b~\rightarrow~?\$$), checks if $$cc + 1$$ bits can be saved in $$b$$, $$0 \leq cc \leq 255$$.
* $$\text{CF3D}$$ — $$\text{BCHKBITSQ}$$ ($$b~x~\rightarrow~?\$$), evaluates if $$x$$ bits can be stored in $$b$$, $$0 \leq x \leq 1023$$.
* $$\text{CF3E}$$ — $$\text{BCHKREFSQ}$$ ($$b~y~\rightarrow~?\$$), determines if $$y$$ references can be housed in $$b$$, $$0 \leq y \leq 7$$.
* $$\text{CF3F}$$ — $$\text{BCHKBITREFSQ}$$ ($$b~x~y~\rightarrow~?\$$), verifies if $$x$$ bits and $$y$$ references can be kept in $$b$$, where $$0 \leq x \leq 1023$$ and $$0 \leq y \leq 7$$.
* $$\text{CF40}$$ — $$\text{STZEROES}$$ ($$b~n~\rightarrow~b_0\$$), places $$n$$ binary zeroes into Builder $$b$$.
* $$\text{CF41}$$ — $$\text{STONES}$$ ($$b~n~\rightarrow~b_0\$$), incorporates $$n$$ binary ones into Builder $$b$$.
* $$\text{CF42}$$ — $$\text{STSAME}$$ ($$b~n~x~\rightarrow~b_0\$$), saves $$n$$ binary xes ($$0 \leq x \leq 1$$) into Builder $$b$$.
* $$\text{CFC0\_xysss}$$ — $$\text{STSLICECONST}~sss$$ ($$b~\rightarrow~b_0\$$), adds a constant subslice $$sss$$ of $$0 \leq x \leq 3$$ references and up to $$8y + 1$$ data bits where $$0 \leq y \leq 7$$. Completion bit is included.
* $$\text{CF81}$$ — $$\text{STSLICECONST}~'0'$$ or $$\text{STZERO}$$ ($$b~\rightarrow~b_0\$$), inserts a single binary zero.
* $$\text{CF83}$$ — $$\text{STSLICECONST}~'1'$$ or $$\text{STONE}$$ ($$b~\rightarrow~b_0\$$), appends a single binary one.
* $$\text{CFA2}$$ — is equivalent to $$\text{STREFCONST}$$.
* $$\text{CFA3}$$ — is nearly the same as $$\text{STSLICECONST}~'1'; \text{STREFCONST}$$.
* $$\text{CFC2}$$ — corresponds to $$\text{STREF2CONST}$$.
* $$\text{CFE2}$$ — $$\text{STREF3CONST}$$.


### A.7.2. Cell deserialization primitives.

* $$\text{D0}$$ — $$\text{CTOS}$$ ($$c~\rightarrow~s$$), transforms a Cell into a Slice. Note that $$c$$ must be either an ordinary cell or an exotic cell (cf. 3.1.2) which is automatically loaded to produce an ordinary cell $$c_0$$, subsequently converted into a Slice.
* $$\text{D1}$$ — $$\text{ENDS}$$ ($$s~\rightarrow~$$), eliminates a Slice $$s$$ from the stack, and raises an exception if it is not empty.
* $$\text{D2cc}$$ — $$\text{LDI}~cc + 1$$ ($$s~\rightarrow~x~s_0$$), extracts (i.e., parses) a signed $$cc + 1$$-bit integer $$x$$ from Slice $$s$$, and returns the remaining part of $$s$$ as $$s_0$$.
* $$\text{D3cc}$$ — $$\text{LDU}~cc + 1$$ ($$s~\rightarrow~x~s_0$$), retrieves an unsigned $$cc + 1$$-bit integer $$x$$ from Slice $$s$$, and returns the residual portion of $$s$$ as $$s_0$$.
* $$\text{D4}$$ — $$\text{LDREF}$$ ($$s~\rightarrow~c~s_0$$), fetches a cell reference $$c$$ from Slice $$s$$, and provides the remaining part of $$s$$ as $$s_0$$.
* $$\text{D5}$$ — $$\text{LDREFRTOS}$$ ($$s \rightarrow s_0~s_{00}$$), equivalent to executing $$\text{LDREF}$$, followed by $$\text{SWAP}$$, then $$\text{CTOS}$$.
* $$\text{D6cc}$$ — $$\text{LDSLICE}~cc + 1$$ ($$s \rightarrow s_{00}~s_0$$), extracts the next $$cc + 1$$ bits of $$s$$ into a separate Slice $$s_{00}$$.
* $$\text{D700}$$ — $$\text{LDIX}$$ ($$s~l \rightarrow x~s_0$$), loads a signed $$l$$-bit ($$0 \leq l \leq 257$$) integer $$x$$ from Slice $$s$$, returning the remainder of $$s$$ as $$s_0$$.
* $$\text{D701}$$ — $$\text{LDUX}$$ ($$s~l \rightarrow x~s_0$$), loads an unsigned $$l$$-bit integer $$x$$ from the first $$l$$ bits of $$s$$, with $$0 \leq l \leq 256$$.
* $$\text{D702}$$ — $$\text{PLDIX}$$ ($$s~l \rightarrow x$$), preloads a signed $$l$$-bit integer from Slice $$s$$, for $$0 \leq l \leq 257$$.
* $$\text{D703}$$ — $$\text{PLDUX}$$ ($$s~l \rightarrow x$$), preloads an unsigned $$l$$-bit integer from $$s$$, for $$0 \leq l \leq 256$$.
* $$\text{D704}$$ — $$\text{LDIXQ}$$ ($$s~l \rightarrow x~s_0~-1~\text{or}~s~0$$), quiet version of $$\text{LDIX}$$, loads a signed $$l$$-bit integer from $$s$$ similarly to $$\text{LDIX}$$, returning a success flag equal to $$-1$$ on success or $$0$$ on failure (if $$s$$ doesn't have $$l$$ bits) instead of throwing a cell underflow exception.
* $$\text{D705}$$ — $$\text{LDUXQ}$$ ($$s~l \rightarrow x~s_0~-1~\text{or}~s~0$$), quiet version of $$\text{LDUX}$$.
* $$\text{D706}$$ — $$\text{PLDIXQ}$$ ($$s~l \rightarrow x~-1~\text{or}~0$$), quiet version of $$\text{PLDIX}$$.
* $$\text{D707}$$ — $$\text{PLDUXQ}$$ ($$s~l \rightarrow x~-1~\text{or}~0$$), quiet version of $$\text{PLDUX}$$.
* $$\text{D708cc}$$ — $$\text{LDI}~cc + 1$$ ($$s \rightarrow x~s_0$$), a longer encoding for $$\text{LDI}$$.
* $$\text{D709cc}$$ — $$\text{LDU}~cc + 1$$ ($$s \rightarrow x~s_0$$), a longer encoding for $$\text{LDU}$$.
* $$\text{D70Acc}$$ — $$\text{PLDI}~cc+ 1$$ ($$s \rightarrow x$$), preloads a signed $$cc+ 1$$-bit integer from Slice $$s$$.
* $$\text{D70Bcc}$$ — $$\text{PLDU}~cc + 1$$ ($$s \rightarrow x$$), preloads an unsigned $$cc + 1$$-bit integer from $$s$$.
* $$\text{D70Ccc}$$ — $$\text{LDIQ}~cc + 1$$ ($$s \rightarrow x~s_0~-1~\text{or}~s~0$$), a quiet version of $$\text{LDI}$$.
* $$\text{D70Dcc}$$ — $$\text{LDUQ}~cc + 1$$ ($$s \rightarrow x~s_0~-1~\text{or}~s~0$$), a quiet version of $$\text{LDU}$$.
* $$\text{D70Ecc}$$ — $$\text{PLDIQ}~cc + 1$$ ($$s \rightarrow x~-1~\text{or}~0$$), a quiet version of $$\text{PLDI}$$.
* $$\text{D70Fcc}$$ — $$\text{PLDUQ}~cc + 1$$ ($$s \rightarrow x~-1~\text{or}~0$$), a quiet version of $$\text{PLDU}$$.
* $$\text{D714\_c}$$ — $$\text{PLDUZ}~32(c + 1)$$ ($$s \rightarrow s~x$$), preloads the first $$32(c + 1)$$ bits of Slice $$s$$ into an unsigned integer $$x
* $$\text{D718}$$ — $$\text{LDSLICEX}$$ ($$s~l \rightarrow s_{00}~s_{0}$$), loads the first $$0 \leq l \leq 1023$$ bits from Slice $$s$$ into a separate Slice $$s_{00}$$, returning the remainder of $$s$$ as $$s_{0}$$.
* $$\text{D719}$$ — $$\text{PLDSLICEX}$$ ($$s~l \rightarrow s_{00}$$), returns the first $$0 \leq l \leq 1023$$ bits of $$s$$ as $$s_{00}$$.
* $$\text{D71A}$$ — $$\text{LDSLICEXQ}$$ ($$s~l \rightarrow s_{00}~s_{0}~-1~\text{or}~s~0$$), a quiet version of $$\text{LDSLICEX}$$.
* $$\text{D71B}$$ — $$\text{PLDSLICEXQ}$$ ($$s~l \rightarrow s_{0}~-1~\text{or}~0$$), a quiet version of $$\text{LDSLICEXQ}$$.
* $$\text{D71Ccc}$$ — $$\text{LDSLICE}~cc + 1$$ ($$s \rightarrow s_{00}~s_{0}$$), a longer encoding for $$\text{LDSLICE}$$.
* $$\text{D71Dcc}$$ — $$\text{PLDSLICE}~cc + 1$$ ($$s \rightarrow s_{00}$$), returns the first $$0 < cc + 1 \leq 256$$ bits of $$s$$ as $$s_{00}$$.
* $$\text{D71Ecc}$$ — $$\text{LDSLICEQ}~cc + 1$$ ($$s \rightarrow s_{00}~s_{0}~-1~\text{or}~s~0$$), a quiet version of $$\text{LDSLICE}$$.
* $$\text{D71Fcc}$$ — $$\text{PLDSLICEQ}~cc + 1$$ ($$s \rightarrow s_{00}~-1~\text{or}~0$$), a quiet version of $$\text{PLDSLICE}$$.
* $$\text{D720}$$ — $$\text{SDCUTFIRST}$$ ($$s~l \rightarrow s_{0}$$), returns the first $$0 \leq l \leq 1023$$ bits of $$s$$. It is equivalent to $$\text{PLDSLICEX}$$.
* $$\text{D721}$$ — $$\text{SDSKIPFIRST}$$ ($$s~l \rightarrow s_{0}$$), returns all but the first $$0 \leq l \leq 1023$$ bits of $$s$$. It is equivalent to $$\text{LDSLICEX};~\text{NIP}$$.
* $$\text{D722}$$ — $$\text{SDCUTLAST}$$ ($$s~l \rightarrow s_{0}$$), returns the last $$0 \leq l \leq 1023$$ bits of $$s$$.
* $$\text{D723}$$ — $$\text{SDSKIPLAST}$$ ($$s~l \rightarrow s_{0}$$), returns all but the last $$0 \leq l \leq 1023$$ bits of $$s$$.
* $$\text{D724}$$ — $$\text{SDSUBSTR}$$ ($$s~l~l_{0} \rightarrow s_{0}$$), returns $$0 \leq l_{0} \leq 1023$$ bits of $$s$$ starting from offset $$0 \leq l \leq 1023$$, thus extracting a bit substring out of the data of $$s$$.
* $$\text{D726}$$ — $$\text{SDBEGINSX}$$ ($$s~s_{0} \rightarrow s_{00}$$), checks whether $$s$$ begins with (the data bits of) $$s_{0}$$, and removes $$s_{0}$$ from $$s$$ on success. On failure throws a cell deserialization exception. Primitive $$\text{SDPFXREV}$$ can be considered a quiet version of $$\text{SDBEGINSX}$$.
* $$\text{D727}$$ — $$\text{SDBEGINSXQ}$$ ($$s~s_{0} \rightarrow s_{00}~-1~\text{or}~s~0$$), a quiet version of $$\text{SDBEGINSX}$$.
* $$\text{D72A\_xsss}$$ — $$\text{SDBEGINS}$$ ($$s \rightarrow s_{00}$$), checks whether $$s$$ begins with constant bitstring $$sss$$ of length $$8x + 3$$ (with continuation bit assumed), where $$0 \leq x \leq 127$$, and removes $$sss$$ from $$s$$ on success.
* $$\text{D72802}$$ — $$\text{SDBEGINS}~'0'$$ ($$s \rightarrow s_{00}$$), checks whether $$s$$ begins with a binary zero.
* $$\text{D72806}$$ — $$\text{SDBEGINS}~'1'$$ ($$s \rightarrow s_{00}$$), checks whether $$s$$ begins with a binary one.
* $$\text{D72E\_xsss}$$ — $$\text{SDBEGINSQ}$$ ($$s \rightarrow s_{00}~-1~\text{or}~s~0$$), a quiet version of $$\text{SDBEGINS}$$.
* $$\text{D730}$$ — $$\text{SCUTFIRST}$$ ($$s~l~r \rightarrow s_{0}$$), returns the first $$0 \leq l \leq 1023$$ bits and first $$0 \leq r \leq 4$$ references of $$s$$.
* $$\text{D731}$$ — $$\text{SSKIPFIRST}$$ ($$s~l~r \rightarrow s_{0}$$).
* $$\text{D732}$$ — $$\text{SCUTLAST}$$ ($$s~l~r \rightarrow s_{0}$$), returns the last $$0 \leq l \leq 1023$$ data bits and last $$0 \leq r \leq 4$$ references of $$s$$.
* $$\text{D733}$$ — $$\text{SSKIPLAST}$$ ($$s~l~r \rightarrow s_{0}$$).
* $$\text{D734}$$ — $$\text{SUBSLICE}$$ ($$s~l~r~l_{0}~r_{0} \rightarrow s_{0}$$), returns $$0 \leq l_{0} \leq 1023$$ bits and $$0 \leq r_{0} \leq 4$$ references from Slice $$s$$, after skipping the first $$0 \leq l \leq 1023$$ bits and first $$0 \leq r \leq 4$$ references.
* $$\text{D736}$$ — $$\text{SPLIT}$$ ($$s~l~r \rightarrow s_{0}~s_{00}$$), splits the first $$0 \leq l \leq 1023$$ data bits and first $$0 \leq r \leq 4$$ references from $$s$$ into $$s_{0}$$, returning the remainder of $$s$$ as $$s_{00}$$.
* $$\text{D737}$$ — $$\text{SPLITQ}$$ ($$s~l~r \rightarrow s_{0}~s_{00}~-1~\text{or}~s~0$$), a quiet version of $$\text{SPLIT}$$.
* $$\text{D739}$$ — $$\text{XCTOS}$$ ($$c \rightarrow s~?$$), transforms an ordinary or exotic cell into a Slice, as if it were an ordinary cell. A flag is returned indicating whether $$c$$ is exotic. If that is the case, its type can later be deserialized from the first eight bits of $$s$$.
* $$\text{D73A}$$ — $$\text{XLOAD}$$ ($$c \rightarrow c_{0}$$), loads an exotic cell $$c$$ and returns an ordinary cell $$c_{0}$$. If $$c$$ is already ordinary, does nothing. If $$c$$ cannot be loaded, throws an exception.
* $$\text{D73B}$$ — $$\text{XLOADQ}$$ ($$c \rightarrow c_{0}~-1~\text{or}~c~0$$), loads an exotic cell $$c$$ as XLOAD, but returns 0 on failure.
* $$\text{D741}$$ — $$\text{SCHKBITS}$$ ($$s~l \rightarrow$$), checks whether there are at least $$l$$ data bits in Slice $$s$$. If this is not the case, throws a cell deserialization (i.e., cell underflow) exception.
* $$\text{D742}$$ — $$\text{SCHKREFS}$$ ($$s~r \rightarrow$$), checks whether there are at least $$r$$ references in Slice $$s$$.
* $$\text{D743}$$ — $$\text{SCHKBITREFS}$$ ($$s~l~r \rightarrow$$), checks whether there are at least $$l$$ data bits and $$r$$ references in Slice $$s$$.
* $$\text{D745}$$ — $$\text{SCHKBITSQ}$$ ($$s~l \rightarrow ?$$), checks whether there are at least $$l$$ data bits in Slice $$s$$.
* $$\text{D746}$$ — $$\text{SCHKREFSQ}$$ ($$s~r \rightarrow ?$$), checks whether there are at least $$r$$ references in Slice $$s$$.
* $$\text{D747}$$ — $$\text{SCHKBITREFSQ}$$ ($$s~l~r \rightarrow ?$$), checks whether there are at least $$l$$ data bits and $$r$$ references in Slice $$s$$.
* $$\text{D748}$$ — $$\text{PLDREFVAR}$$ ($$s~n \rightarrow c$$), returns the $$n$$-th cell reference of Slice $$s$$ for $$0 \leq n \leq 3$$.
* $$\text{D749}$$ — $$\text{SBITS}$$ ($$s \rightarrow l$$), returns the number of data bits in Slice $$s$$.
* $$\text{D74A}$$ — $$\text{SREFS}$$ ($$s \rightarrow r$$), returns the number of references in Slice $$s$$.
* $$\text{D74B}$$ — $$\text{SBITREFS}$$ ($$s \rightarrow l~r$$), returns both the number of data bits and the number of references in $$s$$.
* $$\text{D74E\_n}$$ — $$\text{PLDREFIDX}~n$$ ($$s \rightarrow c$$), returns the $$n$$-th cell reference of Slice $$s$$, where $$0 \leq n \leq 3$$.
* $$\text{D74C}$$ — $$\text{PLDREF}$$ ($$s \rightarrow c$$), preloads the first cell reference of a Slice.
* $$\text{D750}$$ — $$\text{LDILE4}$$ ($$s \rightarrow x~s_{0}$$), loads a little-endian signed 32-bit integer.
* $$\text{D751}$$ — $$\text{LDULE4}$$ ($$s \rightarrow x~s_{0}$$), loads a little-endian unsigned 32-bit integer.
* $$\text{D752}$$ — $$\text{LDILE8}$$ ($$s \rightarrow x~s_{0}$$), loads a little-endian signed 64-bit integer.
* $$\text{D753}$$ — $$\text{LDULE8}$$ ($$s \rightarrow x~s_{0}$$), loads a little-endian unsigned 64-bit integer.
* $$\text{D754}$$ — $$\text{PLDILE4}$$ ($$s \rightarrow x$$), preloads a little-endian signed 32-bit integer.
* $$\text{D755}$$ — $$\text{PLDULE4}$$ ($$s \rightarrow x$$), preloads a little-endian unsigned 32-bit integer.
* $$\text{D756}$$ — $$\text{PLDILE8}$$ ($$s \rightarrow x$$), preloads a little-endian signed 64-bit integer.
* $$\text{D757}$$ — $$\text{PLDULE8}$$ ($$s \rightarrow x$$), preloads a little-endian unsigned 64-bit integer.
* $$\text{D758}$$ — $$\text{LDILE4Q}$$ ($$s \rightarrow x~s_{0}~-1~\text{or}~s~0$$), quietly loads a little-endian signed 32-bit integer.
* $$\text{D759}$$ — $$\text{LDULE4Q}$$ ($$s \rightarrow x~s_{0}~-1~\text{or}~s~0$$), quietly loads a little-endian unsigned 32-bit integer.
* $$\text{D75A}$$ — $$\text{LDILE8Q}$$ ($$s \rightarrow x~s_{0}~-1~\text{or}~s~0$$), quietly loads a little-endian signed 64-bit integer.
* $$\text{D75B}$$ — $$\text{LDULE8Q}$$ ($$s \rightarrow x~s_{0}~-1~\text{or}~s~0$$), quietly loads a little-endian unsigned 64-bit integer.
* $$\text{D75C}$$ — $$\text{PLDILE4Q}$$ ($$s \rightarrow x~-1~\text{or}~0$$), quietly preloads a little-endian signed 32-bit integer.
* $$\text{D75D}$$ — $$\text{PLDULE4Q}$$ ($$s \rightarrow x~-1~\text{or}~0$$), quietly preloads a little-endian unsigned 32-bit integer.
* $$\text{D75E}$$ — $$\text{PLDILE8Q}$$ ($$s \rightarrow x~-1~\text{or}~0$$), quietly preloads a little-endian signed 64-bit integer.
* $$\text{D75F}$$ — $$\text{PLDULE8Q}$$ ($$s \rightarrow x~-1~\text{or}~0$$), quietly preloads a little-endian unsigned 64-bit integer.
* $$\text{D760}$$ — $$\text{LDZEROES}$$ ($$s \rightarrow n~s_{0}$$), returns the count $$n$$ of leading zero bits in $$s$$, and removes these bits from $$s$$.
* $$\text{D761}$$ — $$\text{LDONES}$$ ($$s \rightarrow n~s_{0}$$), returns the count $$n$$ of leading one bits in $$s$$, and removes these bits from $$s$$.
* $$\text{D762}$$ — $$\text{LDSAME}$$ ($$s~x \rightarrow n~s_{0}$$), returns the count $$n$$ of leading bits equal to $$0 \leq x \leq 1$$ in $$s$$, and removes these bits from $$s$$.
* $$\text{D764}$$ — $$\text{SDEPTH}$$ ($$s \rightarrow x$$), returns the depth of Slice $$s$$. If $$s$$ has no references, then $$x = 0$$; otherwise $$x$$ is one plus the maximum of depths of cells referred to from $$s$$.
* $$\text{D765}$$ — $$\text{CDEPTH}$$ ($$c \rightarrow x$$), returns the depth of Cell $$c$$. If $$c$$ has no references, then $$x = 0$$; otherwise $$x$$ is one plus the maximum of depths of cells referred to from $$c$$. If c is a *Null* instead of a *Cell*, returns zero.


## A.8 Continuation and control flow primitives

### A.8.1. Unconditional control flow primitives.

* $$\text{D8}$$ — $$\text{EXECUTE}$$ or $$\text{CALLX}$$ ($$c \rightarrow$$), calls or executes continuation $$c$$ (i.e., $$cc \leftarrow c \circ0~ cc$$).
* $$\text{D9}$$ — $$\text{JMPX}$$ ($$c \rightarrow$$), jumps, or transfers control, to continuation $$c$$ (i.e., $$cc \leftarrow c \circ0~ c0$$, or rather $$cc \leftarrow (c \circ0~ c0) \circ1~ c1$$). The remainder of the previous current continuation $$cc$$ is discarded.
* $$\text{DA}_{pr}$$ — $$\text{CALLXARGS}~ p,r$$ ($$c \rightarrow$$), calls continuation $$c$$ with $$p$$ parameters and expecting $$r$$ return values, $$0 \leq p \leq 15$$, $$0 \leq r \leq 15$$.
* $$\text{DB0}_{p}$$ — $$\text{CALLXARGS}~ p,−1$$ ($$c \rightarrow$$), calls continuation $$c$$ with $$0 \leq p \leq 15$$ parameters, expecting an arbitrary number of return values.
* $$\text{DB1}_{p}$$ — $$\text{JMPXARGS}~ p$$ ($$c \rightarrow$$), jumps to continuation $$c$$, passing only the top $$0 \leq p \leq 15$$ values from the current stack to it (the remainder of the current stack is discarded).
* $$\text{DB2}_{r}$$ — $$\text{RETARGS}~ r$$, returns to $$c0$$, with $$0 \leq r \leq 15$$ return values taken from the current stack.
* $$\text{DB30}$$ — $$\text{RET}$$ or $$\text{RETTRUE}$$, returns to the continuation at $$c0$$ (i.e., performs $$cc \leftarrow c0$$). The remainder of the current continuation $$cc$$ is discarded. Approximately equivalent to $$\text{PUSH}~ c0;~ \text{JMPX}$$.
* $$\text{DB31}$$ — $$\text{RETALT}$$ or $$\text{RETFALSE}$$, returns to the continuation at $$c1$$ (i.e., $$cc \leftarrow c1$$). Approximately equivalent to $$\text{PUSH}~ c1;~ \text{JMPX}$$.
* $$\text{DB32}$$ — $$\text{BRANCH}$$ or $$\text{RETBOOL}$$ ($$f \rightarrow$$), performs $$\text{RETTRUE}$$ if integer $$f \neq 0$$, or $$\text{RETFALSE}$$ if $$f = 0$$.
* $$\text{DB34}$$ — $$\text{CALLCC}$$ ($$c \rightarrow$$), call with current continuation, transfers control to $$c$$, pushing the old value of $$cc$$ into $$c$$’s stack (instead of discarding it or writing it into new $$c0$$).
* $$\text{DB35}$$ — $$\text{JMPXDATA}$$ ($$c \rightarrow$$), similar to $$\text{CALLCC}$$, but the remainder of the current continuation (the old value of $$cc$$) is converted into a Slice before pushing it into the stack of $$c$$.
* $$\text{DB36}_{pr}$$ — $$\text{CALLCCARGS}~ p,r$$ ($$c \rightarrow$$), similar to $$\text{CALLXARGS}$$, but pushes the old value of $$cc$$ (along with the top $$0 \leq p \leq 15$$ values from the original stack) into the stack of newly-invoked continuation $$c$$, setting $$cc.nargs$$ to $$−1 \leq r \leq 14$$.
* $$\text{DB38}$$ — $$\text{CALLXVARARGS}$$ ($$c~ p~ r \rightarrow$$), similar to $$\text{CALLXARGS}$$, but takes $$−1 \leq p, r \leq 254$$ from the stack. The next three operations also take $$p$$ and $$r$$ from the stack, both in the range $$−1 . . . 254$$.
* $$\text{DB39}$$ — $$\text{RETVARARGS}$$ ($$p~ r \rightarrow$$), similar to $$\text{RETARGS}$$.
* $$\text{DB3A}$$ — $$\text{JMPXVARARGS}$$ ($$c~ p~ r \rightarrow$$), similar to $$\text{JMPXARGS}$$.
* $$\text{DB3B}$$ — $$\text{CALLCCVARARGS}$$ ($$c~ p~ r \rightarrow$$), similar to $$\text{CALLCCARGS}$$.
* $$\text{DB3C}$$ — $$\text{CALLREF}$$, equivalent to $$\text{PUSHREFCONT};~ \text{CALLX}$$.
* $$\text{DB3D}$$ — $$\text{JMPREF}$$, equivalent to $$\text{PUSHREFCONT};~ \text{JMPX}$$.
* $$\text{DB3E}$$ — $$\text{JMPREFDATA}$$, equivalent to $$\text{PUSHREFCONT};~ \text{JMPXDATA}$$.
* $$\text{DB3F}$$ — $$\text{RETDATA}$$, equivalent to $$\text{PUSH}~ c0;~ \text{JMPXDATA}$$. In this way, the remainder of the current continuation is converted into a Slice and returned to the caller.

### A.8.2. Conditional control flow primitives.

* $$\text{DC}$$ — $$\text{IFRET}$$ ($$f \to$$), performs a $$\text{RET}$$, but only if integer $$f$$ is non-zero. If $$f$$ is a NaN, throws an integer overflow exception.
* $$\text{DD}$$ — $$\text{IFNOTRET}$$ ($$f \to$$), performs a $$\text{RET}$$, but only if integer $$f$$ is zero.
* $$\text{DE}$$ — $$\text{IF}$$ ($$f~c \to$$), performs $$\text{EXECUTE}$$ for $$c$$ (i.e., executes $$c$$), but only if integer $$f$$ is non-zero. Otherwise simply discards both values.
* $$\text{DF}$$ — $$\text{IFNOT}$$ ($$f~c \to$$), executes continuation $$c$$, but only if integer $$f$$ is zero. Otherwise simply discards both values.
* $$\text{E0}$$ — $$\text{IFJMP}$$ ($$f~c \to$$), jumps to $$c$$ (similarly to $$\text{JMPX}$$), but only if $$f$$ is non-zero.
* $$\text{E1}$$ — $$\text{IFNOTJMP}$$ ($$f~c \to$$), jumps to $$c$$ (similarly to $$\text{JMPX}$$), but only if $$f$$ is zero.
* $$\text{E2}$$ — $$\text{IFELSE}$$ ($$f~c~c_0 \to$$), if integer $$f$$ is non-zero, executes $$c$$, otherwise executes $$c_0$$. Equivalent to $$\text{CONDSELCHK};~\text{EXECUTE}$$.
* $$\text{E300}$$ — $$\text{IFREF}$$ ($$f \to$$), equivalent to $$\text{PUSHREFCONT};~\text{IF}$$, with the optimization that the cell reference is not actually loaded into a Slice and then converted into an ordinary Continuation if $$f = 0$$. Similar remarks apply to the next three primitives.
* $$\text{E301}$$ — $$\text{IFNOTREF}$$ ($$f \to$$), equivalent to $$\text{PUSHREFCONT};~\text{IFNOT}$$.
* $$\text{E302}$$ — $$\text{IFJMPREF}$$ ($$f \to$$), equivalent to $$\text{PUSHREFCONT};~\text{IFJMP}$$.
* $$\text{E303}$$ — $$\text{IFNOTJMPREF}$$ ($$f \to$$), equivalent to $$\text{PUSHREFCONT};~\text{IFNOTJMP}$$.
* $$\text{E304}$$ — $$\text{CONDSEL}$$ ($$f~x~y \to x~\text{or}~y$$), if integer $$f$$ is non-zero, returns $$x$$, otherwise returns $$y$$. Notice that no type checks are performed on $$x$$ and $$y$$; as such, it is more like a conditional stack operation. Roughly equivalent to $$\text{ROT};~\text{ISZERO};~\text{INC};~\text{ROLLX};~\text{NIP}$$.
* $$\text{E305}$$ — $$\text{CONDSELCHK}$$ ($$f~x~y \to x~\text{or}~y$$), same as $$\text{CONDSEL}$$, but first checks whether $$x$$ and $$y$$ have the same type.
* $$\text{E308}$$ — $$\text{IFRETALT}$$ ($$f \to$$), performs $$\text{RETALT}$$ if integer $$f \neq 0$$.
* $$\text{E309}$$ — $$\text{IFNOTRETALT}$$ ($$f \to$$), performs $$\text{RETALT}$$ if integer $$f = 0$$.
* $$\text{E30D}$$ — $$\text{IFREFELSE}$$ ($$f~c \to$$), equivalent to $$\text{PUSHREFCONT};~\text{SWAP};~\text{IFELSE}$$, with the optimization that the cell reference is not actually loaded into a Slice and then converted into an ordinary Continuation if $$f = 0$$. Similar remarks apply to the next two primitives: Cells are converted into Continuations only when necessary.
* $$\text{E30E}$$ — $$\text{IFELSEREF}$$ ($$f~c \to$$), equivalent to $$\text{PUSHREFCONT};~\text{IFELSE}$$.
* $$\text{E30F}$$ — $$\text{IFREFELSEREF}$$ ($$f \to$$), equivalent to $$\text{PUSHREFCONT};~\text{PUSHREFCONT};~\text{IFELSE}$$.
* $$\text{E310–E31F}$$ — reserved for loops with break operators, cf. A.8.3 below.
* $$\text{E39_n}$$ — $$\text{IFBITJMP}~ n$$ ($$x~c \to x$$), checks whether bit $$0 \leq n \leq 31$$ is set in integer $$x$$, and if so, performs $$\text{JMPX}$$ to continuation $$c$$. Value $$x$$ is left in the stack.
* $$\text{E3B_n}$$ — $$\text{IFNBITJMP}~ n$$ ($$x~c \to x$$), jumps to $$c$$ if bit $$0 \leq n \leq 31$$ is not set in integer $$x$$.
* $$\text{E3D_n}$$ — $$\text{IFBITJMPREF}~ n$$ ($$x \to x$$), performs a $$\text{JMPREF}$$ if bit $$0 \leq n \leq 31$$ is set in integer $$x$$.
* $$\text{E3F_n}$$ — $$\text{IFNBITJMPREF}~ n$$ ($$x \to x$$), performs a $$\text{JMPREF}$$ if bit $$0 \leq n \leq 31$$ is not set in integer $$x$$.

### A.8.3. Control flow primitives: loops.

* $$\text{E4}$$ — $$\text{REPEAT}$$ ($$n~c \to$$), executes continuation $$c$$ $$n$$ times, if integer $$n$$ is non-negative. If $$n \geq 2^{31}$$ or $$n < -2^{31}$$, generates a range check exception. Notice that a $$\text{RET}$$ inside the code of $$c$$ works as a continue, not as a break. One should use either alternative (experimental) loops or alternative $$\text{RETALT}$$ (along with a $$\text{SETEXITALT}$$ before the loop) to break out of a loop.
* $$\text{E5}$$ — $$\text{REPEATEND}$$ ($$n \to$$), similar to $$\text{REPEAT}$$, but it is applied to the current continuation $$\text{cc}$$.
* $$\text{E6}$$ — $$\text{UNTIL}$$ ($$c \to$$), executes continuation $$c$$, then pops an integer $$x$$ from the resulting stack. If $$x$$ is zero, performs another iteration of this loop. The actual implementation of this primitive involves an extraordinary continuation $$\text{ec\_until}$$ (cf. 4.1.5) with its arguments set to the body of the loop (continuation $$c$$) and the original current continuation $$\text{cc}$$. This extraordinary continuation is then saved into the savelist of $$c$$ as $$c.c_0$$ and the modified $$c$$ is then executed. The other loop primitives are implemented similarly with the aid of suitable extraordinary continuations.
* $$\text{E7}$$ — $$\text{UNTILEND}$$ ($\to$), similar to $$\text{UNTIL}$$, but executes the current continuation $$\text{cc}$$ in a loop. When the loop exit condition is satisfied, performs a $$\text{RET}$$.
* $$\text{E8}$$ — $$\text{WHILE}$$ ($$c_0~c \to$$), executes $$c_0$$ and pops an integer $$x$$ from the resulting stack. If $$x$$ is zero, exists the loop and transfers control to the original $$\text{cc}$$. If $$x$$ is non-zero, executes $$c$$, and then begins a new iteration.
* $$\text{E9}$$ — $$\text{WHILEEND}$$ ($$c_0 \to$$), similar to $$\text{WHILE}$$, but uses the current continuation $$\text{cc}$$ as the loop body.
* $$\text{EA}$$ — $$\text{AGAIN}$$ ($$c \to$$), similar to $$\text{REPEAT}$$, but executes $$c$$ infinitely many times. A $$\text{RET}$$ only begins a new iteration of the infinite loop, which can be exited only by an exception, or a $$\text{RETALT}$$ (or an explicit $$\text{JMPX}$$).
* $$\text{EB}$$ — $$\text{AGAINEND}$$ ($\to$), similar to $$\text{AGAIN}$$, but performed with respect to the current continuation $$\text{cc}$$.
* $$\text{E314}$$ — $$\text{REPEATBRK}$$ ($$n~c \to$$), similar to $$\text{REPEAT}$$, but also sets $$c_1$$ to the original $$\text{cc}$$ after saving the old value of $$c_1$$ into the savelist of the original $$\text{cc}$$. In this way $$\text{RETALT}$$ could be used to break out of the loop body.
* $$\text{E315}$$ — $$\text{REPEATENDBRK}$$ ($$n \to$$), similar to $$\text{REPEATEND}$$, but also sets $$c_1$$ to the original $$c_0$$ after saving the old value of $$c_1$$ into the savelist of the original $$c_0$$. Equivalent to $$\text{SAMEALTSAVE};~\text{REPEATEND}$$.
* $$\text{E316}$$ — $$\text{UNTILBRK}$$ ($$c \to$$), similar to $$\text{UNTIL}$$, but also modifies $$c_1$$ in the same way as $$\text{REPEATBRK}$$.
* $$\text{E317}$$ — $$\text{UNTILENDBRK}$$ ($\to$), equivalent to $$\text{SAMEALTSAVE};~\text{UNTILEND}$$.
* $$\text{E318}$$ — $$\text{WHILEBRK}$$ ($$c_0~c \to$$), similar to $$\text{WHILE}$$, but also modifies $$c_1$$ in the same way as $$\text{REPEATBRK}$$.
* $$\text{E319}$$ — $$\text{WHILEENDBRK}$$ ($$c \to$$), equivalent to $$\text{SAMEALTSAVE};~\text{WHILEEND}$$.
* $$\text{E31A}$$ — $$\text{AGAINBRK}$$ ($$c \to$$), similar to $$\text{AGAIN}$$, but also modifies $$c_1$$ in the same way as $$\text{REPEATBRK}$$.
* $$\text{E31B}$$ — $$\text{AGAINENDBRK}$$ ($\to$), equivalent to $$\text{SAMEALTSAVE};~\text{AGAINEND}$$.


## A.8.4. Manipulating the stack of continuations.

* $$\text{EC}_{rn}$$ — $$\text{SETCONTARGS}~r,n$$ ($$x_1~x_2~\dots~x_r~c \to c_0$$), similar to $$\text{SETCONTARGS}~r$$, but sets $$c.nargs$$ to the final size of the stack of $$c_0$$ plus $$n$$. In other words, transforms $$c$$ into a closure or a partially applied function, with $$0 \leq n \leq 14$$ arguments missing.
* $$\text{EC}_{0n}$$ — $$\text{SETNUMARGS}~n$$ or $$\text{SETCONTARGS}~0,n$$ ($$c \to c_0$$), sets $$c.nargs$$ to $$n$$ plus the current depth of $$c$$'s stack, where $$0 \leq n \leq 14$$. If $$c.nargs$$ is already set to a non-negative value, does nothing.
* $$\text{EC}_{rF}$$ — $$\text{SETCONTARGS}~r$$ or $$\text{SETCONTARGS}~r,-1$$ ($$x_1~x_2~\dots~x_r~c \to c_0$$), pushes $$0 \leq r \leq 15$$ values $$x_1~\dots~x_r$$ into the stack of (a copy of) the continuation $$c$$, starting with $$x_1$$. If the final depth of $$c$$'s stack turns out to be greater than $$c.nargs$$, a stack overflow exception is generated.
* $$\text{ED}_{0p}$$ — $$\text{RETURNARGS}~p$$ ($\to$), leaves only the top $$0 \leq p \leq 15$$ values in the current stack (somewhat similarly to $$\text{ONLYTOPX}$$), with all the unused bottom values not discarded, but saved into continuation $$c_0$$ in the same way as $$\text{SETCONTARGS}$$ does.
* $$\text{ED}_{10}$$ — $$\text{RETURNVARARGS}$$ ($$p \to$$), similar to $$\text{RETURNARGS}$$, but with Integer $$0 \leq p \leq 255$$ taken from the stack.
* $$\text{ED}_{11}$$ — $$\text{SETCONTVARARGS}$$ ($$x_1~x_2~\dots~x_r~c~r~n \to c_0$$), similar to $$\text{SETCONTARGS}$$, but with $$0 \leq r \leq 255$$ and $$−1 \leq n \leq 255$$ taken from the stack.
* $$\text{ED}_{12}$$ — $$\text{SETNUMVARARGS}$$ ($$c~n \to c_0$$), where $$−1 \leq n \leq 255$$. If $$n = −1$$, this operation does nothing ($$c_0 = c$$). Otherwise its action is similar to $$\text{SETNUMARGS}~n$$, but with $$n$$ taken from the stack.


### A.8.5. Creating simple continuations and closures.

* $$\text{ED}_{1E}$$ — $$\text{BLESS}$$ ($$s \to c$$), transforms a Slice $$s$$ into a simple ordinary continuation $$c$$, with $$c.code = s$$ and an empty stack and savelist.
* $$\text{ED}_{1F}$$ — $$\text{BLESSVARARGS}$$ ($$x_1~\dots~x_r~s~r~n \to c$$), equivalent to $$\text{ROT}$$; $$\text{BLESS}$$; $$\text{ROTREV}$$; $$\text{SETCONTVARARGS}$$.
* $$\text{EE}_{rn}$$ — $$\text{BLESSARGS}~r,~n$$ ($$x_1~\dots~x_r~s \to c$$), where $$0 \leq r \leq 15$$, $$−1 \leq n \leq 14$$, equivalent to $$\text{BLESS}$$; $$\text{SETCONTARGS}~r,~n$$. The value of $$n$$ is represented inside the instruction by the 4-bit integer $$n \mod 16$$.
* $$\text{EE}_{0n}$$ — $$\text{BLESSNUMARGS}~n$$ or $$\text{BLESSARGS}~0,n$$ ($$s \to c$$), also transforms a Slice $$s$$ into a Continuation $$c$$, but sets $$c.nargs$$ to $$0 \leq n \leq 14$$.

### A.8.6. Operations with continuation savelists and control registers.

* $${ED4i}$$~—~PUSH~c(i)~or~PUSHCTR~c(i)~(–~x)$$, pushes the current value of control register $$c(i)$$. If the control register is not supported in the current codepage, or if it does not have a value, an exception is triggered.
* $${ED44}$$~—~PUSH~c4~or~PUSHROOT$$, pushes the “global data root” cell reference, thus enabling access to persistent smart-contract data.
* $${ED5i}$$~—~POP~c(i)~or~POPCTR~c(i)~(x~–)$$, pops a value $$x$$ from the stack and stores it into control register $$c(i)$$, if supported in the current codepage. Notice that if a control register accepts only values of a specific type, a type-checking exception may occur.
* $${ED54}$$~—~POP~c4~or~POPROOT$$, sets the “global data root” cell reference, thus allowing modification of persistent smart-contract data.
* $${ED6i}$$~—~SETCONT~c(i)~or~SETCONTCTR~c(i)~(x~c~–~c_0)$$, stores $$x$$ into the savelist of continuation $$c$$ as $$c(i)$$, and returns the resulting continuation $$c_0$$. Almost all operations with continuations may be expressed in terms of $$SETCONTCTR$$, $$POPCTR$$, and $$PUSHCTR$$.
* $${ED7i}$$~—~SETRETCTR~c(i)~(x~–)$$, equivalent to $$PUSH~c0$$; $$SETCONTCTR~c(i)$$; $$POP~c0$$.
* $${ED8i}$$~—~SETALTCTR~c(i)~(x~–)$$, equivalent to $$PUSH~c1$$; $$SETCONTCTR~c(i)$$; $$POP~c0$$.
* $${ED9i}$$~—~POPSAVE~c(i)~or~POPCTRSAVE~c(i)~(x~–)$$, similar to $$POP~c(i)$$, but also saves the old value of $$c(i)$$ into continuation $$c0$$. Equivalent (up to exceptions) to $$SAVECTR~c(i)$$; $$POP~c(i)$$.
* $${EDAi}$$~—~SAVE~c(i)~or~SAVECTR~c(i)~(–)$$, saves the current value of $$c(i)$$ into the savelist of continuation $$c0$$. If an entry for $$c(i)$$ is already present in the savelist of $$c0$$, nothing is done. Equivalent to $$PUSH~c(i)$$; $$SETRETCTR~c(i)$$.
* $${EDBi}$$~—~SAVEALT~c(i)~or~SAVEALTCTR~c(i)~(–)$$, similar to $$SAVE~c(i)$$, but saves the current value of $$c(i)$$ into the savelist of $$c1$$, not $$c0$$.
* $${EDCi}$$~—~SAVEBOTH~c(i)~or~SAVEBOTHCTR~c(i)~(–)$$, equivalent to $$DUP$$; $$SAVE~c(i)$$; $$SAVEALT~c(i)$$.
* $${EDE0}$$~—~PUSHCTRX~(i~–~x)$$, similar to $$PUSHCTR~c(i)$$, but with $$i$$, $$0 \leq i \leq 255$$, taken from the stack. Notice that this primitive is one of the few “exotic” primitives, which are not polymorphic like stack manipulation primitives, and at the same time do not have well-defined types of parameters and return values, because the type of $$x$$ depends on $$i$$.
* $${EDE1}$$~—~POPCTRX~(x~i~–)$$, similar to $$POPCTR~c(i)$$, but with $$0 \leq i \leq 255$$ from the stack.
* $${EDE2}$$~—~SETCONTCTRX~(x~c~i~–~c_0)$$, similar to $$SETCONTCTR~c(i)$$, but with $$0 \leq i \leq 255$$ from the stack.
* $${EDF0}$$~—~COMPOS~or~BOOLAND~(c~c0~–~c_{00})$$, computes the composition $$c \circ_0 c_0$$, which has the meaning of “perform $$c$$, and, if successful, perform $$c_0$$” (if $$c$$ is a boolean circuit) or simply “perform $$c$$, then $$c_0$$”. Equivalent to $$SWAP$$; $$SETCONT~c0$$.
* $${EDF1}$$~—~COMPOSALT~or~BOOLOR~(c~c0~–~c_{00})$$, computes the alternative composition $$c \circ_1 c_0$$, which has the meaning of “perform $$c$$, and, if not successful, perform $$c_0$$” (if $$c$$ is a boolean circuit). Equivalent to $$SWAP$$; $$SETCONT~c1$$.
* $${EDF2}$$~—~COMPOSBOTH~(c~c0~–~c_{00})$$, computes $$(c \circ_0 c_0) \circ_1 c_0$$, which has the meaning of “compute boolean circuit $$c$$, then compute $$c_0$$, regardless of the result of $$c$$”.
* $${EDF3}$$~—~ATEXIT~(c~–)$$, sets $$c0 \leftarrow c \circ_0 c0$$. In other words, $$c$$ will be executed before exiting current subroutine.
* $${EDF4}$$~—~ATEXITALT~(c~–)$$, sets $$c1 \leftarrow c \circ_1 c1$$. In other words, $$c$$ will be executed before exiting current subroutine by its alternative return path.
* $${EDF5}$$~—~SETEXITALT~(c~–)$$, sets $$c1 \leftarrow (c \circ_0 c0) \circ_1 c1$$. In this way, a subsequent RETALT will first execute $$c$$, then transfer control to the original $$c0$$. This can be used, for instance, to exit from nested loops.
* $${EDF6}$$~—~THENRET~(c~–~c_0)$$, computes $$c_0 := c \circ_0 c0$$
* $${EDF7}$$~—~THENRETALT~(c~–~c_0)$$, computes $$c_0 := c \circ_0 c1$$
* $${EDF8}$$~—~INVERT~(–)$$, interchanges $$c0$$ and $$c1$$.
* $${EDF9}$$~—~BOOLEVAL~(c~–~?)$$, performs $$cc \leftarrow$$ 

### A.8.7. Dictionary subroutine calls and jumps.

* $${F0n}$$~—~CALL~n~or~CALLDICT~n~(–~n)$$, calls the continuation in $$c3$$, pushing integer $$0 \leq n \leq 255$$ into its stack as an argument. Approximately equivalent to $$PUSHINT~n$$; $$PUSH~c3$$; $$EXECUTE$$.
* $${F12_n}$$~—~CALL~n~for~0 \leq n < 2^{14}~(–~n)$$, an encoding of CALL n for larger values of n.
* $${F16_n}$$~—~JMP~n~or~JMPDICT~n~(–~n)$$, jumps to the continuation in $$c3$$, pushing integer $$0 \leq n < 2^{14}$$ as its argument. Approximately equivalent to $$PUSHINT~n$$; $$PUSH~c3$$; $$JMPX$$.
* $${F1A_n}$$~—~PREPARE~n~or~PREPAREDICT~n~(–~n~c)$$, equivalent to $$PUSHINT~n$$; $$PUSH~c3$$, for $$0 \leq n < 2^{14}$$. In this way, CALL n is approximately equivalent to PREPARE n; EXECUTE, and JMP n is approximately equivalent to PREPARE n; JMPX.

## A.9 Exception generating and handling primitives

### A.9.1. Throwing exceptions.

* $${F22_nn}$$~—~THROW~nn~(–~0~nn)$$, throws exception $$0 \leq nn \leq 63$$ with parameter zero. In other words, it transfers control to the continuation in $$c2$$, pushing 0 and $$nn$$ into its stack, and discarding the old stack altogether.
* $${F26_nn}$$~—~THROWIF~nn~(f~–)$$, throws exception $$0 \leq nn \leq 63$$ with parameter zero only if integer $$f \neq 0$$.
* $${F2A_nn}$$~—~THROWIFNOT~nn~(f~–)$$, throws exception $$0 \leq nn \leq 63$$ with parameter zero only if integer $$f = 0$$.
* $${F2C4_nn}$$~—~THROW~nn~for~0 \leq nn < 2^{11}$$, an encoding of THROW $$nn$$ for larger values of $$nn$$.
* $${F2CC_nn}$$~—~THROWARG~nn~(x~–~x~nn)$$, throws exception $$0 \leq nn < 2^{11}$$ with parameter $$x$$, by copying $$x$$ and $$nn$$ into the stack of $$c2$$ and transferring control to $$c2$$.
* $${F2D4_nn}$$~—~THROWIF~nn~(f~–)~for~0 \leq nn < 2^{11}$$.
* $${F2DC_nn}$$~—~THROWARGIF~nn~(x~f~–)$$, throws exception $$0 \leq nn < 2^{11}$$ with parameter $$x$$ only if integer $$f \neq 0$$.
* $${F2E4_nn}$$~—~THROWIFNOT~nn~(f~–)~for~0 \leq nn < 2^{11}$$.
* $${F2EC_nn}$$~—~THROWARGIFNOT~nn~(x~f~–)$$, throws exception $$0 \leq nn < 2^{11}$$ with parameter $$x$$ only if integer $$f = 0$$.
* $${F2F0}$$~—~THROWANY~(n~–~0~n)$$, throws exception $$0 \leq n < 2^{16}$$ with parameter zero. Approximately equivalent to $$PUSHINT~0$$; $$SWAP$$; $$THROWARGANY$$.
* $${F2F1}$$~—~THROWARGANY~(x~n~–~x~n)$$, throws exception $$0 \leq n < 2^{16}$$ with parameter $$x$$, transferring control to the continuation in $$c2$$. Approximately equivalent to $$PUSH~c2$$; $$JMPXARGS~2$$.
* $${F2F2}$$~—~THROWANYIF~(n~f~–)$$, throws exception $$0 \leq n < 2^{16}$$ with parameter zero only if $$f \neq 0$$.
* $${F2F3}$$~—~THROWARGANYIF~(x~n~f~–)$$, throws exception $$0 \leq n < 2^{16}$$ with parameter $$x$$ only if $$f \neq 0$$.
* $${F2F4}$$~—~THROWANYIFNOT~(n~f~–)$$, throws exception $$0 \leq n < 2^{16}$$ with parameter zero only if $$f = 0$$.
* $${F2F5}$$~—~THROWARGANYIFNOT~(x~n~f~–)$$, throws exception $$0 \leq n < 2^{16}$$ with parameter $$x$$ only if $$f = 0$$.

### A.9.2. Catching and handling exceptions.

* $${F2FF}$$~—~TRY~(c~c0~–)$$, sets $$c2$$ to $$c_0$$, first saving the old value of $$c2$$ both into the savelist of $$c_0$$ and into the savelist of the current continuation, which is stored into $$c.c0$$ and $$c_0.c0$$. Then runs $$c$$ similarly to EXECUTE.
* $${F3pr}$$~—~TRYARGS~p,r~(c~c0~–)$$, similar to TRY, but with $$CALLARGS~p,r$$ internally used instead of EXECUTE.

## A.10 Dictionary manipulation primitives

TVM's dictionary support is discussed at length in [$$\mathbf{3.3}$$](\[\$$/mathbf%7B3.3%7D\$$]\(cells-memory-and-persistent-storage.md#3.3-hashmaps-or-dictionaries\)) The basic operations with dictionaries are listed in [$$\mathbf{3.3.10}$$](cells-memory-and-persistent-storage.md#3.3.10.-basic-dictionary-operations.), while the taxonomy of dictionary manipulation primitives is provided in [$$\mathbf{3.3.11}$$](cells-memory-and-persistent-storage.md#3.3.11.-taxonomy-of-dictionary-primitives.) Here we use the concepts and notation introduced in those sections.

Dictionaries admit two different representations as TVM stack values:

* A Slice $$s$$ with a serialization of a TL-B value of type $$\operatorname{Hashmap} E(n, X)$$. In other words, $$s$$ consists either of one bit equal to zero (if the dictionary is empty), or of one bit equal to one and a reference to a Cell containing the root of the binary tree, i.e., a serialized value of type $$\operatorname{Hashmap}(n, X)$$.
* A "maybe Cell" c? , i.e., a value that is either a Cell (containing a serialized value of type $$\operatorname{Hashmap}(n, X)$$ as before) or a $$N u l l$$ (corresponding to an empty dictionary). When a "maybe Cell" $$c$$ ? is used to represent a dictionary, we usually denote it by $$D$$ in the stack notation.

Most of the dictionary primitives listed below accept and return dictionaries in the second form, which is more convenient for stack manipulation. However, serialized dictionaries inside larger TL-B objects use the first representation.

Opcodes starting with F4 and F5 are reserved for dictionary operations.

### A.10.1. Dictionary creation.

* $${6D}$$~—~NEWDICT~(–~D)$$, returns a new empty dictionary. It is an alternative mnemonics for $$PUSHNULL$$.
* $${6E}$$~—~DICTEMPTY~(D~–~?)$$, checks whether dictionary $$D$$ is empty, and returns −1 or 0 accordingly.

### A.10.2. Dictionary serialization and deserialization.

* $${CE}$$~—~ST_DICTS~(s~b~–~b_0)$$, stores a Slice-represented dictionary $$s$$ into Builder $$b$$. It is actually a synonym for $$STSLICE$$.
* $${F400}$$~—~ST_DICT~or~STOPTREF~(D~b~–~b_0)$$, stores dictionary $$D$$ into Builder $$b$$, returning the resulting Builder $$b_0$$. In other words, if $$D$$ is a cell, performs $$STONE$$ and $$STREF$$; if $$D$$ is Null, performs $$NIP$$ and $$STZERO$$; otherwise throws a type checking exception.
* $${F401}$$~—~SKIPDICT~or~SKIPOPTREF~(s~–~s_0)$$, equivalent to $$LDDICT$$; $$NIP$$.
* $${F402}$$~—~LDDICTS~(s~–~s_0~s_{00})$$, loads (parses) a (Slice-represented) dictionary $$s_0$$ from Slice $$s$$, and returns the remainder of $$s$$ as $$s_{00}$$. This is a “split function” for all $$HashmapE(n, X)$$ dictionary types.
* $${F403}$$~—~PLDDICTS~(s~–~s_0)$$, preloads a (Slice-represented) dictionary $$s_0$$ from Slice $$s$$. Approximately equivalent to $$LDDICTS$$; $$DROP$$.
* $${F404}$$~—~LDDICT~or~LDOPTREF~(s~–~D~s_0)$$, loads (parses) a dictionary $$D$$ from Slice $$s$$, and returns the remainder of $$s$$ as $$s_0$$. May be applied to dictionaries or to values of arbitrary $$(ˆY )?$$ types.
* $${F405}$$~—~PLDDICT~or~PLDOPTREF~(s~–~D)$$, preloads a dictionary $$D$$ from Slice $$s$$. Approximately equivalent to $$LDDICT$$; $$DROP$$.
* $${F406}$$~—~LDDICTQ~(s~–~D~s_0~−1~or~s~0)$$, a quiet version of $$LDDICT$$.
* $${F407}$$~—~PLDDICTQ~(s~–~D~−1~or~0)$$, a quiet version of $$PLDDICT$$.

### A.10.3. GET dictionary operations.

* $${F40A}$$~—~DICTGET~(k~D~n~–~x~−1~or~0)$$, looks up key $$k$$ (represented by a Slice, the first $$0 \leq n \leq 1023$$ data bits of which are used as a key) in dictionary $$D$$ of type $$HashmapE(n, X)$$ with $$n$$-bit keys. On success, returns the value found as a Slice $$x$$.
* $${F40B}$$~—~DICTGETREF~(k~D~n~–~c~−1~or~0)$$, similar to $$DICTGET$$, but with a $$LDREF; ENDS$$ applied to $$x$$ on success. This operation is useful for dictionaries of type $$HashmapE(n, ˆY )$$.
* $${F40C}$$~—~DICTIGET~(i~D~n~–~x~−1~or~0)$$, similar to $$DICTGET$$, but with a signed (big-endian) $$n$$-bit Integer $$i$$ as a key. If $$i$$ does not fit into $$n$$ bits, returns 0. If $$i$$ is a NaN, throws an integer overflow exception.
* $${F40D}$$~—~DICTIGETREF~(i~D~n~–~c~−1~or~0)$$, combines $$DICTIGET$$ with DICTGETREF: it uses signed $$n$$-bit Integer $$i$$ as a key and returns a Cell instead of a Slice on success.
* $${F40E}$$~—~DICTUGET~(i~D~n~–~x~−1~or~0)$$, similar to $$DICTIGET$$, but with unsigned (big-endian) $$n$$-bit Integer $$i$$ used as a key.
* $${F40F}$$~—~DICTUGETREF~(i~D~n~–~c~−1~or~0)$$, similar to $$DICTIGETREF$$, but with an unsigned $$n$$-bit Integer key $$i$$.

### A.10.4. Set/RePlace/ADD dictionary operations.

The mnemonics of the following dictionary primitives are constructed in a systematic fashion according to the regular expression DICT\[, I, U] (SET, REPLACE, ADD) \[GET]\[REF] depending on the type of the key used (a Slice or a signed or unsigned Integer), the dictionary operation to be performed, and the way the values are accepted and returned (as Cells or as Slices). Therefore, we provide a detailed description only for some primitives, assuming that this information is sufficient for the reader to understand the precise action of the remaining primitives.

* $${F412}$$~—~DICTSET~(x~k~D~n~–~D_0)$$, sets the value associated with $$n$$-bit key $$k$$ (represented by a Slice as in $$DICTGET$$) in dictionary $$D$$ (also represented by a Slice) to value $$x$$ (again a Slice), and returns the resulting dictionary as $$D_0$$.
* $${F413}$$~—~DICTSETREF~(c~k~D~n~–~D_0)$$, similar to $$DICTSET$$, but with the value set to a reference to Cell $$c$$.
* $${F414}$$~—~DICTISET~(x~i~D~n~–~D_0)$$, similar to $$DICTSET$$, but with the key represented by a (big-endian) signed $$n$$-bit integer $$i$$. If $$i$$ does not fit into $$n$$ bits, a range check exception is generated.
* $${F415}$$~—~DICTISETREF~(c~i~D~n~–~D_0)$$, similar to $$DICTSETREF$$, but with the key a signed $$n$$-bit integer as in $$DICTISET.
* $${F416}$$~—~DICTUSET~(x~i~D~n~–~D_0)$$, similar to DICTISET, but with $$i$$ an unsigned $$n$$-bit integer.
* $${F417}$$~—~DICTUSETREF~(c~i~D~n~–~D_0)$$, similar to DICTISETREF, but with $$i$$ unsigned.
* $${F41A}$$~—~DICTSETGET~(x~k~D~n~–~D_0~y~−1~or~D_0~0)$$, combines $$DICTSET$$ with $$DICTGET$$: it sets the value corresponding to key $$k$$ to $$x$$, but also returns the old value $$y$$ associated with the key in question, if present.
* $${F41B}$$~—~DICTSETGETREF~(c~k~D~n~–~D_0~c_0~−1~or~D_0~0)$$, combines $$DICTSETREF$$ with DICTGETREF similarly to $$DICTSETGET$$.
* $${F41C}$$~—~DICTISETGET~(x~i~D~n~–~D_0~y~−1~or~D_0~0)$$, similar to $$DICTSETGET$$, but with the key represented by a big-endian signed $$n$$-bit Integer $$i$$.
* $${F41D}$$~—~DICTISETGETREF~(c~i~D~n~–~D_0~c_0~−1~or~D_0~0)$$, a version of $$DICTSETGETREF$$ with signed Integer $$i$$ as a key.
* $${F41E}$$~—~DICTUSETGET~(x~i~D~n~–~D_0~y~−1~or~D_0~0)$$, similar to $$DICTISETGET$$, but with $$i$$ an unsigned $$n$$-bit integer.
* $${F41F}$$~—~DICTUSETGETREF~(c~i~D~n~–~D_0~c_0~−1~or~D_0~0)$$.
* $${F422}$$~—~DICTREPLACE~(x~k~D~n~–~D_0~−1~or~D~0)$$, a Replace operation, which is similar to $$DICTSET$$, but sets the value of key $$k$$ in dictionary $$D$$ to $$x$$ only if the key $$k$$ was already present in $$D$$.
* $${F423}$$~—~DICTREPLACEREF~(c~k~D~n~–~D_0~−1~or~D~0)$$, a Replace counterpart of $$DICTSETREF$$.
* $${F424}$$~—~DICTIREPLACE~(x~i~D~n~–~D_0~−1~or~D~0)$$, a version of $$DICTREPLACE$$ with signed $$n$$-bit Integer $$i$$ used as a key.
* $${F425}$$~—~DICTIREPLACEREF~(c~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F426}$$~—~DICTUREPLACE~(x~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F427}$$~—~DICTUREPLACEREF~(c~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F42A}$$~—~DICTREPLACEGET~(x~k~D~n~–~D_0~y~−1~or~D~0)$$, a Replace counterpart of $$DICTSETGET$$: on success, also returns the old value associated with the key in question.
* $${F42B}$$~—~DICTREPLACEGETREF~(c~k~D~n~–~D_0~c_0~−1~or~D~0)$$.
* $${F42C}$$~—~DICTIREPLACEGET~(x~i~D~n~–~D_0~y~−1~or~D~0)$$.
* $${F42D}$$~—~DICTIREPLACEGETREF~(c~i~D~n~–~D_0~c_0~−1~or~D_0~0)$$.
* $${F42E}$$~—~DICTUREPLACEGET~(x~i~D~n~–~D_0~y~−1~or~D_0~0)$$.
* $${F42F}$$~—~DICTUREPLACEGETREF~(c~i~D~n~–~D_0~c_0~−1~or~D_0~0)$$.
* $${F432}$$~—~DICTADD~(x~k~D~n~–~D_0~−1~or~D~0)$$, an Add counterpart of $$DICTSET$$: sets the value associated with key $$k$$ in dictionary $$D$$ to $$x$$, but only if it is not already present in $$D$$.
* $${F433}$$~—~DICTADDREF~(c~k~D~n~–~D_0~−1~or~D~0)$$.
* $${F434}$$~—~DICTIADD~(x~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F435}$$~—~DICTIADDREF~(c~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F436}$$~—~DICTUADD~(x~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F437}$$~—~DICTUADDREF~(c~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F43A}$$~—~DICTADDGET~(x~k~D~n~–~D_0~−1~or~D~y~0)$$, an Add counterpart of $$DICTSETGET$$: sets the value associated with key $$k$$ in dictionary $$D$$ to $$x$$, but only if key $$k$$ is not already present in $$D$$. Otherwise, just returns the old value $$y$$ without changing the dictionary.
* $${F43B}$$~—~DICTADDGETREF~(c~k~D~n~–~D_0~−1~or~D~c_0~0)$$, an Add counterpart of $$DICTSETGETREF$$.
* $${F43C}$$~—~DICTIADDGET~(x~i~D~n~–~D_0~−1~or~D~y~0)$$.
* $${F43D}$$~—~DICTIADDGETREF~(c~i~D~n~–~D_0~−1~or~D~c_0~0)$$.
* $${F43E}$$~—~DICTUADDGET~(x~i~D~n~–~D_0~−1~or~D~y~0)$$.
* $${F43F}$$~—~DICTUADDGETREF~(c~i~D~n~–~D_0~−1~or~D~c_0~0)$$.

### A.10.5. Builder-accepting variants of SET dictionary operations.

The following primitives accept the new value as a Builder $$b$$ instead of a Slice $$x$$, which often is more convenient if the value needs to be serialized from several components computed in the stack. (This is reflected by appending a B to the mnemonics of the corresponding SET primitives that work with Slices.) The net effect is roughly equivalent to converting $$b$$ into a Slice by ENDC; CTOS and executing the corresponding primitive listed in [$$\mathbf{A.10.4}$$](a-instructions-and-opcodes.md#a.10.4.-set-replace-add-dictionary-operations.)

* $${F441}$$~—~DICTSETB~(b~k~D~n~–~D_0)$$.
* $${F442}$$~—~DICTISETB~(b~i~D~n~–~D_0)$$.
* $${F443}$$~—~DICTUSETB~(b~i~D~n~–~D_0)$$.
* $${F445}$$~—~DICTSETGETB~(b~k~D~n~–~D_0~y~−1~or~D_0~0)$$.
* $${F446}$$~—~DICTISETGETB~(b~i~D~n~–~D_0~y~−1~or~D_0~0)$$.
* $${F447}$$~—~DICTUSETGETB~(b~i~D~n~–~D_0~y~−1~or~D_0~0)$$.
* $${F449}$$~—~DICTREPLACEB~(b~k~D~n~–~D_0~−1~or~D~0)$$.
* $${F44A}$$~—~DICTIREPLACEB~(b~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F44B}$$~—~DICTUREPLACEB~(b~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F44D}$$~—~DICTREPLACEGETB~(b~k~D~n~–~D_0~y~−1~or~D~0)$$.
* $${F44E}$$~—~DICTIREPLACEGETB~(b~i~D~n~–~D_0~y~−1~or~D~0)$$.
* $${F44F}$$~—~DICTUREPLACEGETB~(b~i~D~n~–~D_0~y~−1~or~D~0)$$.
* $${F451}$$~—~DICTADDB~(b~k~D~n~–~D_0~−1~or~D~0)$$.
* $${F452}$$~—~DICTIADDB~(b~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F453}$$~—~DICTUADDB~(b~i~D~n~–~D_0~−1~or~D~0)$$.
* $${F455}$$~—~DICTADDGETB~(b~k~D~n~–~D_0~−1~or~D~y~0)$$.
* $${F456}$$~—~DICTIADDGETB~(b~i~D~n~–~D_0~−1~or~D~y~0)$$.
* $${F457}$$~—~DICTUADDGETB~(b~i~D~n~–~D_0~−1~or~D~y~0)$$.

### A.10.6. DeLETE dictionary operations.

* $${F459}$$~—~DICTDEL~(k~D~n~–~D_0~−1~or~D~0)$$, deletes n-bit key, represented by a Slice $$k$$, from dictionary $$D$$. If the key is present, returns the modified dictionary $$D_0$$ and the success flag −1. Otherwise, returns the original dictionary $$D$$ and 0.
* $${F45A}$$~—~DICTIDEL~(i~D~n~–~D_0~?)$$, a version of DICTDEL with the key represented by a signed n-bit Integer $$i$$. If $$i$$ does not fit into n bits, simply returns $$D~0$$ (“key not found, dictionary unmodified”).
* $${F45B}$$~—~DICTUDEL~(i~D~n~–~D_0~?)$$, similar to DICTIDEL, but with $$i$$ an unsigned n-bit integer.
* $${F462}$$~—~DICTDELGET~(k~D~n~–~D_0~x~−1~or~D~0)$$, deletes n-bit key, represented by a Slice $$k$$, from dictionary $$D$$. If the key is present, returns the modified dictionary $$D_0$$, the original value $$x$$ associated with the key $$k$$ (represented by a Slice), and the success flag −1. Otherwise, returns the original dictionary $$D$$ and 0.
* $${F463}$$~—~DICTDELGETREF~(k~D~n~–~D_0~c~−1~or~D~0)$$, similar to $$DICTDELGET$$, but with LDREF; ENDS applied to $$x$$ on success, so that the value returned $$c$$ is a Cell.
* $${F464}$$~—~DICTIDELGET~(i~D~n~–~D_0~x~−1~or~D~0)$$, a variant of primitive $$DICTDELGET$$ with signed n-bit integer $$i$$ as a key.
* $${F465}$$~—~DICTIDELGETREF~(i~D~n~–~D_0~c~−1~or~D~0)$$, a variant of primitive $$DICTIDELGET$$ returning a Cell instead of a Slice.
* $${F466}$$~—~DICTUDELGET~(i~D~n~–~D_0~x~−1~or~D~0)$$, a variant of primitive $$DICTDELGET$$ with unsigned n-bit integer $$i$$ as a key.
* $${F467}$$~—~DICTUDELGETREF~(i~D~n~–~D_0~c~−1~or~D~0)$$, a variant of primitive $$DICTUDELGET$$ returning a Cell instead of a Slice.

### A.10.7. "Maybe reference" dictionary operations.

The following operations assume that a dictionary is used to store values $$c^{\text {? }}$$ of type $$C_{e l l}$$ ? ("Maybe Cell"), which can be used in particular to store dictionaries as values in other dictionaries. The representation is as follows: if $$c^{\text {? }}$$ is a Cell, it is stored as a value with no data bits and exactly one reference to this $$C$$ ell. If $$c^{?}$$ is $$N u l l$$, then the corresponding key must be absent from the dictionary altogether.

* $${F469}$$~—~DICTGETOPTREF~(k~D~n~–~c?)$$, a variant of $$DICTGETREF$$ that returns Null instead of the value $$c?$$ if the key $$k$$ is absent from dictionary $$D$$.
* $${F46A}$$~—~DICTIGETOPTREF~(i~D~n~–~c?)$$, similar to $$DICTGETOPTREF$$, but with the key given by signed n-bit Integer $$i$$. If the key $$i$$ is out of range, also returns Null.
* $${F46B}$$~—~DICTUGETOPTREF~(i~D~n~–~c?)$$, similar to $$DICTGETOPTREF$$, but with the key given by unsigned n-bit Integer $$i$$.
* $${F46D}$$~—~DICTSETGETOPTREF~(c?~k~D~n~–~D_0~c˜?)$$, a variant of both $$DICTGETOPTREF$$ and $$DICTSETGETREF$$ that sets the value corresponding to key $$k$$ in dictionary $$D$$ to $$c?$$ (if $$c?$$ is Null, then the key is deleted instead), and returns the old value $$c˜?$$ (if the key $$k$$ was absent before, returns Null instead).
* $${F46E}$$~—~DICTISETGETOPTREF~(c?~i~D~n~–~D_0~c˜?)$$, similar to primitive $$DICTSETGETOPTREF$$, but using signed n-bit Integer $$i$$ as a key. If $$i$$ does not fit into n bits, throws a range checking exception.
* $${F46F}$$~—~DICTUSETGETOPTREF~(c?~i~D~n~–~D_0~c˜?)$$, similar to primitive $$DICTSETGETOPTREF$$, but using unsigned n-bit Integer $$i$$ as a key.

### A.10.8. Prefix code dictionary operations.

These are some basic operations for constructing prefix code dictionaries (cf. [$$\mathbf{3.4.2}$$](cells-memory-and-persistent-storage.md#3.4.2.-serialization-of-prefix-codes.)). The primary application for prefix code dictionaries is deserializing TL-B serialized data structures, or, more generally, parsing prefix codes. Therefore, most prefix code dictionaries will be constant and created at compile time, not by the following primitives.

Some GET operations for prefix code dictionaries may be found in [$$\mathbf{A.10.11}$$](a-instructions-and-opcodes.md#a.10.11.-special-get-dictionary-and-prefix-code-dictionary-operations-and-constant-dictionaries.) Other prefix code dictionary operations include:

* $${F470}$$~—~PFXDICTSET~(x~k~D~n~–~D_0~−1~or~D~0)$$.
* $${F471}$$~—~PFXDICTREPLACE~(x~k~D~n~–~D_0~−1~or~D~0)$$.
* $${F472}$$~—~PFXDICTADD~(x~k~D~n~–~D_0~−1~or~D~0)$$.
* $${F473}$$~—~PFXDICTDEL~(k~D~n~–~D_0~−1~or~D~0)$$.

These primitives are completely similar to their non-prefix code counterparts $$DICTSET$$ etc (cf. [$$\mathbf{A.10.4}$$](a-instructions-and-opcodes.md#a.10.4.-set-replace-add-dictionary-operations.)), with the obvious difference that even a SET may fail in a prefix code dictionary, so a success flag must be returned by $$PFXDICTSET$$ as well.

### A.10.9. Variants of GetNext and GetPreV operations.

* $${F474}$$~—~DICTGETNEXT~(k~D~n~–~x_0~k_0~−1~or~0)$$, computes the minimal key $$k_0$$ in dictionary $$D$$ that is lexicographically greater than $$k$$, and returns $$k_0$$ (represented by a Slice) along with associated value $$x_0$$ (also represented by a Slice).
* $${F475}$$~—~DICTGETNEXTEQ~(k~D~n~–~x_0~k_0~−1~or~0)$$, similar to $$DICTGETNEXT$$, but computes the minimal key $$k_0$$ that is lexicographically greater than or equal to $$k$$.
* $${F476}$$~—~DICTGETPREV~(k~D~n~–~x_0~k_0~−1~or~0)$$, similar to $$DICTGETNEXT$$, but computes the maximal key $$k_0$$ lexicographically smaller than $$k$$.
* $${F477}$$~—~DICTGETPREVEQ~(k~D~n~–~x_0~k_0~−1~or~0)$$, similar to DICTGETPREV, but computes the maximal key $$k_0$$ lexicographically smaller than or equal to $$k$$.
* $${F478}$$~—~DICTIGETNEXT~(i~D~n~–~x_0~i_0~−1~or~0)$$, similar to $$DICTGETNEXT$$, but interprets all keys in dictionary $$D$$ as big-endian signed n-bit integers, and computes the minimal key $$i_0$$ that is larger than Integer $$i$$ (which does not necessarily fit into n bits).
* $${F479}$$~—~DICTIGETNEXTEQ~(i~D~n~–~x_0~i_0~−1~or~0)$$.
* $${F47A}$$~—~DICTIGETPREV~(i~D~n~–~x_0~i_0~−1~or~0)$$.
* $${F47B}$$~—~DICTIGETPREVEQ~(i~D~n~–~x_0~i_0~−1~or~0)$$.
* $${F47C}$$~—~DICTUGETNEXT~(i~D~n~–~x_0~i_0~−1~or~0)$$, similar to $$DICTGETNEXT$$, but interprets all keys in dictionary $$D$$ as big-endian unsigned n-bit integers, and computes the minimal key $$i_0$$ that is larger than Integer $$i$$ (which does not necessarily fit into n bits, and is not necessarily non-negative).
* $${F47D}$$~—~DICTUGETNEXTEQ~(i~D~n~–~x_0~i_0~−1~or~0)$$.
* $${F47E}$$~—~DICTUGETPREV~(i~D~n~–~x_0~i_0~−1~or~0)$$.
* $${F47F}$$~—~DICTUGETPREVEQ~(i~D~n~–~x_0~i_0~−1~or~0)$$.

### A.10.10. GetMin, GetMax, RemoveMin, RemoveMax operations.

* $${F482}$$~—~DICTMIN~(D~n~–~x~k~−1~or~0)$$, computes the minimal key $$k$$ (represented by a Slice with n data bits) in dictionary $$D$$, and returns $$k$$ along with the associated value $$x$$.
* $${F483}$$~—~DICTMINREF~(D~n~–~c~k~−1~or~0)$$, similar to $$DICTMIN$$, but returns the only reference in the value as a Cell $$c$$.
* $${F484}$$~—~DICTIMIN~(D~n~–~x~i~−1~or~0)$$, somewhat similar to $$DICTMIN$$, but computes the minimal key $$i$$ under the assumption that all keys are big-endian signed n-bit integers. Notice that the key and value returned may differ from those computed by $$DICTMIN$$ and $$DICTUMIN$$.
* $${F485}$$~—~DICTIMINREF~(D~n~–~c~i~−1~or~0)$$.
* $${F486}$$~—~DICTUMIN~(D~n~–~x~i~−1~or~0)$$, similar to $$DICTMIN$$, but returns the key as an unsigned n-bit Integer $$i$$.
* $${F487}$$~—~DICTUMINREF~(D~n~–~c~i~−1~or~0)$$.
* $${F48A}$$~—~DICTMAX~(D~n~–~x~k~−1~or~0)$$, computes the maximal key $$k$$ (represented by a Slice with n data bits) in dictionary $$D$$, and returns $$k$$ along with the associated value $$x$$.
* $${F48B}$$~—~DICTMAXREF~(D~n~–~c~k~−1~or~0)$$.
* $${F48C}$$~—~DICTIMAX~(D~n~–~x~i~−1~or~0)$$.
* $${F48D}$$~—~DICTIMAXREF~(D~n~–~c~i~−1~or~0)$$.
* $${F48E}$$~—~DICTUMAX~(D~n~–~x~i~−1~or~0)$$.
* $${F48F}$$~—~DICTUMAXREF~(D~n~–~c~i~−1~or~0)$$.
* $${F492}$$~—~DICTREMMIN~(D~n~–~D_0~x~k~−1~or~D~0)$$, computes the minimal key $$k$$ (represented by a Slice with n data bits) in dictionary $$D$$, removes $$k$$ from the dictionary, and returns $$k$$ along with the associated value $$x$$ and the modified dictionary $$D_0$$.
* $${F493}$$~—~DICTREMMINREF~(D~n~–~D_0~c~k~−1~or~D~0)$$, similar to $$DICTREMMIN$$, but returns the only reference in the value as a Cell $$c$$.
* $${F494}$$~—~DICTIREMMIN~(D~n~–~D_0~x~i~−1~or~D~0)$$, somewhat similar to $$DICTREMMIN$$, but computes the minimal key $$i$$ under the assumption that all keys are big-endian signed n-bit integers. Notice that the key and value returned may differ from those computed by $$DICTREMMIN$$ and $$DICTUREMMIN$$.
* $${F495}$$~—~DICTIREMMINREF~(D~n~–~D_0~c~i~−1~or~D~0)$$.
* $${F496}$$~—~DICTUREMMIN~(D~n~–~D_0~x~i~−1~or~D~0)$$, similar to $$DICTREMMIN$$, but returns the key as an unsigned n-bit Integer $$i$$.
* $${F497}$$~—~DICTUREMMINREF~(D~n~–~D_0~c~i~−1~or~D~0)$$.
* $${F49A}$$~—~DICTREMMAX~(D~n~–~D_0~x~k~−1~or~D~0)$$, computes the maximal key $$k$$ (represented by a Slice with n data bits) in dictionary $$D$$, removes $$k$$ from the dictionary, and returns $$k$$ along with the associated value $$x$$ and the modified dictionary $$D_0$$.
* $${F49B}$$~—~DICTREMMAXREF~(D~n~–~D_0~c~k~−1~or~D~0)$$.
* $${F49C}$$~—~DICTIREMMAX~(D~n~–~D_0~x~i~−1~or~D~0)$$.
* $${F49D}$$~—~DICTIREMMAXREF~(D~n~–~D_0~c~i~−1~or~D~0)$$.
* $${F49E}$$~—~DICTUREMMAX~(D~n~–~D_0~x~i~−1~or~D~0)$$.
* $${F49F}$$~—~DICTUREMMAXREF~(D~n~–~D_0~c~i~−1~or~D~0)$$.

### A.10.11. Special GET dictionary and prefix code dictionary operations, and constant dictionaries.

* $${F4A0}$$~—~DICTIGETJMP~(i~D~n~–~)$$, similar to $$DICTIGET$$ but with the value BLESSed into a continuation followed by a $$JMPX$$ to it on success. On failure, does nothing.
* $${F4A1}$$~—~DICTUGETJMP~(i~D~n~–~)$$, similar to $$DICTIGETJMP$$, but uses DICTUGET instead of $$DICTIGET$$.
* $${F4A2}$$~—~DICTIGETEXEC~(i~D~n~–~)$$, like $$DICTIGETJMP$$ but with EXECUTE instead of JMPX.
* $${F4A3}$$~—~DICTUGETEXEC~(i~D~n~–~)$$, similar to $$DICTUGETJMP$$, but with $$EXECUTE$$ instead of $$JMPX$$.
* $${F4A6}_n$$~—~DICTPUSHCONST~n~(–~D~n)$$, pushes a non-empty constant dictionary $$ D $$ with key length $$ 0 \leq n \leq 1023 $$. The dictionary is created from the first remaining references of the current continuation.
* $${F4A8}$$~—~PFXDICTGETQ~(s~D~n~–~s_0~x~s_{00}~−1~or~s_0)$$, looks up the unique prefix of Slice $$ s $$ in the prefix code dictionary and returns the prefix and corresponding value.
* $${F4A9}$$~—~PFXDICTGET~(s~D~n~–~s_0~x~s_{00})$$, similar to $$PFXDICTGETQ$$ but throws a cell deserialization failure exception on failure.
* $${F4AA}$$~—~PFXDICTGETJMP~(s~D~n~–~s_0~s_{00}~or~s)$$, similar to $$PFXDICTGETQ$$, but BLESSes the value into a Continuation and jumps to it. On failure, returns $$ s $$ unchanged.
* $${F4AB}$$~—~PFXDICTGETEXEC~(s~D~n~–~s_0~s_{00})$$, similar to $$PFXDICTGETJMP$$ , but EXECutes the found continuation. On failure, throws an exception.
* $${F4AE}_n$$~—~PFXDICTCONSTGETJMP~n~or~PFXDICTSWITCH~n~(s~–~s_0~s_{00}~or~s)$$, combines $$DICTPUSHCONST$$ $$ n $$ with $$PFXDICTGETJMP$$.
* $${F4BC}$$~—~DICTIGETJMPZ~(i~D~n~–~i~or~nothing)$$, a variant of $$DICTIGETJMP$$ returning index $$ i $$ on failure.
* $${F4BD}$$~—~DICTUGETJMPZ~(i~D~n~–~i~or~nothing)$$, a variant of $$DICTUGETJMP$$ returning index $$ i $$ on failure.
* $${F4BE}$$~—~DICTIGETEXECZ~(i~D~n~–~i~or~nothing)$$, a variant of $$DICTIGETEXEC$$ returning index $$ i $$ on failure.
* $${F4BF}$$~—~DICTUGETEXECZ~(i~D~n~–~i~or~nothing)$$, a variant of $$DICTUGETEXEC$$ returning index $$ i $$ on failure.

### A.10.12. SuBDiCT dictionary operations.

* $${F4B1}$$~—~SUBDICTGET~(k~l~D~n~–~D_0)$$, constructs a subdictionary with all keys starting with prefix $$ k $$ in dictionary $$ D $$.
* $${F4B2}$$~—~SUBDICTIGET~(x~l~D~n~–~D_0)$$, variant of $$SUBDICTGET$$ with the prefix represented by a signed big-endian $$ l $$-bit Integer $$ x $$.
* $${F4B3}$$~—~SUBDICTUGET~(x~l~D~n~–~D_0)$$, variant of $$SUBDICTGET$$ with the prefix represented by an unsigned big-endian $$ l $$-bit Integer $$ x $$.
* $${F4B5}$$~—~SUBDICTRPGET~(k~l~D~n~–~D_0)$$, like $$SUBDICTGET$$ but removes the common prefix $$ k $$ from all keys in the new dictionary.
* $${F4B6}$$~—~SUBDICTIRPGET~(x~l~D~n~–~D_0)$$, variant of $$SUBDICTRPGET$$ with the prefix represented by a signed big-endian $$ l $$-bit Integer $$ x $$.
* $${F4B7}$$~—~SUBDICTURPGET~(x~l~D~n~–~D_0)$$, variant of $$SUBDICTRPGET$$ with the prefix represented by an unsigned big-endian $$ l $$-bit Integer $$ x $$.

## A.11 Application-specific primitives

Opcode range F8... FB is reserved for the application-specific primitives. When TVM is used to execute TVM smart contracts, these applicationspecific primitives are in fact TVM Blockchain-specific.

A.11.1. External actions and access to blockchain configuration data. Some of the primitives listed below pretend to produce some externally visible actions, such as sending a message to another smart contract. In fact, the execution of a smart contract in TVM never has any effect apart from a modification of the TVM state. All external actions are collected into a linked list stored in special register c5 ("output actions"). Additionally, some primitives use the data kept in the first component of the Tuple stored in c7 ("root of temporary data", cf. $$\mathbf{1 . 3 . 2}$$ ). Smart contracts are free to modify any other data kept in the cell $$c 7$$, provided the first reference remains intact (otherwise some application-specific primitives would be likely to throw exceptions when invoked).

Most of the primitives listed below use 16-bit opcodes.

### A.11.2. Gas-related primitives.

Of the following primitives, only the first two are "pure" in the sense that they do not use c5 or c7.

* $${F800}$$~—~ACCEPT$$, sets the current gas limit to its maximum allowed value and resets the gas credit to zero.
* $${F801}$$~—~SETGASLIMIT~(g~–~)$$, sets the current gas limit to the minimum of $$ g $$ and $$ gm $$, and resets the gas credit to zero.
* $${F802}$$~—~BUYGAS~(x~–~)$$, computes the gas amount that can be bought for $$ x $$ nanograms and sets the gas limit accordingly.
* $${F804}$$~—~GRAMTOGAS~(x~–~g)$$, computes the gas amount that can be bought for $$ x $$ nanograms.
* $${F805}$$~—~GASTOGRAM~(g~–~x)$$, computes the price of $$ g $$ gas in nanograms.
* $$F806-F80E~-$$ Reserved for gas-related primitives.
* $$F80F~-~COMMIT$$ ($$-$$), commits the current state of registers $$c4$$ ("persistent data") and $$c5$$ ("actions") so that the current execution is considered "successful" with the saved values even if an exception is thrown later.


### A.11.3. Pseudo-random number generator primitives.

The pseudorandom number generator uses the random seed (parameter $$\# 6$$, cf. A.11.4), an unsigned 256-bit Integer, and (sometimes) other data kept in c7. The initial value of the random seed before a smart contract is executed in TVM Blockchain is a hash of the smart contract address and the global block random seed. If there are several runs of the same smart contract inside a block, then all of these runs will have the same random seed. This can be fixed, for example, by running LTIME; ADDRAND before using the pseudorandom number generator for the first time.

* $${F810}$$~—~RANDU256~(–~x)$$, generates a pseudo-random unsigned 256-bit Integer $$ x $$.
* $${F811}$$~—~RAND~(y~–~z)$$, generates a pseudo-random integer $$ z $$ in the range $$ 0 $$ to $$ y - 1 $$.
* $${F814}$$~—~SETRAND~(x~–~)$$, sets the random seed to unsigned 256-bit Integer $$ x $$.
* $${F815}$$~—~ADDRAND~(x~–~)$$, mixes unsigned 256-bit Integer $$ x $$ into the random seed.
* $$F810–F81F~—$$ Reserved for pseudo-random number generator primitives.

### A.11.4. Configuration primitives.

The following primitives read configuration data provided in the Tuple stored in the first component of the Tuple at c7. Whenever TVM is invoked for executing TVM Blockchain smart contracts, this Tuple is initialized by a SmartContractInfo structure; configuration primitives assume that it has remained intact.

* $${F82i}$$~—~GETPARAM~i~(–~x)$$, returns the $$ i $$-th parameter from the Tuple provided at $$ c7 $$.
* $${F823}$$~—~NOW~(–~x)$$, returns the current Unix time as an Integer.
* $${F824}$$~—~BLOCKLT~(–~x)$$, returns the starting logical time of the current block.
* $${F825}$$~—~LTIME~(–~x)$$, returns the logical time of the current transaction.
* $${F826}$$~—~RANDSEED~(–~x)$$, returns the current random seed as an unsigned 256-bit Integer.
* $${F827}$$~—~BALANCE~(–~t)$$, returns the remaining balance of the smart contract.
* $${F828}$$~—~MYADDR~(–~s)$$, returns the internal address of the current smart contract.
* $${F829}$$~—~CONFIGROOT~(–~D)$$, returns the Maybe Cell $$ D $$ with the current global configuration dictionary.
* $${F830}$$~—~CONFIGDICT~(–~D~32)$$, returns the global configuration dictionary along with its key length.
* $${F832}$$~—~CONFIGPARAM~(i~–~c~−1~or~0)$$, returns the value of the global configuration parameter with integer index $$ i $$.
* $${F833}$$~—~CONFIGOPTPARAM~(i~–~c?)$$, returns the value of the global configuration parameter with integer index $$ i $$.

### A.11.5. Global variable primitives. 

The "global variables" may be helpful in implementing some high-level smart-contract languages. They are in fact stored as components of the Tuple at c7: the $$k$$-th global variable simply is the $$k$$-th component of this Tuple, for $$1 \leq k \leq 254$$. By convention, the 0 -th component is used for the "configuration parameters" of A.11.4, so it is not available as a global variable.

* $${F840}$$~—~GETGLOBVAR~(k~–~x)$$, returns the $$ k $$-th global variable.
* $${F85_k}$$~—~GETGLOB~k~(–~x)$$, returns the $$ k $$-th global variable.
* $${F860}$$~—~SETGLOBVAR~(x~k~–~)$$, assigns $$ x $$ to the $$ k $$-th global variable.
* $${F87_k}$$~—~SETGLOB~k~(x~–~)$$, assigns $$ x $$ to the $$ k $$-th global variable.

### A.11.6. Hashing and cryptography primitives.

* $${F900}$$~—~HASHCU~(c~–~x)$$, computes the representation hash of a Cell $$ c $$ and returns it as a 256-bit unsigned integer $$ x $$.
* $${F901}$$~—~HASHSU~(s~–~x)$$, computes the hash of a Slice $$ s $$ and returns it as a 256-bit unsigned integer $$ x $$.
* $${F902}$$~—~SHA256U~(s~–~x)$$, computes sha256 of the data bits of Slice $$ s $$.
* $${F910}$$~—~CHKSIGNU~(h~s~k~–~?)$$, checks the Ed25519-signature $$ s $$ of a hash $$ h $$ using public key $$ k $$.
* $${F911}$$~—~CHKSIGNS~(d~s~k~–~?)$$, checks whether $$ s $$ is a valid Ed25519-signature of the data portion of Slice $$ d $$ using public key $$ k $$.
* $$F912-F93F~-$$ Reserved for hashing and cryptography primitives.

### A.11.7. Miscellaneous primitives.

* $${F940}$$ — $$CDATASIZEQ~(c~n~–~x~y~z~−1~or~0)$$, recursively computes the count of distinct cells $$x$$, data bits $$y$$, and cell references $$z$$ in the dag rooted at Cell $$c$$, effectively returning the total storage used by this dag taking into account the identification of equal cells. The values of $$x$$, $$y$$, and $$z$$ are computed by a depth-first traversal of this dag, with a hash table of visited cell hashes used to prevent visits of already-visited cells. The total count of visited cells $$x$$ cannot exceed non-negative Integer $$n$$; otherwise the computation is aborted before visiting the (n + 1)-st cell and a zero is returned to indicate failure. If $$c$$ is Null, returns $$x = y = z = 0$$.
* $${F941}$$ — $$CDATASIZE~(c~n~–~x~y~z)$$, a non-quiet version of $$CDATASIZEQ$$ that throws a cell overflow exception (8) on failure.
* $${F942}$$ — $$SDATASIZEQ~(s~n~–~x~y~z~−1~or~0)$$, similar to $$CDATASIZEQ$$, but accepting a Slice $$s$$ instead of a Cell. The returned value of $$x$$ does not take into account the cell that contains the slice $$s$$ itself; however, the data bits and the cell references of $$s$$ are accounted for in $$y$$ and $$z$$.
* $${F943}$$ — $$SDATASIZE~(s~n~–~x~y~z)$$, a non-quiet version of $$SDATASIZEQ$$ that throws a cell overflow exception (8) on failure.
* $${F944–F97F}$$ — Reserved for miscellaneous TON-specific primitives that do not fall into any other specific category.

### A.11.8. Currency manipulation primitives.

* $${FA00}$$ — $$LDGRAMS~or~LDVARUINT16~(s~–~x~s0)$$, loads (deserializes) a Gram or VarUInteger 16 amount from CellSlice $$s$$, and returns the amount as Integer $$x$$ along with the remainder $$s0$$ of $$s$$. The expected serialization of $$x$$ consists of a 4-bit unsigned big-endian integer $$l$$, followed by an $$8l$$-bit unsigned big-endian representation of $$x$$. The net effect is approximately equivalent to $$LDU~4;~SWAP;~LSHIFT~3;~LDUX$$.
* $${FA01}$$ — $$LDVARINT16~(s~–~x~s0)$$, similar to $$LDVARUINT16$$, but loads a signed Integer $$x$$. Approximately equivalent to $$LDU~4;~SWAP;~LSHIFT~3;~LDIX$$.
* $${FA02}$$ — $$STGRAMS~or~STVARUINT16~(b~x~–~b0)$$, stores (serializes) an Integer $$x$$ in the range $$0 \ldots 2^{120−1}$$ into Builder $$b$$, and returns the resulting Builder $$b0$$. The serialization of $$x$$ consists of a 4-bit unsigned big-endian integer $$l$$, which is the smallest integer $$l \geq 0$$, such that $$x < 2^{8l}$$, followed by an $$8l$$-bit unsigned big-endian representation of $$x$$. If $$x$$ does not belong to the supported range, a range check exception is thrown.
* $${FA03}$$ — $$STVARINT16~(b~x~–~b0)$$, similar to $$STVARUINT16$$, but serializes a signed Integer $$x$$ in the range $$−2^{119} \ldots 2^{119} − 1$$.
* $${FA04}$$ — $$LDVARUINT32~(s~–~x~s0)$$, loads (deserializes) a VarUInteger 32 from CellSlice $$s$$, and returns the deserialized value as an Integer $$0 \leq x < 2^{248}$$. The expected serialization of $$x$$ consists of a 5-bit unsigned big-endian integer $$l$$, followed by an $$8l$$-bit unsigned big-endian representation of $$x$$. The net effect is approximately equivalent to $$LDU~5;~SWAP;~SHIFT~3;~LDUX$$.
* $${FA05}$$ — $$LDVARINT32~(s~–~x~s0)$$, deserializes a VarInteger 32 from CellSlice $$s$$, and returns the deserialized value as an Integer $$−2^{247} \leq x < 2^{247}$$.
* $${FA06}$$ — $$STVARUINT32~(b~x~–~b0)$$, serializes an Integer $$0 \leq x < 2^{248}$$ as a VarUInteger 32.
* $${FA07}$$ — $$STVARINT32~(b~x~–~b0)$$, serializes an Integer $$−2^{247} \leq x < 2^{247}$$ as a VarInteger 32.
* $${FA08–FA1F}$$ — Reserved for currency manipulation primitives.

### A.11.9. Message and address manipulation primitives.

The message and address manipulation primitives listed below serialize and deserialize values according to the following TL-B scheme (cf. [$$\mathbf{3.3.4}$$](a-instructions-and-opcodes.md#3.3.4.-brief-explanation-of-tl-b-schemes.)):

```
addr_none$00 = MsgAddressExt;
addr_extern$01 len: (## 8) external_address: (bits len)
MsgAddressExt;
anycast_info$_ depth: (#<= 30) { depth >= 1 }
rewrite_pfx: (bits depth) = Anycast;
addr_std$10 anycast: (Maybe Anycast)
workchain_id:int8 address:bits256 = MsgAddressInt;
addr_var$ll anycast: (Maybe Anycast) addr_len: (## 9)
workchain_id:int32 address: (bits addr_len) = MsgAddressInt;
_ _:MsgAddressInt = MsgAddress;
_ _:MsgAddressExt = MsgAddress;

int_msg_info$0 ihr_disabled:Bool bounce:Bool bounced:Bool
src:MsgAddress dest:MsgAddressInt
value:CurrencyCollection ihr_fee:Grams fwd_fee:Grams
created_lt:uint64 created_at:uint32 = CommonMsgInfoRelaxed;
ext_out_msg_info$1l src:MsgAddress dest:MsgAddressExt
created_lt:uint64 created_at:uint32 = CommonMsgInfoRelaxed;

```

A deserialized MsgAddress is represented by a Tuple $$$t$$$ as follows:

* addr\_none is represented by $$$t=(0)$$, i.e., a Tuple containing exactly one Integer equal to zero.
* addr\_extern is represented by $$$t=(1, s)$$, where Slice $$s$$ contains the field external\_address. In other words, $$$t$$$ is a pair (a Tuple consisting of two entries), containing an Integer equal to one and Slice s.
* addr\_std is represented by $$$t=(2, u, x, s)$$, where $$u$$ is either a $$N u l l$$ (if anycast is absent) or a Slice $$s^{\prime}$$ containing rewrite\_pfx (if anycast is present). Next, Integer $$x$$ is the workchain\_id, and Slice s contains the address. - addr\_var is represented by $$$t=(3, u, x, s)$$, where $$u, x$$, and $$s$$ have the same meaning as for addr\_std.

The following primitives, which use the above conventions, are defined:

* $${FA40}$$ — $$LDMSGADDR~(s~–~s0~s00)$$, loads from CellSlice $$s$$ the only prefix that is a valid MsgAddress, and returns both this prefix $$s0$$ and the remainder $$s00$$ of $$s$$ as CellSlices.
* $${FA41}$$ — $$LDMSGADDRQ~(s~–~s0~s00~−1~or~s~0)$$, a quiet version of $$LDMSGADDR$$: on success, pushes an extra −1; on failure, pushes the original $$s$$ and a zero.
* $${FA42}$$ — $$PARSEMSGADDR~(s~–~t)$$, decomposes CellSlice $$s$$ containing a valid MsgAddress into a Tuple $$t$$ with separate fields of this MsgAddress. If $$s$$ is not a valid MsgAddress, a cell deserialization exception is thrown.
* $${FA43}$$ — $$PARSEMSGADDRQ~(s~–~t~−1~or~0)$$, a quiet version of $$PARSEMSGADDR$$: returns a zero on error instead of throwing an exception.
* $${FA44}$$ — $$REWRITESTDADDR~(s~–~x~y)$$, parses CellSlice $$s$$ containing a valid MsgAddressInt (usually a msg_addr_std), applies rewriting from the anycast (if present) to the same-length prefix of the address, and returns both the workchain $$x$$ and the 256-bit address $$y$$ as Integers. If the address is not 256-bit, or if $$s$$ is not a valid serialization of MsgAddressInt, throws a cell deserialization exception.
* $${FA45}$$ — $$REWRITESTDADDRQ~(s~–~x~y~−1~or~0)$$, a quiet version of primitive $$REWRITESTDADDR$$.
* $${FA46}$$ — $$REWRITEVARADDR~(s~–~x~s0)$$, a variant of $$REWRITESTDADDR$$ that returns the (rewritten) address as a Slice $$s$$, even if it is not exactly 256 bit long (represented by a msg_addr_var).
* $${FA47}$$ — $$REWRITEVARADDRQ~(s~–~x~s0~−1~or~0)$$, a quiet version of primitive $$REWRITEVARADDR$$.
* $${FA48–FA5F}$$ — Reserved for message and address manipulation primitives.

### A.11.10. Outbound message and output action primitives.

* $${FB00}$$ — $$SENDRAWMSG~(c~x~–~)$$, sends a raw message contained in Cell $$c$$, which should contain a correctly serialized object Message X, with the only exception that the source address is allowed to have dummy value addr_none (to be automatically replaced with the current smart-contract address), and ihr_fee, fwd_fee, created_lt and created_at fields can have arbitrary values (to be rewritten with correct values during the action phase of the current transaction). Integer parameter $$x$$ contains the flags. Currently $$x = 0$$ is used for ordinary messages; $$x = 128$$ is used for messages that are to carry all the remaining balance of the current smart contract (instead of the value originally indicated in the message); $$x = 64$$ is used for messages that carry all the remaining value of the inbound message in addition to the value initially indicated in the new message (if bit 0 is not set, the gas fees are deducted from this amount); $$x0 = x + 1$$ means that the sender wants to pay transfer fees separately; $$x0 = x + 2$$ means that any errors arising while processing this message during the action phase should be ignored. Finally, $$x0 = x + 32$$ means that the current account must be destroyed if its resulting balance is zero. This flag is usually employed together with +128.
* $${FB02}$$ — $$RAWRESERVE~(x~y~–~)$$, creates an output action which would reserve exactly $$x$$ nanograms (if $$y = 0$$), at most $$x$$ nanograms (if $$y = 2$$), or all but $$x$$ nanograms (if $$y = 1$$ or $$y = 3$$), from the remaining balance of the account. It is roughly equivalent to creating an outbound message carrying $$x$$ nanograms (or $$b − x$$ nanograms, where $$b$$ is the remaining balance) to oneself, so that the subsequent output actions would not be able to spend more money than the remainder. Bit +2 in $$y$$ means that the external action does not fail if the specified amount cannot be reserved; instead, all remaining balance is reserved. Bit +8 in $$y$$ means $$x ← −x$$ before performing any further actions. Bit +4 in $$y$$ means that $$x$$ is increased by the original balance of the current account (before the compute phase), including all extra currencies, before performing any other checks and actions. Currently $$x$$ must be a non-negative integer, and $$y$$ must be in the range $$0 \ldots 15$$.
* $${FB03}$$ — $$RAWRESERVEX~(x~D~y~–~)$$, similar to $$RAWRESERVE$$, but also accepts a dictionary $$D$$ (represented by a Cell or Null) with extra currencies. In this way currencies other than Grams can be reserved.
* $${FB04}$$ — $$SETCODE~(c~–~)$$, creates an output action that would change this smart contract code to that given by Cell $$c$$. Notice that this change will take effect only after the successful termination of the current run of the smart contract.
* $${FB06}$$ — $$SETLIBCODE~(c~x~–~)$$, creates an output action that would modify the collection of this smart contract libraries by adding or removing library with code given in Cell $$c$$. If $$x = 0$$, the library is actually removed if it was previously present in the collection (if not, this action does nothing). If $$x = 1$$, the library is added as a private library, and if $$x = 2$$, the library is added as a public library (and becomes available to all smart contracts if the current smart contract resides in the masterchain); if the library was present in the collection before, its public/private status is changed according to $$x$$. Values of $$x$$ other than $$0 \ldots 2$$ are invalid.
* $${FB07}$$ — $$CHANGELIB~(h~x~–~)$$, creates an output action similarly to $$SETLIBCODE$$, but instead of the library code accepts its hash as an unsigned 256-bit integer $$h$$. If $$x 6= 0$$ and the library with hash $$h$$ is absent from the library collection of this smart contract, this output action will fail.
* $${FB08–FB3F}$$ — Reserved for output action primitives.

## A.12 Debug primitives

Opcodes beginning with $$\mathrm{FE}$$ are reserved for the debug primitives. These primitives have known fixed operation length, and behave as (multibyte) NOP operations. In particular, they never change the stack contents, and never throw exceptions, unless there are not enough bits to completely decode the opcode. However, when invoked in a TVM instance with debug mode enabled, these primitives can produce specific output into the text debug log of the TVM instance, never affecting the TVM state (so that from the perspective of TVM the behavior of debug primitives in debug mode is exactly the same). For instance, a debug primitive might dump all or some of the values near the top of the stack, display the current state of TVM and so on.

### A.12.1. Debug primitives as multibyte NOPs.

* $${FEnn}$$ — $$DEBUG~nn$$, for $$0 \leq nn < 240$$, is a two-byte NOP.
* $${FEFnssss}$$ — $$DEBUGSTR~ssss$$, for $$0 \leq n < 16$$, is an $$(n + 3)$$-byte NOP, with the $$(n + 1)$$-byte “contents string” $$ssss$$ skipped as well.

### A.12.2. Debug primitives as operations without side-effect. 

Next we describe the debug primitives that might (and actually are) implemented in a version of TVM. Notice that another TVM implementation is free to use these codes for other debug purposes, or treat them as multibyte NOPs. Whenever these primitives need some arguments from the stack, they inspect these arguments, but leave them intact in the stack. If there are insufficient values in the stack, or they have incorrect types, debug primitives may output error messages into the debug log, or behave as NOPs, but they cannot throw exceptions.

* $${FE00}$$ — $$DUMPSTK$$, dumps the stack (at most the top 255 values) and shows the total stack depth.
* $${FE0n}$$ — $$DUMPSTKTOP~n$$, $$1 \leq n < 15$$, dumps the top $$n$$ values from the stack, starting from the deepest of them. If there are $$d < n$$ values available, dumps only $$d$$ values.
* $${FE10}$$ — $$HEXDUMP$$, dumps $$s0$$ in hexadecimal form, be it a Slice or an Integer.
* $${FE11}$$ — $$HEXPRINT$$, similar to $$HEXDUMP$$, except the hexadecimal representation of $$s0$$ is not immediately output, but rather concatenated to an output text buffer.
* $${FE12}$$ — $$BINDUMP$$, dumps $$s0$$ in binary form, similarly to $$HEXDUMP$$.
* $${FE13}$$ — $$BINPRINT$$, outputs the binary representation of $$s0$$ to a text buffer.
* $${FE14}$$ — $$STRDUMP$$, dumps the Slice at $$s0$$ as an UTF-8 string.
* $${FE15}$$ — $$STRPRINT$$, similar to $$STRDUMP$$, but outputs the string into a text buffer (without carriage return).
* $${FE1E}$$ — $$DEBUGOFF$$, disables all debug output until it is re-enabled by a $$DEBUGON$$. More precisely, this primitive increases an internal counter, which disables all debug operations (except $$DEBUGOFF$$ and $$DEBUGON$$) when strictly positive.
* $${FE1F}$$ — $$DEBUGON$$, enables debug output (in a debug version of TVM).
* $${FE2n}$$ — $$DUMP~s(n)$$, $$0 \leq n < 15$$, dumps $$s(n)$$.
* $${FE3n}$$ — $$PRINT~s(n)$$, $$0 \leq n < 15$$, concatenates the text representation of $$s(n)$$ (without any leading or trailing spaces or carriage returns) to a text buffer which will be output before the output of any other debug operation.
* $${FEC0–FEEF}$$ — Use these opcodes for custom/experimental debug operations.
* $${FEFnssss}$$ — $$DUMPTOSFMT~ssss$$, dumps $$s0$$ formatted according to the $$(n + 1)$$-byte string $$ssss$$. This string might contain (a prefix of) the name of a TL-B type supported by the debugger. If the string begins with a zero byte, simply outputs it (without the first byte) into the debug log. If the string begins with a byte equal to one, concatenates it to a buffer, which will be output before the output of any other debug operation (effectively outputs a string without a carriage return).
* $${FEFn00ssss}$$ — $$LOGSTR~ssss$$, string $$ssss$$ is $$n$$ bytes long.
* $${FEF000}$$ — $$LOGFLUSH$$, flushes all pending debug output from the buffer into the debug log.
* $${FEFn01ssss}$$ — $$PRINTSTR~ssss$$, string $$ssss$$ is $$n$$ bytes long.

### A.13 Codepage primitives

The following primitives, which begin with byte FF, typically are used at the very beginning of a smart contract's code or a library subroutine to select another TVM codepage. Notice that we expect all codepages to contain these primitives with the same codes, otherwise switching back to another codepage might be impossible (cf. [$$\mathbf{5.1.8}$$](codepages-and-instruction-encoding.md#5.1.8.-setting-the-codepage-in-the-code-itself.)).

* $${FFnn}$$ — $$SETCP~nn$$, selects TVM codepage $$0 \leq nn < 240$$. If the codepage is not supported, throws an invalid opcode exception.
* $${FF00}$$ — $$SETCP0$$, selects TVM (test) codepage zero as described in this document.
* $${FFFz}$$ — $$SETCP~z~−~16$$, selects TVM codepage $$z − 16$$ for $$1 \leq z \leq 15$$. Negative codepages $$−13 \ldots − 1$$ are reserved for restricted versions of TVM needed to validate runs of TVM in other codepages as explained in B.2.6. Negative codepage $$−14$$ is reserved for experimental codepages, not necessarily compatible between different TVM implementations, and should be disabled in the production versions of TVM.
* $${FFF0}$$ — $$SETCPX~(c~–~)$$, selects codepage $$c$$ with $$−2^{15} \leq c < 2^{15}$$ passed in the top of the stack.
