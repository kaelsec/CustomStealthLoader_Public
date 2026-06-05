> Personal Research Project
>
> A study of Stage-0 loader architecture, execution context, and observable artifacts in modern EDR-monitored environments.
>
> This repository contains research documents only. Source code is maintained separately in a private repository.
# Custom Stage-0 Loader Research Project

> 현재 연구는 V6 아키텍처를 기준으로 Stage-0 Loader 계층까지 진행되었으며, 본 저장소는 해당 연구 결과와 설계 과정을 문서화합니다.

## Why This Project?

OSCP 준비 및 실전 침투 테스트를 수행하면서 공개 도구와 프레임워크를 사용하는 것만으로는 로더의 실제 동작 원리와 운영체제 내부 메커니즘을 충분히 이해하기 어렵다는 한계를 느꼈습니다.

이에 따라 Windows Internals, 실행 흐름(Execution Flow), 메모리 관리, 그리고 현대 EDR이 관찰하는 다양한 지점들을 직접 이해하기 위해 로더를 설계하고 구현하는 연구를 시작하게 되었습니다.

본 저장소는 그 과정에서 얻은 연구 결과와 설계 기록을 정리한 공간입니다.

---

## Repository Contents

본 저장소에는 다음과 같은 연구 문서가 포함되어 있습니다.

* Architecture Whitepaper
* Development Journal
* Troubleshooting & Lessons Learned
* Research Notes

실제 구현 코드는 별도의 Private Repository에서 관리하고 있으며, 본 저장소는 연구 결과, 설계 의사결정, 그리고 구현 과정에서 얻은 학습 내용을 문서화하기 위한 목적으로 운영됩니다.

---

## Research Areas

본 프로젝트를 통해 다음과 같은 주제를 중점적으로 연구하였습니다.

* Windows Internals
* Process Memory Management
* API Resolution
* Execution Context
* Process & Thread Telemetry
* Call Stack Analysis
* Loader Architecture Design

---

## Current Status

현재 연구는 V6 아키텍처를 기준으로 Stage-0 Loader 영역까지 진행되었습니다.

향후에는 다음 분야를 중심으로 학습과 연구를 이어갈 예정입니다.

* Advanced Windows Internals
* C2 Framework Operations
* Adversary Emulation
* Detection Engineering
* Red Team Tradecraft

---

## Disclaimer

본 프로젝트는 교육 및 연구 목적으로 수행되었습니다.

저장소에 포함된 문서들은 운영체제 내부 구조, 소프트웨어 아키텍처, 실행 흐름 분석에 대한 이해를 돕기 위해 작성되었으며, 비인가 접근이나 불법적인 활동을 지원하기 위한 목적이 아닙니다.

---
# Custom Stage-0 Loader Research Project

> The research currently concludes at the Stage-0 Loader layer (V6), and this repository documents the resulting architecture, implementation journey, and lessons learned.

## Why This Project?

While preparing for offensive security certifications and conducting penetration testing engagements, I realized that relying solely on public tools provided limited understanding of how modern loaders actually operate.

To better understand Windows internals, execution flow, memory management, and EDR observation surfaces, I began implementing and documenting my own loader architecture as a learning project.

This repository is the result of that research journey.

---

## Repository Contents

* Architecture Whitepaper
* Development Journal
* Troubleshooting & Lessons Learned
* Research Notes

The actual implementation is maintained in a separate private repository.

This public repository is intended to document the design decisions, research findings, and engineering lessons learned throughout the project.

---

## Research Areas

* Windows Internals
* Process Memory Management
* API Resolution
* Execution Context
* Process & Thread Telemetry
* Call Stack Analysis
* Loader Architecture Design

---

## Current Status

The research currently concludes at the Stage-0 Loader layer (V6).

Future areas of study include:

* Advanced Windows Internals
* C2 Framework Operations
* Adversary Emulation
* Detection Engineering
* Red Team Tradecraft

---

## Disclaimer

This project was conducted for educational and research purposes.

The materials in this repository are intended to support the study of operating system internals, software architecture, and execution flow analysis. They are not intended to facilitate unauthorized access, misuse, or unlawful activities.

