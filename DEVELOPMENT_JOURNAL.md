# V6 Loader Development Journal & Engineering Notes

본 문서는 V3 프로토타입부터 V6 최종 아키텍처까지의 개발 과정과 주요 엔지니어링 이슈를 정리한 기술 기록이다.

문서는 기능 개발 이력(Development Timeline)과 구현 과정에서 발생한 문제 해결 기록(Troubleshooting & Lessons Learned)으로 구분된다.

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

본 장에서는 V3부터 V6까지의 개발 과정에서 발생한 주요 기술적 문제와 해결 과정을 정리한다.

단순히 버그 수정 내역을 나열하는 것이 아니라, 문제 발생 원인과 설계 변경의 배경을 함께 기록하여 향후 유사한 구현 시 참고할 수 있는 엔지니어링 지식 베이스(Knowledge Base)로 활용하는 것을 목적으로 한다.

---

# Runtime Stability

## Non-Volatile Register Corruption During Stack Spoofing

### Problem

V6 단계에서 Advanced SpookStack을 적용하는 과정에서 원인 불명의 런타임 크래시가 반복적으로 발생했다.

특정 API 호출 직후 즉시 크래시가 발생하는 것이 아니라, 호출 이후 수십~수백 줄의 코드가 추가로 실행된 뒤 예외가 발생하였다. 이로 인해 초기에는 Stack Alignment 문제 또는 Return Address 손상 문제로 의심되었으나, 분석 결과 실제 원인은 다른 영역에 존재하였다.

### Root Cause Analysis

ASM Stub(`SpookedSyscall5`, `SpookCall`) 내부에서는 실제 Return Address를 임시 보관하기 위해 R12 레지스터를 사용하고 있었다.

그러나 x64 Windows ABI 규약상 R12는 Non-Volatile Register로 분류된다.

즉, 호출된 함수는 해당 레지스터 값을 보존해야 하며, 호출자는 함수 호출 이후에도 동일한 값이 유지될 것을 전제로 동작한다.

Stack Spoofing 로직은 이 규칙을 위반한 채 R12를 임시 저장 공간으로 활용하였고, 결과적으로 호출자의 컨텍스트가 손상되면서 예측 불가능한 시점에 크래시가 발생하였다.

### Resolution

문제 해결을 위해 ASM Stub 전체를 재검토하였다.

우선 R12 사용을 중단하고 Volatile Register인 R10으로 전면 교체하였다.

이후 Return Address 저장 및 복원 로직을 재설계하여 Caller Context를 침범하지 않도록 수정하였다.

또한 ROP Gadget Offset에 따라 Stack Frame이 달라질 수 있으므로, Gadget 탐색 결과를 기반으로 Stack Frame을 동적으로 재구성하는 로직을 추가하였다.

### Result

수정 이후 장시간 반복 테스트에서 동일한 크래시는 재현되지 않았다.

Windows 10 및 Windows 11 환경 모두에서 동일한 실행 결과를 확인하였으며, Stack Spoofing 적용 상태에서도 안정적으로 원래 제어 흐름으로 복귀함을 검증하였다.

### Key Takeaway

Stack Spoofing 자체보다 ABI 규약 준수가 운영 안정성에 훨씬 큰 영향을 미친다는 사실을 확인하였다.

고급 회피 기법도 결국 운영체제 호출 규약 위에서 동작하며, 이러한 기초 원칙이 무너지면 어떠한 우회 기법도 안정성을 보장할 수 없다.

---

## BCryptDecrypt IV Corruption

### Problem

AES 복호화 루틴은 정상적으로 동작하는 것처럼 보였으나, 특정 빌드에서는 쉘코드가 정상적으로 복호화되지 않는 현상이 발생하였다.

동일한 암호문과 키를 사용함에도 불구하고 실행 결과가 매번 달라지는 비결정적 현상이 관찰되었다.

### Root Cause Analysis

