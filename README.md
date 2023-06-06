# Introduction

## Abstract

The aim of this text is to provide a description of the Threaded Open Network Virtual Machine (TON VM or TVM), used to execute smart contracts in the TON Blockchain.

## Introduction

The primary purpose of the Threaded Open Network Virtual Machine (TON VM or TVM) is to execute smart-contract code in the TON Blockchain. TVM must support all operations required to parse incoming messages and persistent data, and to create new messages and modify persistent data.

Additionally, TVM must meet the following requirements:

* It must provide for possible future extensions and improvements while retaining backward compatibility and interoperability, because the code of a smart contract, once committed into the blockchain, must continue working in a predictable manner regardless of any future modifications to the VM.
* It must strive to attain high "(virtual) machine code" density, so that the code of a typical smart contract occupies as little persistent blockchain storage as possible.
* It must be completely deterministic. In other words, each run of the same code with the same input data must produce the same result, regardless of specific software and hardware used$$|^{1}$$&#x20;

The design of TVM is guided by these requirements. While this document describes a preliminary and experimental version of TVM the backward compatibility mechanisms built into the system allow us to be relatively unconcerned with the efficiency of the operation encoding used for TVM code in this preliminary version.

TVM is not intended to be implemented in hardware (e.g., in a specialized microprocessor chip); rather, it should be implemented in software running on conventional hardware. This consideration lets us incorporate some highlevel concepts and operations in TVM that would require convoluted microcode in a hardware implementation but pose no significant problems for a software implementation. Such operations are useful for achieving high code density and minimizing the byte (or storage cell) profile of smart-contract code when deployed in the TON Blockchain.

$${ }^{1}$$ For example, there are no floating-point arithmetic operations (which could be efficiently implemented using hardware-supported double type on most modern CPUs) present in TVM, because the result of performing such operations is dependent on the specific underlying hardware implementation and rounding mode settings. Instead, TVM supports special integer arithmetic operations, which can be used to simulate fixed-point arithmetic if needed.

$${ }^{2}$$ The production version will likely require some tweaks and modifications prior to launch, which will become apparent only after using the experimental version in the test environment for some time.

## Overview

This chapter provides an overview of the main features and design principles of TVM. More detail on each topic is provided in subsequent chapters.

### Notation for bitstrings

The following notation is used for bit strings (or bitstrings)-i.e., finite strings consisting of binary digits (bits), 0 and 1 -throughout this document.

### 1.0.1. Hexadecimal notation for bitstrings. 

When the length of a bitstring is a multiple of four, we subdivide it into groups of four bits and represent each group by one of sixteen hexadecimal digits $$0-9, \mathrm{~A}-\mathrm{F}$$ in the usual manner: $$0_{16} \leftrightarrow 0000,1_{16}$$ $$\leftrightarrow 0001, \ldots$$, $$F_{16} \leftrightarrow 1111$$. The resulting hexadecimal string is our equivalent representation for the original binary string.

### 1.0.2. Bitstrings of lengths not divisible by four. 

If the length of a binary string is not divisible by four, we augment it by one 1 and several (maybe zero) 0s at the end, so that its length becomes divisible by four, and then transform it into a string of hexadecimal digits as described above. To indicate that such a transformation has taken place, a special "completion tag" - is added to the end of the hexadecimal string. The reverse transformation (applied if the completion tag is present) consists in first replacing each hexadecimal digit by four corresponding bits, and then removing all trailing zeroes (if any) and the last 1 immediately preceding them (if the resulting bitstring is non-empty at this point).

Notice that there are several admissible hexadecimal representations for the same bitstring. Among them, the shortest one is "canonical". It can be deterministically obtained by the above procedure.

For example, 8A corresponds to binary string 10001010, while 8A\_ and 8A0\_ both correspond to 100010. An empty bitstring may be represented by either', '8\_', '0\_,', ', or '00\_'.

### 1.0.3. Emphasizing that a string is a hexadecimal representation of a bitstring. 

