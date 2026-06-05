# V6 Loader Development Journal & Engineering Notes

This document summarizes the development process of the V6 Loader architecture from the initial V3 prototype through the final V6 implementation.

The document is divided into two sections:

* **Development Timeline** – Major architectural milestones and feature evolution.
* **Troubleshooting & Lessons Learned** – Engineering challenges encountered during implementation and the corresponding solutions.

---

# Development Timeline

## V3

* Initial DLL Proxying
* AES Payload Loading

## V4

* Halo's Gate
* Module Stomping
* Threadless Injection

## V5

* Full IAT Hiding
* Jenkins Hash Resolver
* String-less Execution

## V6

* Deferred Execution
* Advanced SpookStack
* Dynamic Stack Re-alignment

---

# Troubleshooting & Lessons Learned

This section documents the major engineering issues encountered throughout the development of V3 through V6.

Rather than serving as a simple bug-fix log, the goal is to capture the root causes, design decisions, and implementation lessons learned during development. The resulting knowledge base can serve as a reference for future implementations facing similar challenges.

---

# Runtime Stability

## Non-Volatile Register Corruption During Stack Spoofing

### Problem

During the implementation of Advanced SpookStack in V6, intermittent runtime crashes began occurring without any obvious trigger.

The crashes did not occur immediately after a sensitive API invocation. Instead, execution often continued normally for dozens or even hundreds of instructions before eventually terminating with an exception.

Because of this delayed failure pattern, initial investigation focused on potential stack alignment issues or return address corruption. Further analysis revealed that the root cause originated elsewhere.

### Root Cause Analysis

The assembly stubs (`SpookedSyscall5`, `SpookCall`) temporarily stored return addresses inside the R12 register.

Under the Windows x64 ABI, however, R12 is classified as a non-volatile register.

This means that a callee is required to preserve its value across function calls, while callers assume that its contents remain unchanged after execution returns.

The stack spoofing implementation violated this convention by using R12 as temporary storage. As a result, caller context was silently corrupted, leading to unpredictable crashes later in the execution flow.

### Resolution

The assembly stubs were reviewed and refactored.

R12 usage was eliminated entirely and replaced with R10, which is classified as a volatile register under the x64 ABI.

The return address preservation logic was redesigned to ensure that caller-owned state remained untouched.

Additionally, because stack layout varies depending on the selected ROP gadget offset, a dynamic stack reconstruction mechanism was introduced to rebuild stack frames based on the discovered gadget characteristics.

### Result

After applying the modifications, the crash could no longer be reproduced during extended testing.

Consistent behavior was observed across both Windows 10 and Windows 11 environments, and execution reliably returned to the original control flow even when stack spoofing was enabled.

### Key Takeaway

Operational stability is often more dependent on ABI compliance than on the sophistication of the evasion technique itself.

Regardless of complexity, advanced execution-flow manipulation techniques ultimately operate within the constraints imposed by the operating system calling convention.

---

## BCryptDecrypt IV Corruption

### Problem

The AES decryption routine appeared to function correctly, yet certain builds produced inconsistent decryption results.

Identical ciphertext and key material occasionally yielded different outputs across executions, resulting in non-deterministic behavior.

### Root Cause Analysis

Investigation revealed that the Windows CNG API function `BCryptDecrypt()` modifies the supplied IV buffer internally.

The original implementation reused the same IV buffer for both the size-calculation call and the actual decryption call.

Consequently, the IV contents were altered during the first invocation, causing the second invocation to operate on modified data and produce corrupted plaintext.

### Resolution

The original IV was preserved separately, and a fresh local copy was created before each invocation of `BCryptDecrypt()`.

Every decryption operation was modified to operate on an independent IV instance.

### Result

The decryption process became fully deterministic, producing identical output across all test environments.

### Key Takeaway

Even documented APIs should not be assumed to treat input parameters as immutable.

Cryptographic interfaces may modify caller-provided buffers internally, making defensive parameter handling an important implementation practice.

---

## NtProtectVirtualMemory Parameter Reuse Bug

### Problem

During Module Stomping operations, memory protection changes occasionally targeted unexpected memory regions.

In severe cases, the process terminated with access violation exceptions.

### Root Cause Analysis

Unlike many Win32 APIs, `NtProtectVirtualMemory()` modifies both `BaseAddress` and `RegionSize` as output parameters.

The original implementation reused these modified values in subsequent invocations.

As a result, later calls operated on unintended address ranges rather than the original target region.

### Resolution

A fresh local copy of both parameters was created before each invocation.

The original values remained unchanged while temporary variables were supplied exclusively to the Native API call.

### Result

Memory protection transitions became reliable, and the exceptions previously observed during Module Stomping were completely eliminated.

### Key Takeaway

Native APIs should not be assumed to behave identically to their Win32 counterparts.

When working with lower-level interfaces, parameter mutation and side effects must be explicitly validated rather than assumed.

---

## 64-bit Absolute JMP Requirement

### Problem

Following the implementation of Threadless Injection, hook reliability issues were observed on certain systems.

The problem became particularly noticeable when ASLR was enabled.

### Root Cause Analysis

The original implementation relied on relative JMP instructions.

On x64 systems, relative jumps are limited to a ±2 GB address range.

Because ASLR can place modules anywhere within the virtual address space, the destination occasionally exceeded this limitation.

### Resolution

The relative JMP implementation was replaced with an FF 25–based absolute JMP mechanism.

This allowed control flow to be redirected to arbitrary 64-bit addresses regardless of module placement.

### Result

Hook execution became stable across all tested environments, independent of ASLR behavior.

The overall reliability of the Threadless Injection implementation improved significantly.

### Key Takeaway

Addressing strategies that function reliably in 32-bit environments may encounter structural limitations on modern x64 systems.

ASLR-aware design considerations must be incorporated from the earliest stages of implementation.
