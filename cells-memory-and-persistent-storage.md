# Cells, memory, and persistent storage

This chapter briefly describes TVM cells, used to represent all data structures inside the TVM memory and its persistent storage, and the basic operations used to create cells, write (or serialize) data into them, and read (or deserialize) data from them.

## Generalities on cells

This section presents a classification and general descriptions of cell types.

### 3.1.1. TVM memory and persistent storage consist of cells.

Recall that the TVM memory and persistent storage consist of (TVM) cells. Each cell contains up to 1023 bits of data and up to four references to other cells $${ }^{11}$$ Circular references are forbidden and cannot be created by means of TVM (cf. 2.3.5). In this way, all cells kept in TVM memory and persistent storage constitute a directed acyclic graph (DAG).

### 3.1.2. Ordinary and exotic cells.

Apart from the data and references, a cell has a cell type, encoded by an integer $$-1 \ldots 255$$. A cell of type -1 is called ordinary; such cells do not require any special processing. Cells of other types are called exotic, and may be loaded - automatically replaced by other cells when an attempt to deserialize them (i.e., to convert them into a Slice by a CTOS instruction) is made. They may also exhibit a non-trivial behavior when their hashes are computed.

The most common use for exotic cells is to represent some other cells - for instance, cells present in an external library, or pruned from the original tree of cells when a Merkle proof has been created.

The type of an exotic cell is stored as the first eight bits of its data. If an exotic cell has less than eight data bits, it is invalid.

### 3.1.3. The level of a cell.

Every cell $$c$$ has another attribute LVL $$(c)$$ called its (de Brujn) level, which currently takes integer values in the range $$0 \ldots 3$$.

$${ }^{11}$$ From the perspective of low-level cell operations, these data bits and cell references are not intermixed. In other words, an (ordinary) cell essentially is a couple consisting of a list of up to 1023 bits and of a list of up to four cell references, without prescribing an order in which the references and the data bits should be deserialized, even though TL-B schemes appear to suggest such an order. The level of an ordinary cell is always equal to the maximum of the levels of all its children $$c_{i}$$ :

$$
\operatorname{LVL}(c)=\max _{1 \leq i \leq r} \operatorname{LVL}\left(c_{i}\right)
$$

for an ordinary cell $$c$$ containing $$r$$ references to cells $$c_{1}, \ldots, c_{r}$$. If $$r=0$$, $$\operatorname{LVL}(c)=0$$. Exotic cells may have different rules for setting their level.

A cell's level affects the number of higher hashes it has. More precisely, a level $$l$$ cell has $$l$$ higher hashes $$\mathrm{HASH}_{1}(c), \ldots, \mathrm{HASH}_{l}(c)$$ in addition to its representation hash $$\mathrm{HASH}(c)=\mathrm{HASH}_{\infty}(c)$$. Cells of non-zero level appear inside Merkle proofs and Merkle updates, after some branches of the tree of cells representing a value of an abstract data type are pruned.

### 3.1.4. Standard cell representation.

When a cell needs to be transferred by a network protocol or stored in a disk file, it must be serialized. The standard representation CellRepr $$(c)=\operatorname{CELlRepR}_{\infty}(c)$$ of a cell $$c$$ as an octet (byte) sequence is constructed as follows:

1. Two descriptor bytes $$d_{1}$$ and $$d_{2}$$ are serialized first. Byte $$d_{1}$$ equals $$r+8 s+32 l$$, where $$0 \leq r \leq 4$$ is the quantity of cell references contained in the cell, $$0 \leq l \leq 3$$ is the level of the cell, and $$0 \leq s \leq 1$$ is 1 for exotic cells and 0 for ordinary cells. Byte $$d_{2}$$ equals $$\lfloor b / 8\rfloor+\lceil b / 8\rceil$$, where $$0 \leq b \leq 1023$$ is the quantity of data bits in $$c$$.
2. Then the data bits are serialized as $$\lceil b / 8\rceil 8$$-bit octets (bytes). If $$b$$ is not a multiple of eight, a binary 1 and up to six binary 0s are appended to the data bits. After that, the data is split into $$\lceil b / 8\rceil$$ eight-bit groups, and each group is interpreted as an unsigned big-endian integer 0 . . 255 and stored into an octet.
3. Finally, each of the $$r$$ cell references is represented by 32 bytes containing the 256-bit representation hash $$\operatorname{HASH}\left(c_{i}\right)$$, explained below in 3.1.5. of the cell $$c_{i}$$ referred to.

In this way, $$2+\lceil b / 8\rceil+32 r$$ bytes of CELlRepR $$(c)$$ are obtained.

### 3.1.5. The representation hash of a cell.

The 256-bit representation hash or simply hash $$\mathrm{HASH}(c)$$ of a cell $$c$$ is recursively defined as the SHA256 of the standard representation of the cell $$c$$ :

$$
\operatorname{HASH}(c):=\operatorname{SHA} 256(\operatorname{CELlREPR}(c))
$$

