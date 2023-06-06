# B Formal properties and specifications of TVM

This appendix discusses certain formal properties of TVM that are necessary for executing smart contracts in the TON Blockchain and validating such executions afterwards.

## B.1 Serialization of the TVM state

Recall that a virtual machine used for executing smart contracts in a blockchain must be deterministic, otherwise the validation of each execution would require the inclusion of all intermediate steps of the execution into a block, or at least of the choices made when indeterministic operations have been performed.

Furthermore, the state of such a virtual machine must be (uniquely) serializable, so that even if the state itself is not usually included in a block, its hash is still well-defined and can be included into a block for verification purposes.

### B.1.1. TVM stack values. 

TVM stack values can be serialized as follows:

```
vm_stk_tinyint#01 value:int64 = VmStackValue;
vm_stk_int#0201_ value:int257 = VmStackValue;
vm_stk_nan#02FF = VmStackValue;
vm_stk_cell#03 cell:^Cell = VmStackValue;
_ cell:^Cell st_bits:(## 10) end_bits:(## 10)
{ st_bits <= end_bits }
st_ref:(#<= 4) end_ref:(#<= 4)
{ st_ref <= end_ref } = VmCellSlice;
vm_stk_slice#04 _:VmCellSlice = VmStackValue;
vm_stk_builder#05 cell:^Cell = VmStackValue;
vm_stk_cont#06 cont:VmCont = VmStackValue;
```

Of these, vm_stk_tinyint is never used by TVM in codepage zero; it is used only in restricted modes.

### B.1.2. TVM stack. 

The TVM stack can be serialized as follows:

![](https://cdn.mathpix.com/cropped/2023_06_02_174e9ec2591c06b3f394g-142.jpg?height=212&width=1329&top_left_y=2154&top_left_x=365)

### B.1.3. TVM control registers. 

Control registers in TVM can be serialized as follows:

```
_ cregs: (HashmapE 4 VmStackValue) = VmSaveList;
```

### B.1.4. TVM gas limits. 

Gas limits in TVM can be serialized as follows:

![](https://cdn.mathpix.com/cropped/2023_06_02_174e9ec2591c06b3f394g-143.jpg?height=164&width=1048&top_left_y=746&top_left_x=366)

### B.1.5. TVM library environment. 

The TVM library environment can be serialized as follows:

```
_ libraries: (HashmapE 256 -Cell) = VmLibraries;
```

### B.1.6. TVM continuations. 

Continuations in TVM can be serialized as follows:

![](https://cdn.mathpix.com/cropped/2023_06_02_174e9ec2591c06b3f394g-143.jpg?height=672&width=1353&top_left_y=1281&top_left_x=362)

![](https://cdn.mathpix.com/cropped/2023_06_02_174e9ec2591c06b3f394g-144.jpg?height=306&width=1349&top_left_y=453&top_left_x=366)

When TVM is initialized, its state is described by a vms_init, usually with step set to zero. The step function of TVM does nothing to a vms_finished state, and transforms all other states into vms_running, vms_exception, or vms_finished, with step increased by one.

## B.2 Step function of TVM

A formal specification of TVM would be completed by the definition of a step function $$f:$$ VmState $$\rightarrow$$ VmState. This function deterministically transforms a valid VM state into a valid subsequent VM state, and is allowed to throw exceptions or return an invalid subsequent state if the original state was invalid.

### B.2.1. A high-level definition of the step function. 

We might present a very long formal definition of the TVM step function in a high-level functional programming language. Such a specification, however, would mostly be useful as a reference for the (human) developers. We have chosen another approach, better adapted to automated formal verification by computers.

### B.2.2. An operational definition of the step function. 

Notice that the step function $$f$$ is a well-defined computable function from trees of cells into trees of cells. As such, it can be computed by a universal Turing machine. Then a program $$P$$ computing $$f$$ on such a machine would provide a machinecheckable specification of the step function $$f$$. This program $$P$$ effectively is an emulator of TVM on this Turing machine.

### B.2.3. A reference implementation of the TVM emulator inside TVM. 

We see that the step function of TVM may be defined by a reference implementation of a TVM emulator on another machine. An obvious idea is to use TVM itself, since it is well-adapted to working with trees of cells. However, an emulator of TVM inside itself is not very useful if we have doubts about a particular implementation of TVM and want to check it. For instance, if such an emulator interpreted a DICTISET instruction simply by invoking this instruction itself, then a bug in the underlying implementation of TVM would remain unnoticed.

### B.2.4. Reference implementation inside a minimal version of TVM. 

We see that using TVM itself as a host machine for a reference implementation of TVM emulator would yield little insight. A better idea is to define a stripped-down version of $$T V M$$, which supports only the bare minimum of primitives and 64-bit integer arithmetic, and provide a reference implementation $$P$$ of the TVM step function $$f$$ for this stripped-down version of TVM.

In that case, one must carefully implement and check only a handful of primitives to obtain a stripped-down version of TVM, and compare the reference implementation $$P$$ running on this stripped-down version to the full custom TVM implementation being verified. In particular, if there are any doubts about the validity of a specific run of a custom TVM implementation, they can now be easily resolved with the aid of the reference implementation.

### B.2.5. Relevance for the TON Blockchain. 

The TON Blockchain adopts this approach to validate the runs of TVM (e.g., those used for processing inbound messages by smart contracts) when the validators' results do not match one another. In this case, a reference implementation of TVM, stored inside the masterchain as a configurable parameter (thus defining the current revision of TVM), is used to obtain the correct result.

### B.2.6. Codepage -1 . 

Codepage -1 of TVM is reserved for the strippeddown version of TVM. Its main purpose is to execute the reference implementation of the step function of the full TVM. This codepage contains only special versions of arithmetic primitives working with "tiny integers" (64-bit signed integers); therefore, TVM's 257-bit Integer arithmetic must be defined in terms of 64-bit arithmetic. Elliptic curve cryptography primitives are also implemented directly in codepage -1 , without using any third-party libraries. Finally, a reference implementation of the SHA256 hash function is also provided in codepage -1 .

### B.2.7. Codepage -2 . 

This bootstrapping process could be iterated even further, by providing an emulator of the stripped-down version of TVM written for an even simpler version of TVM that supports only boolean values (or integers 0 and 1) -a "codepage - 2". All 64-bit arithmetic used in codepage -1 would then need to be defined by means of boolean operations, thus providing a reference implementation for the stripped-down version of TVM used in codepage -1 . In this way, if some of the TON Blockchain validators did not agree on the results of their 64-bit arithmetic, they could regress to this reference implementation to find the correct answer $${ }^{30}$$

$${ }^{30}$$ The preliminary version of TVM does not use codepage -2 for this purpose. This may change in the future.