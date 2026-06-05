> Personal Research Project
>
> A study of Stage-0 loader architecture, execution context, and observable artifacts in modern EDR-monitored environments.
>
> This repository contains research documents only. Source code is maintained separately in a private repository.

# Custom Stage-0 Loader Research Project

## 소개

본 저장소는 현대 EDR 환경에서 초기 실행 단계(Stage-0)가 어떠한 방식으로 관찰되고 분석되는지를 이해하기 위해 진행한 개인 연구 프로젝트를 정리한 공간입니다.

OSCP 준비 및 실전 침투 테스트 과정에서 공개적으로 알려진 도구와 프레임워크를 사용하는 것만으로는 내부 동작 원리를 충분히 이해하기 어렵다는 한계를 느꼈고, 이를 계기로 로더(Loader)의 실행 구조와 운영체제 내부 동작을 직접 연구하게 되었습니다.

본 프로젝트는 특정 보안 제품의 우회를 목표로 하기보다는 다음과 같은 주제를 학습하기 위한 목적으로 진행되었습니다.

* Windows Internals
* Process Memory Management
* API Resolution
* Execution Context
* Thread & Process Telemetry
* Call Stack Analysis
* Modern EDR Observation Surface

프로젝트는 V3부터 V6까지 단계적으로 발전하였으며, 각 버전에서 새로운 설계 요소를 추가하고 구현 과정에서 발생한 문제를 분석하며 개선해 왔습니다.

---

## Repository Scope

이 저장소에는 다음과 같은 연구 문서가 포함됩니다.

* Architecture Whitepaper
* Development Journal
* Troubleshooting Notes
* Design Decisions
* Research References

실제 구현 코드는 별도의 Private Repository에서 관리하고 있으며, 본 저장소는 연구 결과와 설계 과정을 문서화하기 위한 목적으로 운영됩니다.

---

## Research Focus

프로젝트를 통해 다음과 같은 주제를 중점적으로 연구하였습니다.

* PEB / EAT 기반 API Resolution
* Hash-based Import Resolution
* Runtime Context Awareness
* Deferred Execution Models
* Memory Residency Considerations
* Call Stack Context Analysis
* Loader Architecture Design

---

## Future Research

현재 연구는 Stage-0 Loader 영역까지 진행되었으며, 앞으로는 다음 분야를 중심으로 학습을 이어갈 예정입니다.

* Advanced Windows Internals
* C2 Framework Operations
* Adversary Emulation
* Detection Engineering
* Red Team Tradecraft
* Threat-Informed Offensive Security

---

## Disclaimer

본 프로젝트는 교육 및 연구 목적으로 수행되었습니다.

저장소에 포함된 문서들은 운영체제 내부 구조와 실행 메커니즘에 대한 이해를 목적으로 작성되었으며, 실제 환경에서의 무단 접근이나 비인가 활동을 지원하기 위한 목적이 아닙니다.

---

# Custom Stage-0 Loader Research Project

## Overview

This repository documents a personal research project focused on understanding how modern Endpoint Detection and Response (EDR) solutions observe and analyze the initial execution phase of software.

While preparing for offensive security certifications and conducting penetration testing engagements, I found that relying solely on publicly available tools often provided limited visibility into the underlying execution mechanisms. This project was started as an effort to better understand Windows internals and loader architecture through hands-on implementation and experimentation.

The primary goal of this research is not to bypass any specific security product, but to study and understand:

* Windows Internals
* Process Memory Management
* API Resolution Mechanisms
* Execution Context
* Thread and Process Telemetry
* Call Stack Analysis
* Modern EDR Observation Surfaces

The project evolved incrementally from V3 to V6, with each version introducing new architectural concepts and lessons learned throughout the development process.

---

## Repository Scope

This repository contains:

* Architecture Whitepapers
* Development Journals
* Troubleshooting Notes
* Design Discussions
* Research References

The actual implementation is maintained in a separate private repository. This public repository is intended to document the research findings, architectural decisions, and lessons learned throughout the project.

---

## Research Focus

Topics explored throughout this project include:

* PEB / EAT-based API Resolution
* Hash-based Import Resolution
* Runtime Context Awareness
* Deferred Execution Models
* Memory Residency Considerations
* Call Stack Context Analysis
* Loader Architecture Design

---

## Future Research

The current research scope concludes at the Stage-0 Loader layer.

Future areas of study include:

* Advanced Windows Internals
* C2 Framework Operations
* Adversary Emulation
* Detection Engineering
* Red Team Tradecraft
* Threat-Informed Offensive Security

---

## Disclaimer

This project was conducted for educational and research purposes.

The materials provided in this repository are intended to support the study of operating system internals, execution flow analysis, and software architecture concepts. They are not intended to facilitate unauthorized access, misuse, or unlawful activities.