Sometimes we need to emphasize that a string of hexadecimal digits (with or without a \_ at the end) is the hexadecimal representation of a bitstring. In such cases, we either prepend $$x$$ to the resulting string (e.g., $$\mathrm{x} 8 \mathrm{~A}$$ ), or prepend $$\mathrm{x}\{$$ and append $$\}$$ (e.g., $$x\left\{2 D\right.$$ D\_\_ $$_{-}$$, which is 00101101100). This should not be confused with hexadecimal numbers, usually prepended by $$0 x$$ (e.g., 0x2D9 or $$0 \times 2 \mathrm{~d} 9$$, which is the integer 729$$)$$.

### 1.0.4. Serializing a bitstring into a sequence of octets. 

When a bitstring needs to be represented as a sequence of 8-bit bytes (octets), which take values in integers $$0 . .255$$, this is achieved essentially in the same fashion as above: we split the bitstring into groups of eight bits and interpret each group as the binary representation of an integer $$0 \ldots 255$$. If the length of the bitstring is not a multiple of eight, the bitstring is augmented by a binary 1 and up to seven binary 0s before being split into groups. The fact that such a completion has been applied is usually reflected by a "completion tag" bit.

For instance, 00101101100 corresponds to the sequence of two octets (0x2d, 0x90) (hexadecimal), or $$(45,144)$$ (decimal), along with a completion tag bit equal to 1 (meaning that the completion has been applied), which must be stored separately.

In some cases, it is more convenient to assume the completion is enabled by default rather than store an additional completion tag bit separately. Under such conventions, $$8 n$$-bit strings are represented by $$n+1$$ octets, with the last octet always equal to $$0 \times 80=128$$.

### TVM is a stack machine

First of all, $$T V M$$ is a stack machine. This means that, instead of keeping values in some "variables" or "general-purpose registers", they are kept in a (LIFO) stack, at least from the "low-level" (TVM) perspective $$!^{3}$$

Most operations and user-defined functions take their arguments from the top of the stack, and replace them with their result. For example, the integer addition primitive (built-in operation) ADD does not take any arguments describing which registers or immediate values should be added together and where the result should be stored. Instead, the two top values are taken from the stack, they are added together, and their sum is pushed into the stack in their place.

$${ }^{3}$$ A high-level smart-contract language might create a visibility of variables for the ease of programming; however, the high-level source code working with variables will be translated into TVM machine code keeping all the values of these variables in the TVM stack. 1.1.1. TVM values. The entities that can be stored in the TVM stack will be called TVM values, or simply values for brevity. They belong to one of several predefined value types. Each value belongs to exactly one value type. The values are always kept on the stack along with tags uniquely determining their types, and all built-in TVM operations (or primitives) only accept values of predefined types.

For example, the integer addition primitive ADD accepts only two integer values, and returns one integer value as a result. One cannot supply ADD with two strings instead of two integers expecting it to concatenate these strings or to implicitly transform the strings into their decimal integer values; any attempt to do so will result in a run-time type-checking exception.

### Static typing, dynamic typing, and run-time type checking.

In some respects TVM performs a kind of dynamic typing using run-time type checking. However, this does not make the TVM code a "dynamically typed language" like PHP or Javascript, because all primitives accept values and return results of predefined (value) types, each value belongs to strictly one type, and values are never implicitly converted from one type to another. If, on the other hand, one compares the TVM code to the conventional microprocessor machine code, one sees that the TVM mechanism of value tagging prevents, for example, using the address of a string as a numberor, potentially even more disastrously, using a number as the address of a string - thus eliminating the possibility of all sorts of bugs and security vulnerabilities related to invalid memory accesses, usually leading to memory corruption and segmentation faults. This property is highly desirable for a VM used to execute smart contracts in a blockchain. In this respect, TVM's insistence on tagging all values with their appropriate types, instead of reinterpreting the bit sequence in a register depending on the needs of the operation it is used in, is just an additional run-time type-safety mechanism.An alternative would be to somehow analyze the smart-contract code for type correctness and type safety before allowing its execution in the VM, or even before allowing it to be uploaded into the blockchain as the code of a smart contract. Such a static analysis of code for a Turing-complete machine appears to be a time-consuming and non-trivial problem (likely to be equivalent to the stopping problem for Turing machines), something we would rather avoid in a blockchain smart-contract context.

One should bear in mind that one always can implement compilers from statically typed high-level smart-contract languages into the TVM code (and we do expect that most smart contracts for TON will be written in such languages), just as one can compile statically typed languages into conventional machine code (e.g., x86 architecture). If the compiler works correctly, the resulting machine code will never generate any run-time type-checking exceptions. All type tags attached to values processed by TVM will always have expected values and may be safely ignored during the analysis of the resulting TVM code, apart from the fact that the run-time generation and verification of these type tags by TVM will slightly slow down the execution of the TVM code.

### 1.1.3. Preliminary list of value types. 

A preliminary list of value types supported by TVM is as follows:

* Integer - Signed 257-bit integers, representing integer numbers in the range $$-2^{256} \ldots 2^{256}-1$$, as well as a special "not-a-number" value $$N a N$$.
* Cell - A TVM cell consists of at most 1023 bits of data, and of at most four references to other cells. All persistent data (including TVM code) in the TON Blockchain is represented as a collection of TVM cells (cf. \[1, 2.5.14]).
* Tuple - An ordered collection of up to 255 components, having arbitrary value types, possibly distinct. May be used to represent nonpersistent values of arbitrary algebraic data types.
* Null - A type with exactly one value $$\perp$$, used for representing empty lists, empty branches of binary trees, absence of return value in some situations, and so on.
* Slice - A TVM cell slice, or slice for short, is a contiguous "sub-cell" of an existing cell, containing some of its bits of data and some of its references. Essentially, a slice is a read-only view for a subcell of a cell. Slices are used for unpacking data previously stored (or serialized) in a cell or a tree of cells.
* Builder - A TVM cell builder, or builder for short, is an "incomplete" cell that supports fast operations of appending bitstrings and cell references at its end. Builders are used for packing (or serializing) data from the top of the stack into new cells (e.g., before transferring them to persistent storage). - Continuation - Represents an "execution token" for TVM, which may be invoked (executed) later. As such, it generalizes function addresses (i.e., function pointers and references), subroutine return addresses, instruction pointer addresses, exception handler addresses, closures, partial applications, anonymous functions, and so on.

This list of value types is incomplete and may be extended in future revisions of TVM without breaking the old TVM code, due mostly to the fact that all originally defined primitives accept only values of types known to them and will fail (generate a type-checking exception) if invoked on values of new types. Furthermore, existing value types themselves can also be extended in the future: for example, 257-bit Integer might become 513-bit LongInteger, with originally defined arithmetic primitives failing if either of the arguments or the result does not fit into the original subtype Integer. Backward compatibility with respect to the introduction of new value types and extension of existing value types will be discussed in more detail later (cf. 5.1.4).

### Categories of TVM instructions

TVM instructions, also called primitives and sometimes (built-in) operations, are the smallest operations atomically performed by TVM that can be present in the TVM code. They fall into several categories, depending on the types of values (cf. 1.1.3) they work on. The most important of these categories are:

* Stack (manipulation) primitives - Rearrange data in the TVM stack, so that the other primitives and user-defined functions can later be called with correct arguments. Unlike most other primitives, they are polymorphic, i.e., work with values of arbitrary types.
* Tuple (manipulation) primitives - Construct, modify, and decompose Tuples. Similarly to the stack primitives, they are polymorphic.
* Constant or literal primitives - Push into the stack some "constant" or "literal" values embedded into the TVM code itself, thus providing arguments to the other primitives. They are somewhat similar to stack primitives, but are less generic because they work with values of specific types. - Arithmetic primitives - Perform the usual integer arithmetic operations on values of type Integer.
* Cell (manipulation) primitives - Create new cells and store data in them (cell creation primitives) or read data from previously created cells (cell parsing primitives). Because all memory and persistent storage of TVM consists of cells, these cell manipulation primitives actually correspond to "memory access instructions" of other architectures. Cell creation primitives usually work with values of type Builder, while cell parsing primitives work with Slices.
* Continuation and control flow primitives - Create and modify Continuations, as well as execute existing Continuations in different ways, including conditional and repeated execution.
* Custom or application-specific primitives - Efficiently perform specific high-level actions required by the application (in our case, the TON Blockchain), such as computing hash functions, performing elliptic curve cryptography, sending new blockchain messages, creating new smart contracts, and so on. These primitives correspond to standard library functions rather than microprocessor instructions.

### Control registers

While TVM is a stack machine, some rarely changed values needed in almost all functions are better passed in certain special registers, and not near the top of the stack. Otherwise, a prohibitive number of stack reordering operations would be required to manage all these values.

To this end, the TVM model includes, apart from the stack, up to 16 special control registers, denoted by c0 to c15, or c $$(0)$$ to $$c(15)$$. The original version of TVM makes use of only some of these registers; the rest may be supported later.

### 1.3.1. Values kept in control registers. 

The values kept in control registers are of the same types as those kept on the stack. However, some control registers accept only values of specific types, and any attempt to load a value of a different type will lead to an exception.

### 1.3.2. List of control registers. 

The original version of TVM defines and uses the following control registers: - c0 - Contains the next continuation or return continuation (similar to the subroutine return address in conventional designs). This value must be a Continuation.

* c1 - Contains the alternative (return) continuation; this value must be a Continuation. It is used in some (experimental) control flow primitives, allowing TVM to define and call "subroutines with two exit points".
* c2 - Contains the exception handler. This value is a Continuation, invoked whenever an exception is triggered.
* c3 - Contains the current dictionary, essentially a hashmap containing the code of all functions used in the program. For reasons explained later in $$\mathbf{4 . 6}$$, this value is also a Continuation, not a Cell as one might expect.
* c4 - Contains the root of persistent data, or simply the data. This value is a Cell. When the code of a smart contract is invoked, c4 points to the root cell of its persistent data kept in the blockchain state. If the smart contract needs to modify this data, it changes c4 before returning.
* c5 - Contains the output actions. It is also a Cell initialized by a reference to an empty cell, but its final value is considered one of the smart contract outputs. For instance, the SENDMSG primitive, specific for the TON Blockchain, simply inserts the message into a list stored in the output actions.
* c7 - Contains the root of temporary data. It is a Tuple, initialized by a reference to an empty Tuple before invoking the smart contract and discarded after its termination $$\bigsqcup^{4}$$

More control registers may be defined in the future for specific TON Blockchain or high-level programming language purposes, if necessary.

$${ }^{4}$$ In the TON Blockchain context, c7 is initialized with a singleton Tuple, the only component of which is a Tuple containing blockchain-specific data. The smart contract is free to modify c7 to store its temporary data provided the first component of this Tuple remains intact.

### Total state of TVM (SCCCG)

The total state of TVM consists of the following components:

* Stack (cf. 1.1) - Contains zero or more values (cf. 1.1.1), each belonging to one of value types listed in 1.1.3
* Control registers c0-c15 - Contain some specific values as described in 1.3.2. (Only seven control registers are used in the current version.)
* Current continuation cc - Contains the current continuation (i.e., the code that would be normally executed after the current primitive is completed). This component is similar to the instruction pointer register (ip) in other architectures.
* Current codepage cp-A special signed 16-bit integer value that selects the way the next TVM opcode will be decoded. For example, future versions of TVM might use different codepages to add new opcodes while preserving backward compatibility.
* Gas limits gas - Contains four signed 64-bit integers: the current gas limit $$g_{l}$$, the maximal gas limit $$g_{m}$$, the remaining gas $$g_{r}$$, and the gas credit $$g_{c}$$. Always $$0 \leq g_{l} \leq g_{m}, g_{c} \geq 0$$, and $$g_{r} \leq g_{l}+g_{c} ; g_{c}$$ is usually initialized by zero, $$g_{r}$$ is initialized by $$g_{l}+g_{c}$$ and gradually decreases as the TVM runs. When $$g_{r}$$ becomes negative or if the final value of $$g_{r}$$ is less than $$g_{c}$$, an out of gas exception is triggered.

Notice that there is no "return stack" containing the return addresses of all previously called but unfinished functions. Instead, only control register c0 is used. The reason for this will be explained later in 4.1.9.

Also notice that there are no general-purpose registers, because TVM is a stack machine (cf. 1.1). So the above list, which can be summarized as "stack, control, continuation, codepage, and gas" (SCCCG), similarly to the classical SECD machine state ("stack, environment, control, dump"), is indeed the total state of TVM 5

$${ }^{5}$$ Strictly speaking, there is also the current library context, which consists of a dictionary with 256-bit keys and cell values, used to load library reference cells of 3.1.7.

### Integer arithmetic

All arithmetic primitives of TVM operate on several arguments of type Integer, taken from the top of the stack, and return their results, of the same type, into the stack. Recall that Integer represents all integer values in the range $$-2^{256} \leq x<2^{256}$$, and additionally contains a special value $$\mathrm{NaN}$$ ("nota-number").

If one of the results does not fit into the supported range of integersor if one of the arguments is a $$\mathrm{NaN}$$-then this result or all of the results are replaced by a $$\mathrm{NaN}$$, and (by default) an integer overflow exception is generated. However, special "quiet" versions of arithmetic operations will simply produce NaNs and keep going. If these NaNs end up being used in a "non-quiet" arithmetic operation, or in a non-arithmetic operation, an integer overflow exception will occur.

### 1.5.1. Absence of automatic conversion of integers. 

Notice that TVM Integers are "mathematical" integers, and not 257-bit strings interpreted differently depending on the primitive used, as is common for other machine code designs. For example, TVM has only one multiplication primitive MUL, rather than two (MUL for unsigned multiplication and IMUL for signed multiplication) as occurs, for example, in the popular x86 architecture.

### 1.5.2. Automatic overflow checks. 

Notice that all TVM arithmetic primitives perform overflow checks of the results. If a result does not fit into the Integer type, it is replaced by a NaN, and (usually) an exception occurs. In particular, the result is not automatically reduced modulo $$2^{256}$$ or $$2^{257}$$, as is common for most hardware machine code architectures.

### 1.5.3. Custom overflow checks. 

In addition to automatic overflow checks, TVM includes custom overflow checks, performed by primitives FITS $$n$$ and UFITS $$n$$, where $$1 \leq n \leq 256$$. These primitives check whether the value on (the top of) the stack is an integer $$x$$ in the range $$-2^{n-1} \leq x<2^{n-1}$$ or $$0 \leq x<2^{n}$$, respectively, and replace the value with a $$\mathrm{NaN}$$ and (optionally) generate an integer overflow exception if this is not the case. This greatly simplifies the implementation of arbitrary $$n$$-bit integer types, signed or unsigned: the programmer or the compiler must insert appropriate FITS or UFITS primitives either after each arithmetic operation (which is more reasonable, but requires more checks) or before storing computed values and returning them from functions. This is important for smart contracts, where unexpected integer overflows happen to be among the most common source of bugs.

### 1.5.4. Reduction modulo $$2^{n}$$. 

TVM also has a primitive MODPOW $$2 n$$, which reduces the integer at the top of the stack modulo $$2^{n}$$, with the result ranging from 0 to $$2^{n}-1$$.

### 1.5.5. Integer is 257-bit, not 256-bit. 

One can understand now why TVM's Integer is (signed) 257-bit, not 256-bit. The reason is that it is the smallest integer type containing both signed 256-bit integers and unsigned 256-bit integers, which does not require automatic reinterpreting of the same 256-bit string depending on the operation used (cf. $$\mathbf{1 . 5 . 1}$$ ).

### 1.5.6. Division and rounding. 

The most important division primitives are DIV, MOD, and DIVMOD. All of them take two numbers from the stack, $$x$$ and $$y$$ ( $$y$$ is taken from the top of the stack, and $$x$$ is originally under it), compute the quotient $$q$$ and remainder $$r$$ of the division of $$x$$ by $$y$$ (i.e., two integers such that $$x=y q+r$$ and $$|r|<|y|$$ ), and return either $$q, r$$, or both of them. If $$y$$ is zero, then all of the expected results are replaced by NaNs, and (usually) an integer overflow exception is generated.

The implementation of division in TVM somewhat differs from most other implementations with regards to rounding. By default, these primitives round to $$-\infty$$, meaning that $$q=\lfloor x / y\rfloor$$, and $$r$$ has the same sign as $$y$$. (Most conventional implementations of division use "rounding to zero" instead, meaning that $$r$$ has the same sign as $$x$$.) Apart from this "floor rounding", two other rounding modes are available, called "ceiling rounding" (with $$q=\lceil x / y\rceil$$, and $$r$$ and $$y$$ having opposite signs) and "nearest rounding" (with $$q=\lfloor x / y+1 / 2\rfloor$$ and $$|r| \leq|y| / 2$$ ). These rounding modes are selected by using other division primitives, with letters $$C$$ and $$R$$ appended to their mnemonics. For example, DIVMODR computes both the quotient and the remainder using rounding to the nearest integer.

### 1.5.7. Combined multiply-divide, multiply-shift, and shift-divide operations. 

To simplify implementation of fixed-point arithmetic, TVM supports combined multiply-divide, multiply-shift, and shift-divide operations with double-length (i.e., 514-bit) intermediate product. For example, MULDIVMODR takes three integer arguments from the stack, $$a, b$$, and $$c$$, first computes $$a b$$ using a 514-bit intermediate result, and then divides $$a b$$ by $$c$$ using rounding to the nearest integer. If $$c$$ is zero or if the quotient does not fit into Integer, either two NaNs are returned, or an integer overflow exception is generated, depending on whether a quiet version of the operation has been used. Otherwise, both the quotient and the remainder are pushed into the stack.