Notice that cyclic cell references are not allowed and cannot be created by means of the TVM (cf. [2.3.5](stacks/#2.3.5.-absence-of-circular-references.)), so this recursion always ends, and the representation hash of any cell is well-defined.

### 3.1.6. The higher hashes of a cell.

Recall that a cell $$c$$ of level $$l$$ has $$l$$ higher hashes $$\mathrm{HASH}_{i}(c), 1 \leq i \leq l$$, as well. Exotic cells have their own rules for computing their higher hashes. Higher hashes $$\mathrm{HASH}_{i}(c)$$ of an ordinary cell $$c$$ are computed similarly to its representation hash, but using the higher hashes $$\operatorname{HASH}_{i}\left(c_{j}\right)$$ of its children $$c_{j}$$ instead of their representation hashes $$\operatorname{HASH}\left(c_{j}\right)$$. By convention, we set $$\operatorname{HASH}_{\infty}(c):=\operatorname{HASH}(c)$$, and $$\operatorname{HASH}_{i}(c):=$$ $$\operatorname{HASH}_{\infty}(c)=\operatorname{HASH}(c)$$ for all $$i>l, l^{12}$$

### 3.1.7. Types of exotic cells.

TVM currently supports the following cell types:

* Type -1: Ordinary cell-Contains up to 1023 bits of data and up to four cell references.
* Type 1: Pruned branch cell c- May have any level $$1 \leq l \leq 3$$. It contains exactly $$8+256 l$$ data bits: first an 8-bit integer equal to 1 (representing the cell's type), then its $$l$$ higher hashes $$\mathrm{HASH}_{1}(c), \ldots$$, $$\mathrm{HASH}_{l}(c)$$. The level $$l$$ of a pruned branch cell may be called its de Brujn index, because it determines the outer Merkle proof or Merkle update during the construction of which the branch has been pruned. An attempt to load a pruned branch cell usually leads to an exception.
* Type 2: Library reference cell-Always has level 0, and contains 8+256 data bits, including its 8-bit type integer 2 and the representation hash $$\operatorname{HAsh}\left(c^{\prime}\right)$$ of the library cell being referred to. When loaded, a library reference cell may be transparently replaced by the cell it refers to, if found in the current library context.
* Type 3: Merkle proof cell c-Has exactly one reference $$c_{1}$$ and level $$0 \leq l \leq 3$$, which must be one less than the level of its only child $$c_{1}$$ :

$$
\operatorname{LVL}(c)=\max \left(\operatorname{LVL}\left(c_{1}\right)-1,0\right)
$$

$${ }^{12}$$ From a theoretical perspective, we might say that a cell $$c$$ has an infinite sequence of hashes $$\left(\mathrm{HASH}_{i}(c)\right)_{i \geq 1}$$, which eventually stabilizes: $$\mathrm{HASH}_{i}(c) \rightarrow \mathrm{HASH}_{\infty}(c)$$. Then the level $$l$$ is simply the largest index $$i$$, such that $$\operatorname{HASH}_{i}(c) \neq \mathrm{HASH}_{\infty}(c)$$. The $$8+256$$ data bits of a Merkle proof cell contain its 8-bit type integer 3 , followed by $$\mathrm{HASH}_{1}\left(c_{1}\right)$$ (assumed to be equal to $$\operatorname{HASH}\left(c_{1}\right)$$ if $$\left.\operatorname{LVL}\left(c_{1}\right)=0\right)$$. The higher hashes $$\mathrm{HASH}_{i}(c)$$ of $$c$$ are computed similarly to the higher hashes of an ordinary cell, but with $$\mathrm{HASH}_{i+1}\left(c_{1}\right)$$ used instead of $$\mathrm{HASH}_{i}\left(c_{1}\right)$$. When loaded, a Merkle proof cell is replaced by $$c_{1}$$.

* Type 4: Merkle update cell c-Has two children $$c_{1}$$ and $$c_{2}$$. Its level $$0 \leq l \leq 3$$ is given by

$$
\operatorname{LVL}(c)=\max \left(\operatorname{LVL}\left(c_{1}\right)-1, \operatorname{LVL}\left(c_{2}\right)-1,0\right)
$$

A Merkle update behaves like a Merkle proof for both $$c_{1}$$ and $$c_{2}$$, and contains $$8+256+256$$ data bits with $$\mathrm{HASH}_{1}\left(c_{1}\right)$$ and $$\mathrm{HASH}_{1}\left(c_{2}\right)$$. However, an extra requirement is that all pruned branch cells $$c^{\prime}$$ that are descendants of $$c_{2}$$ and are bound by $$c$$ must also be descendants of $$c_{1} \underline{1}^{13}$$ When a Merkle update cell is loaded, it is replaced by $$c_{2}$$.

### 3.1.8. All values of algebraic data types are trees of cells.

Arbitrary values of arbitrary algebraic data types (e.g., all types used in functional programming languages) can be serialized into trees of cells (of level 0), and such representations are used for representing such values within TVM. The copy-on-write mechanism (cf. [2.3.2](stacks/#2.3.2.-efficient-implementation-of-dup-and-push-instructions-using-copy-on-write.)) allows TVM to identify cells containing the same data and references, and to keep only one copy of such cells. This actually transforms a tree of cells into a directed acyclic graph (with the additional property that all its vertices be accessible from a marked vertex called the "root"). However, this is a storage optimization rather than an essential property of TVM. From the perspective of a TVM code programmer, one should think of TVM data structures as trees of cells.

### 3.1.9. TVM code is a tree of cells.

The TVM code itself is also represented by a tree of cells. Indeed, TVM code is simply a value of some complex algebraic data type, and as such, it can be serialized into a tree of cells.

The exact way in which the TVM code (e.g., TVM assembly code) is transformed into a tree of cells is explained later (cf. [4.1.4](control-flow-continuations-and-exceptions/#4.1.4.-normal-work-of-tvm-or-the-main-loop.) and [5.2](codepages-and-instruction-encoding/#instruction-encoding)), in sections discussing control flow instructions, continuations, and TVM instruction encoding.

$${ }^{13} \mathrm{~A}$$ pruned branch cell $$c^{\prime}$$ of level $$l$$ is bound by a Merkle (proof or update) cell $$c$$ if there are exactly $$l$$ Merkle cells on the path from $$c$$ to its descendant $$c^{\prime}$$, including $$c$$.

### 3.1.10. "Everything is a bag of cells" paradigm.

All the data used by the TVM Blockchain, including the blocks themselves and the blockchain state, can be represented - and are represented - as collections, or "bags", of cells. We see that TVM's structure of data (cf. [3.1.8](cells-memory-and-persistent-storage/#3.1.8.-all-values-of-algebraic-data-types-are-trees-of-cells.)) and code (cf. [3.1.9](cells-memory-and-persistent-storage/#3.1.9.-tvm-code-is-a-tree-of-cells.)) nicely fits into this "everything is a bag of cells" paradigm. In this way, TVM can naturally be used to execute smart contracts in the TVM Blockchain, and the TVM Blockchain can be used to store the code and persistent data of these smart contracts between invocations of TVM. (Of course, both TVM and the TVM Blockchain have been designed so that this would become possible.)

## Data manipulation instructions and cells

The next large group of TVM instructions consists of data manipulation instructions, also known as cell manipulation instructions or simply cell instructions. They correspond to memory access instructions of other architectures.

### 3.2.1. Classes of cell manipulation instructions.

The TVM cell instructions are naturally subdivided into two principal classes:

* Cell creation instructions or serialization instructions, used to construct new cells from values previously kept in the stack and previously constructed cells.
* Cell parsing instructions or deserialization instructions, used to extract data previously stored into cells by cell creation instructions.

Additionally, there are exotic cell instructions used to create and inspect exotic cells (cf. [3.1.2](cells-memory-and-persistent-storage/#3.1.2.-ordinary-and-exotic-cells.)), which in particular are used to represent pruned branches of Merkle proofs and Merkle proofs themselves.

### 3.2.2. Builder and Slice values.

Cell creation instructions usually work with Builder values, which can be kept only in the stack (cf. [1.1.3](cells-memory-and-persistent-storage.md#1.1.3.-preliminary-list-of-value-types.)). Such values represent partially constructed cells, for which fast operations for appending bitstrings, integers, other cells, and references to other cells can be defined. Similarly, cell parsing instructions make heavy use of Slice values, which represent either the remainder of a partially parsed cell, or a value (subcell) residing inside such a cell and extracted from it by a parsing instruction. 3.2.3. Builder and Slice values exist only as stack values. Notice that Builder and Slice objects appear only as values in a TVM stack. They cannot be stored in "memory" (i.e., trees of cells) or "persistent storage" (which is also a bag of cells). In this sense, there are far more Cell objects than Builder or Slice objects in a TVM environment, but, somewhat paradoxically, a TVM program sees Builder and Slice objects in its stack more often than Cells. In fact, a TVM program does not have much use for Cell values, because they are immutable and opaque; all cell manipulation primitives require that a Cell value be transformed into either a Builder or a Slice first, before it can be modified or inspected.

### 3.2.4. TVM has no separate Bitstring value type.

Notice that TVM offers no separate bitstring value type. Instead, bitstrings are represented by Slices that happen to have no references at all, but can still contain up to 1023 data bits.

### 3.2.5. Cells and cell primitives are bit-oriented, not byte-oriented.

An important point is that TVM regards data kept in cells as sequences (strings, streams) of (up to 1023) bits, not of bytes. In other words, TVM is a bit-oriented machine, not a byte-oriented machine. If necessary, an application is free to use, say, 21-bit integer fields inside records serialized into TVM cells, thus using fewer persistent storage bytes to represent the same data.

### 3.2.6. Taxonomy of cell creation (serialization) primitives.

Cell creation primitives usually accept a Builder argument and an argument representing the value to be serialized. Additional arguments controlling some aspects of the serialization process (e.g., how many bits should be used for serialization) can be also provided, either in the stack or as an immediate value inside the instruction. The result of a cell creation primitive is usually another Builder, representing the concatenation of the original builder and the serialization of the value provided.

Therefore, one can suggest a classification of cell serialization primitives according to the answers to the following questions:

* Which is the type of values being serialized?
* How many bits are used for serialization? If this is a variable number, does it come from the stack, or from the instruction itself? - What happens if the value does not fit into the prescribed number of bits? Is an exception generated, or is a success flag equal to zero silently returned in the top of stack?
* What happens if there is insufficient space left in the Builder? Is an exception generated, or is a zero success flag returned along with the unmodified original Builder?

The mnemonics of cell serialization primitives usually begin with ST. Subsequent letters describe the following attributes:

* The type of values being serialized and the serialization format (e.g., I for signed integers, $$U$$ for unsigned integers).
* The source of the field width in bits to be used (e.g., X for integer serialization instructions means that the bit width $$n$$ is supplied in the stack; otherwise it has to be embedded into the instruction as an immediate value).
* The action to be performed if the operation cannot be completed (by default, an exception is generated; "quiet" versions of serialization instructions are marked by a $$Q$$ letter in their mnemonics).

This classification scheme is used to create a more complete taxonomy of cell serialization primitives, which can be found in $$\mathbf{A . 7 . 1}$$

### 3.2.7. Integer serialization primitives.

Integer serialization primitives can be classified according to the above taxonomy as well. For example:

* There are signed and unsigned (big-endian) integer serialization primitives.
* The size $$n$$ of the bit field to be used ( $$1 \leq n \leq 257$$ for signed integers, $$0 \leq n \leq 256$$ for unsigned integers) can either come from the top of stack or be embedded into the instruction itself.
* If the integer $$x$$ to be serialized is not in the range $$-2^{n-1} \leq x<2^{n-1}$$ (for signed integer serialization) or $$0 \leq x<2^{n}$$ (for unsigned integer serialization), a range check exception is usually generated, and if $$n$$ bits cannot be stored into the provided Builder, a cell overflow exception is generated. - Quiet versions of serialization instructions do not throw exceptions; instead, they push -1 on top of the resulting Builder upon success, or return the original Builder with 0 on top of it to indicate failure.

Integer serialization instructions have mnemonics like STU 20 ("store an unsigned 20-bit integer value") or STIXQ ("quietly store an integer value of variable length provided in the stack"). The full list of these instructionsincluding their mnemonics, descriptions, and opcodes - is provided in $$\mathbf{A . 7 . 1}$$

### 3.2.8. Integers in cells are big-endian by default.

Notice that the default order of bits in Integers serialized into Cells is big-endian, not littleendian.$$^{14}$$ In this respect $$T V M$$ is a big-endian machine. However, this affects only the serialization of integers inside cells. The internal representation of the Integer value type is implementation-dependent and irrelevant for the operation of TVM. Besides, there are some special primitives such as STULE for (de)serializing little-endian integers, which must be stored into an integral number of bytes (otherwise "little-endianness" does not make sense, unless one is also willing to revert the order of bits inside octets). Such primitives are useful for interfacing with the little-endian world-for instance, for parsing custom-format messages arriving to a TVM Blockchain smart contract from the outside world.

### 3.2.9. Other serialization primitives.

Other cell creation primitives serialize bitstrings (i.e., cell slices without references), either taken from the stack or supplied as literal arguments; cell slices (which are concatenated to the cell builder in an obvious way); other Builders (which are also concatenated); and cell references $$(\mathrm{STREF})$$.

### 3.2.10. Other cell creation primitives.

In addition to the cell serialization primitives for certain built-in value types described above, there are simple primitives that create a new empty Builder and push it into the stack (NEWC), or transform a Builder into a Cell (ENDC), thus finishing the cell creation process. An ENDC can be combined with a STREF into a single instruction ENDCST, which finishes the creation of a cell and immediately stores a reference to it in an "outer" Builder. There are also primitives that obtain the quantity of data bits or references already stored in a Builder, and check how many data bits or references can be stored.

$${ }^{14}$$Negative numbers are represented using two's complement. For instance, integer -17 is serialized by instruction STI 8 into bitstring $$\mathrm{xEF}$$.

### 3.2.11. Taxonomy of cell deserialisation primitives.

Cell parsing, or deserialization, primitives can be classified as described in 3.2.6, with the following modifications:

* They work with Slices (representing the remainder of the cell being parsed) instead of Builders.
* They return deserialized values instead of accepting them as arguments.
* They may come in two flavors, depending on whether they remove the deserialized portion from the Slice supplied ("fetch operations") or leave it unmodified ("prefetch operations").
* Their mnemonics usually begin with LD (or PLD for prefetch operations) instead of ST.

For example, an unsigned big-endian 20-bit integer previously serialized into a cell by a STU 20 instruction is likely to be deserialized later by a matching LDU 20 instruction.

Again, more detailed information about these instructions is provided in $$\mathbf{A . 7 . 2}$$.

### 3.2.12. Other cell slice primitives.

In addition to the cell deserialisation primitives outlined above, TVM provides some obvious primitives for initializing and completing the cell deserialization process. For instance, one can convert a Cell into a Slice (CTOS), so that its deserialisation might begin; or check whether a Slice is empty, and generate an exception if it is not (ENDS); or deserialize a cell reference and immediately convert it into a Slice (LDREFTOS, equivalent to two instructions LDREF and CTOS).

### 3.2.13. Modifying a serialized value in a cell.

The reader might wonder how the values serialized inside a cell may be modified. Suppose a cell contains three serialized 29-bit integers, $$(x, y, z)$$, representing the coordinates of a point in space, and we want to replace $$y$$ with $$y^{\prime}=y+1$$, leaving the other coordinates intact. How would we achieve this?

TVM does not offer any ways to modify existing values (cf. 2.3.4 and 2.3.5, so our example can only be accomplished with a series of operations as follows:

1. Deserialize the original cell into three Integers $$x, y, z$$ in the stack (e.g., by CTOS; LDI 29; LDI 29; LDI 29; ENDS). 2. Increase $$y$$ by one (e.g., by SWAP; INC; SWAP).
2. Finally, serialize the resulting Integers into a new cell (e.g., by XCHG s2; NEWC; STI 29; STI 29; STI 29; ENDC).

### 3.2.14. Modifying the persistent storage of a smart contract.

If the TVM code wants to modify its persistent storage, represented by the tree of cells rooted at $$c 4$$, it simply needs to rewrite control register c4 by the root of the tree of cells containing the new value of its persistent storage. (If only part of the persistent storage needs to be modified, cf. 3.2.13.)

## Hashmaps, or dictionaries

Hashmaps, or dictionaries, are a specific data structure represented by a tree of cells. Essentially, a hashmap represents a map from keys, which are bitstrings of either fixed or variable length, into values of an arbitrary type $$X$$, in such a way that fast lookups and modifications be possible. While any such structure might be inspected or modified with the aid of generic cell serialization and deserialization primitives, TVM introduces special primitives to facilitate working with these hashmaps.

### 3.3.1. Basic hashmap types.

The two most basic hashmap types predefined in TVM are HashmapE $$n X$$ or HashmapE( $$n, X)$$, which represents a partially defined map from $$n$$-bit strings (called keys) for some fixed $$0 \leq$$ $$n \leq 1023$$ into values of some type $$X$$, and $$\operatorname{Hashmap}(n, X)$$, which is similar to HashmapE $$(n, X)$$ but is not allowed to be empty (i.e., it must contain at least one key-value pair).

Other hashmap types are also available-for example, one with keys of arbitrary length up to some predefined bound (up to 1023 bits).

### 3.3.2. Hashmaps as Patricia trees.

The abstract representation of a hashmap in TVM is a Patricia tree, or a compact binary trie. It is a binary tree with edges labelled by bitstrings, such that the concatenation of all edge labels on a path from the root to a leaf equals a key of the hashmap. The corresponding value is kept in this leaf (for hashmaps with keys of fixed length), or optionally in the intermediate vertices as well (for hashmaps with keys of variable length). Furthermore, any intermediate vertex must have two children, and the label of the left child must begin with a binary zero, while the label of the right child must begin with a binary one. This enables us not to store the first bit of the edge labels explicitly. It is easy to see that any collection of key-value pairs (with distinct keys) is represented by a unique Patricia tree.

### 3.3.3. Serialization of hashmaps.

The serialization of a hashmap into a tree of cells (or, more generally, into a Slice) is defined by the following TL-B scheme:

![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-038.jpg?height=1116\&width=1351\&top\_left\_y=757\&top\_left\_x=362)

### 3.3.4. Brief explanation of TL-B schemes.

The right-hand side of each "equation" is a type, either simple (such as Bit or True) or parametrized (such as Hashmap $$n X$$ ). The parameters of a type must be either natural numbers (i.e., non-negative integers, which are required to fit into 32 bits in practice), such as $$n$$ in Hashmap $$n X$$, or other types, such as $$X$$ in Hashmap $$n X$$.

The left-hand side of each equation describes a way to define, or even to serialize, a value of the type indicated in the right-hand side. Such a description begins with the name of a constructor, such as hm\_edge or hml\_long, immediately followed by an optional constructor tag, such as $$\#\_\$$ or $$\$10\$$, which describes the bitstring used to encode (serialize) the constructor in question. Such tags may be given in either binary (after a dollar sign) or hexadecimal notation (after a hash sign), using the conventions described in $$\mathbf{1 . 0}$$. If a tag is not explicitly provided, TL-B computes a default 32-bit constructor tag by hashing the text of the "equation" defining this constructor in a certain fashion. Therefore, empty tags must be explicitly provided by $$\#\_\$$or $$\$\_\$$. All constructor names must be distinct, and constructor tags for the same type must constitute a prefix code (otherwise the deserialization would not be unique).

The constructor and its optional tag are followed by field definitions. Each field definition is of the form ident : type-expr, where ident is an identifier with the name of the field $${ }^{16}$$ (replaced by an underscore for anonymous fields), and type-expr is the field's type. The type provided here is a type expression, which may include simple types or parametrized types with suitable parameters. Variables - i.e., the (identifiers of the) previously defined fields of types \\# (natural numbers) or Type (type of types)-may be used as parameters for the parametrized types. The serialization process recursively serializes each field according to its type, and the serialization of a value ultimately consists of the concatenation of bitstrings representing the constructor (i.e., the constructor tag) and the field values.

Some fields may be implicit. Their definitions are surrounded by curly braces, which indicate that the field is not actually present in the serialization, but that its value must be deduced from other data (usually the parameters of the type being serialized).

Some occurrences of "variables" (i.e., already-defined fields) are prefixed by a tilde. This indicates that the variable's occurrence is used in the opposite way of the default behavior: in the left-hand side of the equation, it means that the variable will be deduced (computed) based on this occurrence, instead of substituting its previously computed value; in the right-hand side, conversely, it means that the variable will not be deduced from the type being serialized, but rather that it will be computed during the deserialization pro-

$${ }^{16}$$ The field's name is useful for representing values of the type being defined in humanreadable form, but it does not affect the binary serialization. cess. In other words, a tilde transforms an "input argument" into an "output argument", and vice versa. $${ }^{17}$$

Finally, some equalities may be included in curly brackets as well. These are certain "equations", which must be satisfied by the "variables" included in them. If one of the variables is prefixed by a tilde, its value will be uniquely determined by the values of all other variables participating in the equation (which must be known at this point) when the definition is processed from the left to the right.

A caret (^) preceding a type $$X$$ means that instead of serializing a value of type $$X$$ as a bitstring inside the current cell, we place this value into a separate cell, and add a reference to it into the current cell. Therefore $${ }^{\wedge} X$$ means "the type of references to cells containing values of type $$X$$ ".

Parametrized type $$\#<=p$$ with $$p: \#$$ (this notation means " $$p$$ of type \\#", i.e., a natural number) denotes the subtype of the natural numbers type $$\#$$, consisting of integers $$0 \ldots p$$; it is serialized into $$\left\lceil\log _{2}(p+1)\right\rceil$$ bits as an unsigned big-endian integer. Type \\# by itself is serialized as an unsigned 32-bit integer. Parametrized type \\#\\# $$b$$ with $$b: \#<=31$$ is equivalent to \\#<= $$2^{b}-1$$ (i.e., it is an unsigned $$b$$-bit integer).

### 3.3.5. Application to the serialization of hashmaps.

Let us explain the net result of applying the general rules described in 3.3.4 to the TL-B scheme presented in $$\mathbf{3 . 3 . 3}$$

Suppose we wish to serialize a value of type HashmapE $$n X$$ for some integer $$0 \leq n \leq 1023$$ and some type $$X$$ (i.e., a dictionary with $$n$$-bit keys and values of type $$X$$, admitting an abstract representation as a Patricia tree $$(\operatorname{cf} .3 .3 .2)$$ ).

First of all, if our dictionary is empty, it is serialized into a single binary 0 , which is the tag of nullary constructor hme\_empty. Otherwise, its serialization consists of a binary 1 (the tag of hme\_root), along with a reference to a cell containing the serialization of a value of type Hashmap $$n X$$ (i.e., a necessarily non-empty dictionary).

The only way to serialize a value of type Hashmap $$n X$$ is given by the hm\_edge constructor, which instructs us to serialize first the label label of the edge leading to the root of the subtree under consideration (i.e., the common prefix of all keys in our (sub)dictionary). This label is of type HmLabel $$l^{\perp} n$$, which means that it is a bitstring of length at most $$n$$, serialized in such a way that the true length $$l$$ of the label, $$0 \leq l \leq n$$, becomes known from

$${ }^{17}$$ This is the "linear negation" operation $$(-)^{\perp}$$ of linear logic, hence our notation . the serialization of the label. (This special serialization method is described separately in $$\mathbf{3 . 3 . 6}$$.)

The label must be followed by the serialization of a node of type HashmapNode $$m X$$, where $$m=n-l$$. It corresponds to a vertex of the Patricia tree, representing a non-empty subdictionary of the original dictionary with $$m$$-bit keys, obtained by removing from all the keys of the original subdictionary their common prefix of length $$l$$.

If $$m=0$$, a value of type HashmapNode $$0 X$$ is given by the hmn\_leaf constructor, which describes a leaf of the Patricia tree or, equivalently, a subdictionary with 0-bit keys. A leaf simply consists of the corresponding value of type $$X$$ and is serialized accordingly.

On the other hand, if $$m>0$$, a value of type HashmapNode $$m X$$ corresponds to a fork (i.e., an intermediate node) in the Patricia tree, and is given by the hmn\_fork constructor. Its serialization consists of left and right, two references to cells containing values of type Hashmap $$m-1 X$$, which correspond to the left and the right child of the intermediate node in question-or, equivalently, to the two subdictionaries of the original dictionary consisting of key-value pairs with keys beginning with a binary 0 or a binary 1, respectively. Because the first bit of all keys in each of these subdictionaries is known and fixed, it is removed, and the resulting (necessarily non-empty) subdictionaries are recursively serialized as values of type Hashmap $$m-1 X$$.

### 3.3.6. Serialization of labels.

There are several ways to serialize a label of length at most $$n$$, if its exact length is $$l \leq n$$ (recall that the exact length must be deducible from the serialization of the label itself, while the upper bound $$n$$ is known before the label is serialized or deserialized). These ways are described by the three constructors hml\_short, hml\_long, and hml\_same of type HmLabel $$l^{\perp} n$$ :

* hml\_short - Describes a way to serialize "short" labels, of small length $$l \leq n$$. Such a serialization consists of a binary 0 (the constructor tag of $$\mathrm{hml}$$ \_short), followed by $$l$$ binary 1 s and one binary 0 (the unary representation of the length $$l$$ ), followed by $$l$$ bits comprising the label itself.
* hml\_long - Describes a way to serialize "long" labels, of arbitrary length $$l \leq n$$. Such a serialization consists of a binary 10 (the constructor tag of hml\_long), followed by the big-endian binary representation of the length $$0 \leq l \leq n$$ in $$\left\lceil\log _{2}(n+1)\right\rceil$$ bits, followed by $$l$$ bits comprising the label itself.
* hml\_same - Describes a way to serialize "long" labels, consisting of $$l$$ repetitions of the same bit $$v$$. Such a serialization consists of 11 (the constructor tag of $$\mathrm{hml}$$ same), followed by the bit $$v$$, followed by the length $$l$$ stored in $$\left\lceil\log _{2}(n+1)\right\rceil$$ bits as before.

Each label can always be serialized in at least two different fashions, using hml\_short or hml\_long constructors. Usually the shortest serialization (and in the case of a tie-the lexicographically smallest among the shortest) is preferred and is generated by TVM hashmap primitives, while the other variants are still considered valid.

This label encoding scheme has been designed to be efficient for dictionaries with "random" keys (e.g., hashes of some data), as well as for dictionaries with "regular" keys (e.g., big-endian representations of integers in some range).

### 3.3.7. An example of dictionary serialization.

Consider a dictionary with three 16-bit keys 13, 17, and 239 (considered as big-endian integers) and corresponding 16-bit values 169,289 , and 57121.

In binary form:

$$0000000000001101 \Rightarrow 0000000010101001$$ $$0000000000010001 \Rightarrow 0000000100100001$$ $$0000000011101111 \Rightarrow 1101111100100001$$

The corresponding Patricia tree consists of a root $$A$$, two intermediate nodes $$B$$ and $$C$$, and three leaf nodes $$D, E$$, and $$F$$, corresponding to 13,17 , and 239 , respectively. The root $$A$$ has only one child, $$B$$; the label on the edge $$A B$$ is $$00000000=0^{8}$$. The node $$B$$ has two children: its left child is an intermediate node $$C$$ with the edge $$B C$$ labelled by (0)00, while its right child is the leaf $$F$$ with $$B F$$ labelled by (1)1101111. Finally, $$C$$ has two leaf children $$D$$ and $$E$$, with $$C D$$ labelled by (0)1101 and $$C E-$$ by (1)0001.

The corresponding value of type HashmapE 16 (\\#\\# 16) may be written in human-readable form as:

![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-042.jpg?height=161\&width=1254\&top\_left\_y=2205\&top\_left\_x=379)

![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-043.jpg?height=360\&width=1114\&top\_left\_y=449\&top\_left\_x=451)

The serialization of this data structure into a tree of cells consists of six cells with the following binary data contained in them: $$\mathrm{A}:=1$$ A. $$0:=11001000$$ A.0.0:=0 11000 A.0.0.0:=10 10011010000000010101001 A.0.0.1:=1010000010000000100100001 A.0.1:=10 11111011111101111100100001

Here $$A$$ is the root cell, $$A .0$$ is the cell at the first reference of $$A, A .1$$ is the cell at the second reference of $$A$$, and so on. This tree of cells can be represented more compactly using the hexadecimal notation described in $$\mathbf{1 . 0}$$ using indentation to reflect the tree-of-cells structure:

![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-043.jpg?height=301\&width=262\&top\_left\_y=1538\&top\_left\_x=365)

A total of 93 data bits and 5 references in 6 cells have been used to serialize this dictionary. Notice that a straightforward representation of three 16bit keys and their corresponding 16-bit values would already require 96 bits (albeit without any references), so this particular serialization turns out to be quite efficient.

### 3.3.8. Ways to describe the serialization of type $$X$$.

Notice that the built-in TVM primitives for dictionary manipulation need to know something about the serialization of type $$X$$; otherwise, they would not be able to work correctly with Hashmap $$n X$$, because values of type $$X$$ are immediately contained in the Patricia tree leaf cells. There are several options available to describe the serialization of type $$X$$ :

* The simplest case is when $$X=^{\wedge} Y$$ for some other type $$Y$$. In this case the serialization of $$X$$ itself always consists of one reference to a cell, which in fact must contain a value of type $$Y$$, something that is not relevant for dictionary manipulation primitives.
* Another simple case is when the serialization of any value of type $$X$$ always consists of $$0 \leq b \leq 1023$$ data bits and $$0 \leq r \leq 4$$ references. Integers $$b$$ and $$r$$ can then be passed to a dictionary manipulation primitive as a simple description of $$X$$. (Notice that the previous case corresponds to $$b=0, r=1$$.)
* A more sophisticated case can be described by four integers $$1 \leq b_{0}, b_{1} \leq$$ 1023, $$0 \leq r_{0}, r_{1} \leq 4$$, with $$b_{i}$$ and $$r_{i}$$ used when the first bit of the serialization equals $$i$$. When $$b_{0}=b_{1}$$ and $$r_{0}=r_{1}$$, this case reduces to the previous one.
* Finally, the most general description of the serialization of a type $$X$$ is given by a splitting function split $$_{X}$$ for $$X$$, which accepts one Slice parameter $$s$$, and returns two Slices, $$s^{\prime}$$ and $$s^{\prime \prime}$$, where $$s^{\prime}$$ is the only prefix of $$s$$ that is the serialization of a value of type $$X$$, and $$s^{\prime \prime}$$ is the remainder of $$s$$. If no such prefix exists, the splitting function is expected to throw an exception. Notice that a compiler for a high-level language, which supports some or all algebraic TL-B types, is likely to automatically generate splitting functions for all types defined in the program.

### 3.3.9. A simplifying assumption on the serialization of $$X$$.

One may notice that values of type $$X$$ always occupy the remaining part of an hm\_edge/hme\_leaf cell inside the serialization of a HashmapE $$n X$$. Therefore, if we do not insist on strict validation of all dictionaries accessed, we may assume that everything left unparsed in an hm\_edge/hme\_leaf cell after deserializing its label is a value of type $$X$$. This greatly simplifies the creation of dictionary manipulation primitives, because in most cases they turn out not to need any information about $$X$$ at all.

### 3.3.10. Basic dictionary operations.

Let us present a classification of basic operations with dictionaries (i.e., values $$D$$ of type Hashmap $$E \quad X)$$ : - $$\operatorname{Get}(D, k)$$ - Given $$D: \operatorname{Hashmap} E(n, X)$$ and a key $$k: n \cdot$$ bit, returns the corresponding value $$D[k]: X^{?}$$ kept in $$D$$.

* $$\operatorname{Set}(D, k, x)$$ - Given $$D: \operatorname{Hashmap} E(n, X)$$, a key $$k: n$$ · bit, and a value $$x: X$$, sets $$D^{\prime}[k]$$ to $$x$$ in a copy $$D^{\prime}$$ of $$D$$, and returns the resulting dictionary $$D^{\prime}$$ (cf. 2.3.4).
* $$\operatorname{AdD}(D, k, x)$$ - Similar to SET, but adds the key-value pair $$(k, x)$$ to $$D$$ only if key $$k$$ is absent in $$D$$.
* $$\operatorname{Replace}(D, k, x)$$ - Similar to Set, but changes $$D^{\prime}[k]$$ to $$x$$ only if key $$k$$ is already present in $$D$$.
* Getset, Getadd, GetReplace - Similar to Set, Add, and RePLACE, respectively, but returns the old value of $$D[k]$$ as well.
* $$\operatorname{Delete}(D, k)$$ - Deletes key $$k$$ from dictionary $$D$$, and returns the resulting dictionary $$D^{\prime}$$.
* Getmin $$(D)$$, Getmax $$(D)$$ - Gets the minimal or maximal key $$k$$ from dictionary $$D$$, along with the associated value $$x: X$$.
* Removemin $$(D)$$, Removemax $$(D)$$ - Similar to GetMin and GetMAX, but also removes the key in question from dictionary $$D$$, and returns the modified dictionary $$D^{\prime}$$. May be used to iterate over all elements of $$D$$, effectively using (a copy of) $$D$$ itself as an iterator.
* GetNext $$(D, k)$$ - Computes the minimal key $$k^{\prime}>k$$ (or $$k^{\prime} \geq k$$ in a variant) and returns it along with the corresponding value $$x^{\prime}: X$$. May be used to iterate over all elements of $$D$$.
* $$\operatorname{GetPrev}(D, k)$$ - Computes the maximal key $$k^{\prime}<k$$ (or $$k^{\prime} \leq k$$ in a variant) and returns it along with the corresponding value $$x^{\prime}: X$$.
* Empty $$(n)$$ - Creates an empty dictionary $$D: \operatorname{Hashmap} E(n, X)$$.
* IsEmpty $$(D)$$ - Checks whether a dictionary is empty.
* $$\operatorname{Create}\left(n,\left\{\left(k_{i}, x_{i}\right)\right\}\right)$$ - Given $$n$$, creates a dictionary from a list $$\left(k_{i}, x_{i}\right)$$ of key-value pairs passed in stack. - $$\operatorname{GetSubdict}\left(D, l, k_{0}\right)$$ - Given $$D: \operatorname{Hashmap} E(n, X)$$ and some $$l$$-bit string $$k_{0}: l \cdot$$ bit for $$0 \leq l \leq n$$, returns subdictionary $$D^{\prime}=D / k_{0}$$ of $$D$$, consisting of keys beginning with $$k_{0}$$. The result $$D^{\prime}$$ may be of either type $$\operatorname{Hashmap} E(n, X)$$ or type $$\operatorname{Hashmap} E(n-l, X)$$.
* ReplaceSubdict $$\left(D, l, k_{0}, D^{\prime}\right)$$ - Given $$D: \operatorname{Hashmap} E(n, X), 0 \leq$$ $$l \leq n, k_{0}: l \cdot$$ bit, and $$D^{\prime}: \operatorname{Hashmap} E(n-l, X)$$, replaces with $$D^{\prime}$$ the subdictionary $$D / k_{0}$$ of $$D$$ consisting of keys beginning with $$k_{0}$$, and returns the resulting dictionary $$D^{\prime \prime}: \operatorname{Hash} \operatorname{map} E(n, X)$$. Some variants of REPLACESUBDICT may also return the old value of the subdictionary $$D / k_{0}$$ in question.
* $$\operatorname{DeleteSubdict}\left(D, l, k_{0}\right)$$ - Equivalent to ReplaceSubdict with $$D^{\prime}$$ being an empty dictionary.
* $$\operatorname{Split}(D)$$ - Given $$D$$ : HashmapE(n,X), returns $$D_{0}:=D / 0$$ and $$D_{1}:=D / 1:$$ Hashmap $$E(n-1, X)$$, the two subdictionaries of $$D$$ consisting of all keys beginning with 0 and 1, respectively.
* $$\operatorname{Merge}\left(D_{0}, D_{1}\right)$$ - Given $$D_{0}$$ and $$D_{1}: \operatorname{Hashmap} E(n-1, X)$$, computes $$D: \operatorname{Hashmap} E(n, X)$$, such that $$D / 0=D_{0}$$ and $$D / 1=D_{1}$$.
* Foreach $$(D, f)$$ - Executes a function $$f$$ with two arguments $$k$$ and $$x$$, with $$(k, x)$$ running over all key-value pairs of a dictionary $$D$$ in lexicographical order $${ }^{18}$$
* ForeachRev $$(D, f)$$ - Similar to Foreach, but processes all keyvalue pairs in reverse order.
* TreeReduce $$(D, o, f, g)$$ - Given $$D: \operatorname{Hashmap} E(n, X)$$, a value $$o: X$$, and two functions $$f: X \rightarrow Y$$ and $$g: Y \times Y \rightarrow Y$$, performs a "tree reduction" of $$D$$ by first applying $$f$$ to all the leaves, and then using $$g$$ to compute the value corresponding to a fork starting from the values assigned to its children. $${ }^{19}$$

$${ }^{18}$$ In fact, $$f$$ may receive $$m$$ extra arguments and return $$m$$ modified values, which are passed to the next invocation of $$f$$. This may be used to implement "map" and "reduce" operations with dictionaries.

$${ }^{19}$$ Versions of this operation may be introduced where $$f$$ and $$g$$ receive an additional bitstring argument, equal to the key (for leaves) or to the common prefix of all keys (for forks) in the corresponding subtree. 3.3.11. Taxonomy of dictionary primitives. The dictionary primitives, described in detail in $$\mathbf{A . 1 0}$$ can be classified according to the following categories:

* Which dictionary operation (cf. 3.3.10 do they perform?
* Are they specialized for the case $$X={ }^{\wedge} Y$$ ? If so, do they represent values of type $$Y$$ by Cells or by Slices? (Generic versions always represent values of type $$X$$ as Slices.)
* Are the dictionaries themselves passed and returned as Cells or as Slices? (Most primitives represent dictionaries as Slices.)
* Is the key length $$n$$ fixed inside the primitive, or is it passed in the stack?
* Are the keys represented by Slices, or by signed or unsigned Integers?

In addition, TVM includes special serialization/deserialization primitives, such as STDICT, LDDICT, and PLDDICT. They can be used to extract a dictionary from a serialization of an encompassing object, or to insert a dictionary into such a serialization.

## Hashmaps with variable-length keys

TVM provides some support for dictionaries, or hashmaps, with variablelength keys, in addition to its support for dictionaries with fixed-length keys (as described in 3.3 above).

### 3.4.1. Serialization of dictionaries with variable-length keys.

The serialization of a VarHashmap into a tree of cells (or, more generally, into a Slice) is defined by a TL-B scheme, similar to that described in $$\mathbf{3 . 3 . 3}$$

![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-047.jpg?height=412\&width=1322\&top\_left\_y=1952\&top\_left\_x=369)

![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-048.jpg?height=412\&width=1073\&top\_left\_y=450\&top\_left\_x=362)

### 3.4.2. Serialization of prefix codes.

One special case of a dictionary with variable-length keys is that of a prefix code, where the keys cannot be prefixes of each other. Values in such dictionaries may occur only in the leaves of a Patricia tree.

The serialization of a prefix code is defined by the following TL-B scheme:

![](https://cdn.mathpix.com/cropped/2023\_06\_02\_174e9ec2591c06b3f394g-048.jpg?height=570\&width=1370\&top\_left\_y=1163\&top\_left\_x=364)