원인을 추적한 결과 Windows CNG API의 BCryptDecrypt 함수가 전달받은 IV 버퍼를 내부적으로 수정한다는 사실을 확인하였다.

초기 구현에서는 크기 계산용 호출과 실제 복호화 호출에 동일한 IV 버퍼를 재사용하였다.

결과적으로 첫 번째 호출 이후 IV 값이 변경되었고, 두 번째 호출 시 변형된 IV가 사용되면서 복호화 결과가 손상되었다.

### Resolution

원본 IV를 별도로 유지하고, BCryptDecrypt 호출 직전에 항상 로컬 복사본을 생성하여 전달하도록 수정하였다.

모든 복호화 호출은 독립적인 IV 복사본을 사용하도록 변경하였다.

### Result

복호화 결과가 완전히 결정론적으로 변하였으며, 모든 테스트 환경에서 동일한 결과를 얻을 수 있었다.

### Key Takeaway

공식 API라 하더라도 입력 파라미터가 불변(Immutable)일 것이라고 가정해서는 안 된다.

특히 암호화 API는 내부 상태를 변경하는 경우가 존재하므로 문서화되지 않은 부작용까지 고려한 구현이 필요하다.

---

## NtProtectVirtualMemory Parameter Reuse Bug

### Problem

Module Stomping 과정에서 메모리 권한 변경이 정상적으로 수행되지 않거나, 예상하지 못한 메모리 영역의 권한이 변경되는 현상이 발생하였다.

심한 경우 접근 위반 예외로 이어졌다.

### Root Cause Analysis

NtProtectVirtualMemory는 일반적인 Win32 API와 달리 BaseAddress와 RegionSize를 Out Parameter 형태로 수정한다.

초기 구현에서는 첫 번째 호출 이후 변경된 값을 그대로 유지한 상태에서 두 번째 호출에 재사용하였다.

결과적으로 실제 의도한 영역이 아닌 다른 주소 범위가 권한 변경 대상으로 전달되었다.

### Resolution

각 호출마다 BaseAddress와 RegionSize의 로컬 복사본을 새로 생성하여 전달하도록 수정하였다.

원본 값은 유지하고 Native API 호출에만 임시 변수를 사용하도록 구조를 변경하였다.

### Result

권한 변경 로직이 안정적으로 동작하기 시작했으며 Module Stomping 과정에서 발생하던 예외가 완전히 제거되었다.

### Key Takeaway

Native API는 Win32 API와 동일하게 동작할 것이라는 가정을 해서는 안 된다.

문서화 수준이 낮은 인터페이스일수록 입력값 변경 여부와 부작용을 직접 검증하는 과정이 필수적이다.

---

## 64-bit Absolute JMP Requirement

### Problem

Threadless Injection 구현 이후 일부 환경에서 후킹이 정상적으로 동작하지 않았다.

특히 ASLR이 활성화된 환경에서 점프 대상 주소에 도달하지 못하는 현상이 발생하였다.

### Root Cause Analysis

초기 구현은 Relative JMP를 사용하고 있었다.

그러나 x64 환경에서는 ±2GB 범위 제약이 존재하며, ASLR에 의해 모듈 배치가 변경될 경우 해당 범위를 벗어날 수 있다.

### Resolution

상대 JMP를 제거하고 FF 25 기반의 Absolute JMP 구조를 적용하였다.

이를 통해 실제 메모리 배치와 무관하게 임의의 64비트 주소로 점프할 수 있도록 수정하였다.

### Result

ASLR 여부와 관계없이 안정적으로 후킹이 동작하게 되었으며, Threadless Injection의 신뢰성이 크게 향상되었다.

### Key Takeaway

x64 환경에서는 ASLR을 고려한 주소 설계가 필수적이다.

32비트 환경에서 자연스럽게 동작하던 기법도 64비트 주소 공간에서는 구조적 한계를 가질 수 있으며, 설계 단계부터 이를 고려해야 한다.
