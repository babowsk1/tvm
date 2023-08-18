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
* $$0i~-~\text{XCHG}~s(i)~\text{or}~\text{XCHG}~s0,s(i),~\text{interchanges~the~top~of~the~stack~with}~s(i),~1~≤~i~≤~15.$$
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

* $$55ij~-~\text{BLKSWAP}~i+1,j+1,~\text{permutes two blocks}~{s}(j+i+1)~\ldots~{s}(j+1)$$
* $$\text{and}~{s}(j)~\ldots~{s}0,~\text{for}~0 ≤ i, j ≤ 15.$$
* $$\text{Equivalent~to}~\text{REVERSE}~i+1,j+1;~\text{REVERSE}~j+1,0;~\text{REVERSE}~i+j+2,0.$$
* $$5513~-~\text{ROT2~or~2ROT}~(a~b~c~d~e~f~–~c~d~e~f~a~b),~\text{rotates~the~three}$$
* $$\text{topmost~pairs~of~stack~entries.}$$
* $$550i~-~\text{ROLL}~i+1,~\text{rotates~the~top}~i+1~\text{stack~entries.}~\text{Equivalent~to}$$
* $$\text{BLKSWAP}~1,i+1.$$
* $$55i0~-~\text{ROLLREV}~i+1~\text{or}~-ROLL~i+1,~\text{rotates~the~top}~i+1$$
* $$\text{stack~entries~in~the~other~direction.}~\text{Equivalent~to}~\text{BLKSWAP}~i+1,1.$$
* $$56ii~-~\text{PUSH}~{s}(ii)~\text{for}~0 ≤ ii ≤ 255.$$
* $$57ii~-~\text{POP}~{s}(ii)~\text{for}~0 ≤ ii ≤ 255.$$
* $$58~-~\text{ROT}~(a~b~c~–~b~c~a)$$, equivalent to $$\text{BLKSWAP}~1,2~\text{or~to}~\text{XCHG2}~s2,s1$$.
* $$59~-~\text{ROTREV~or}~-ROT~(a~b~c~–~c~a~b),~\text{equivalent~to}~\text{BLKSWAP}~2,1~\text{or~to}$$
* $$\text{XCHG2}~{s}(2),{s}(2).$$
* $$5A~-~\text{SWAP2~or~2SWAP}~(a~b~c~d~–~c~d~a~b),~\text{equivalent~to}~\text{BLKSWAP}~2,2~\text{or}$$
* $$\text{to}~\text{XCHG2}~{s}(3),{s}(2).$$
* $$5B~-~\text{DROP2~or~2DROP}~(a~b~–~),~\text{equivalent~to}~\text{DROP;}~\text{DROP.}$$
* $$5C~-~\text{DUP2~or~2DUP}~(a~b~–~a~b~a~b),~\text{equivalent~to}~\text{PUSH2}~{s}(1),{s}(0).$$
* $$5D~-~\text{OVER2~or~2OVER}~(a~b~c~d~–~a~b~c~d~a~b),~\text{equivalent~to}~\text{PUSH2}~{s}(3),{s}(2).$$
* $$5Eij~-~\text{REVERSE}~i+2,j,~\text{reverses~the~order~of}~{s}(j+i+1)~\ldots~{s}(j)~\text{for}$$
* $$0 ≤ i, j ≤ 15;~\text{equivalent~to~a~sequence~of}~\frac{b}{2}~\text{XCHGs.}$$
* $$5F0i~-~\text{BLKDROP}~i,~\text{equivalent~to}~\text{DROP}~\text{performed}~i~\text{times.}$$
* $$5Fij~-~\text{BLKPUSH}~i,j,~\text{equivalent~to}~\text{PUSH}~{s}(j)~\text{performed}~i~\text{times,}~1 ≤$$
* $$i ≤ 15,~0 ≤ j ≤ 15.$$
* $$60~-~\text{PICK~or~PUSHX,~pops~integer}~i~\text{from~the~stack,~then~performs}~\text{PUSH}~{s}(i).$$
* $$61~-~\text{ROLLX,~pops~integer}~i~\text{from~the~stack,~then~performs}~\text{BLKSWAP}~1,i.$$

## A.3 Tuple, List, and Null primitives

Tuples are ordered collections consisting of at most 255 TVM stack values of arbitrary types (not necessarily the same). Tuple primitives create, modify, and unpack Tuples; they manipulate values of arbitrary types in the process, similarly to the stack primitives. We do not recommend using Tuples of more than 15 elements.

When a Tuple $$t$$ contains elements $$x_{1}, \ldots, x_{n}$$ (in that order), we write $$t=\left(x_{1}, \ldots, x_{n}\right)$$; number $$n \geq 0$$ is the length of Tuple $$t$$. It is also denoted by $$|t|$$. Tuples of length two are called pairs, and Tuples of length three are triples.

Lisp-style lists are represented with the aid of pairs, i.e., tuples consisting of exactly two elements. An empty list is represented by a Null value, and a non-empty list is represented by pair $$(h, t)$$, where $$h$$ is the first element of the list, and $$t$$ is its tail.

### A.3.1. Null primitives.

The following primitives work with (the only) value $$\perp$$ of type $$N u l l$$, useful for representing empty lists, empty branches of binary trees, and absence of values in Maybe $$X$$ types. An empty Tuple created by NIL could have been used for the same purpose; however, Null is more efficient and costs less gas.

* 6D - NULL or PUSHNULL $$(-\perp)$$, pushes the only value of type $$N u l l$$.
* 6E - ISNULL $$(x-?)$$, checks whether $$x$$ is a $$N u l l$$, and returns -1 or 0 accordingly.

### A.3.2. Tuple primitives.

* 6F0 $$n$$ - TUPLE $$n\left(x_{1} \ldots x_{n}-t\right)$$, creates a new Tuple $$t=\left(x_{1}, \ldots, x_{n}\right)$$ containing $$n$$ values $$x_{1}, \ldots, x_{n}$$, where $$0 \leq n \leq 15$$
* 6F00 - NIL $$(-t)$$, pushes the only Tuple $$t=()$$ of length zero.
* 6F01 - SINGLE $$(x-t)$$, creates a singleton $$t:=(x)$$, i.e., a Tuple of length one.
* 6F02 - PAIR or CONS $$(x y-t)$$, creates pair $$t:=(x, y)$$.
* 6F03 - TRIPLE $$(x y z-t)$$, creates triple $$t:=(x, y, z)$$.
* 6F1 $$k$$ - INDEX $$k(t-x)$$, returns the $$k$$-th element of a Tuple $$t$$, where $$0 \leq k \leq 15$$. In other words, returns $$x_{k+1}$$ if $$t=\left(x_{1}, \ldots, x_{n}\right)$$. If $$k \geq n$$, throws a range check exception.
* 6F10 - FIRST or CAR $$(t-x)$$, returns the first element of a Tuple.
* 6F11 - SECOND or CDR $$(t-y)$$, returns the second element of a Tuple.
* 6F12 - THIRD $$(t-z)$$, returns the third element of a Tuple. - 6F2n - UNTUPLE $$n\left(t-x_{1} \ldots x_{n}\right)$$, unpacks a Tuple $$t=\left(x_{1}, \ldots, x_{n}\right)$$ of length equal to $$0 \leq n \leq 15$$. If $$t$$ is not a Tuple, of if $$|t| \neq n$$, a type check exception is thrown.
* 6F21 - UNSINGLE $$(t-x)$$, unpacks a singleton $$t=(x)$$.
* 6F22 - UNPAIR or UNCONS $$(t-x y)$$, unpacks a pair $$t=(x, y)$$.
* 6F23 - UNTRIPLE $$(t-x y z)$$, unpacks a triple $$t=(x, y, z)$$.
* 6F3 $$k$$ - UNPACKFIRST $$k\left(t-x_{1} \ldots x_{k}\right)$$, unpacks first $$0 \leq k \leq 15$$ elements of a Tuple $$t$$. If $$|t|<k$$, throws a type check exception.
* 6F30 - CHKTUPLE $$(t-)$$, checks whether $$t$$ is a Tuple.
* 6F4n-EXPLODE $$n\left(t-x_{1} \ldots x_{m} m\right)$$, unpacks a Tuple $$t=\left(x_{1}, \ldots, x_{m}\right)$$ and returns its length $$m$$, but only if $$m \leq n \leq 15$$. Otherwise throws a type check exception.
* 6F5 $$k$$ - SETINDEX $$k\left(t x-t^{\prime}\right)$$, computes Tuple $$t^{\prime}$$ that differs from $$t$$ only at position $$t_{k+1}^{\prime}$$, which is set to $$x$$. In other words, $$\left|t^{\prime}\right|=|t|, t_{i}^{\prime}=t_{i}$$ for $$i \neq k+1$$, and $$t_{k+1}^{\prime}=x$$, for given $$0 \leq k \leq 15$$. If $$k \geq|t|$$, throws a range check exception.
* 6F50 - SETFIRST $$\left(t x-t^{\prime}\right)$$, sets the first component of Tuple $$t$$ to $$x$$ and returns the resulting Tuple $$t^{\prime}$$.
* 6F51 - SETSECOND $$\left(t x-t^{\prime}\right)$$, sets the second component of Tuple $$t$$ to $$x$$ and returns the resulting Tuple $$t^{\prime}$$.
* 6F52 - SETTHIRD $$\left(t x-t^{\prime}\right)$$, sets the third component of Tuple $$t$$ to $$x$$ and returns the resulting Tuple $$t^{\prime}$$.
* 6F6 $$k$$ - INDEXQ $$k(t-x)$$, returns the $$k$$-th element of a Tuple $$t$$, where $$0 \leq k \leq 15$$. In other words, returns $$x_{k+1}$$ if $$t=\left(x_{1}, \ldots, x_{n}\right)$$. If $$k \geq n$$, or if $$t$$ is $$N u l l$$, returns a Null instead of $$x$$.
* 6F7 $$k$$ - SETINDEXQ $$k\left(t x-t^{\prime}\right)$$, sets the $$k$$-th component of Tuple $$t$$ to $$x$$, where $$0 \leq k<16$$, and returns the resulting Tuple $$t^{\prime}$$. If $$|t| \leq k$$, first extends the original Tuple to length $$k+1$$ by setting all new components to Null. If the original value of $$t$$ is Null, treats it as an empty Tuple. If $$t$$ is not Null or Tuple, throws an exception. If $$x$$ is Null and either $$|t| \leq k$$ or $$t$$ is $$N u l l$$, then always returns $$t^{\prime}=t$$ (and does not consume tuple creation gas).
* 6F80 - TUPLEVAR $$\left(x_{1} \ldots x_{n} n-t\right)$$, creates a new Tuple $$t$$ of length $$n$$ similarly to TUPLE, but with $$0 \leq n \leq 255$$ taken from the stack.
* 6F81 - INDEXVAR $$(t k-x)$$, similar to INDEX $$k$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* 6F82 - UNTUPLEVAR $$\left(t n-x_{1} \ldots x_{n}\right)$$, similar to UNTUPLE $$n$$, but with $$0 \leq n \leq 255$$ taken from the stack.
* 6F83 - UNPACKFIRSTVAR $$\left(t n-x_{1} \ldots x_{n}\right)$$, similar to UNPACKFIRST $$n$$, but with $$0 \leq n \leq 255$$ taken from the stack.
* 6F84 - EXPLODEVAR $$\left(t n-x_{1} \ldots x_{m} m\right)$$, similar to EXPLODE $$n$$, but with $$0 \leq n \leq 255$$ taken from the stack.
* 6F85 - SETINDEXVAR $$\left(t x k-t^{\prime}\right)$$, similar to SETINDEX $$k$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* 6F86 - INDEXVARQ $$(t k-x)$$, similar to INDEXQ $$n$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* 6F87 - SETINDEXVARQ ( $$\left.t x k-t^{\prime}\right)$$, similar to SETINDEXQ $$k$$, but with $$0 \leq k \leq 254$$ taken from the stack.
* 6F88 - TLEN $$(t-n)$$, returns the length of a Tuple.
* 6F89 - QTLEN $$(t-n$$ or -1$$)$$, similar to TLEN, but returns -1 if $$t$$ is not a Tuple.
* 6F8A - ISTUPLE $$(t-?)$$, returns -1 or 0 depending on whether $$t$$ is a Tuple.
* 6F8B - LAST $$(t-x)$$, returns the last element $$t_{|t|}$$ of a non-empty Tuple $$t$$.
* 6F8C - TPUSH or COMMA $$\left(t x-t^{\prime}\right)$$, appends a value $$x$$ to a Tuple $$t=$$ $$\left(x_{1}, \ldots, x_{n}\right)$$, but only if the resulting Tuple $$t^{\prime}=\left(x_{1}, \ldots, x_{n}, x\right)$$ is of length at most 255. Otherwise throws a type check exception. - 6F8D - TPOP $$\left(t-t^{\prime} x\right)$$, detaches the last element $$x=x_{n}$$ from a nonempty Tuple $$t=\left(x_{1}, \ldots, x_{n}\right)$$, and returns both the resulting Tuple $$t^{\prime}=$$ $$\left(x_{1}, \ldots, x_{n-1}\right)$$ and the original last element $$x$$.
* 6FA0 - NULLSWAPIF $$(x-x$$ or $$\perp x)$$, pushes a $$N u l l$$ under the topmost Integer $$x$$, but only if $$x \neq 0$$.
* 6FA1 - NULLSWAPIFNOT $$(x-x$$ or $$\perp x)$$, pushes a $$N u l l$$ under the topmost Integer $$x$$, but only if $$x=0$$. May be used for stack alignment after quiet primitives such as PLDUXQ.
* 6FA2 - NULLROTRIF $$(x y-x y$$ or $$\perp x y)$$, pushes a Null under the second stack entry from the top, but only if the topmost Integer $$y$$ is non-zero.
* 6FA3 - NULLROTRIFNOT $$(x y-x y$$ or $$\perp x y)$$, pushes a Null under the second stack entry from the top, but only if the topmost Integer $$y$$ is zero. May be used for stack alignment after quiet primitives such as LDUXQ.
* 6FA4 - NULLSWAPIF2 $$(x-x$$ or $$\perp \perp x)$$, pushes two Nulls under the topmost Integer $$x$$, but only if $$x \neq 0$$. Equivalent to NULLSWAPIF; NULLSWAPIF.
* 6FA5 - NULLSWAPIFNOT2 $$(x-x$$ or $$\perp \perp x)$$, pushes two Nulls under the topmost Integer $$x$$, but only if $$x=0$$. Equivalent to NULLSWAPIFNOT; NULLSWAPIFNOT.
* 6FA6 - NULLROTRIF2 $$(x y-x y$$ or $$\perp \perp x y)$$, pushes two Nulls under the second stack entry from the top, but only if the topmost Integer $$y$$ is non-zero. Equivalent to NULLROTRIF; NULLROTRIF.
* 6FA7 - NULLROTRIFNOT2 $$(x y-x y$$ or $$\perp \perp x y)$$, pushes two Nulls under the second stack entry from the top, but only if the topmost Integer $$y$$ is zero. Equivalent to NULLROTRIFNOT; NULLROTRIFNOT.
* 6FBij-INDEX2 $$i, j(t-x)$$, recovers $$x=\left(t_{i+1}\right)_{j+1}$$ for $$0 \leq i, j \leq 3$$. Equivalent to INDEX $$i$$; INDEX $$j$$.
* 6FB4 - CADR $$(t-x)$$, recovers $$x=\left(t_{2}\right)_{1}$$. - 6 FB5 - CDDR $$(t-x)$$, recovers $$x=\left(t_{2}\right)_{2}$$.
* 6FE\_ $$i j k-$$ INDEX3 $$i, j, k(t-x)$$, recovers $$x=\left(\left(t_{i+1}\right)_{j+1}\right)_{k+1}$$ for $$0 \leq$$ $$i, j, k \leq 3$$. Equivalent to INDEX2 $$i, j ;$$ INDEX $$k$$.
* 6FD4 - CADDR $$(t-x)$$, recovers $$x=\left(\left(t_{2}\right)_{2}\right)_{1}$$.
* 6 FD5 - CDDDR $$(t-x)$$, recovers $$x=\left(\left(t_{2}\right)_{2}\right)_{2}$$.

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

