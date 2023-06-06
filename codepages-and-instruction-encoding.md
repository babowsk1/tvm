# Codepages and instruction encoding

This chapter describes the codepage mechanism, which allows TVM to be flexible and extendable while preserving backward compatibility with respect to previously generated code.

We also discuss some general considerations about instruction encodings (applicable to arbitrary machine code, not just TVM), as well as the implications of these considerations for TVM and the choices made while designing TVM's (experimental) codepage zero. The instruction encodings themselves are presented later in Appendix $$\mathbf{A}$$.

## Codepages and interoperability of different TVM versions

The codepages are an essential mechanism of backward compatibility and of future extensions to TVM. They enable transparent execution of code written for different revisions of TVM, with transparent interaction between instances of such code. The mechanism of the codepages, however, is general and powerful enough to enable some other originally unintended applications.

### 5.1.1. 

Codepages in continuations. Every ordinary continuation contains a 16-bit codepage field $$\mathrm{cp}$$ (cf. 4.1.1), which determines the codepage that will be used to execute its code. If a continuation is created by a PUSHCONT (cf. 4.2.3 or similar primitive, it usually inherits the current codepage (i.e., the codepage of cc)

### 5.1.2. 

Current codepage. The current codepage cp (cf. 1.4) is the codepage of the current continuation cc. It determines the way the next instruction will be decoded from cc.code, the remainder of the current continuation's code. Once the instruction has been decoded and executed, it determines the next value of the current codepage. In most cases, the current codepage is left unchanged.

On the other hand, all primitives that switch the current continuation load the new value of $$\mathrm{cp}$$ from the new current continuation. In this way, all code in continuations is always interpreted exactly as it was intended to be.

$${ }^{25}$$ This is not exactly true. A more precise statement is that usually the codepage of the newly-created continuation is a known function of the current codepage. 

### 5.1.3. 

Different versions of TVM may use different codepages. Different versions of TVM may use different codepages for their code. For example, the original version of TVM might use codepage zero. A newer version might use codepage one, which contains all the previously defined opcodes, along with some newly defined ones, using some of the previously unused opcode space. A subsequent version might use yet another codepage, and so on.

However, a newer version of TVM will execute old code for codepage zero exactly as before. If the old code contained an opcode used for some new operations that were undefined in the original version of TVM, it will still generate an invalid opcode exception, because the new operations are absent in codepage zero.

### 5.1.4. 

Changing the behavior of old operations. New codepages can also change the effects of some operations present in the old codepages while preserving their opcodes and mnemonics.

For example, imagine a future 513-bit upgrade of TVM (replacing the current 257-bit design). It might use a 513-bit Integer type within the same arithmetic primitives as before. However, while the opcodes and instructions in the new codepage would look exactly like the old ones, they would work differently, accepting 513-bit integer arguments and results. On the other hand, during the execution of the same code in codepage zero, the new machine would generate exceptions whenever the integers used in arithmetic and other primitives do not fit into 257 bits $${ }^{26}$$ In this way, the upgrade would not change the behavior of the old code.

### 5.1.5. 

Improving instruction encoding. Another application for codepages is to change instruction encodings, reflecting improved knowledge of the actual frequencies of such instructions in the code base. In this case, the new codepage will have exactly the same instructions as the old one, but with different encodings, potentially of differing lengths. For example, one might create an experimental version of the first version of TVM, using a

$${ }^{26}$$ This is another important mechanism of backward compatibility. All values of newlyadded types, as well as values belonging to extended original types that do not belong to the original types (e.g., 513-bit integers that do not fit into 257 bits in the example above), are treated by all instructions (except stack manipulation instructions, which are naturally polymorphic, cf. 2.2.6 in the old codepages as "values of incorrect type", and generate type-checking exceptions accordingly. (prefix) bitcode instead of the original bytecode, aiming to achieve higher code density.

### 5.1.6. 

Making instruction encoding context-dependent. Another way of using codepages to improve code density is to use several codepages with different subsets of the whole instruction set defined in each of them, or with the whole instruction set defined, but with different length encodings for the same instructions in different codepages.

Imagine, for instance, a "stack manipulation" codepage, where stack manipulation primitives have short encodings at the expense of all other operations, and a "data processing" codepage, where all other operations are shorter at the expense of stack manipulation operations. If stack manipulation operations tend to come one after another, we can automatically switch to "stack manipulation" codepage after executing any such instruction. When a data processing instruction occurs, we switch back to "data processing" codepage. If conditional probabilities of the class of the next instruction depending on the class of the previous instruction are considerably different from corresponding unconditional probabilities, this techniqueautomatically switching into stack manipulation mode to rearrange the stack with shorter instructions, then switching back-might considerably improve the code density.

### 5.1.7. 

Using codepages for status and control flags. Another potential application of multiple codepages inside the same revision of TVM consists in switching between several codepages depending on the result of the execution of some instructions.

For example, imagine a version of TVM that uses two new codepages, 2 and 3. Most operations do not change the current codepage. However, the integer comparison operations will switch to codepage 2 if the condition is false, and to codepage 3 if it is true. Furthermore, a new operation ?EXECUTE, similar to EXECUTE, will indeed be equivalent to EXECUTE in codepage 3 , but will instead be a DROP in codepage 2. Such a trick effectively uses bit 0 of the current codepage as a status flag.

Alternatively, one might create a couple of codepages-say, 4 and 5 which differ only in their cell deserialisation primitives. For instance, in codepage 4 they might work as before, while in codepage 5 they might deserialize data not from the beginning of a Slice, but from its end. Two new instructions-say, CLD and STD-might be used for switching to codepage 4 or codepage 5. Clearly, we have now described a status flag, affecting the execution of some instructions in a certain new manner.

### 5.1.8. 

Setting the codepage in the code itself. For convenience, we reserve some opcode in all codepages-say, FF $$n$$-for the instruction SETCP $$n$$, with $$n$$ from 0 to 255 (cf. A.13). Then by inserting such an instruction into the very beginning of (the main function of) a program (e.g., a TON Blockchain smart contract) or a library function, we can ensure that the code will always be executed in the intended codepage.

## Instruction encoding

This section discusses the general principles of instruction encoding valid for all codepages and all versions of TVM. Later, 5.3 discusses the choices made for the experimental "codepage zero".

### 5.2.1. 

Instructions are encoded by a binary prefix code. All complete instructions (i.e., instructions along with all their parameters, such as the names of stack registers $$s(i)$$ or other embedded constants) of a TVM codepage are encoded by a binary prefix code. This means that a (finite) binary string (i.e., a bitstring) corresponds to each complete instruction, in such a way that binary strings corresponding to different complete instructions do not coincide, and no binary string among the chosen subset is a prefix of another binary string from this subset.

### 5.2.2. 

Determining the first instruction from a code stream. As a consequence of this encoding method, any binary string admits at most one prefix, which is an encoding of some complete instruction. In particular, the code cc.code of the current continuation (which is a Slice, and thus a bitstring along with some cell references) admits at most one such prefix, which corresponds to the (uniquely determined) instruction that TVM will execute first. After execution, this prefix is removed from the code of the current continuation, and the next instruction can be decoded.

### 5.2.3. 

Invalid opcode. If no prefix of cc.code encodes a valid instruction in the current codepage, an invalid opcode exception is generated (cf. 4.5.7). However, the case of an empty cc.code is treated separately as explained in 4.1.4 (the exact behavior may depend on the current codepage). 

### 5.2.4. 

Special case: end-of-code padding. As an exception to the above rule, some codepages may accept some values of cc. code that are too short to be valid instruction encodings as additional variants of NOP, thus effectively using the same procedure for them as for an empty cc.code. Such bitstrings may be used for padding the code near its end.

For example, if binary string 00000000 (i.e., x00, cf. $$\mathbf{1 . 0 . 3}$$ is used in a codepage to encode NOP, its proper prefixes cannot encode any instructions. So this codepage may accept 0, 00, 000, ..., 0000000 as variants of NOP if this is all that is left in cc.code, instead of generating an invalid opcode exception.

Such a padding may be useful, for example, if the PUSHCONT primitive (cf. 4.2.3 creates only continuations with code consisting of an integral number of bytes, but not all instructions are encoded by an integral number of bytes.

### 5.2.5. 

TVM code is a bitcode, not a bytecode. Recall that TVM is a bit-oriented machine in the sense that its Cells (and Slices) are naturally considered as sequences of bits, not just of octets (bytes), cf. 3.2.5. Because the TVM code is also kept in cells (cf. 3.1.9 and $$\mathbf{4 . 1 . 4}$$ ), there is no reason to use only bitstrings of length divisible by eight as encodings of complete instructions. In other words, generally speaking, the TVM code is a bitcode, not a bytecode.

That said, some codepages (such as our experimental codepage zero) may opt to use a bytecode (i.e., to use only encodings consisting of an integral number of bytes) - either for simplicity, or for the ease of debugging and of studying memory (i.e., cell) dumps. $${ }^{27}$$

### 5.2.6. 

Opcode space used by a complete instruction. Recall from coding theory that the lengths of bitstrings $$l_{i}$$ used in a binary prefix code satisfy Kraft-McMillan inequality $$\sum_{i} 2^{-l_{i}} \leq 1$$. This is applicable in particular to the (complete) instruction encoding used by a TVM codepage. We say that a particular complete instruction (or, more precisely, the encoding of a complete instruction) utilizes the portion $$2^{-l}$$ of the opcode space, if it is encoded by an $$l$$-bit string. One can see that all complete instructions together utilize at most 1 (i.e., "at most the whole opcode space").

$${ }^{27}$$ If the cell dumps are hexadecimal, encodings consisting of an integral number of hexadecimal digits (i.e., having length divisible by four bits) might be equally convenient.

## INSTRUCTION ENCODING

### 5.2.7. 

Opcode space used by an instruction, or a class of instructions. The above terminology is extended to instructions (considered with all admissible values of their parameters), or even classes of instructions (e.g., all arithmetic instructions). We say that an (incomplete) instruction, or a class of instructions, occupies portion $$\alpha$$ of the opcode space, if $$\alpha$$ is the sum of the portions of the opcode space occupied by all complete instructions belonging to that class.

### 5.2.8. 

Opcode space for bytecodes. A useful approximation of the above definitions is as follows: Consider all 256 possible values for the first byte of an instruction encoding. Suppose that $$k$$ of these values correspond to the specific instruction or class of instructions we are considering. Then this instruction or class of instructions occupies approximately the portion $$k / 256$$ of the opcode space.

This approximation shows why all instructions cannot occupy together more than the portion $$256 / 256=1$$ of the opcode space, at least without compromising the uniqueness of instruction decoding.

### 5.2.9. 

Almost optimal encodings. Coding theory tells us that in an optimally dense encoding, the portion of the opcode space used by a complete instruction $$\left(2^{-l}\right.$$, if the complete instruction is encoded in $$l$$ bits) should be approximately equal to the probability or frequency of its occurrence in real programs. $${ }^{28}$$ The same should hold for (incomplete) instructions, or primitives (i.e., generic instructions without specified values of parameters), and for classes of instructions.

### 5.2.10. 

Example: stack manipulation primitives. For instance, if stack manipulation instructions constitute approximately half of all instructions in a typical TVM program, one should allocate approximately half of the opcode space for encoding stack manipulation instructions. One might reserve the first bytes ("opcodes") 0x00-0x7f for such instructions. If a quarter of these instructions are XCHG, it would make sense to reserve 0x00-0x1f for XCHGs. Similarly, if half of all XCHGs involve the top of stack s0, it would make sense to use 0x00-0x0f to encode XCHG $$\mathrm{s} 0, \mathrm{~s}(i)$$.

### 5.2.11. 

Simple encodings of instructions. In most cases, simple encodings of complete instructions are used. Simple encodings begin with a fixed

$${ }^{28}$$ Notice that it is the probability of occurrence in the code that counts, not the probability of being executed. An instruction occurring in the body of a loop executed a million times is still counted only once. bitstring called the opcode of the instruction, followed by, say, 4-bit fields containing the indices $$i$$ of stack registers $$\mathrm{s}(i)$$ specified in the instruction, followed by all other constant (literal, immediate) parameters included in the complete instruction. While simple encodings may not be exactly optimal, they admit short descriptions, and their decoding and encoding can be easily implemented.

If a (generic) instruction uses a simple encoding with an $$l$$-bit opcode, then the instruction will utilize $$2^{-l}$$ portion of the opcode space. This observation might be useful for considerations described in 5.2.9 and 5.2.10

### 5.2.12. 

Optimizing code density further: Huffman codes. One might construct optimally dense binary code for the set of all complete instructions, provided their probabilities or frequences in real code are known. This is the well-known Huffman code (for the given probability distribution). However, such code would be highly unsystematic and hard to decode.

### 5.2.13. 

Practical instruction encodings. In practice, instruction encodings used in TVM and other virtual machines offer a compromise between code density and ease of encoding and decoding. Such a compromise may be achieved by selecting simple encodings (cf. 5.2.11) for all instructions (maybe with separate simple encodings for some often used variants, such as XCHG $$\mathrm{s} 0, \mathrm{~s}(i)$$ among all XCHG $$\mathrm{s}(i), \mathrm{s}(j)$$ ), and allocating opcode space for such simple encodings using the heuristics outlined in $$\mathbf{5 . 2 . 9}$$ and $$\mathbf{5 . 2 . 1 0}$$ this is the approach currently used in TVM.

## Instruction encoding in codepage zero

This section provides details about the experimental instruction encoding for codepage zero, as described elsewhere in this document (cf. Appendix $$\mathbf{A}$$ ) and used in the preliminary test version of TVM.

### 5.3.1. 

Upgradability. First of all, even if this preliminary version somehow gets into the production version of the TON Blockchain, the codepage mechanism (cf. 5.1) enables us to introduce better versions later without compromising backward compatibility. $${ }^{29}$$ So in the meantime, we are free to experiment.

$${ }^{29}$$ Notice that any modifications after launch cannot be done unilaterally; rather they would require the support of at least two-thirds of validators. 5.3.2. Choice of instructions. We opted to include many "experimental" and not strictly necessary instructions in codepage zero just to see how they might be used in real code. For example, we have both the basic (cf. 2.2.1) and the compound (cf. 2.2.3) stack manipulation primitives, as well as some "unsystematic" ones such as ROT (mostly borrowed from Forth). If such primitives are rarely used, their inclusion just wastes some part of the opcode space and makes the encodings of other instructions slightly less effective, something we can afford at this stage of TVM's development.

### 5.3.3. 

Using experimental instructions. Some of these experimental instructions have been assigned quite long opcodes, just to fit more of them into the opcode space. One should not be afraid to use them just because they are long; if these instructions turn out to be useful, they will receive shorter opcodes in future revisions. Codepage zero is not meant to be finetuned in this respect.

### 5.3.4. 

Choice of bytecode. We opted to use a bytecode (i.e., to use encodings of complete instructions of lengths divisible by eight). While this may not produce optimal code density, because such a length restriction makes it more difficult to match portions of opcode space used for the encoding of instructions with estimated frequencies of these instructions in TVM code (cf. 5.2.11 and 5.2.9), such an approach has its advantages: it admits a simpler instruction decoder and simplifies debugging (cf. 5.2.5).

After all, we do not have enough data on the relative frequencies of different instructions right now, so our code density optimizations are likely to be very approximate at this stage. The ease of debugging and experimenting and the simplicity of implementation are more important at this point.

### 5.3.5. 

Simple encodings for all instructions. For similar reasons, we opted to use simple encodings for all instructions (cf. 5.2.11 and 5.2.13), with separate simple encodings for some very frequently used subcases as outlined in 5.2.13. That said, we tried to distribute opcode space using the heuristics described in 5.2 .9 and 5.2 .10

### 5.3.6. 

Lack of context-dependent encodings. This version of TVM also does not use context-dependent encodings (cf. 5.1.6. . They may be added at a later stage, if deemed useful.

### 5.3.7. 

The list of all instructions. The list of all instructions available in codepage zero, along with their encodings and (in some cases) short descriptions, may be found in Appendix $$\mathbf{A}$$.