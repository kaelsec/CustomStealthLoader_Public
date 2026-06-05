# Architectural Design of a Custom Stage-0 Loader for Modern EDR Environments (V6)

## 1. Introduction: The Evolution of Endpoint Defense

Modern Endpoint Detection and Response (EDR) solutions have evolved beyond traditional file signature-based detection. Contemporary platforms analyze a wide range of runtime artifacts, including process memory characteristics, execution flow, call stack information, module loading behaviors, and API invocation patterns.

In this environment, the role of a loader extends beyond simply executing code. It becomes an engineering problem focused on managing the observable artifacts generated during the initial execution phase.

V6 Loader was developed as a research project from this perspective. Its primary objective is to reduce observable indicators exposed during early-stage execution while maintaining consistent execution flow throughout the staging process.

This document describes the major architectural components of the loader according to its execution lifecycle and discusses the engineering considerations behind each design decision.

---

## 2. Phase 1: Static Analysis Considerations and API Resolution

### Design Objective

Minimize functional and behavioral indicators exposed during static analysis.

### Design Approach

V6 Loader explores alternative approaches to runtime API resolution in order to reduce reliance on conventional operating system mechanisms.

Additionally, efforts were made to minimize string exposure and reduce the amount of information that can be collected through static inspection.

### Comparison with Conventional Approaches

Commonly used API resolution mechanisms include:

* `LoadLibrary`
* `LoadLibraryEx`
* `GetProcAddress`

### Design Rationale

Standard API-based resolution mechanisms are simple to implement and easy to maintain.

However, import metadata and embedded strings may provide useful clues during static analysis and can assist analysts in inferring program functionality.

For this reason, the project focused on reducing information exposure prior to execution whenever practical.

---

## 3. Phase 2: Runtime Monitoring Considerations

### Design Objective

Reduce execution artifacts observable within user-mode monitoring environments.

### Design Approach

The project investigates approaches that reduce dependency on specific runtime interfaces while considering the broader execution context in which system operations occur.

### Comparison with Conventional Approaches

Common approaches include:

* Standard operating system API invocation paths
* Direct system service invocation mechanisms

### Design Rationale

Modern EDR platforms analyze not only which APIs are invoked, but also the execution context, memory origin, and call path associated with those invocations.

As a result, understanding how an operation is performed can be just as important as the operation itself.

---

## 4. Phase 3: Initial Execution and Process Telemetry

### Design Objective

Reduce unnecessary process and thread-related events during the initial execution phase.

### Design Approach

The architecture minimizes activities performed during initialization and attempts to leverage existing execution paths already present within the host process.

### Comparison with Conventional Approaches

Typical execution mechanisms include:

* `CreateThread`
* `CreateRemoteThread`
* `NtCreateThreadEx`
* APC-based execution techniques

### Design Rationale

Thread creation and cross-process execution mechanisms are highly observable events from the perspective of operating system telemetry and security monitoring platforms.

Accordingly, this project explored approaches that reduce reliance on creating new execution flows and instead utilize existing execution contexts whenever possible.

---

## 5. Phase 4: Memory Residency and Memory Analysis Considerations

### Design Objective

Reduce the analytical surface associated with executable memory regions.

### Design Approach

The architecture considers both the characteristics of executable memory regions and their relationship to existing module structures within the target process.

### Comparison with Conventional Approaches

Common techniques include:

* `VirtualAlloc`
* `VirtualAllocEx`
* Reflective Loading

### Design Rationale

The creation of executable memory, memory protection transitions, and relationships between executable code and loaded modules are all areas commonly examined during memory analysis.

Consequently, the project considered not only how code is executed, but also how executable memory appears from an observational perspective.

---

## 6. Phase 5: Call Stack and Execution Context Management

### Design Objective

Reduce abnormal execution artifacts observable through call stack analysis.

### Design Approach

The architecture considers how execution flow and call context may appear during the analysis of sensitive system operations.

### Comparison with Conventional Approaches

Typical implementations rely on:

* Direct function calls
* Conventional API call chains

### Design Rationale

Modern EDR solutions frequently leverage call stack analysis to determine execution origin and calling relationships.

For this reason, execution consistency and contextual integrity were treated as important design considerations alongside functional implementation.

---

## Architecture Summary

### Phase 1

* PEB/EAT Resolver
* Jenkins Hash Resolver

### Phase 2

* Halo's Gate

### Phase 3

* Deferred Threadless Execution

### Phase 4

* Module Stomping

### Phase 5

* SpookStack

---

## 7. Conclusion: Architectural Scope and Design Philosophy

V6 Loader is a research project focused on the Initial Execution Phase of a loader architecture.

The objective of this project is not to target any specific security product, but rather to examine the various observation points commonly utilized within modern EDR environments and explore architectural decisions influenced by those observations.

To achieve this goal, the project incorporates the following design components:

* PEB/EAT-based API Resolution
* Hash-based Import Resolution
* Dynamic Indirect Syscalls (Halo's Gate)
* Deferred Threadless Execution
* Module-Backed Memory Staging (Module Stomping)
* Call Stack Context Management (Stack Spoofing)

Each component was selected to address different categories of observable artifacts, including static analysis indicators, runtime monitoring data, process telemetry, memory analysis characteristics, and call stack visibility.

Ultimately, the project approaches loader development as an engineering problem and serves as a case study exploring how execution flow, memory residency, and API interaction patterns may be designed within increasingly telemetry-rich defensive environments.