* 88 - PUSHREF, pushes the first reference of cc.code into the stack as a Cell (and removes this reference from the current continuation).
* 89 - PUSHREFSLICE, similar to PUSHREF, but converts the cell into a Slice.
* 8A - PUSHREFCONT, similar to PUSHREFSLICE, but makes a simple ordinary Continuation out of the cell.
* 8Bxsss - PUSHSLICE sss, pushes the (prefix) subslice of cc . code consisting of its first $$8 x+4$$ bits and no references (i.e., essentially a bitstring), where $$0 \leq x \leq 15$$. A completion tag is assumed, meaning that all trailing zeroes and the last binary one (if present) are removed from this bitstring. If the original bitstring consists only of zeroes, an empty slice will be pushed.
* 8B08 - PUSHSLICE x8\_, pushes an empty slice (bitstring '”).
* 8B04 - PUSHSLICE $$\mathrm{x}_{-}$$, pushes bitstring ' 0 '.
* 8BOC - PUSHSLICE $$\mathrm{xC}_{-}$$, pushes bitstring ' 1 '.
* 8Crxxssss - PUSHSLICE ssss, pushes the (prefix) subslice of cc.code consisting of its first $$1 \leq r+1 \leq 4$$ references and up to first $$8 x x+1$$ bits of data, with $$0 \leq x x \leq 31$$. A completion tag is also assumed. - 8C01 is equivalent to PUSHREFSLICE.
* 8Drxxsssss - PUSHSLICE sssss, pushes the subslice of cc. code consisting of $$0 \leq r \leq 4$$ references and up to $$8 x x+6$$ bits of data, with $$0 \leq x x \leq 127$$. A completion tag is assumed.
* 8DE\_ \_ unused (reserved).
* 8F\_rXxcccc - PUSHCONT cccc, where cccc is the simple ordinary continuation made from the first $$0 \leq r \leq 3$$ references and the first $$0 \leq x x \leq 127$$ bytes of cc.code.
* $$9 x \operatorname{xcc}$$ - PUSHCONT ccc, pushes an $$x$$-byte continuation for $$0 \leq x \leq 15$$.

## A.5 Arithmetic primitives

### A.5.1. Addition, subtraction, multiplication.

* $$\mathrm{AO}-\mathrm{ADD}(x y-x+y)$$, adds together two integers.
* $$\mathrm{A} 1-\operatorname{SUB}(x y-x-y)$$.
* A2 - $$\operatorname{SUBR}(x y-y-x)$$, equivalent to SWAP; SUB.
* A3 - NEGATE $$(x--x)$$, equivalent to MULCONST -1 or to ZERO; SUBR. Notice that it triggers an integer overflow exception if $$x=-2^{256}$$.
* A4 - INC $$(x-x+1)$$, equivalent to ADDCONST 1.
* A5 - DEC $$(x-x-1)$$, equivalent to ADDCONST -1 .
* A6cc-ADDCONST $$c c(x-x+c c),-128 \leq c c \leq 127$$.
* A7cc- MULCONST $$c c(x-x \cdot c c),-128 \leq c c \leq 127$$
* A8 - MUL $$(x y-x y)$$.

### A.5.2. Division.

The general encoding of a DIV, DIVMOD, or MOD operation is A9mscdf, with an optional pre-multiplication and an optional replacement of the division or multiplication by a shift. Variable one- or two-bit fields $$m, s, c, d$$, and $$f$$ are as follows: - $$0 \leq m \leq 1$$ - Indicates whether there is pre-multiplication (MULDIV operation and its variants), possibly replaced by a left shift.

* $$0 \leq s \leq 2$$ - Indicates whether either the multiplication or the division have been replaced by shifts: $$s=0$$-no replacement, $$s=1$$-division replaced by a right shift, $$s=2$$-multiplication replaced by a left shift (possible only for $$m=1$$ ).
* $$0 \leq c \leq 1$$ - Indicates whether there is a constant one-byte argument $$t t$$ for the shift operator (if $$s \neq 0$$ ). For $$s=0, c=0$$. If $$c=1$$, then $$0 \leq t t \leq 255$$, and the shift is performed by $$t t+1$$ bits. If $$s \neq 0$$ and $$c=0$$, then the shift amount is provided to the instruction as a top-of-stack Integer in range $$0 \ldots 256$$.
* $$1 \leq d \leq 3$$ - Indicates which results of division are required: 1-only the quotient, 2 -only the remainder, 3-both.
* $$0 \leq f \leq 2$$ - Rounding mode: 0-floor, 1-nearest integer, 2-ceiling (cf. [$$\mathbf{1.5.6}$$](./))

Examples:

* A904 - DIV $$(x y-q:=\lfloor x / y\rfloor)$$.
* A905- DIVR $$\left(x y-q^{\prime}:=\lfloor x / y+1 / 2\rfloor\right)$$.
* A906 - DIVC $$\left(x y-q^{\prime \prime}:=\lceil x / y\rceil\right)$$.
* A908 - MOD $$(x y-r)$$, where $$q:=\lfloor x / y\rfloor, r:=x \bmod y:=x-y q$$.
* A90C - DIVMOD $$(x y-q r)$$, where $$q:=\lfloor x / y\rfloor, r:=x-y q$$.
* A90D - DIVMODR $$\left(x y-q^{\prime} r^{\prime}\right)$$, where $$q^{\prime}:=\lfloor x / y+1 / 2\rfloor, r^{\prime}:=x-y q^{\prime}$$.
* A90E - DIVMODC $$\left(x y-q^{\prime \prime} r^{\prime \prime}\right)$$, where $$q^{\prime \prime}:=\lceil x / y\rceil, r^{\prime \prime}:=x-y q^{\prime \prime}$$.
* A924 - same as RSHIFT: $$\left(x y-\left\lfloor x \cdot 2^{-y}\right\rfloor\right)$$ for $$0 \leq y \leq 256$$.
* A934tt - same as RSHIFT $$t t+1:\left(x-\left\lfloor x \cdot 2^{-t t-1}\right\rfloor\right)$$.
* A938tt - MODPOW2 $$t t+1:\left(x-x \bmod 2^{t t+1}\right)$$.
* A985 - MULDIVR $$\left(x y z-q^{\prime}\right)$$, where $$q^{\prime}=\lfloor x y / z+1 / 2\rfloor$$. - A98C - MULDIVMOD $$(x y z-q r)$$, where $$q:=\lfloor x \cdot y / z\rfloor, r:=x \cdot y \bmod z$$ (same as $$* /$$ MOD in Forth).
* A9A4 - MULRSHIFT $$\left(x y z-\left\lfloor x y \cdot 2^{-z}\right\rfloor\right)$$ for $$0 \leq z \leq 256$$.
* A9A5 - MULRSHIFTR $$\left(x y z-\left\lfloor x y \cdot 2^{-z}+1 / 2\right\rfloor\right)$$ for $$0 \leq z \leq 256$$.
* A9B4tt - MULRSHIFT $$t t+1\left(x y-\left\lfloor x y \cdot 2^{-t t-1}\right\rfloor\right)$$.
* A9B5tt - MULRSHIFTR $$t t+1\left(x y-\left\lfloor x y \cdot 2^{-t t-1}+1 / 2\right\rfloor\right)$$.
* A9C4 - LSHIFTDIV $$\left(x y z-\left\lfloor 2^{z} x / y\right\rfloor\right)$$ for $$0 \leq z \leq 256$$.
* A9C5 - LSHIFTDIVR $$\left(x y z-\left\lfloor 2^{z} x / y+1 / 2\right\rfloor\right)$$ for $$0 \leq z \leq 256$$.
* A9D4tt - LSHIFTDIV $$t t+1\left(x y-\left\lfloor 2^{t t+1} x / y\right\rfloor\right)$$.
* A9D5tt - LSHIFTDIVR $$t t+1\left(x y-\left\lfloor 2^{t t+1} x / y+1 / 2\right\rfloor\right)$$.

The most useful of these operations are DIV, DIVMOD, MOD, DIVR, DIVC, MODPOW2 $$t$$, and RSHIFTR $$t$$ (for integer arithmetic); and MULDIVMOD, MULDIV, MULDIVR, LSHIFTDIVR $$t$$, and MULRSHIFTR $$t$$ (for fixed-point arithmetic).

### A.5.3. Shifts, logical operations.

* AAcc-LSHIFT $$c c+1\left(x-x \cdot 2^{c c+1}\right), 0 \leq c c \leq 255$$.
* AAO0 - LSHIFT 1, equivalent to MULCONST 2 or to Forth's 2\*.
* $$\mathrm{ABcc}-\mathrm{RSHIFT} c c+1\left(x-\left\lfloor x \cdot 2^{-c c-1}\right\rfloor\right), 0 \leq c c \leq 255$$.
* AC - LSHIFT $$\left(x y-x \cdot 2^{y}\right), 0 \leq y \leq 1023$$.
* $$\mathrm{AD}-\mathrm{RSHIFT}\left(x y-\left\lfloor x \cdot 2^{-y}\right\rfloor\right), 0 \leq y \leq 1023$$.
* AE - POW2 $$\left(y-2^{y}\right), 0 \leq y \leq 1023$$, equivalent to ONE; SWAP; LSHIFT.
* $$\mathrm{AF}$$ - reserved.
* BO - AND $$(x y-x \& y)$$, bitwise "and" of two signed integers $$x$$ and $$y$$, sign-extended to infinity.
* B1 - OR $$(x y-x \vee y)$$, bitwise "or" of two integers.
* B2 - XOR $$(x y-x \oplus y)$$, bitwise "xor" of two integers.
* B3 - NOT $$(x-x \oplus-1=-1-x)$$, bitwise "not" of an integer.
* B4cc - FITS $$c c+1(x-x)$$, checks whether $$x$$ is a $$c c+1$$-bit signed integer for $$0 \leq c c \leq 255$$ (i.e., whether $$-2^{c c} \leq x<2^{c c}$$ ). If not, either triggers an integer overflow exception, or replaces $$x$$ with a NaN (quiet version).
* B400 - FITS 1 or CHKBOOL $$(x-x)$$, checks whether $$x$$ is a "boolean value" (i.e., either 0 or -1 ).
* B5cc-UFITS $$c c+1(x-x)$$, checks whether $$x$$ is a $$c c+1$$-bit unsigned integer for $$0 \leq c c \leq 255$$ (i.e., whether $$0 \leq x<2^{c c+1}$$ )
* B500 - UFITS 1 or CHKBIT, checks whether $$x$$ is a binary digit (i.e., zero or one).
* B600 - FITSX $$(x c-x)$$, checks whether $$x$$ is a $$c$$-bit signed integer for $$0 \leq c \leq 1023$$
* B601 - UFITSX $$(x c-x)$$, checks whether $$x$$ is a $$c$$-bit unsigned integer for $$0 \leq c \leq 1023$$
* B602 - BITSIZE $$(x-c)$$, computes smallest $$c \geq 0$$ such that $$x$$ fits into a $$c$$-bit signed integer $$\left(-2^{c-1} \leq c<2^{c-1}\right)$$.
* B603 - UBITSIZE $$(x-c)$$, computes smallest $$c \geq 0$$ such that $$x$$ fits into a $$c$$-bit unsigned integer $$\left(0 \leq x<2^{c}\right)$$, or throws a range check exception.
* B608 - MIN $$(x y-x$$ or $$y$$ ), computes the minimum of two integers $$x$$ and $$y$$.
* B609 - MAX $$(x y-x$$ or $$y)$$, computes the maximum of two integers $$x$$ and $$y$$.
* B60A - MINMAX or INTSORT2 $$(x y-x y$$ or $$y x)$$, sorts two integers. Quiet version of this operation returns two NaNs if any of the arguments are NaNs.
* $$\mathrm{B} 60 \mathrm{~B}-\operatorname{ABS}(x-|x|)$$, computes the absolute value of an integer $$x$$. A.5.4. Quiet arithmetic primitives. We opted to make all arithmetic operations "non-quiet" (signaling) by default, and create their quiet counterparts by means of a prefix. Such an encoding is definitely sub-optimal. It is not yet clear whether it should be done in this way, or in the opposite way by making all arithmetic operations quiet by default, or whether quiet and non-quiet operations should be given opcodes of equal length; this can only be settled by practice.
* B7xx - QUIET prefix, transforming any arithmetic operation into its "quiet" variant, indicated by prefixing a $$Q$$ to its mnemonic. Such operations return NaNs instead of throwing integer overflow exceptions if the results do not fit in Integers, or if one of their arguments is a NaN. Notice that this does not extend to shift amounts and other parameters that must be within a small range (e.g., 0-1023). Also notice that this does not disable type-checking exceptions if a value of a type other than Integer is supplied.
* B7AO - QADD $$(x y-x+y)$$, always works if $$x$$ and $$y$$ are Integers, but returns a $$\mathrm{NaN}$$ if the addition cannot be performed.
* B7A904 - QDIV $$(x y-\lfloor x / y\rfloor)$$, returns a NaN if $$y=0$$, or if $$y=-1$$ and $$x=-2^{256}$$, or if either of $$x$$ or $$y$$ is a NaN.
* B7BO - QAND $$(x y-x \& y)$$, bitwise "and" (similar to AND), but returns a NaN if either $$x$$ or $$y$$ is a NaN instead of throwing an integer overflow exception. However, if one of the arguments is zero, and the other is a $$\mathrm{NaN}$$, the result is zero.
* B7B1 - QOR $$(x y-x \vee y)$$, bitwise "or". If $$x=-1$$ or $$y=-1$$, the result is always -1 , even if the other argument is a $$\mathrm{NaN}$$.
* B7B507 - QUFITS $$8\left(x-x^{\prime}\right)$$, checks whether $$x$$ is an unsigned byte (i.e., whether $$0 \leq x<2^{8}$$ ), and replaces $$x$$ with a $$\mathrm{NaN}$$ if this is not the case; leaves $$x$$ intact otherwise (i.e., if $$x$$ is an unsigned byte).

## A.6 Comparison primitives

### A.6.1. Integer comparison.

All integer comparison primitives return integer -1 ("true") or 0 ("false") to indicate the result of the comparison. We do not define their "boolean circuit" counterparts, which would transfer control to $$c 0$$ or $$c 1$$ depending on the result of the comparison. If needed, such instructions can be simulated with the aid of RETBOOL.

Quiet versions of integer comparison primitives are also available, encoded with the aid of the QUIET prefix (B7). If any of the integers being compared are NaNs, the result of a quiet comparison will also be a $$\mathrm{NaN}$$ ("undefined"), instead of a -1 ("yes") or 0 ("no"), thus effectively supporting ternary logic.

* B8 - SGN $$(x-\operatorname{sgn}(x))$$, computes the sign of an integer $$x:-1$$ if $$x<0$$, 0 if $$x=0,1$$ if $$x>0$$.
* B9 - LESS $$(x y-x<y)$$, returns -1 if $$x<y, 0$$ otherwise.
* BA - EQUAL $$(x y-x=y)$$, returns -1 if $$x=y, 0$$ otherwise.
* $$\mathrm{BB}-\mathrm{LEQ}(x y-x \leq y)$$.
* $$\mathrm{BC}-\operatorname{GREATER}(x y-x>y)$$.
* $$\mathrm{BD}-\mathrm{NEQ}(x y-x \neq y)$$, equivalent to EQUAL; NOT.
* $$\mathrm{BE}-\mathrm{GEQ}(x y-x \geq y)$$, equivalent to LESS; NOT.
* $$\mathrm{BF}-\operatorname{CMP}(x y-\operatorname{sgn}(x-y))$$, computes the sign of $$x-y:-1$$ if $$x<y$$, 0 if $$x=y, 1$$ if $$x>y$$. No integer overflow can occur here unless $$x$$ or $$y$$ is a $$\mathrm{NaN}$$.
* COyy-EQINT $$y y(x-x=y y)$$ for $$-2^{7} \leq y y<2^{7}$$.
* COOO - ISZERO, checks whether an integer is zero. Corresponds to Forth's $$0=$$.
* C1yy-LESSINT $$y y(x-x<y y)$$ for $$-2^{7} \leq y y<2^{7}$$.
* C100 - ISNEG, checks whether an integer is negative. Corresponds to Forth's $$0<$$.
* C101 - ISNPOS, checks whether an integer is non-positive.
* C2yy-GTINT $$y y(x-x>y y)$$ for $$-2^{7} \leq y y<2^{7}$$.
* C200 - ISPOS, checks whether an integer is positive. Corresponds to Forth's 0>. - $$\mathrm{C} 2 \mathrm{FF}$$ - ISNNEG, checks whether an integer is non-negative.
* C3yy-NEQINT $$y y(x-x \neq y y)$$ for $$-2^{7} \leq y y<2^{7}$$.
* C4 - ISNAN $$(x-x=\mathrm{NaN})$$, checks whether $$x$$ is a NaN.
* C5-CHKNAN $$(x-x)$$, throws an arithmetic overflow exception if $$x$$ is a $$\mathrm{NaN}$$.
* C6 - reserved for integer comparison.

### A.6.2. Other comparison.

Most of these "other comparison" primitives actually compare the data portions of Slices as bitstrings.

* C700 - SEMPTY $$(s-s=\emptyset)$$, checks whether a Slice $$s$$ is empty (i.e., contains no bits of data and no cell references).
* C701 - SDEMPTY $$(s-s \approx \emptyset)$$, checks whether Slice $$s$$ has no bits of data.
* C702 - SREMPTY $$(s-r(s)=0)$$, checks whether Slice s has no references.
* C703 - SDFIRST $$\left(s-s_{0}=1\right)$$, checks whether the first bit of Slice $$s$$ is a one.
* C704 - SDLEXCMP $$\left(s s^{\prime}-c\right)$$, compares the data of $$s$$ lexicographically with the data of $$s^{\prime}$$, returning $$-1,0$$, or 1 depending on the result.
* C705 - SDEQ $$\left(s s^{\prime}-s \approx s^{\prime}\right)$$, checks whether the data parts of $$s$$ and $$s^{\prime}$$ coincide, equivalent to SDLEXCMP; ISZERO.
* C708- $$\operatorname{SDPFX}\left(s s^{\prime}-?\right)$$, checks whether $$s$$ is a prefix of $$s^{\prime}$$.
* C709-SDPFXREV $$\left(s s^{\prime}-\right.$$ ?), checks whether $$s^{\prime}$$ is a prefix of $$s$$, equivalent to SWAP; SDPFX.
* C70A - SDPPFX $$\left(s s^{\prime}-\right.$$ ?), checks whether $$s$$ is a proper prefix of $$s^{\prime}$$ (i.e., a prefix distinct from $$\left.s^{\prime}\right)$$.
* C70B - SDPPFXREV $$\left(s s^{\prime}-\right.$$ ?), checks whether $$s^{\prime}$$ is a proper prefix of $$s$$.

## A.7. Cell primitives

* C70C - SDSFX $$\left(s s^{\prime}-\right.$$ ?), checks whether $$s$$ is a suffix of $$s^{\prime}$$.
* C70D - SDSFXREV $$\left(s s^{\prime}-\right.$$ ?), checks whether $$s^{\prime}$$ is a suffix of $$s$$.
* C70E - SDPSFX $$\left(s s^{\prime}-\right.$$ ?), checks whether $$s$$ is a proper suffix of $$s^{\prime}$$.
* C70F - SDPSFXREV $$\left(s s^{\prime}-?\right)$$, checks whether $$s^{\prime}$$ is a proper suffix of $$s$$.
* C710 - SDCNTLEADO $$(s-n)$$, returns the number of leading zeroes in $$s$$.
* C711 - SDCNTLEAD1 $$(s-n)$$, returns the number of leading ones in $$s$$.
* C712 - SDCNTTRAILO $$(s-n)$$, returns the number of trailing zeroes in $$s$$.
* C713 - SDCNTTRAIL1 $$(s-n)$$, returns the number of trailing ones in $$s$$.

The cell primitives are mostly either cell serialization primitives, which work with Builders, or cell deserialization primitives, which work with Slices.

### A.7.1. Cell serialization primitives.

All these primitives first check whether there is enough space in the Builder, and only then check the range of the value being serialized.

* C8 - NEWC $$(-b)$$, creates a new empty Builder.
* C9 - ENDC $$(b-c)$$, converts a Builder into an ordinary Cell.
* CAcc - STI $$c c+1\left(x b-b^{\prime}\right)$$, stores a signed $$c c+$$ 1-bit integer $$x$$ into Builder $$b$$ for $$0 \leq c c \leq 255$$, throws a range check exception if $$x$$ does not fit into $$c c+1$$ bits.
* $$\mathrm{CBcc}$$ - STU $$c c+1\left(x b-b^{\prime}\right)$$, stores an unsigned $$c c+$$ 1-bit integer $$x$$ into Builder $$b$$. In all other respects it is similar to STI.
* CC - STREF $$\left(c b-b^{\prime}\right)$$, stores a reference to Cell c into Builder $$b$$.
* CD - STBREFR or ENDCST $$\left(b b^{\prime \prime}-b\right)$$, equivalent to ENDC; SWAP; STREF.
* CE - STSLICE $$\left(s b-b^{\prime}\right)$$, stores Slice $$s$$ into Builder $$b$$.
* CF00 - STIX $$\left(x b l-b^{\prime}\right)$$, stores a signed $$l$$-bit integer $$x$$ into $$b$$ for $$0 \leq l \leq 257$$
* CF01 - STUX $$\left(x b l-b^{\prime}\right)$$, stores an unsigned $$l$$-bit integer $$x$$ into $$b$$ for $$0 \leq l \leq 256$$
* CF02 - STIXR $$\left(b x l-b^{\prime}\right)$$, similar to STIX, but with arguments in a different order.
* CF03 - STUXR $$\left(b x l-b^{\prime}\right)$$, similar to STUX, but with arguments in a different order.
* CF04 - STIXQ $$\left(x b l-x b f\right.$$ or $$\left.b^{\prime} 0\right)$$, a quiet version of STIX. If there is no space in $$b$$, sets $$b^{\prime}=b$$ and $$f=-1$$. If $$x$$ does not fit into $$l$$ bits, sets $$b^{\prime}=b$$ and $$f=1$$. If the operation succeeds, $$b^{\prime}$$ is the new Builder and $$f=0$$. However, $$0 \leq l \leq 257$$, with a range check exception if this is not so.
* CF05 - STUXQ $$\left(x b l-b^{\prime} f\right)$$.
* CF06 - STIXRQ $$\left(b x l-b x f\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CF07 - STUXRQ $$\left(b x l-b x f\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CF08cc - a longer version of STI $$c c+1$$.
* CF09cc - a longer version of STU $$c c+1$$.
* CFOAcc - STIR $$c c+1\left(b x-b^{\prime}\right)$$, equivalent to SWAP; STI $$c c+1$$.
* CFOBcc-STUR $$c c+1\left(b x-b^{\prime}\right)$$, equivalent to SWAP; STU $$c c+1$$.
* CFOCcc-STIQ $$c c+1\left(x b-x b f\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CFODcc-STUQ $$c c+1\left(x b-x b f\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CFOEcc-STIRQ $$c c+1\left(b x-b x f\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CFOF $$c c-$$ STURQ $$c c+1\left(b x-b x f\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CF10 - a longer version of STREF $$\left(c b-b^{\prime}\right)$$.
* CF11 - STBREF $$\left(b^{\prime} b-b^{\prime \prime}\right)$$, equivalent to SWAP; STBREFREV. - CF12 - a longer version of STSLICE $$\left(s b-b^{\prime}\right)$$.
* CF13 - STB $$\left(b^{\prime} b-b^{\prime \prime}\right)$$, appends all data from Builder $$b^{\prime}$$ to Builder $$b$$.
* CF14 - STREFR $$\left(b c-b^{\prime}\right)$$.
* CF15 - STBREFR $$\left(b b^{\prime}-b^{\prime \prime}\right)$$, a longer encoding of STBREFR.
* CF16 - STSLICER $$\left(b s-b^{\prime}\right)$$.
* CF17 - STBR $$\left(b b^{\prime}-b^{\prime \prime}\right)$$, concatenates two Builders, equivalent to SWAP; STB.
* CF18 - STREFQ $$\left(c b-c b-1\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CF19 - STBREFQ $$\left(b^{\prime} b-b^{\prime} b-1\right.$$ or $$\left.b^{\prime \prime} 0\right)$$.
* CF1A - STSLICEQ $$\left(s b-s b-1\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CF1B - STBQ $$\left(b^{\prime} b-b^{\prime} b-1\right.$$ or $$\left.b^{\prime \prime} 0\right)$$.
* CF1C - STREFRQ $$\left(b c-b c-1\right.$$ or $$\left.b^{\prime} 0\right)$$.
* CF1D - STBREFRQ $$\left(b b^{\prime}-b b^{\prime}-1\right.$$ or $$\left.b^{\prime \prime} 0\right)$$.
* CF1E - STSLICERQ ( $$b s-b s-1$$ or $$\left.b^{\prime \prime} 0\right)$$.
* CF1F - $$\operatorname{STBRQ}\left(b b^{\prime}-b b^{\prime}-1\right.$$ or $$\left.b^{\prime \prime} 0\right)$$.
* CF20 - STREFCONST, equivalent to PUSHREF; STREFR.
* CF21 - STREF2CONST, equivalent to STREFCONST; STREFCONST.
* CF23 - ENDXC $$(b x-c)$$, if $$x \neq 0$$, creates a special or exotic cell (cf. 3.1.2 from Builder b. The type of the exotic cell must be stored in the first 8 bits of $$b$$. If $$x=0$$, it is equivalent to ENDC. Otherwise some validity checks on the data and references of $$b$$ are performed before creating the exotic cell.
* CF28 - STILE4 $$\left(x b-b^{\prime}\right)$$, stores a little-endian signed 32-bit integer.
* CF29 - STULE4 $$\left(x b-b^{\prime}\right)$$, stores a little-endian unsigned 32-bit integer.
* CF2A - STILE8 $$\left(x b-b^{\prime}\right)$$, stores a little-endian signed 64-bit integer.
* CF2B - STULE8 $$\left(x b-b^{\prime}\right)$$, stores a little-endian unsigned 64-bit integer.
* CF30 - BDEPTH $$(b-x)$$, returns the depth of Builder b. If no cell references are stored in $$b$$, then $$x=0$$; otherwise $$x$$ is one plus the maximum of depths of cells referred to from $$b$$.
* CF31 - BBITS $$(b-x)$$, returns the number of data bits already stored in Builder $$b$$.
* CF32 - BREFS $$(b-y)$$, returns the number of cell references already stored in $$b$$.
* CF33 - BBITREFS $$(b-x y)$$, returns the numbers of both data bits and cell references in $$b$$.
* CF35 - BREMBITS $$\left(b-x^{\prime}\right)$$, returns the number of data bits that can still be stored in $$b$$.
* CF36 - BREMREFS $$\left(b-y^{\prime}\right)$$.
* CF37 - BREMBITREFS $$\left(b-x^{\prime} y^{\prime}\right)$$.
* CF38cc - BCHKBITS $$c c+1(b-)$$, checks whether $$c c+1$$ bits can be stored into $$b$$, where $$0 \leq c c \leq 255$$.
* CF39 - BCHKBITS $$(b x-)$$, checks whether $$x$$ bits can be stored into $$b$$, $$0 \leq x \leq 1023$$. If there is no space for $$x$$ more bits in $$b$$, or if $$x$$ is not within the range $$0 \ldots 1023$$, throws an exception.
* CF3A - BCHKREFS ( $$b y-$$ ), checks whether $$y$$ references can be stored into $$b, 0 \leq y \leq 7$$
* CF3B - BCHKBITREFS ( $$b x y-)$$, checks whether $$x$$ bits and $$y$$ references can be stored into $$b, 0 \leq x \leq 1023,0 \leq y \leq 7$$
* CF3Ccc- BCHKBITSQ $$c c+1(b-$$ ?), checks whether $$c c+1$$ bits can be stored into $$b$$, where $$0 \leq c c \leq 255$$.
* CF3D - BCHKBITSQ ( $$b x-$$ ?), checks whether $$x$$ bits can be stored into $$b, 0 \leq x \leq 1023$$
* CF3E - BCHKREFSQ (by-?), checks whether $$y$$ references can be stored into $$b, 0 \leq y \leq 7$$. - CF3F - BCHKBITREFSQ ( $$b x y$$-?), checks whether $$x$$ bits and $$y$$ references can be stored into $$b, 0 \leq x \leq 1023,0 \leq y \leq 7$$
* CF40 - STZEROES $$\left(b n-b^{\prime}\right)$$, stores $$n$$ binary zeroes into Builder $$b$$.
* CF41 - STONES $$\left(b n-b^{\prime}\right)$$, stores $$n$$ binary ones into Builder $$b$$.
* CF42 - STSAME $$\left(b n x-b^{\prime}\right)$$, stores $$n$$ binary xes $$(0 \leq x \leq 1)$$ into Builder b.
* CFC0\_xysss - STSLICECONST sss $$\left(b-b^{\prime}\right)$$, stores a constant subslice sss consisting of $$0 \leq x \leq 3$$ references and up to $$8 y+1$$ data bits, with $$0 \leq y \leq 7$$. Completion bit is assumed.
* CF81 - STSLICECONST ' 0 ' or STZERO $$\left(b-b^{\prime}\right)$$, stores one binary zero.
* CF83 - STSLICECONST ' 1 ' or STONE $$\left(b-b^{\prime}\right)$$, stores one binary one.
* CFA2 - equivalent to STREFCONST.
* CFA3 - almost equivalent to STSLICECONST '1'; STREFCONST.
* $$\mathrm{CFC} 2$$ - equivalent to STREF2CONST.
* CFE2 - STREF3CONST.

### A.7.2. Cell deserialization primitives.

* DO - CTOS $$(c-s)$$, converts a Cell into a Slice. Notice that $$c$$ must be either an ordinary cell, or an exotic cell (cf. 3.1.2) which is automatically loaded to yield an ordinary cell $$c^{\prime}$$, converted into a Slice afterwards.
* D1 - ENDS $$(s-)$$, removes a Slice $$s$$ from the stack, and throws an exception if it is not empty.
* D2cc - LDI $$c c+1\left(s-x s^{\prime}\right)$$, loads (i.e., parses) a signed $$c c+$$ 1-bit integer $$x$$ from Slice $$s$$, and returns the remainder of $$s$$ as $$s^{\prime}$$.
* D3cc - LDU $$c c+1\left(s-x s^{\prime}\right)$$, loads an unsigned $$c c+$$ 1-bit integer $$x$$ from Slice s.
* D4 - LDREF $$\left(s-c s^{\prime}\right)$$, loads a cell reference $$c$$ from $$s$$.
* D5 - LDREFRTOS $$\left(s-s^{\prime} s^{\prime \prime}\right)$$, equivalent to LDREF; SWAP; CTOS.
* D6cc - LDSLICE $$c c+1\left(s-s^{\prime \prime} s^{\prime}\right)$$, cuts the next $$c c+1$$ bits of $$s$$ into a separate Slice $$s^{\prime \prime}$$
* D700 - LDIX $$\left(s l-x s^{\prime}\right)$$, loads a signed $$l$$-bit $$(0 \leq l \leq 257)$$ integer $$x$$ from Slice $$s$$, and returns the remainder of $$s$$ as $$s^{\prime}$$.
* D701 - LDUX $$\left(s l-x s^{\prime}\right)$$, loads an unsigned $$l$$-bit integer $$x$$ from (the first $$l$$ bits of) $$s$$, with $$0 \leq l \leq 256$$
* D702 - PLDIX $$(s l-x)$$, preloads a signed $$l$$-bit integer from Slice $$s$$, for $$0 \leq l \leq 257$$
* D703 - PLDUX $$(s l-x)$$, preloads an unsigned $$l$$-bit integer from $$s$$, for $$0 \leq l \leq 256$$
* D704 - LDIXQ ( $$s l-x s^{\prime}-1$$ or $$s$$ ), quiet version of LDIX: loads a signed $$l$$-bit integer from $$s$$ similarly to LDIX, but returns a success flag, equal to -1 on success or to 0 on failure (if $$s$$ does not have $$l$$ bits), instead of throwing a cell underflow exception.
* D705 - LDUXQ $$\left(s l-x s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, quiet version of LDUX.
* D706 - PLDIXQ $$(s l-x-1$$ or 0$$)$$, quiet version of PLDIX.
* D707 - PLDUXQ $$(s l-x-1$$ or 0$$)$$, quiet version of PLDUX.
* D708cc - LDI $$c c+1\left(s-x s^{\prime}\right)$$, a longer encoding for LDI.
* D709cc - LDU $$c c+1\left(s-x s^{\prime}\right)$$, a longer encoding for LDU.
* D70Acc-PLDI $$c c+1(s-x)$$, preloads a signed $$c c+1$$-bit integer from Slice s.
* D70Bcc - PLDU $$c c+1(s-x)$$, preloads an unsigned $$c c+$$ 1-bit integer from $$s$$.
* D70Ccc-LDIQ $$c c+1\left(s-x s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of LDI.
* D70Dcc - LDUQ $$c c+1\left(s-x s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of LDU.
* D70Ecc - PLDIQ $$c c+1(s-x-1$$ or 0$$)$$, a quiet version of PLDI.
* D70F $$c c-$$ PLDUQ $$c c+1(s-x-1$$ or 0$$)$$, a quiet version of PLDU.
* D714\_c - PLDUZ $$32(c+1)(s-s x)$$, preloads the first 32(c+1) bits of Slice $$s$$ into an unsigned integer $$x$$, for $$0 \leq c \leq 7$$. If $$s$$ is shorter than necessary, missing bits are assumed to be zero. This operation is intended to be used along with IFBITJMP and similar instructions.
* D718 - LDSLICEX $$\left(s l-s^{\prime \prime} s^{\prime}\right)$$, loads the first $$0 \leq l \leq 1023$$ bits from Slice $$s$$ into a separate Slice $$s^{\prime \prime}$$, returning the remainder of $$s$$ as $$s^{\prime}$$.
* D719 - PLDSLICEX $$\left(s l-s^{\prime \prime}\right)$$, returns the first $$0 \leq l \leq 1023$$ bits of $$s$$ as $$s^{\prime \prime}$$.
* D71A - LDSLICEXQ $$\left(s l-s^{\prime \prime} s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of LDSLICEX.
* D71B - PLDSLICEXQ ( $$s l-s^{\prime}-1$$ or 0 ), a quiet version of LDSLICEXQ.
* D71Ccc - LDSLICE $$c c+1\left(s-s^{\prime \prime} s^{\prime}\right)$$, a longer encoding for LDSLICE.
* D71Dcc - PLDSLICE $$c c+1\left(s-s^{\prime \prime}\right)$$, returns the first $$0<c c+1 \leq 256$$ bits of $$s$$ as $$s^{\prime \prime}$$.
* D71Ecc - LDSLICEQ $$c c+1\left(s-s^{\prime \prime} s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of LDSLICE.
* D71Fcc - PLDSLICEQ $$c c+1\left(s-s^{\prime \prime}-1\right.$$ or 0$$)$$, a quiet version of PLDSLICE.
* D720 - SDCUTFIRST $$\left(s l-s^{\prime}\right)$$, returns the first $$0 \leq l \leq 1023$$ bits of $$s$$. It is equivalent to PLDSLICEX.
* D721 - SDSKIPFIRST $$\left(s l-s^{\prime}\right)$$, returns all but the first $$0 \leq l \leq 1023$$ bits of $$s$$. It is equivalent to LDSLICEX; NIP.
* D722 - SDCUTLAST $$\left(s l-s^{\prime}\right)$$, returns the last $$0 \leq l \leq 1023$$ bits of $$s$$.
* D723 - SDSKIPLAST $$\left(s l-s^{\prime}\right)$$, returns all but the last $$0 \leq l \leq 1023$$ bits of $$s$$.
* D724 - SDSUBSTR $$\left(s l l^{\prime}-s^{\prime}\right)$$, returns $$0 \leq l^{\prime} \leq 1023$$ bits of $$s$$ starting from offset $$0 \leq l \leq 1023$$, thus extracting a bit substring out of the data of $$s$$. - D726 - SDBEGINSX $$\left(s s^{\prime}-s^{\prime \prime}\right)$$, checks whether $$s$$ begins with (the data bits of) $$s^{\prime}$$, and removes $$s^{\prime}$$ from $$s$$ on success. On failure throws a cell deserialization exception. Primitive SDPFXREV can be considered a quiet version of SDBEGINSX.
* D727 - SDBEGINSXQ $$\left(s s^{\prime}-s^{\prime \prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of SDBEGINSX.
* D72A\_xsss - SDBEGINS $$\left(s-s^{\prime \prime}\right)$$, checks whether $$s$$ begins with constant bitstring sss of length $$8 x+3$$ (with continuation bit assumed), where $$0 \leq x \leq 127$$, and removes sss from $$s$$ on success.
* D72802 - SDBEGINS '0' $$\left(s-s^{\prime \prime}\right)$$, checks whether $$s$$ begins with a binary zero.
* D72806 - SDBEGINS ' 1 ' $$\left(s-s^{\prime \prime}\right)$$, checks whether $$s$$ begins with a binary one.
* D72E\_ $$x s s s$$ - SDBEGINSQ $$\left(s-s^{\prime \prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of SDBEGINS.
* D730 - SCUTFIRST $$\left(s l r-s^{\prime}\right)$$, returns the first $$0 \leq l \leq 1023$$ bits and first $$0 \leq r \leq 4$$ references of $$s$$.
* D731 - SSKIPFIRST $$\left(s l r-s^{\prime}\right)$$.
* D732 - SCUTLAST $$\left(s l r-s^{\prime}\right)$$, returns the last $$0 \leq l \leq 1023$$ data bits and last $$0 \leq r \leq 4$$ references of $$s$$.
* D733 - SSKIPLAST $$\left(s l r-s^{\prime}\right)$$.
* D734 - SUBSLICE $$\left(s l r l^{\prime} r^{\prime}-s^{\prime}\right)$$, returns $$0 \leq l^{\prime} \leq 1023$$ bits and $$0 \leq r^{\prime} \leq 4$$ references from Slice s, after skipping the first $$0 \leq l \leq 1023$$ bits and first $$0 \leq r \leq 4$$ references.
* D736 - SPLIT $$\left(s l r-s^{\prime} s^{\prime \prime}\right)$$, splits the first $$0 \leq l \leq 1023$$ data bits and first $$0 \leq r \leq 4$$ references from $$s$$ into $$s^{\prime}$$, returning the remainder of $$s$$ as $$s^{\prime \prime}$$.
* D737-SPLITQ $$\left(s l r-s^{\prime} s^{\prime \prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of SPLIT.
* D739 - XCTOS $$(c-s$$ ?), transforms an ordinary or exotic cell into a Slice, as if it were an ordinary cell. A flag is returned indicating whether $$c$$ is exotic. If that be the case, its type can later be deserialized from the first eight bits of $$s$$.
* D73A - XLOAD $$\left(c-c^{\prime}\right)$$, loads an exotic cell $$c$$ and returns an ordinary cell $$c^{\prime}$$. If $$c$$ is already ordinary, does nothing. If $$c$$ cannot be loaded, throws an exception.
* D73B - XLOADQ $$\left(c-c^{\prime}-1\right.$$ or $$\left.c 0\right)$$, loads an exotic cell $$c$$ as XLOAD, but returns 0 on failure.
* D741 - SCHKBITS $$(s l-)$$, checks whether there are at least $$l$$ data bits in Slice s. If this is not the case, throws a cell deserialisation (i.e., cell underflow) exception.
* D742 - SCHKREFS $$(s r-)$$, checks whether there are at least $$r$$ references in Slice s.
* D743 - SCHKBITREFS ( $$s l r-)$$, checks whether there are at least $$l$$ data bits and $$r$$ references in Slice $$s$$.
* D745 - SCHKBITSQ $$(s l-?)$$, checks whether there are at least $$l$$ data bits in Slice $$s$$.
* D746 - SCHKREFSQ $$(s r-?)$$, checks whether there are at least $$r$$ references in Slice s.
* D747 - SCHKBITREFSQ ( $$s l r-?$$ ), checks whether there are at least $$l$$ data bits and $$r$$ references in Slice s.
* D748 - PLDREFVAR $$(s n-c)$$, returns the $$n$$-th cell reference of Slice $$s$$ for $$0 \leq n \leq 3$$.
* D749 - SBITS $$(s-l)$$, returns the number of data bits in Slice s.
* D74A - SREFS $$(s-r)$$, returns the number of references in Slice s.
* D74B - SBITREFS $$(s-l r)$$, returns both the number of data bits and the number of references in $$s$$.
* D74E\_ $$n-$$ PLDREFIDX $$n(s-c)$$, returns the $$n$$-th cell reference of Slice $$s$$, where $$0 \leq n \leq 3$$.
* D74C - PLDREF $$(s-c)$$, preloads the first cell reference of a Slice.
* D750 - LDILE4 $$\left(s-x s^{\prime}\right)$$, loads a little-endian signed 32-bit integer.
* D751 - LDULE4 $$\left(s-x s^{\prime}\right)$$, loads a little-endian unsigned 32-bit integer.
* D752 - LDILE8 $$\left(s-x s^{\prime}\right)$$, loads a little-endian signed 64-bit integer.
* D753 - LDULE8 $$\left(s-x s^{\prime}\right)$$, loads a little-endian unsigned 64-bit integer.
* D754 - PLDILE4 $$(s-x)$$, preloads a little-endian signed 32-bit integer.
* D755 - PLDULE4 $$(s-x)$$, preloads a little-endian unsigned 32-bit integer.
* D756 - PLDILE8 $$(s-x)$$, preloads a little-endian signed 64-bit integer.
* D757 - PLDULE8 $$(s-x)$$, preloads a little-endian unsigned 64-bit integer.
* D758 - LDILE4Q $$\left(s-x s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, quietly loads a little-endian signed 32-bit integer.
* D759 - LDULE4Q $$\left(s-x s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, quietly loads a little-endian unsigned 32-bit integer.
* D75A - LDILE8Q $$\left(s-x s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, quietly loads a little-endian signed 64-bit integer.
* D75B - LDULE8Q $$\left(s-x s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, quietly loads a little-endian unsigned 64-bit integer.
* D75C - PLDILE4Q $$(s-x-1$$ or 0), quietly preloads a little-endian signed 32-bit integer.
* D75D - PLDULE4Q $$(s-x-1$$ or 0$$)$$, quietly preloads a little-endian unsigned 32-bit integer.
* D75E - PLDILE8Q $$(s-x-1$$ or 0), quietly preloads a little-endian signed 64-bit integer.
* D75F - PLDULE8Q $$(s-x-1$$ or 0), quietly preloads a little-endian unsigned 64-bit integer.
* D760 - LDZEROES $$\left(s-n s^{\prime}\right)$$, returns the count $$n$$ of leading zero bits in $$s$$, and removes these bits from $$s$$.
* D761 - LDONES $$\left(s-n s^{\prime}\right)$$, returns the count $$n$$ of leading one bits in $$s$$, and removes these bits from $$s$$.
* D762 - LDSAME $$\left(s x-n s^{\prime}\right)$$, returns the count $$n$$ of leading bits equal to $$0 \leq x \leq 1$$ in $$s$$, and removes these bits from $$s$$.
* D764 - SDEPTH $$(s-x)$$, returns the depth of Slice $$s$$. If $$s$$ has no references, then $$x=0$$; otherwise $$x$$ is one plus the maximum of depths of cells referred to from $$s$$.
* D765 - CDEPTH $$(c-x)$$, returns the depth of Cell c. If $$c$$ has no references, then $$x=0$$; otherwise $$x$$ is one plus the maximum of depths of cells referred to from $$c$$. If $$c$$ is a $$N u l l$$ instead of a Cell, returns zero.

## A.8 Continuation and control flow primitives

### A.8.1. Unconditional control flow primitives.

* D8 - EXECUTE or CALLX $$(c-)$$, calls or executes continuation $$c$$ (i.e., $$\left.\mathrm{cc} \leftarrow c \circ_{0} \mathrm{cc}\right)$$.
* D9 - JMPX $$(c-)$$, jumps, or transfers control, to continuation $$c$$ (i.e., $$c c \leftarrow c \circ_{0} c 0$$, or rather $$\left.c c \leftarrow\left(c \circ_{0} c 0\right) \circ_{1} c 1\right)$$. The remainder of the previous current continuation $$\mathrm{cc}$$ is discarded.
* DApr - CALLXARGS $$p, r(c-)$$, calls continuation $$c$$ with $$p$$ parameters and expecting $$r$$ return values, $$0 \leq p \leq 15,0 \leq r \leq 15$$.
* DBO $$p$$ - CALLXARGS $$p,-1(c-)$$, calls continuation $$c$$ with $$0 \leq p \leq 15$$ parameters, expecting an arbitrary number of return values.
* DB1 $$p$$ - JMPXARGS $$p(c-)$$, jumps to continuation $$c$$, passing only the top $$0 \leq p \leq 15$$ values from the current stack to it (the remainder of the current stack is discarded).
* DB2 $$r$$ - RETARGS $$r$$, returns to $$\mathrm{c} 0$$, with $$0 \leq r \leq 15$$ return values taken from the current stack.
* DB30 - RET or RETTRUE, returns to the continuation at c0 (i.e., performs cc $$\leftarrow c 0)$$. The remainder of the current continuation $$c c$$ is discarded. Approximately equivalent to PUSH c0; JMPX. - DB31 - RETALT or RETFALSE, returns to the continuation at c1 (i.e., $$c c \leftarrow c 1)$$. Approximately equivalent to PUSH $$c 1$$; JMPX.
* DB32 - BRANCH or RETBOOL $$(f-)$$, performs RETTRUE if integer $$f \neq 0$$, or RETFALSE if $$f=0$$.
* DB34 - CALLCC $$(c-)$$, call with current continuation, transfers control to $$c$$, pushing the old value of cc into $$c$$ 's stack (instead of discarding it or writing it into new c0).
* DB35 - JMPXDATA $$(c-)$$, similar to CALLCC, but the remainder of the current continuation (the old value of cc) is converted into a Slice before pushing it into the stack of $$c$$.
* DB36pr - CALLCCARGS $$p, r(c-)$$, similar to CALLXARGS, but pushes the old value of cc (along with the top $$0 \leq p \leq 15$$ values from the original stack) into the stack of newly-invoked continuation $$c$$, setting cc. nargs to $$-1 \leq r \leq 14$$
* DB38 - CALLXVARARGS ( c p r - ), similar to CALLXARGS, but takes $$-1 \leq p, r \leq 254$$ from the stack. The next three operations also take $$p$$ and $$r$$ from the stack, both in the range $$-1 \ldots 254$$.
* DB39 - RETVARARGS $$(p r-)$$, similar to RETARGS.
* DB3A - JMPXVARARGS ( c p r-), similar to JMPXARGS.
* DB3B - CALLCCVARARGS ( c pr-), similar to CALLCCARGS.
* DB3C - CALLREF, equivalent to PUSHREFCONT; CALLX.
* DB3D - JMPREF, equivalent to PUSHREFCONT; JMPX.
* DB3E - JMPREFDATA, equivalent to PUSHREFCONT; JMPXDATA.
* DB3F - RETDATA, equivalent to PUSH c0; JMPXDATA. In this way, the remainder of the current continuation is converted into a Slice and returned to the caller.

### A.8.2. Conditional control flow primitives.

* DC - IFRET $$(f-)$$, performs a RET, but only if integer $$f$$ is non-zero. If $$f$$ is a $$N a N$$, throws an integer overflow exception. - DD - IFNOTRET $$(f-)$$, performs a RET, but only if integer $$f$$ is zero.
* DE - IF $$(f c-)$$, performs EXECUTE for $$c$$ (i.e., executes $$c)$$, but only if integer $$f$$ is non-zero. Otherwise simply discards both values.
* DF - IFNOT $$(f c-)$$, executes continuation $$c$$, but only if integer $$f$$ is zero. Otherwise simply discards both values.
* EO - IFJMP ( $$f c-)$$, jumps to $$c$$ (similarly to JMPX), but only if $$f$$ is non-zero.
* E1 - IFNOTJMP ( $$f c-)$$, jumps to $$c$$ (similarly to JMPX), but only if $$f$$ is zero.
* E2 - IFELSE ( $$\left.f c c^{\prime}-\right)$$, if integer $$f$$ is non-zero, executes $$c$$, otherwise executes $$c^{\prime}$$. Equivalent to CONDSELCHK; EXECUTE.
* E300 - IFREF $$(f-)$$, equivalent to PUSHREFCONT; IF, with the optimization that the cell reference is not actually loaded into a Slice and then converted into an ordinary Continuation if $$f=0$$. Similar remarks apply to the next three primitives.
* E301 - IFNOTREF $$(f-)$$, equivalent to PUSHREFCONT; IFNOT.
* E302 - IFJMPREF $$(f-)$$, equivalent to PUSHREFCONT; IFJMP.
* E303 - IFNOTJMPREF $$(f-)$$, equivalent to PUSHREFCONT; IFNOTJMP.
* E304 - CONDSEL $$(f x y-x$$ or $$y$$ ), if integer $$f$$ is non-zero, returns $$x$$, otherwise returns $$y$$. Notice that no type checks are performed on $$x$$ and $$y$$; as such, it is more like a conditional stack operation. Roughly equivalent to ROT; ISZERO; INC; ROLLX; NIP.
* E305 - CONDSELCHK $$(f x y-x$$ or $$y)$$, same as CONDSEL, but first checks whether $$x$$ and $$y$$ have the same type.
* E308 - IFRETALT $$(f-)$$, performs RETALT if integer $$f \neq 0$$.
* E309 - IFNOTRETALT $$(f-)$$, performs RETALT if integer $$f=0$$.
* E3OD - IFREFELSE $$(f c-)$$, equivalent to PUSHREFCONT; SWAP; IFELSE, with the optimization that the cell reference is not actually loaded into a Slice and then converted into an ordinary Continuation if $$f=0$$. Similar remarks apply to the next two primitives: Cells are converted into Continuations only when necessary.
* E3OE - IFELSEREF $$(f c-)$$, equivalent to PUSHREFCONT; IFELSE.
* E3OF - IFREFELSEREF $$(f-)$$, equivalent to PUSHREFCONT; PUSHREFCONT; IFELSE.
* E310-E31F - reserved for loops with break operators, cf. A.8.3 below.
* E39\_ $$n$$ - IFBITJMP $$n(x c-x)$$, checks whether bit $$0 \leq n \leq 31$$ is set in integer $$x$$, and if so, performs JMPX to continuation $$c$$. Value $$x$$ is left in the stack.
* E3B\_ $$n-$$ IFNBITJMP $$n(x c-x)$$, jumps to $$c$$ if bit $$0 \leq n \leq 31$$ is not set in integer $$x$$.
* E3D\_ $$n$$ - IFBITJMPREF $$n(x-x)$$, performs a JMPREF if bit $$0 \leq n \leq 31$$ is set in integer $$x$$.
* E3F\_ $$n$$ - IFNBITJMPREF $$n(x-x)$$, performs a JMPREF if bit $$0 \leq n \leq 31$$ is not set in integer $$x$$.

### A.8.3. Control flow primitives: loops.

Most of the loop primitives listed below are implemented with the aid of extraordinary continuations, such as ec\_until (cf. 4.1.5), with the loop body and the original current continuation cc stored as the arguments to this extraordinary continuation. Typically a suitable extraordinary continuation is constructed, and then saved into the loop body continuation savelist as c0; after that, the modified loop body continuation is loaded into cc and executed in the usual fashion. All of these loop primitives have $$* \mathrm{BRK}$$ versions, adapted for breaking out of a loop; they additionally set $$c 1$$ to the original current continuation (or original c0 for $$*$$ ENDBRK versions), and save the old c1 into the savelist of the original current continuation (or of the original c0 for $$*$$ ENDBRK versions).

* E4 - REPEAT $$(n c-)$$, executes continuation $$c n$$ times, if integer $$n$$ is non-negative. If $$n \geq 2^{31}$$ or $$n<-2^{31}$$, generates a range check exception. Notice that a RET inside the code of $$c$$ works as a continue, not as a break. One should use either alternative (experimental) loops or alternative RETALT (along with a SETEXITALT before the loop) to break out of a loop.
* E5 - REPEATEND $$(n-)$$, similar to REPEAT, but it is applied to the current continuation cc.
* E6 - UNTIL $$(c-)$$, executes continuation $$c$$, then pops an integer $$x$$ from the resulting stack. If $$x$$ is zero, performs another iteration of this loop. The actual implementation of this primitive involves an extraordinary continuation ec\_until (cf. 4.1.5) with its arguments set to the body of the loop (continuation $$c$$ ) and the original current continuation cc. This extraordinary continuation is then saved into the savelist of $$c$$ as $$c . c 0$$ and the modified $$c$$ is then executed. The other loop primitives are implemented similarly with the aid of suitable extraordinary continuations.
* E7 - UNTILEND $$(-)$$, similar to UNTIL, but executes the current continuation cc in a loop. When the loop exit condition is satisfied, performs a RET.
* E8 - WHILE $$\left(c^{\prime} c-\right)$$, executes $$c^{\prime}$$ and pops an integer $$x$$ from the resulting stack. If $$x$$ is zero, exists the loop and transfers control to the original cc. If $$x$$ is non-zero, executes $$c$$, and then begins a new iteration.
* E9 - WHILEEND $$\left(c^{\prime}-\right)$$, similar to WHILE, but uses the current continuation cc as the loop body.
* EA - AGAIN $$(c-)$$, similar to REPEAT, but executes $$c$$ infinitely many times. A RET only begins a new iteration of the infinite loop, which can be exited only by an exception, or a RETALT (or an explicit JMPX).
* EB - AGAINEND $$(-)$$, similar to AGAIN, but performed with respect to the current continuation cc.
* E314 - REPEATBRK $$(n c-)$$, similar to REPEAT, but also sets c1 to the original cc after saving the old value of $$c 1$$ into the savelist of the original cc. In this way RETALT could be used to break out of the loop body. - E315 - REPEATENDBRK $$(n-)$$, similar to REPEATEND, but also sets c1 to the original c0 after saving the old value of c1 into the savelist of the original c0. Equivalent to SAMEALTSAVE; REPEATEND.
* E316 - UNTILBRK $$(c-)$$, similar to UNTIL, but also modifies c1 in the same way as REPEATBRK.
* E317 - UNTILENDBRK ( - ), equivalent to SAMEALTSAVE; UNTILEND.
* E318 - WHILEBRK $$\left(c^{\prime} c-\right)$$, similar to WHILE, but also modifies c1 in the same way as REPEATBRK.
* E319 - WHILEENDBRK $$(c-)$$, equivalent to SAMEALTSAVE; WHILEEND.
* E31A - AGAINBRK $$(c-)$$, similar to AGAIN, but also modifies c1 in the same way as REPEATBRK.
* E31B - AGAINENDBRK ( - ), equivalent to SAMEALTSAVE; AGAINEND.

## A.8.4. Manipulating the stack of continuations.

* ECrn - SETCONTARGS $$r, n\left(x_{1} x_{2} \ldots x_{r} c-c^{\prime}\right)$$, similar to SETCONTARGS $$r$$, but sets $$c$$.nargs to the final size of the stack of $$c^{\prime}$$ plus $$n$$. In other words, transforms $$c$$ into a closure or a partially applied function, with $$0 \leq n \leq 14$$ arguments missing.
* ECO $$n$$ - SETNUMARGS $$n$$ or SETCONTARGS $$0, n\left(c-c^{\prime}\right)$$, sets $$c$$. nargs to $$n$$ plus the current depth of $$c^{\prime}$$ s stack, where $$0 \leq n \leq 14$$. If $$c$$.nargs is already set to a non-negative value, does nothing.
* ECrF - SETCONTARGS $$r$$ or SETCONTARGS $$r,-1\left(x_{1} x_{2} \ldots x_{r} c-c^{\prime}\right)$$, pushes $$0 \leq r \leq 15$$ values $$x_{1} \ldots x_{r}$$ into the stack of (a copy of) the continuation $$c$$, starting with $$x_{1}$$. If the final depth of $$c$$ 's stack turns out to be greater than $$c$$.nargs, a stack overflow exception is generated.
* EDO $$p$$ - RETURNARGS $$p(-)$$, leaves only the top $$0 \leq p \leq 15$$ values in the current stack (somewhat similarly to ONLYTOPX), with all the unused bottom values not discarded, but saved into continuation c0 in the same way as SETCONTARGS does.
* ED10 - RETURNVARARGS $$(p-)$$, similar to RETURNARGS, but with Integer $$0 \leq p \leq 255$$ taken from the stack. - ED11 - SETCONTVARARGS $$\left(x_{1} x_{2} \ldots x_{r} c r n-c^{\prime}\right)$$, similar to SETCONTARGS, but with $$0 \leq r \leq 255$$ and $$-1 \leq n \leq 255$$ taken from the stack.
* ED12 - SETNUMVARARGS $$\left(c n-c^{\prime}\right)$$, where $$-1 \leq n \leq 255$$. If $$n=-1$$, this operation does nothing $$\left(c^{\prime}=c\right)$$. Otherwise its action is similar to SETNUMARGS $$n$$, but with $$n$$ taken from the stack.

### A.8.5. Creating simple continuations and closures.

* ED1E - BLESS $$(s-c)$$, transforms a Slice $$s$$ into a simple ordinary continuation $$c$$, with $$c$$.code $$=s$$ and an empty stack and savelist.
* ED1F - BLESSVARARGS $$\left(x_{1} \ldots x_{r} s r n-c\right)$$, equivalent to ROT; BLESS; ROTREV; SETCONTVARARGS.
* EErn - BLESSARGS $$r, n\left(x_{1} \ldots x_{r} s-c\right)$$, where $$0 \leq r \leq 15,-1 \leq$$ $$n \leq 14$$, equivalent to BLESS; SETCONTARGS $$r, n$$. The value of $$n$$ is represented inside the instruction by the 4-bit integer $$n \bmod 16$$.
* EEO $$n$$ - BLESSNUMARGS $$n$$ or BLESSARGS $$0, n(s-c)$$, also transforms a Slice $$s$$ into a Continuation $$c$$, but sets c.nargs to $$0 \leq n \leq 14$$.

### A.8.6. Operations with continuation savelists and control registers.

* ED4 $$i$$ - PUSH $$\mathrm{c}(i)$$ or PUSHCTR $$\mathrm{c}(i)(-x)$$, pushes the current value of control register $$\mathrm{c}(i)$$. If the control register is not supported in the current codepage, or if it does not have a value, an exception is triggered.
* ED44 - PUSH c4 or PUSHR00T, pushes the "global data root" cell reference, thus enabling access to persistent smart-contract data.
* ED5 $$i$$ - POP $$\mathrm{c}(i)$$ or POPCTR $$\mathrm{c}(i)(x-)$$, pops a value $$x$$ from the stack and stores it into control register $$c(i)$$, if supported in the current codepage. Notice that if a control register accepts only values of a specific type, a type-checking exception may occur.
* ED54 - POP c4 or POPROOT, sets the "global data root" cell reference, thus allowing modification of persistent smart-contract data.
* ED6 $$i$$ - SETCONT $$\mathrm{c}(i)$$ or SETCONTCTR $$\mathrm{c}(i)\left(x c-c^{\prime}\right)$$, stores $$x$$ into the savelist of continuation $$c$$ as $$c(i)$$, and returns the resulting continuation $$c^{\prime}$$. Almost all operations with continuations may be expressed in terms of SETCONTCTR, POPCTR, and PUSHCTR.
* ED7i - SETRETCTR c $$(i)(x-)$$, equivalent to PUSH c0; SETCONTCTR $$\mathrm{c}(i) ;$$ POP c0.
* ED8i - SETALTCTR c $$(i)(x-)$$, equivalent to PUSH c1; SETCONTCTR $$\mathrm{c}(i) ;$$ POP c0.
* ED9 $$i$$ - POPSAVE $$\mathrm{c}(i)$$ or POPCTRSAVE $$\mathrm{c}(i)(x-)$$, similar to POP $$\mathrm{c}(i)$$, but also saves the old value of $$c(i)$$ into continuation co. Equivalent (up to exceptions) to SAVECTR $$\mathrm{c}(i)$$; POP $$\mathrm{c}(i)$$.
* EDA $$i$$ - SAVE $$c(i)$$ or SAVECTR $$c(i)(-)$$, saves the current value of $$c(i)$$ into the savelist of continuation c0. If an entry for $$c(i)$$ is already present in the savelist of c0, nothing is done. Equivalent to PUSH $$c(i)$$; SETRETCTR $$c(i)$$.
* EDBi - SAVEALT $$\mathrm{c}(i)$$ or SAVEALTCTR $$\mathrm{c}(i)(-)$$, similar to SAVE $$\mathrm{c}(i)$$, but saves the current value of $$c(i)$$ into the savelist of $$c 1$$, not $$c 0$$.
* EDC $$i$$ - SAVEBOTH $$c(i)$$ or SAVEBOTHCTR $$c(i)(-)$$, equivalent to DUP; SAVE $$\mathrm{c}(i)$$; SAVEALT $$\mathrm{c}(i)$$.
* EDEO - PUSHCTRX $$(i-x)$$, similar to PUSHCTR $$\mathrm{c}(i)$$, but with $$i, 0 \leq i \leq$$ 255 , taken from the stack. Notice that this primitive is one of the few "exotic" primitives, which are not polymorphic like stack manipulation primitives, and at the same time do not have well-defined types of parameters and return values, because the type of $$x$$ depends on $$i$$.
* EDE1 - POPCTRX $$(x i-)$$, similar to POPCTR $$\mathrm{c}(i)$$, but with $$0 \leq i \leq 255$$ from the stack.
* EDE2 - SETCONTCTRX $$\left(x c i-c^{\prime}\right)$$, similar to SETCONTCTR $$c(i)$$, but with $$0 \leq i \leq 255$$ from the stack.
* EDFO - COMPOS or BOOLAND $$\left(c c^{\prime}-c^{\prime \prime}\right)$$, computes the composition $$c \circ_{0} c^{\prime}$$, which has the meaning of "perform $$c$$, and, if successful, perform $$c^{\prime}$$ " (if $$c$$ is a boolean circuit) or simply "perform $$c$$, then $$c$$ ". Equivalent to SWAP; SETCONT cO.
* EDF1 - COMPOSALT or BOOLOR $$\left(c c^{\prime}-c^{\prime \prime}\right)$$, computes the alternative composition $$c \circ_{1} c^{\prime}$$, which has the meaning of "perform $$c$$, and, if not successful, perform $$c^{\prime \prime \prime}$$ (if $$c$$ is a boolean circuit). Equivalent to SWAP; SETCONT c1.
* EDF2 - COMPOSBOTH $$\left(c c^{\prime}-c^{\prime \prime}\right)$$, computes $$\left(c \circ_{0} c^{\prime}\right) \circ_{1} c^{\prime}$$, which has the meaning of "compute boolean circuit $$c$$, then compute $$c^{\prime}$$, regardless of the result of $$c$$ ".
* EDF3 - ATEXIT $$(c-)$$, sets $$c 0 \leftarrow c \circ_{0} c 0$$. In other words, $$c$$ will be executed before exiting current subroutine.
* EDF4 - ATEXITALT $$(c-)$$, sets $$c 1 \leftarrow c \circ_{1} c 1$$. In other words, $$c$$ will be executed before exiting current subroutine by its alternative return path.
* EDF5 - SETEXITALT $$(c-)$$, sets $$c 1 \leftarrow\left(c \circ_{0} c 0\right) \circ_{1} c 1$$. In this way, a subsequent RETALT will first execute $$c$$, then transfer control to the original c0. This can be used, for instance, to exit from nested loops.
* EDF6 - THENRET $$\left(c-c^{\prime}\right)$$, computes $$c^{\prime}:=c \circ_{0}$$ c0
* EDF7 - THENRETALT $$\left(c-c^{\prime}\right)$$, computes $$c^{\prime}:=c \circ_{0} c 1$$
* EDF8 - INVERT ( - ), interchanges c0 and c1.
* EDF9 - BOOLEVAL $$(c-?)$$, performs cc $$\leftarrow\left(c \circ_{0}\left((\mathrm{PUSH}-1) \circ_{0} \mathrm{cc}\right)\right) \circ_{1}$$ $$\left((\right.$$ PUSH 0$$\left.) \circ_{0} \mathrm{Cc}\right)$$. If $$c$$ represents a boolean circuit, the net effect is to evaluate it and push either -1 or 0 into the stack before continuing.
* EDFA - SAMEALT $$(-)$$, sets $$c_{1}:=c_{0}$$. Equivalent to PUSH c0;POP c1.
* EDFB - SAMEALTSAVE $$(-)$$, sets $$c_{1}:=c_{0}$$, but first saves the old value of $$c_{1}$$ into the savelist of $$c_{0}$$. Equivalent to SAVE $$c 1$$; SAMEALT.
* EErn - BLESSARGS $$r, n\left(x_{1} \ldots x_{r} s-c\right)$$, described in [$$\mathbf{A.8.4}$$](a-instructions-and-opcodes.md#a.8.4.-manipulating-the-stack-of-continuations.).

### A.8.7. Dictionary subroutine calls and jumps.

* F0n - CALL $$n$$ or CALLDICT $$n(-n)$$, calls the continuation in c3, pushing integer $$0 \leq n \leq 255$$ into its stack as an argument. Approximately equivalent to PUSHINT $$n$$; PUSH c3; EXECUTE.
* F12\_ $$n-$$ CALL $$n$$ for $$0 \leq n<2^{14}(-n)$$, an encoding of CALL $$n$$ for larger values of $$n$$. - F16\_ $$n$$ - JMP $$n$$ or JMPDICT $$n(-n)$$, jumps to the continuation in c3, pushing integer $$0 \leq n<2^{14}$$ as its argument. Approximately equivalent to PUSHINT $$n$$; PUSH c3; JMPX.
* F1A\_ $$n$$ - PREPARE $$n$$ or PREPAREDICT $$n(-n c)$$, equivalent to PUSHINT $$n$$; PUSH c3, for $$0 \leq n<2^{14}$$. In this way, CALL $$n$$ is approximately equivalent to PREPARE $$n$$; EXECUTE, and JMP $$n$$ is approximately equivalent to PREPARE $$n$$; JMPX. One might use, for instance, CALLARGS or CALLCC instead of EXECUTE here.

## A.9 Exception generating and handling primitives

### A.9.1. Throwing exceptions.

* F22\_nn-THROW $$n n(-0 n n)$$, throws exception $$0 \leq n n \leq 63$$ with parameter zero. In other words, it transfers control to the continuation in c2, pushing 0 and $$n n$$ into its stack, and discarding the old stack altogether.
* F26\_nn - THROWIF $$n n(f-)$$, throws exception $$0 \leq n n \leq 63$$ with parameter zero only if integer $$f \neq 0$$.
* F2A\_nn-THROWIFNOT $$n n(f-)$$, throws exception $$0 \leq n n \leq 63$$ with parameter zero only if integer $$f=0$$.
* F2C4\_nn - THROW $$n n$$ for $$0 \leq n n<2^{11}$$, an encoding of THROW $$n n$$ for larger values of $$n n$$.
* F2CC\_nn - THROWARG $$n n(x-x n n)$$, throws exception $$0 \leq n n<$$ $$2^{11}$$ with parameter $$x$$, by copying $$x$$ and $$n n$$ into the stack of $$c 2$$ and transferring control to c2.
* F2D4\_nn-THROWIF $$n n(f-)$$ for $$0 \leq n n<2^{11}$$.
* F2DC\_ $$n n-$$ THROWARGIF $$n n(x f-)$$, throws exception $$0 \leq n n<2^{11}$$ with parameter $$x$$ only if integer $$f \neq 0$$.
* F2E4\_nn-THROWIFNOT $$n n(f-)$$ for $$0 \leq n n<2^{11}$$.
* F2EC\_nn - THROWARGIFNOT $$n n(x f-)$$, throws exception $$0 \leq n n<$$ $$2^{11}$$ with parameter $$x$$ only if integer $$f=0$$. - F2F0 - THROWANY $$(n-0 n)$$, throws exception $$0 \leq n<2^{16}$$ with parameter zero. Approximately equivalent to PUSHINT 0; SWAP; THROWARGANY.
* F2F1 - THROWARGANY $$(x n-x n)$$, throws exception $$0 \leq n<2^{16}$$ with parameter $$x$$, transferring control to the continuation in c2. Approximately equivalent to PUSH c2; JMPXARGS 2.
* F2F2 - THROWANYIF ( $$n f-)$$, throws exception $$0 \leq n<2^{16}$$ with parameter zero only if $$f \neq 0$$.
* F2F3 - THROWARGANYIF $$(x n f-)$$, throws exception $$0 \leq n<2^{16}$$ with parameter $$x$$ only if $$f \neq 0$$.
* F2F4 - THROWANYIFNOT ( $$n f-)$$, throws exception $$0 \leq n<2^{16}$$ with parameter zero only if $$f=0$$.
* F2F5 - THROWARGANYIFNOT $$(x n f-)$$, throws exception $$0 \leq n<2^{16}$$ with parameter $$x$$ only if $$f=0$$.

### A.9.2. Catching and handling exceptions.

* F2FF - TRY $$\left(c c^{\prime}-\right)$$, sets c2 to $$c^{\prime}$$, first saving the old value of c2 both into the savelist of $$c^{\prime}$$ and into the savelist of the current continuation, which is stored into $$c . c 0$$ and $$c^{\prime} . c 0$$. Then runs $$c$$ similarly to EXECUTE. If $$c$$ does not throw any exceptions, the original value of c2 is automatically restored on return from $$c$$. If an exception occurs, the execution is transferred to $$c^{\prime}$$, but the original value of c2 is restored in the process, so that $$c^{\prime}$$ can re-throw the exception by THROWANY if it cannot handle it by itself.
* F3pr - TRYARGS $$p, r\left(c c^{\prime}-\right)$$, similar to TRY, but with CALLARGS $$p, r$$ internally used instead of EXECUTE. In this way, all but the top $$0 \leq p \leq 15$$ stack elements will be saved into current continuation's stack, and then restored upon return from either $$c$$ or $$c^{\prime}$$, with the top $$0 \leq r \leq 15$$ values of the resulting stack of $$c$$ or $$c^{\prime}$$ copied as return values.

## A.10 Dictionary manipulation primitives

TVM's dictionary support is discussed at length in [$$\mathbf{3.3}$$](\[\$$/mathbf%7B3.3%7D\$$]\(cells-memory-and-persistent-storage.md#3.3-hashmaps-or-dictionaries\)) The basic operations with dictionaries are listed in [$$\mathbf{3.3.10}$$](cells-memory-and-persistent-storage.md#3.3.10.-basic-dictionary-operations.), while the taxonomy of dictionary manipulation primitives is provided in [$$\mathbf{3.3.11}$$](cells-memory-and-persistent-storage.md#3.3.11.-taxonomy-of-dictionary-primitives.) Here we use the concepts and notation introduced in those sections.

Dictionaries admit two different representations as TVM stack values:

* A Slice $$s$$ with a serialization of a TL-B value of type $$\operatorname{Hashmap} E(n, X)$$. In other words, $$s$$ consists either of one bit equal to zero (if the dictionary is empty), or of one bit equal to one and a reference to a Cell containing the root of the binary tree, i.e., a serialized value of type $$\operatorname{Hashmap}(n, X)$$.
* A "maybe Cell" c? , i.e., a value that is either a Cell (containing a serialized value of type $$\operatorname{Hashmap}(n, X)$$ as before) or a $$N u l l$$ (corresponding to an empty dictionary). When a "maybe Cell" $$c$$ ? is used to represent a dictionary, we usually denote it by $$D$$ in the stack notation.

Most of the dictionary primitives listed below accept and return dictionaries in the second form, which is more convenient for stack manipulation. However, serialized dictionaries inside larger TL-B objects use the first representation.

Opcodes starting with F4 and F5 are reserved for dictionary operations.

### A.10.1. Dictionary creation.

* 6D - NEWDICT $$(-D)$$, returns a new empty dictionary. It is an alternative mnemonics for PUSHNULL, cf. [$$\mathbf{A.3.1}$$](a-instructions-and-opcodes.md#a.3.1-null-primitives.).
* 6E - DICTEMPTY $$(D-?)$$, checks whether dictionary $$D$$ is empty, and returns -1 or 0 accordingly. It is an alternative mnemonics for ISNULL, cf. [$$\mathbf{A.3.1}$$](a-instructions-and-opcodes.md#a.3.1-null-primitives.).

### A.10.2. Dictionary serialization and deserialization.

* CE - STDICTS $$\left(s b-b^{\prime}\right)$$, stores a Slice-represented dictionary $$s$$ into Builder $$b$$. It is actually a synonym for STSLICE.
* F400 - STDICT or STOPTREF $$\left(D b-b^{\prime}\right)$$, stores dictionary $$D$$ into Builder $$b$$, returing the resulting Builder $$b^{\prime}$$. In other words, if $$D$$ is a cell, performs STONE and STREF; if $$D$$ is $$N u l l$$, performs NIP and STZERO; otherwise throws a type checking exception.
* F401 - SKIPDICT or SKIPOPTREF $$\left(s-s^{\prime}\right)$$, equivalent to LDDICT; NIP. - F402 - LDDICTS $$\left(s-s^{\prime} s^{\prime \prime}\right)$$, loads (parses) a (Slice-represented) dictionary $$s^{\prime}$$ from Slice $$s$$, and returns the remainder of $$s$$ as $$s^{\prime \prime}$$. This is a "split function" for all HashmapE $$(n, X)$$ dictionary types.
* F403 - PLDDICTS $$\left(s-s^{\prime}\right)$$, preloads a (Slice-represented) dictionary $$s^{\prime}$$ from Slice s. Approximately equivalent to LDDICTS; DROP.
* F404 - LDDICT or LDOPTREF $$\left(s-D s^{\prime}\right)$$, loads (parses) a dictionary $$D$$ from Slice $$s$$, and returns the remainder of $$s$$ as $$s^{\prime}$$. May be applied to dictionaries or to values of arbitrary $$\left({ }^{\wedge} Y\right)^{?}$$ types.
* F405 - PLDDICT or PLDOPTREF $$(s-D)$$, preloads a dictionary $$D$$ from Slice s. Approximately equivalent to LDDICT; DROP.
* F406 - LDDICTQ $$\left(s-D s^{\prime}-1\right.$$ or $$\left.s 0\right)$$, a quiet version of LDDICT.
* F407 - PLDDICTQ $$(s-D-1$$ or 0$$)$$, a quiet version of PLDDICT.

### A.10.3. GET dictionary operations.

* F40A - DICTGET ( $$k D n-x-1$$ or 0$$)$$, looks up key $$k$$ (represented by a Slice, the first $$0 \leq n \leq 1023$$ data bits of which are used as a key) in dictionary $$D$$ of type Hashmap $$E(n, X)$$ with $$n$$-bit keys. On success, returns the value found as a Slice $$x$$.
* F4OB - DICTGETREF ( $$k D n-c-1$$ or 0$$)$$, similar to DICTGET, but with a LDREF; ENDS applied to $$x$$ on success. This operation is useful for dictionaries of type $$\operatorname{Hashmap} E\left(n,{ }^{\wedge} Y\right)$$.
* F40C - DICTIGET ( $$i D n-x-1$$ or 0$$)$$, similar to DICTGET, but with a signed (big-endian) $$n$$-bit Integer $$i$$ as a key. If $$i$$ does not fit into $$n$$ bits, returns 0 . If $$i$$ is a NaN, throws an integer overflow exception.
* F4OD - DICTIGETREF ( $$i D n-c-1$$ or 0$$)$$, combines DICTIGET with DICTGETREF: it uses signed $$n$$-bit Integer $$i$$ as a key and returns a Cell instead of a Slice on success.
* F40E - DICTUGET ( $$i$$ D $$n-x-1$$ or 0$$)$$, similar to DICTIGET, but with unsigned (big-endian) $$n$$-bit Integer $$i$$ used as a key.
* F4OF - DICTUGETREF ( $$i D n-c-1$$ or 0$$)$$, similar to DICTIGETREF, but with an unsigned $$n$$-bit Integer key $$i$$.

### A.10.4. Set/RePlace/ADD dictionary operations.

The mnemonics of the following dictionary primitives are constructed in a systematic fashion according to the regular expression DICT\[, I, U] (SET, REPLACE, ADD) \[GET]\[REF] depending on the type of the key used (a Slice or a signed or unsigned Integer), the dictionary operation to be performed, and the way the values are accepted and returned (as Cells or as Slices). Therefore, we provide a detailed description only for some primitives, assuming that this information is sufficient for the reader to understand the precise action of the remaining primitives.

* F412 - DICTSET $$\left(x k D n-D^{\prime}\right)$$, sets the value associated with $$n$$-bit key $$k$$ (represented by a Slice as in DICTGET) in dictionary $$D$$ (also represented by a Slice) to value $$x$$ (again a Slice), and returns the resulting dictionary as $$D^{\prime}$$.
* F413 - DICTSETREF $$\left(c k D n-D^{\prime}\right)$$, similar to DICTSET, but with the value set to a reference to $$C e l l$$ c.
* F414 - DICTISET $$\left(x i D n-D^{\prime}\right)$$, similar to DICTSET, but with the key represented by a (big-endian) signed $$n$$-bit integer $$i$$. If $$i$$ does not fit into $$n$$ bits, a range check exception is generated.
* F415 - DICTISETREF ( c i D n-D $${ }^{\prime}$$ ), similar to DICTSETREF, but with the key a signed $$n$$-bit integer as in DICTISET.
* F416 - DICTUSET $$\left(x i D n-D^{\prime}\right)$$, similar to DICTISET, but with $$i$$ an unsigned $$n$$-bit integer.
* F417 - DICTUSETREF ( i $$i n-D^{\prime}$$ ), similar to DICTISETREF, but with $$i$$ unsigned.
* F41A - DICTSETGET $$\left(x k D n-D^{\prime} y-1\right.$$ or $$\left.D^{\prime} 0\right)$$, combines DICTSET with DICTGET: it sets the value corresponding to key $$k$$ to $$x$$, but also returns the old value $$y$$ associated with the key in question, if present.
* F41B - DICTSETGETREF ( $$c k D n-D^{\prime} c^{\prime}-1$$ or $$\left.D^{\prime} 0\right)$$, combines DICTSETREF with DICTGETREF similarly to DICTSETGET.
* F41C - DICTISETGET $$\left(x i D n-D^{\prime} y-1\right.$$ or $$\left.D^{\prime} 0\right)$$, similar to DICTSETGET, but with the key represented by a big-endian signed $$n$$-bit Integer $$i$$. - F41D - DICTISETGETREF ( c i D n- $$D^{\prime} c^{\prime}-1$$ or $$D^{\prime} 0$$ ), a version of DICTSETGETREF with signed Integer $$i$$ as a key.
* F41E - DICTUSETGET $$\left(x i D n-D^{\prime} y-1\right.$$ or $$\left.D^{\prime} 0\right)$$, similar to DICTISETGET, but with $$i$$ an unsigned $$n$$-bit integer.
* $$\mathrm{F} 41 \mathrm{~F}$$ - DICTUSETGETREF $$\left(c i D n-D^{\prime} c^{\prime}-1\right.$$ or $$\left.D^{\prime} 0\right)$$.
* F422 - DICTREPLACE $$\left(x k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$, a RePlaCE operation, which is similar to DICTSET, but sets the value of key $$k$$ in dictionary $$D$$ to $$x$$ only if the key $$k$$ was already present in $$D$$.
* F423 - DICTREPLACEREF ( $$c k D n-D^{\prime}-1$$ or $$\left.D 0\right)$$, a REPlaCE counterpart of DICTSETREF.
* F424 - DICTIREPLACE $$\left(x i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$, a version of DICTREPLACE with signed $$n$$-bit Integer $$i$$ used as a key.
* F425 - DICTIREPLACEREF $$\left(c i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F426 - DICTUREPLACE $$\left(x i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F427 - DICTUREPLACEREF ( c $$i D n-D^{\prime}-1$$ or $$D 0$$ ).
* F42A - DICTREPLACEGET $$\left(x k D n-D^{\prime} y-1\right.$$ or $$\left.D 0\right)$$, a REPlaCE counterpart of DICTSETGET: on success, also returns the old value associated with the key in question.
* F42B - DICTREPLACEGETREF $$\left(c k D n-D^{\prime} c^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* $$\mathrm{F} 42 \mathrm{C}$$ - DICTIREPLACEGET $$\left(x i D n-D^{\prime} y-1\right.$$ or $$\left.D 0\right)$$.
* F42D - DICTIREPLACEGETREF ( $$c i D n-D^{\prime} c^{\prime}-1$$ or $$\left.D 0\right)$$.
* $$\mathrm{F} 42 \mathrm{E}$$ - DICTUREPLACEGET $$\left(x i D n-D^{\prime} y-1\right.$$ or $$\left.D 0\right)$$.
* $$\mathrm{F} 42 \mathrm{~F}$$ - DICTUREPLACEGETREF ( $$c i D n-D^{\prime} c^{\prime}-1$$ or $$\left.D 0\right)$$.
* F432 - DICTADD $$\left(x k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$, an ADD counterpart of DICTSET: sets the value associated with key $$k$$ in dictionary $$D$$ to $$x$$, but only if it is not already present in $$D$$.
* $$\mathrm{F} 433$$ - DICTADDREF $$\left(c k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$. - F434 - DICTIADD $$\left(x i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F435 - DICTIADDREF $$\left(c i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F436 - DICTUADD $$\left(x i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F437 - DICTUADDREF $$\left(c i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F43A - DICTADDGET $$\left(x k D n-D^{\prime}-1\right.$$ or $$\left.D y 0\right)$$, an ADD counterpart of DICTSETGET: sets the value associated with key $$k$$ in dictionary $$D$$ to $$x$$, but only if key $$k$$ is not already present in $$D$$. Otherwise, just returns the old value $$y$$ without changing the dictionary.
* F43B - DICTADDGETREF ( $$c k D n-D^{\prime}-1$$ or $$\left.D c^{\prime} 0\right)$$, an ADD counterpart of DICTSETGETREF.
* F43C - DICTIADDGET $$\left(x i D n-D^{\prime}-1\right.$$ or $$\left.D y 0\right)$$.
* F43D - DICTIADDGETREF ( c $$i D n-D^{\prime}-1$$ or $$\left.D c^{\prime} 0\right)$$.
* F43E - DICTUADDGET $$\left(x i D n-D^{\prime}-1\right.$$ or $$\left.D y 0\right)$$.
* F43F - DICTUADDGETREF ( $$c i D n-D^{\prime}-1$$ or $$\left.D c^{\prime} 0\right)$$.

### A.10.5. Builder-accepting variants of SET dictionary operations.

The following primitives accept the new value as a Builder $$b$$ instead of a Slice $$x$$, which often is more convenient if the value needs to be serialized from several components computed in the stack. (This is reflected by appending a B to the mnemonics of the corresponding SET primitives that work with Slices.) The net effect is roughly equivalent to converting $$b$$ into a Slice by ENDC; CTOS and executing the corresponding primitive listed in [$$\mathbf{A.10.4}$$](a-instructions-and-opcodes.md#a.10.4.-set-replace-add-dictionary-operations.)

* F441 - DICTSETB $$\left(b k D n-D^{\prime}\right)$$.
* F442 - DICTISETB $$\left(b i D n-D^{\prime}\right)$$.
* F443 - DICTUSETB $$\left(b i D n-D^{\prime}\right)$$.
* F445 - DICTSETGETB $$\left(b k D n-D^{\prime} y-1\right.$$ or $$\left.D^{\prime} 0\right)$$.
* F446 - DICTISETGETB $$\left(b i D n-D^{\prime} y-1\right.$$ or $$\left.D^{\prime} 0\right)$$.
* F447 - DICTUSETGETB $$\left(b i D n-D^{\prime} y-1\right.$$ or $$D^{\prime} 0$$ ). - $$\mathrm{F} 449$$ - DICTREPLACEB $$\left(b k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F44A - DICTIREPLACEB ( $$b i D n-D^{\prime}-1$$ or $$\left.D 0\right)$$.
* F44B - DICTUREPLACEB ( $$b i D n-D^{\prime}-1$$ or $$D 0$$ ).
* F44D - DICTREPLACEGETB $$\left(b k D n-D^{\prime} y-1\right.$$ or $$\left.D 0\right)$$.
* F44E - DICTIREPLACEGETB (bi $$i n-D^{\prime} y-1$$ or $$\left.D 0\right)$$.
* F44F - DICTUREPLACEGETB (bi $$i n-D^{\prime} y-1$$ or $$\left.D 0\right)$$.
* F451 - DICTADDB $$\left(b k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F452 - DICTIADDB $$\left(b i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* $$\mathrm{F} 453$$ - DICTUADDB $$\left(b i D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F455 - DICTADDGETB $$\left(b k D n-D^{\prime}-1\right.$$ or $$\left.D y 0\right)$$.
* F456 - DICTIADDGETB ( $$b i D n-D^{\prime}-1$$ or $$\left.D y 0\right)$$.
* F457 - DICTUADDGETB ( $$b i D n-D^{\prime}-1$$ or $$\left.D y 0\right)$$.

### A.10.6. DeLETE dictionary operations.

* F459 - DICTDEL $$\left(k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$, deletes $$n$$-bit key, represented by a Slice $$k$$, from dictionary $$D$$. If the key is present, returns the modified dictionary $$D^{\prime}$$ and the success flag -1 . Otherwise, returns the original dictionary $$D$$ and 0 .
* F45A - DICTIDEL $$\left(i D n-D^{\prime}\right.$$ ?), a version of DICTDEL with the key represented by a signed $$n$$-bit Integer $$i$$. If $$i$$ does not fit into $$n$$ bits, simply returns $$D 0$$ ("key not found, dictionary unmodified").
* F45B - DICTUDEL $$\left(i D n-D^{\prime}\right.$$ ?), similar to DICTIDEL, but with $$i$$ an unsigned $$n$$-bit integer.
* F462 - DICTDELGET $$\left(k D n-D^{\prime} x-1\right.$$ or $$\left.D 0\right)$$, deletes $$n$$-bit key, represented by a Slice $$k$$, from dictionary $$D$$. If the key is present, returns the modified dictionary $$D^{\prime}$$, the original value $$x$$ associated with the key $$k$$ (represented by a Slice), and the success flag -1 . Otherwise, returns the original dictionary $$D$$ and 0 . - F463 - DICTDELGETREF $$\left(k D n-D^{\prime} c-1\right.$$ or $$\left.D 0\right)$$, similar to DICTDELGET, but with LDREF; ENDS applied to $$x$$ on success, so that the value returned $$c$$ is a Cell.
* F464 - DICTIDELGET ( $$i D n-D^{\prime} x-1$$ or $$\left.D 0\right)$$, a variant of primitive DICTDELGET with signed $$n$$-bit integer $$i$$ as a key.
* F465 - DICTIDELGETREF ( $$i D n-D^{\prime} c-1$$ or $$\left.D 0\right)$$, a variant of primitive DICTIDELGET returning a Cell instead of a Slice.
* F466 - DICTUDELGET ( $$i D n-D^{\prime} x-1$$ or $$D 0$$ ), a variant of primitive DICTDELGET with unsigned $$n$$-bit integer $$i$$ as a key.
* F467 - DICTUDELGETREF ( $$i D n-D^{\prime} c-1$$ or $$D$$ 0), a variant of primitive DICTUDELGET returning a Cell instead of a Slice.

### A.10.7. "Maybe reference" dictionary operations.

The following operations assume that a dictionary is used to store values $$c^{\text {? }}$$ of type $$C_{e l l}$$ ? ("Maybe Cell"), which can be used in particular to store dictionaries as values in other dictionaries. The representation is as follows: if $$c^{\text {? }}$$ is a Cell, it is stored as a value with no data bits and exactly one reference to this $$C$$ ell. If $$c^{?}$$ is $$N u l l$$, then the corresponding key must be absent from the dictionary altogether.

* F469 - DICTGETOPTREF $$\left(k D n-c^{?}\right)$$, a variant of DICTGETREF that returns $$N u l l$$ instead of the value $$c^{?}$$ if the key $$k$$ is absent from dictionary $$D$$.
* F46A - DICTIGETOPTREF ( $$i D n-c^{\text {? }}$$ ), similar to DICTGETOPTREF, but with the key given by signed $$n$$-bit Integer $$i$$. If the key $$i$$ is out of range, also returns $$N u l l$$.
* F46B - DICTUGETOPTREF ( $$i D n-c^{\text {? }}$$ ), similar to DICTGETOPTREF, but with the key given by unsigned $$n$$-bit Integer $$i$$.
* F46D - DICTSETGETOPTREF $$\left(c^{?} k D n-D^{\prime} \tilde{c}^{?}\right)$$, a variant of both DICTGETOPTREF and DICTSETGETREF that sets the value corresponding to key $$k$$ in dictionary $$D$$ to $$c^{?}$$ (if $$c^{?}$$ is $$N u l l$$, then the key is deleted instead), and returns the old value $$\tilde{c}^{?}$$ (if the key $$k$$ was absent before, returns $$N u l l$$ instead). - F46E - DICTISETGETOPTREF $$\left(c^{?} i D n-D^{\prime} \tilde{c}^{?}\right)$$, similar to primitive DICTSETGETOPTREF, but using signed $$n$$-bit Integer $$i$$ as a key. If $$i$$ does not fit into $$n$$ bits, throws a range checking exception.
* F46F - DICTUSETGETOPTREF $$\left(c^{?} i D n-D^{\prime} \tilde{c}^{?}\right)$$, similar to primitive DICTSETGETOPTREF, but using unsigned $$n$$-bit Integer $$i$$ as a key.

### A.10.8. Prefix code dictionary operations.

These are some basic operations for constructing prefix code dictionaries (cf. [$$\mathbf{3.4.2}$$](cells-memory-and-persistent-storage.md#3.4.2.-serialization-of-prefix-codes.)). The primary application for prefix code dictionaries is deserializing TL-B serialized data structures, or, more generally, parsing prefix codes. Therefore, most prefix code dictionaries will be constant and created at compile time, not by the following primitives.

Some GET operations for prefix code dictionaries may be found in [$$\mathbf{A.10.11}$$](a-instructions-and-opcodes.md#a.10.11.-special-get-dictionary-and-prefix-code-dictionary-operations-and-constant-dictionaries.) Other prefix code dictionary operations include:

* F470 - PFXDICTSET $$\left(x k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F471 - PFXDICTREPLACE $$\left(x k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F472 - PFXDICTADD $$\left(x k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.
* F473 - PFXDICTDEL $$\left(k D n-D^{\prime}-1\right.$$ or $$\left.D 0\right)$$.

These primitives are completely similar to their non-prefix code counterparts DICTSET etc (cf. [$$\mathbf{A.10.4}$$](a-instructions-and-opcodes.md#a.10.4.-set-replace-add-dictionary-operations.)), with the obvious difference that even a SET may fail in a prefix code dictionary, so a success flag must be returned by PFXDICTSET as well.

### A.10.9. Variants of GetNext and GetPreV operations.

* F474 - DICTGETNEXT ( $$k D n-x^{\prime} k^{\prime}-1$$ or 0 ), computes the minimal key $$k^{\prime}$$ in dictionary $$D$$ that is lexicographically greater than $$k$$, and returns $$k^{\prime}$$ (represented by a Slice) along with associated value $$x^{\prime}$$ (also represented by a Slice).
* F475 - DICTGETNEXTEQ ( $$k D n-x^{\prime} k^{\prime}-1$$ or 0$$)$$, similar to DICTGETNEXT, but computes the minimal key $$k^{\prime}$$ that is lexicographically greater than or equal to $$k$$.
* F476 - DICTGETPREV ( $$k D n-x^{\prime} k^{\prime}-1$$ or 0$$)$$, similar to DICTGETNEXT, but computes the maximal key $$k^{\prime}$$ lexicographically smaller than $$k$$. - F477 - DICTGETPREVEQ ( $$k D n-x^{\prime} k^{\prime}-1$$ or 0$$)$$, similar to DICTGETPREV, but computes the maximal key $$k^{\prime}$$ lexicographically smaller than or equal to $$k$$.
* F478 - DICTIGETNEXT ( $$i D n-x^{\prime} i^{\prime}-1$$ or 0$$)$$, similar to DICTGETNEXT, but interprets all keys in dictionary $$D$$ as big-endian signed $$n$$-bit integers, and computes the minimal key $$i^{\prime}$$ that is larger than Integer $$i$$ (which does not necessarily fit into $$n$$ bits).
* F479 - DICTIGETNEXTEQ $$\left(i D n-x^{\prime} i^{\prime}-1\right.$$ or 0$$)$$.
* F47A - DICTIGETPREV $$\left(i D n-x^{\prime} i^{\prime}-1\right.$$ or 0$$)$$.
* F47B - DICTIGETPREVEQ $$\left(i D n-x^{\prime} i^{\prime}-1\right.$$ or 0$$)$$.
* F47C - DICTUGETNEXT ( $$i D n-x^{\prime} i^{\prime}-1$$ or 0$$)$$, similar to DICTGETNEXT, but interprets all keys in dictionary $$D$$ as big-endian unsigned $$n$$-bit integers, and computes the minimal key $$i^{\prime}$$ that is larger than Integer $$i$$ (which does not necessarily fit into $$n$$ bits, and is not necessarily nonnegative).
* F47D - DICTUGETNEXTEQ $$\left(i D n-x^{\prime} i^{\prime}-1\right.$$ or 0$$)$$.
* F47E - DICTUGETPREV $$\left(i D n-x^{\prime} i^{\prime}-1\right.$$ or 0$$)$$.
* F47F - DICTUGETPREVEQ $$\left(i D n-x^{\prime} i^{\prime}-1\right.$$ or 0$$)$$.

### A.10.10. GetMin, GetMax, RemoveMin, RemoveMax operations.

* F482 - DICTMIN ( $$D n-x k-1$$ or 0$$)$$, computes the minimal key $$k$$ (represented by a Slice with $$n$$ data bits) in dictionary $$D$$, and returns $$k$$ along with the associated value $$x$$.
* F483 - DICTMINREF ( $$D n-c k-1$$ or 0$$)$$, similar to DICTMIN, but returns the only reference in the value as a Cell c.
* F484 - DICTIMIN ( $$D n-x i-1$$ or 0 ), somewhat similar to DICTMIN, but computes the minimal key $$i$$ under the assumption that all keys are big-endian signed $$n$$-bit integers. Notice that the key and value returned may differ from those computed by DICTMIN and DICTUMIN. - F485 - DICTIMINREF $$(D n-c i-1$$ or 0$$)$$.
* F486 - DICTUMIN ( $$D n-x i-1$$ or 0 ), similar to DICTMIN, but returns the key as an unsigned $$n$$-bit Integer $$i$$.
* F487 - DICTUMINREF $$(D n-c i-1$$ or 0$$)$$.
* F48A - DICTMAX ( $$D n-x k-1$$ or 0$$)$$, computes the maximal key $$k$$ (represented by a Slice with $$n$$ data bits) in dictionary $$D$$, and returns $$k$$ along with the associated value $$x$$.
* F48B - DICTMAXREF (Dn-ck-1 or 0$$)$$.
* F48C - DICTIMAX $$(D n-x i-1$$ or 0$$)$$.
* F48D - DICTIMAXREF $$(D n-c i-1$$ or 0$$)$$.
* F48E - DICTUMAX $$(D n-x i-1$$ or 0$$)$$.
* F48F - DICTUMAXREF $$(D n-c i-1$$ or 0$$)$$.
* F492 - DICTREMMIN ( $$D n-D^{\prime} x k-1$$ or $$D 0$$ ), computes the minimal key $$k$$ (represented by a Slice with $$n$$ data bits) in dictionary $$D$$, removes $$k$$ from the dictionary, and returns $$k$$ along with the associated value $$x$$ and the modified dictionary $$D^{\prime}$$.
* F493 - DICTREMMINREF ( $$D n-D^{\prime} c k-1$$ or $$D 0$$ ), similar to DICTREMMIN, but returns the only reference in the value as a Cell c.
* F494 - DICTIREMMIN ( $$D n-D^{\prime} x i-1$$ or $$\left.D 0\right)$$, somewhat similar to DICTREMMIN, but computes the minimal key $$i$$ under the assumption that all keys are big-endian signed $$n$$-bit integers. Notice that the key and value returned may differ from those computed by DICTREMMIN and DICTUREMMIN.
* F495 - DICTIREMMINREF $$\left(D n-D^{\prime} c i-1\right.$$ or $$\left.D 0\right)$$.
* F496 - DICTUREMMIN ( $$D n-D^{\prime} x i-1$$ or $$D 0$$ ), similar to DICTREMMIN, but returns the key as an unsigned $$n$$-bit Integer $$i$$.
* F497 - DICTUREMMINREF $$\left(D n-D^{\prime} c i-1\right.$$ or $$\left.D 0\right)$$. - F49A - DICTREMMAX ( $$D n-D^{\prime} x k-1$$ or $$\left.D 0\right)$$, computes the maximal key $$k$$ (represented by a Slice with $$n$$ data bits) in dictionary $$D$$, removes $$k$$ from the dictionary, and returns $$k$$ along with the associated value $$x$$ and the modified dictionary $$D^{\prime}$$.
* F49B - DICTREMMAXREF $$\left(D n-D^{\prime} c k-1\right.$$ or $$\left.D 0\right)$$.
* F49C - DICTIREMMAX $$\left(D n-D^{\prime} x i-1\right.$$ or $$\left.D 0\right)$$.
* F49D - DICTIREMMAXREF $$\left(D n-D^{\prime} c i-1\right.$$ or $$\left.D 0\right)$$.
* F49E - DICTUREMMAX $$\left(D n-D^{\prime} x i-1\right.$$ or $$\left.D 0\right)$$.
* F49F - DICTUREMMAXREF $$\left(D n-D^{\prime} c i-1\right.$$ or $$\left.D 0\right)$$.

### A.10.11. Special GET dictionary and prefix code dictionary operations, and constant dictionaries.

* F4A0 - DICTIGETJMP $$(i D n-)$$, similar to DICTIGET (cf. [$$\mathbf{A.10.12}$$](a-instructions-and-opcodes.md#a.10.12.-subdict-dictionary-operations.)), but with $$x$$ BLESSed into a continuation with a subsequent JMPX to it on success. On failure, does nothing. This is useful for implementing switch/case constructions.- F4A1 - DICTUGETJMP $$(i D n-)$$, similar to DICTIGETJMP, but performs DICTUGET instead of DICTIGET.
* F4A2 - DICTIGETEXEC $$(i D n-)$$, similar to DICTIGETJMP, but with EXECUTE instead of JMPX.
* F4A3 - DICTUGETEXEC $$(i D n-)$$, similar to DICTUGETJMP, but with EXECUTE instead of JMPX.
* F4A6\_ $$n-$$ DICTPUSHCONST $$n(-D n)$$, pushes a non-empty constant ![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-126.jpg?height=57\&width=1260\&top\_left\_y=1934\&top\_left\_x=476) stored as a part of the instruction. The dictionary itself is created from the first of remaining references of the current continuation. In this way, the complete DICTPUSHCONST instruction can be obtained by first serializing $$\mathrm{xF}_{4} \mathrm{A8}_{-}$$, then the non-empty dictionary itself (one 1 bit and a cell reference), and then the unsigned 10-bit integer $$n$$ (as if by a STU 10 instruction). An empty dictionary can be pushed by a NEWDICT primitive (cf. [$$\mathbf{A.10.1}$$](a-instructions-and-opcodes.md#a.10.1.-dictionary-creation.)) instead.
* F4A8 - PFXDICTGETQ ( $$s D n-s^{\prime} x s^{\prime \prime}-1$$ or $$s$$ ), looks up the unique prefix of Slice $$s$$ present in the prefix code dictionary (cf. [$$\mathbf{3.4.2}$$](cells-memory-and-persistent-storage.md#3.4.2.-serialization-of-prefix-codes.)) represented by $$C e l l^{?} D$$ and $$0 \leq n \leq 1023$$. If found, the prefix of $$s$$ is returned as $$s^{\prime}$$, and the corresponding value (also a Slice) as $$x$$. The remainder of $$s$$ is returned as a Slice $$s^{\prime \prime}$$. If no prefix of $$s$$ is a key in prefix code dictionary $$D$$, returns the unchanged $$s$$ and a zero flag to indicate failure.
* F4A9 - PFXDICTGET ( $$\left.s D n-s^{\prime} x s^{\prime \prime}\right)$$, similar to PFXDICTGET, but throws a cell deserialization failure exception on failure.
* F4AA - PFXDICTGETJMP ( $$s D n-s^{\prime} s^{\prime \prime}$$ or $$\left.s\right)$$, similar to PFXDICTGETQ, but on success BLESSes the value $$x$$ into a Continuation and transfers control to it as if by a JMPX. On failure, returns $$s$$ unchanged and continues execution.
* F4AB - PFXDICTGETEXEC $$\left(s D n-s^{\prime} s^{\prime \prime}\right)$$, similar to PFXDICTGETJMP, but EXECutes the continuation found instead of jumping to it. On failure, throws a cell deserialization exception.
* F4AE\_ $$n$$ - PFXDICTCONSTGETJMP $$n$$ or PFXDICTSWITCH $$n\left(s-s^{\prime} s^{\prime \prime}\right.$$ or $$s)$$, combines DICTPUSHCONST $$n$$ for $$0 \leq n \leq 1023$$ with PFXDICTGETJMP.
* F4BC - DICTIGETJMPZ ( $$i D n-i$$ or nothing), a variant of DICTIGETJMP that returns index $$i$$ on failure.
* F4BD - DICTUGETJMPZ ( $$i D n-i$$ or nothing), a variant of DICTUGETJMP that returns index $$i$$ on failure.
* F4BE - DICTIGETEXECZ $$(i D n-i$$ or nothing), a variant of DICTIGETEXEC that returns index $$i$$ on failure.
* F4BF - DICTUGETEXECZ ( $$i D n-i$$ or nothing), a variant of DICTUGETEXEC that returns index $$i$$ on failure.

### A.10.12. SuBDiCT dictionary operations.

* F4B1 - SUBDICTGET $$\left(k l D n-D^{\prime}\right)$$, constructs a subdictionary consisting of all keys beginning with prefix $$k$$ (represented by a Slice, the first $$0 \leq l \leq n \leq 1023$$ data bits of which are used as a key) of length $$l$$ in dictionary $$D$$ of type $$\operatorname{Hashmap} E(n, X)$$ with $$n$$-bit keys. On success, returns the new subdictionary of the same type $$\operatorname{HashmapE}(n, X)$$ as a Slice $$D^{\prime}$$.
* F4B2 - SUBDICTIGET $$\left(x l D n-D^{\prime}\right)$$, variant of SUBDICTGET with the prefix represented by a signed big-endian $$l$$-bit Integer $$x$$, where necessarily $$l \leq 257$$.
* F4B3 - SUBDICTUGET $$\left(x l D n-D^{\prime}\right)$$, variant of SUBDICTGET with the prefix represented by an unsigned big-endian $$l$$-bit Integer $$x$$, where necessarily $$l \leq 256$$
* F4B5 - SUBDICTRPGET $$\left(k l D n-D^{\prime}\right)$$, similar to SUBDICTGET, but removes the common prefix $$k$$ from all keys of the new dictionary $$D^{\prime}$$, which becomes of type HashmapE $$(n-l, X)$$.
* F4B6 - SUBDICTIRPGET $$\left(x l D n-D^{\prime}\right)$$, variant of SUBDICTRPGET with the prefix represented by a signed big-endian $$l$$-bit Integer $$x$$, where necessarily $$l \leq 257$$.
* F4B7 - SUBDICTURPGET $$\left(x l D n-D^{\prime}\right)$$, variant of SUBDICTRPGET with the prefix represented by an unsigned big-endian $$l$$-bit Integer $$x$$, where necessarily $$l \leq 256$$.
* F4BC-F4BF - used by DICT ... Z primitives in A.10.11

## A.11 Application-specific primitives

Opcode range F8... FB is reserved for the application-specific primitives. When TVM is used to execute TVM smart contracts, these applicationspecific primitives are in fact TVM Blockchain-specific.

A.11.1. External actions and access to blockchain configuration data. Some of the primitives listed below pretend to produce some externally visible actions, such as sending a message to another smart contract. In fact, the execution of a smart contract in TVM never has any effect apart from a modification of the TVM state. All external actions are collected into a linked list stored in special register c5 ("output actions"). Additionally, some primitives use the data kept in the first component of the Tuple stored in c7 ("root of temporary data", cf. $$\mathbf{1 . 3 . 2}$$ ). Smart contracts are free to modify any other data kept in the cell $$c 7$$, provided the first reference remains intact (otherwise some application-specific primitives would be likely to throw exceptions when invoked).

Most of the primitives listed below use 16-bit opcodes.

### A.11.2. Gas-related primitives.

Of the following primitives, only the first two are "pure" in the sense that they do not use c5 or c7.

* F800 - ACCEPT, sets current gas limit $$g_{l}$$ to its maximal allowed value $$g_{m}$$, and resets the gas credit $$g_{c}$$ to zero (cf. $$\mathbf{1 . 4}$$ ), decreasing the value of $$g_{r}$$ by $$g_{c}$$ in the process. In other words, the current smart contract agrees to buy some gas to finish the current transaction. This action is required to process external messages, which bring no value (hence no gas) with themselves.
* F801 - SETGASLIMIT $$(g-)$$, sets current gas limit $$g_{l}$$ to the minimum of $$g$$ and $$g_{m}$$, and resets the gas credit $$g_{c}$$ to zero. If the gas consumed so far (including the present instruction) exceeds the resulting value of $$g_{l}$$, an (unhandled) out of gas exception is thrown before setting new gas limits. Notice that SETGASLIMIT with an argument $$g \geq 2^{63}-1$$ is equivalent to ACCEPT.
* F802 - BUYGAS $$(x-)$$, computes the amount of gas that can be bought for $$x$$ nanograms, and sets $$g_{l}$$ accordingly in the same way as SETGASLIMIT.
* F804 - GRAMTOGAS $$(x-g)$$, computes the amount of gas that can be bought for $$x$$ nanograms. If $$x$$ is negative, returns 0 . If $$g$$ exceeds $$2^{63}-1$$, it is replaced with this value.
* F805 - GASTOGRAM $$(g-x)$$, computes the price of $$g$$ gas in nanograms.
* F806-F80E - Reserved for gas-related primitives.
* F80F - COMMIT ( - ), commits the current state of registers c4 ("persistent data") and c5 ("actions") so that the current execution is considered "successful" with the saved values even if an exception is thrown later.

### A.11.3. Pseudo-random number generator primitives.

The pseudorandom number generator uses the random seed (parameter $$\# 6$$, cf. A.11.4), an unsigned 256-bit Integer, and (sometimes) other data kept in c7. The initial value of the random seed before a smart contract is executed in TVM Blockchain is a hash of the smart contract address and the global block random seed. If there are several runs of the same smart contract inside a block, then all of these runs will have the same random seed. This can be fixed, for example, by running LTIME; ADDRAND before using the pseudorandom number generator for the first time.

* F810 - RANDU256 $$(-x)$$, generates a new pseudo-random unsigned 256-bit Integer $$x$$. The algorithm is as follows: if $$r$$ is the old value of the random seed, considered as a 32-byte array (by constructing the big-endian representation of an unsigned 256-bit integer), then its SHA5 $$12(r)$$ is computed; the first 32 bytes of this hash are stored as the new value $$r^{\prime}$$ of the random seed, and the remaining 32 bytes are returned as the next random value $$x$$.
* F811 - RAND $$(y-z)$$, generates a new pseudo-random integer $$z$$ in the range $$0 \ldots y-1$$ (or $$y \ldots-1$$, if $$y<0$$ ). More precisely, an unsigned random value $$x$$ is generated as in RAND256U; then $$z:=\left\lfloor x y / 2^{256}\right\rfloor$$ is computed. Equivalent to RANDU256; MULRSHIFT 256.
* F814 - SETRAND $$(x-)$$, sets the random seed to unsigned 256-bit Integer $$x$$.
* F815 - ADDRAND $$(x-)$$, mixes unsigned 256-bit Integer $$x$$ into the random seed $$r$$ by setting the random seed to SHA256 of the concatenation of two 32-byte strings: the first with the big-endian representation of the old seed $$r$$, and the second with the big-endian representation of $$x$$.
* F810-F81F - Reserved for pseudo-random number generator primitives.

### A.11.4. Configuration primitives.

The following primitives read configuration data provided in the Tuple stored in the first component of the Tuple at c7. Whenever TVM is invoked for executing TVM Blockchain smart contracts, this Tuple is initialized by a SmartContractInfo structure; configuration primitives assume that it has remained intact.

* F82i - GETPARAM $$i(-x)$$, returns the $$i$$-th parameter from the Tuple provided at $$c 7$$ for $$0 \leq i<16$$. Equivalent to PUSH $$c 7$$; FIRST; INDEX $$i$$. If one of these internal operations fails, throws an appropriate type checking or range checking exception. - F823 - NOW $$(-x)$$, returns the current Unix time as an Integer. If it is impossible to recover the requested value starting from $$c 7$$, throws a type checking or range checking exception as appropriate. Equivalent to GETPARAM 3.
* F824 - BLOCKLT $$(-x)$$, returns the starting logical time of the current block. Equivalent to GETPARAM 4.
* F825 - LTIME $$(-x)$$, returns the logical time of the current transaction. Equivalent to GETPARAM 5.
* F826 - RANDSEED $$(-x)$$, returns the current random seed as an unsigned 256-bit Integer. Equivalent to GETPARAM 6.
* F827 - BALANCE $$(-t)$$, returns the remaining balance of the smart contract as a Tuple consisting of an Integer (the remaining Gram balance in nanograms) and a Maybe Cell (a dictionary with 32-bit keys representing the balance of "extra currencies"). Equivalent to GETPARAM 7. Note that RAW primitives such as SENDRAWMSG do not update this field.
* F828 - MYADDR $$(-s)$$, returns the internal address of the current smart contract as a Slice with a MsgAddress Int. If necessary, it can be parsed further using primitives such as PARSESTDADDR or REWRITESTDADDR. Equivalent to GETPARAM 8.
* F829 - CONFIGROOT $$(-D)$$, returns the Maybe Cell D with the current global configuration dictionary. Equivalent to GETPARAM 9.
* F830 - CONFIGDICT ( - D 32), returns the global configuration dictionary along with its key length (32). Equivalent to CONFIGROOT; PUSHINT 32.
* F832 - CONFIGPARAM $$(i-c-1$$ or 0$$)$$, returns the value of the global configuration parameter with integer index $$i$$ as a Cell c, and a flag to indicate success. Equivalent to CONFIGDICT; DICTIGETREF.
* F833 - CONFIGOPTPARAM $$\left(i-c^{?}\right)$$, returns the value of the global configuration parameter with integer index $$i$$ as a Maybe Cell $$c^{?}$$. Equivalent to CONFIGDICT; DICTIGETOPTREF. - F820-F83F - Reserved for configuration primitives.

A.11.5. Global variable primitives. The "global variables" may be helpful in implementing some high-level smart-contract languages. They are in fact stored as components of the Tuple at c7: the $$k$$-th global variable simply is the $$k$$-th component of this Tuple, for $$1 \leq k \leq 254$$. By convention, the 0 -th component is used for the "configuration parameters" of A.11.4, so it is not available as a global variable.

* F840 - GETGLOBVAR $$(k-x)$$, returns the $$k$$-th global variable for $$0 \leq$$ $$k<255$$. Equivalent to PUSH c7; SWAP; INDEXVARQ (cf. A.3.2.
* F85\_ $$k$$ - GETGLOB $$k(-x)$$, returns the $$k$$-th global variable for $$1 \leq$$ $$k \leq 31$$. Equivalent to PUSH $$c 7 ;$$ INDEXQ $$k$$.
* F860 - SETGLOBVAR $$(x k-)$$, assigns $$x$$ to the $$k$$-th global variable for $$0 \leq k<255$$. Equivalent to PUSH c7; ROTREV; SETINDEXVARQ; POP c7.
* F87\_ $$k$$ - SETGLOB $$k(x-)$$, assigns $$x$$ to the $$k$$-th global variable for $$1 \leq k \leq 31$$. Equivalent to PUSH $$c 7$$; SWAP; SETINDEXQ $$k$$; POP $$c 7$$

### A.11.6. Hashing and cryptography primitives.

* F900 - HASHCU $$(c-x)$$, computes the representation hash (cf. 3.1.5) of a Cell $$c$$ and returns it as a 256-bit unsigned integer $$x$$. Useful for signing and checking signatures of arbitrary entities represented by a tree of cells.
* F901 - HASHSU $$(s-x)$$, computes the hash of a Slice $$s$$ and returns it as a 256-bit unsigned integer $$x$$. The result is the same as if an ordinary cell containing only data and references from $$s$$ had been created and its hash computed by HASHCU.
* F902 - SHA256U $$(s-x)$$, computes SHA256 of the data bits of Slice s. If the bit length of $$s$$ is not divisible by eight, throws a cell underflow exception. The hash value is returned as a 256-bit unsigned integer $$x$$.
* F910 - CHKSIGNU ( $$h s k$$-?), checks the Ed25519-signature $$s$$ of a hash $$h$$ (a 256-bit unsigned integer, usually computed as the hash of some data) using public key $$k$$ (also represented by a 256-bit unsigned integer). The signature $$s$$ must be a Slice containing at least 512 data bits; only the first 512 bits are used. The result is -1 if the signature is valid, 0 otherwise. Notice that CHKSIGNU is equivalent to ROT; NEWB; STU 256; ENDB; NEWC; ROTREV; CHKSIGNS, i.e., to CHKSIGNS with the first argument $$d$$ set to 256-bit Slice containing $$h$$. Therefore, if $$h$$ is computed as the hash of some data, these data are hashed twice, the second hashing occurring inside CHKSIGNS.
* F911 - CHKSIGNS $$(d s k-?)$$, checks whether $$s$$ is a valid Ed25519signature of the data portion of Slice $$d$$ using public key $$k$$, similarly to CHKSIGNU. If the bit length of Slice $$d$$ is not divisible by eight, throws a cell underflow exception. The verification of Ed25519 signatures is the standard one, with SHA256 used to reduce $$d$$ to the 256 -bit number that is actually signed.
* F912-F93F - Reserved for hashing and cryptography primitives.

### A.11.7. Miscellaneous primitives.

* F940 - CDATASIZEQ ( $$c n-x y z-1$$ or 0 ), recursively computes the count of distinct cells $$x$$, data bits $$y$$, and cell references $$z$$ in the dag rooted at Cell c, effectively returning the total storage used by this dag taking into account the identification of equal cells. The values of $$x, y$$, and $$z$$ are computed by a depth-first traversal of this dag, with a hash table of visited cell hashes used to prevent visits of already-visited cells. The total count of visited cells $$x$$ cannot exceed non-negative Integer $$n$$; otherwise the computation is aborted before visiting the $$(n+1)$$-st cell and a zero is returned to indicate failure. If $$c$$ is $$N u l l$$, $$\operatorname{returns} x=y=z=0$$.
* F941 - CDATASIZE $$(c n-x y z)$$, a non-quiet version of CDATASIZEQ that throws a cell overflow exception (8) on failure.
* F942 - SDATASIZEQ ( $$s n-x y z-1$$ or 0$$)$$, similar to CDATASIZEQ, but accepting a Slice $$s$$ instead of a Cell. The returned value of $$x$$ does not take into account the cell that contains the slice $$s$$ itself; however, the data bits and the cell references of $$s$$ are accounted for in $$y$$ and $$z$$.
* F943 - SDATASIZE $$(s n-x y z)$$, a non-quiet version of SDATASIZEQ that throws a cell overflow exception (8) on failure. - F944-F97F - Reserved for miscellaneous TVM-specific primitives that do not fall into any other specific category.

### A.11.8. Currency manipulation primitives.

* FAO0 - LDGRAMS or LDVARUINT16 $$\left(s-x s^{\prime}\right)$$, loads (deserializes) a Gram or VarUInteger 16 amount from CellSlice s, and returns the amount as Integer $$x$$ along with the remainder $$s^{\prime}$$ of $$s$$. The expected serialization of $$x$$ consists of a 4-bit unsigned big-endian integer $$l$$, followed by an $$8 l$$-bit unsigned big-endian representation of $$x$$. The net effect is approximately equivalent to LDU 4; SWAP; LSHIFT 3; LDUX.
* FA01 - LDVARINT16 $$\left(s-x s^{\prime}\right)$$, similar to LDVARUINT16, but loads a signed Integer $$x$$. Approximately equivalent to LDU 4; SWAP; LSHIFT 3; LDIX.
* FA02 - STGRAMS or STVARUINT16 $$\left(b x-b^{\prime}\right)$$, stores (serializes) an Integer $$x$$ in the range $$0 \ldots 2^{120}-1$$ into Builder $$b$$, and returns the resulting Builder $$b^{\prime}$$. The serialization of $$x$$ consists of a 4-bit unsigned big-endian integer $$l$$, which is the smallest integer $$l \geq 0$$, such that $$x<2^{8 l}$$, followed by an $$8 l$$-bit unsigned big-endian representation of $$x$$. If $$x$$ does not belong to the supported range, a range check exception is thrown.
* FA03 - STVARINT16 $$\left(b x-b^{\prime}\right)$$, similar to STVARUINT16, but serializes a signed Integer $$x$$ in the range $$-2^{119} \ldots 2^{119}-1$$.
* FA04 - LDVARUINT32 $$\left(s-x s^{\prime}\right)$$, loads (deserializes) a VarUInteger 32 from CellSlice $$s$$, and returns the deserialized value as an Integer $$0 \leq x<2^{248}$$. The expected serialization of $$x$$ consists of a 5 -bit unsigned big-endian integer $$l$$, followed by an $$8 l$$-bit unsigned big-endian representation of $$x$$. The net effect is approximately equivalent to LDU 5; SWAP; SHIFT 3; LDUX.
* FA05 - LDVARINT32 $$\left(s-x s^{\prime}\right)$$, deserializes a VarInteger 32 from CellSlice s, and returns the deserialized value as an Integer $$-2^{247} \leq$$ $$x<2^{247}$$.
* FA06 - STVARUINT32 $$\left(b x-b^{\prime}\right)$$, serializes an Integer $$0 \leq x<2^{248}$$ as a VarUInteger 32. - FA07 - STVARINT32 $$\left(b x-b^{\prime}\right)$$, serializes an Integer $$-2^{247} \leq x<2^{247}$$ as a VarInteger 32.
* FA08-FA1F - Reserved for currency manipulation primitives.

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

A deserialized MsgAddress is represented by a Tuple $$t$$ as follows:

* addr\_none is represented by $$t=(0)$$, i.e., a Tuple containing exactly one Integer equal to zero.
* addr\_extern is represented by $$t=(1, s)$$, where Slice $$s$$ contains the field external\_address. In other words, $$t$$ is a pair (a Tuple consisting of two entries), containing an Integer equal to one and Slice s.
* addr\_std is represented by $$t=(2, u, x, s)$$, where $$u$$ is either a $$N u l l$$ (if anycast is absent) or a Slice $$s^{\prime}$$ containing rewrite\_pfx (if anycast is present). Next, Integer $$x$$ is the workchain\_id, and Slice s contains the address. - addr\_var is represented by $$t=(3, u, x, s)$$, where $$u, x$$, and $$s$$ have the same meaning as for addr\_std.

The following primitives, which use the above conventions, are defined:

* FA40 - LDMSGADDR $$\left(s-s^{\prime} s^{\prime \prime}\right)$$, loads from CellSlice $$s$$ the only prefix that is a valid MsgAddress, and returns both this prefix $$s^{\prime}$$ and the remainder $$s^{\prime \prime}$$ of $$s$$ as CellSlices.
* FA41 - LDMSGADDRQ $$\left(s-s^{\prime} s^{\prime \prime}-1\right.$$ or $$s$$ 0), a quiet version of LDMSGADDR: on success, pushes an extra -1 ; on failure, pushes the original $$s$$ and a zero.
* FA42 - PARSEMSGADDR $$(s-t)$$, decomposes CellSlice s containing a valid MsgAddress into a Tuple $$t$$ with separate fields of this MsgAddress. If $$s$$ is not a valid MsgAddress, a cell deserialization exception is thrown.
* FA43 - PARSEMSGADDRQ $$(s-t-1$$ or 0$$)$$, a quiet version of PARSEMSGADDR: returns a zero on error instead of throwing an exception.
* FA44 - REWRITESTDADDR $$(s-x y)$$, parses CellSlice s containing a valid MsgAddressint (usually a msg\_addr\_std), applies rewriting from the anycast (if present) to the same-length prefix of the address, and returns both the workchain $$x$$ and the 256-bit address $$y$$ as Integers. If the address is not 256-bit, or if $$s$$ is not a valid serialization of MsgAddress Int, throws a cell deserialization exception.
* FA45 - REWRITESTDADDRQ $$(s-x y-1$$ or 0 ), a quiet version of primitive REWRITESTDADDR.
* FA46 - REWRITEVARADDR $$\left(s-x s^{\prime}\right)$$, a variant of REWRITESTDADDR that returns the (rewritten) address as a Slice s, even if it is not exactly 256 bit long (represented by a msg\_addr\_var).
* FA47 - REWRITEVARADDRQ $$\left(s-x s^{\prime}-1\right.$$ or 0$$)$$, a quiet version of primitive REWRITEVARADDR.
* FA48-FA5F - Reserved for message and address manipulation primitives.

### A.11.10. Outbound message and output action primitives.

* FBOO - SENDRAWMSG $$(c x-)$$, sends a raw message contained in Cell $$c$$, which should contain a correctly serialized object Message $$X$$, with the only exception that the source address is allowed to have dummy value addr\_none (to be automatically replaced with the current smartcontract address), and ihr\_fee, fwd\_fee, created\_lt and created\_at fields can have arbitrary values (to be rewritten with correct values during the action phase of the current transaction). Integer parameter $$x$$ contains the flags. Currently $$x=0$$ is used for ordinary messages; $$x=128$$ is used for messages that are to carry all the remaining balance of the current smart contract (instead of the value originally indicated in the message); $$x=64$$ is used for messages that carry all the remaining value of the inbound message in addition to the value initially indicated in the new message (if bit 0 is not set, the gas fees are deducted from this amount); $$x^{\prime}=x+1$$ means that the sender wants to pay transfer fees separately; $$x^{\prime}=x+2$$ means that any errors arising while processing this message during the action phase should be ignored. Finally, $$x^{\prime}=x+32$$ means that the current account must be destroyed if its resulting balance is zero. This flag is usually employed together with +128 .
* FB02 - RAWRESERVE $$(x y-)$$, creates an output action which would reserve exactly $$x$$ nanograms (if $$y=0$$ ), at most $$x$$ nanograms (if $$y=2$$ ), or all but $$x$$ nanograms (if $$y=1$$ or $$y=3$$ ), from the remaining balance of the account. It is roughly equivalent to creating an outbound message carrying $$x$$ nanograms (or $$b-x$$ nanograms, where $$b$$ is the remaining balance) to oneself, so that the subsequent output actions would not be able to spend more money than the remainder. Bit +2 in $$y$$ means that the external action does not fail if the specified amount cannot be reserved; instead, all remaining balance is reserved. Bit +8 in $$y$$ means $$x \leftarrow-x$$ before performing any further actions. Bit +4 in $$y$$ means that $$x$$ is increased by the original balance of the current account (before the compute phase), including all extra currencies, before performing any other checks and actions. Currently $$x$$ must be a non-negative integer, and $$y$$ must be in the range $$0 \ldots 15$$.
* FB03 - RAWRESERVEX $$(x D y-)$$, similar to RAWRESERVE, but also accepts a dictionary $$D$$ (represented by a Cell or $$N u l l)$$ with extra currencies. In this way currencies other than Grams can be reserved. - FBO4 - SETCODE $$(c-)$$, creates an output action that would change this smart contract code to that given by Cell c. Notice that this change will take effect only after the successful termination of the current run of the smart contract.
* FB06 - SETLIBCODE $$(c x-)$$, creates an output action that would modify the collection of this smart contract libraries by adding or removing library with code given in Cell c. If $$x=0$$, the library is actually removed if it was previously present in the collection (if not, this action does nothing). If $$x=1$$, the library is added as a private library, and if $$x=2$$, the library is added as a public library (and becomes available to all smart contracts if the current smart contract resides in the masterchain); if the library was present in the collection before, its public/private status is changed according to $$x$$. Values of $$x$$ other than $$0 \ldots 2$$ are invalid.
* FB07 - CHANGELIB $$(h x-)$$, creates an output action similarly to SETLIBCODE, but instead of the library code accepts its hash as an unsigned 256-bit integer $$h$$. If $$x \neq 0$$ and the library with hash $$h$$ is absent from the library collection of this smart contract, this output action will fail.
* FB08-FB3F - Reserved for output action primitives.

## A.12 Debug primitives

Opcodes beginning with $$\mathrm{FE}$$ are reserved for the debug primitives. These primitives have known fixed operation length, and behave as (multibyte) NOP operations. In particular, they never change the stack contents, and never throw exceptions, unless there are not enough bits to completely decode the opcode. However, when invoked in a TVM instance with debug mode enabled, these primitives can produce specific output into the text debug log of the TVM instance, never affecting the TVM state (so that from the perspective of TVM the behavior of debug primitives in debug mode is exactly the same). For instance, a debug primitive might dump all or some of the values near the top of the stack, display the current state of TVM and so on.

### A.12.1. Debug primitives as multibyte NOPs.

* FEnn - DEBUG $$n n$$, for $$0 \leq n n<240$$, is a two-byte NOP.
* FEFnssss - DEBUGSTR ssss, for $$0 \leq n<16$$, is an $$(n+3)$$-byte NOP, with the $$(n+1)$$-byte "contents string" ssss skipped as well.

A.12.2. Debug primitives as operations without side-effect. Next we describe the debug primitives that might (and actually are) implemented in a version of TVM. Notice that another TVM implementation is free to use these codes for other debug purposes, or treat them as multibyte NOPs. Whenever these primitives need some arguments from the stack, they inspect these arguments, but leave them intact in the stack. If there are insufficient values in the stack, or they have incorrect types, debug primitives may output error messages into the debug log, or behave as NOPs, but they cannot throw exceptions.

* FEOO - DUMPSTK, dumps the stack (at most the top 255 values) and shows the total stack depth.
* FEO $$n$$ - DUMPSTKTOP $$n, 1 \leq n<15$$, dumps the top $$n$$ values from the stack, starting from the deepest of them. If there are $$d<n$$ values available, dumps only $$d$$ values.
* FE10 - HEXDUMP, dumps s0 in hexadecimal form, be it a Slice or an Integer.
* FE11 - HEXPRINT, similar to HEXDUMP, except the hexadecimal representation of s0 is not immediately output, but rather concatenated to an output text buffer.
* FE12 - BINDUMP, dumps s0 in binary form, similarly to HEXDUMP.
* FE13 - BINPRINT, outputs the binary representation of s0 to a text buffer.
* FE14 - STRDUMP, dumps the Slice at s0 as an UTF-8 string.
* FE15 - STRPRINT, similar to STRDUMP, but outputs the string into a text buffer (without carriage return).
* FE1E - DEBUGOFF, disables all debug output until it is re-enabled by a DEBUGON. More precisely, this primitive increases an internal counter, which disables all debug operations (except DEBUGOFF and DEBUGON) when strictly positive. - FE1F - DEBUGON, enables debug output (in a debug version of TVM).
* FE2 $$n-$$ DUMP $$\mathrm{s}(n), 0 \leq n<15$$, dumps $$\mathrm{s}(n)$$.
* FE3n-PRINT $$\mathrm{s}(n), 0 \leq n<15$$, concatenates the text representation of $$s(n)$$ (without any leading or trailing spaces or carriage returns) to a text buffer which will be output before the output of any other debug operation.
* FECO-FEEF - Use these opcodes for custom/experimental debug operations.
* FEFnssss - DUMPTOSFMT ssss, dumps s0 formatted according to the $$(n+1)$$-byte string ssss. This string might contain (a prefix of) the name of a TL-B type supported by the debugger. If the string begins with a zero byte, simply outputs it (without the first byte) into the debug log. If the string begins with a byte equal to one, concatenates it to a buffer, which will be output before the output of any other debug operation (effectively outputs a string without a carriage return).
* FEFn00ssss - LOGSTR ssss, string ssss is $$n$$ bytes long.
* FEF000 - LOGFLUSH, flushes all pending debug output from the buffer into the debug log.
* FEFn01ssss - PRINTSTR ssss, string ssss is $$n$$ bytes long.

### A.13 Codepage primitives

The following primitives, which begin with byte FF, typically are used at the very beginning of a smart contract's code or a library subroutine to select another TVM codepage. Notice that we expect all codepages to contain these primitives with the same codes, otherwise switching back to another codepage might be impossible (cf. [$$\mathbf{5.1.8}$$](codepages-and-instruction-encoding.md#5.1.8.-setting-the-codepage-in-the-code-itself.)).

* FFnn - SETCP $$n n$$, selects TVM codepage $$0 \leq n n<240$$. If the codepage is not supported, throws an invalid opcode exception.
* FFOO - SETCPO, selects TVM (test) codepage zero as described in this document.
* $$\mathrm{FFF} z$$ - SETCP $$z-16$$, selects TVM codepage $$z-16$$ for $$1 \leq z \leq 15$$. Negative codepages $$-13 \ldots-1$$ are reserved for restricted versions of TVM needed to validate runs of TVM in other codepages as explained in [$$\mathbf{B.2.6}$$](b-formal-properties-and-specifications-of-tvm.md#b.2.6.-codepage-1-.). Negative codepage -14 is reserved for experimental codepages, not necessarily compatible between different TVM implementations, and should be disabled in the production versions of TVM.
* FFFO - SETCPX $$(c-)$$, selects codepage $$c$$ with $$-2^{15} \leq c<2^{15}$$ passed in the top of the stack.
