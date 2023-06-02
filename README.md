# Introduction

## Abstract

The aim of this text is to provide a description of the Telegram Open Network Virtual Machine (TON VM or TVM), used to execute smart contracts in the TON Blockchain.

## Introduction

The primary purpose of the Telegram Open Network Virtual Machine (TON VM or TVM) is to execute smart-contract code in the TON Blockchain. TVM must support all operations required to parse incoming messages and persistent data, and to create new messages and modify persistent data.

Additionally, TVM must meet the following requirements:

* It must provide for possible future extensions and improvements while retaining backward compatibility and interoperability, because the code of a smart contract, once committed into the blockchain, must continue working in a predictable manner regardless of any future modifications to the VM.
* It must strive to attain high "(virtual) machine code" density, so that the code of a typical smart contract occupies as little persistent blockchain storage as possible.
* It must be completely deterministic. In other words, each run of the same code with the same input data must produce the same result, regardless of specific software and hardware used$$|^{1}$$

The design of TVM is guided by these requirements. While this document describes a preliminary and experimental version of TVM the backward compatibility mechanisms built into the system allow us to be relatively unconcerned with the efficiency of the operation encoding used for TVM code in this preliminary version.

TVM is not intended to be implemented in hardware (e.g., in a specialized microprocessor chip); rather, it should be implemented in software running on conventional hardware. This consideration lets us incorporate some highlevel concepts and operations in TVM that would require convoluted microcode in a hardware implementation but pose no significant problems for a software implementation. Such operations are useful for achieving high code density and minimizing the byte (or storage cell) profile of smart-contract code when deployed in the TON Blockchain.

({ }^{1}) For example, there are no floating-point arithmetic operations (which could be efficiently implemented using hardware-supported double type on most modern CPUs) present in TVM, because the result of performing such operations is dependent on the specific underlying hardware implementation and rounding mode settings. Instead, TVM supports special integer arithmetic operations, which can be used to simulate fixed-point arithmetic if needed.

$${ }^{2}$$ The production version will likely require some tweaks and modifications prior to launch, which will become apparent only after using the experimental version in the test environment for some time.
