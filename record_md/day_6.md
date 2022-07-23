# Chapter 5 - CPU Exceptions(예외)

Rust OS DEV[https://os.phil-opp.com/cpu-exceptions/]

## 목표

    CPU 예외는 잘못된 메모리 주소에 액세스하거나 0으로 나눌 때와 같이 다양한 잘못된 상황에서 발생합니다.
    이에 대응하기 위해 핸들러 기능을 제공 하는 Interrupt Descriptor Table을 설정해야 합니다.
    오늘의 목표는 커널은 중단점 예외 를 포착하고 이후에 정상적인 실행을 재개하는 것입니다.

## 용어 풀이 관련 링크

### Interrupt Descriptor Table[https://ko.wikipedia.org/wiki/인터럽트_서술자_테이블]

### CPU Exceptions

Rust CPU 예외[https://yongj.in/rust%20os/rust-os-cpu-exceptions/]
CPU 예외 1[https://luv-n-interest.tistory.com/997]
CPU 예외 2[https://jihyewoo.tistory.com/18?category=936350]

## CPU Exceptions의 종류

OS Dev[https://wiki.osdev.org/Exceptions]

<p align="center"><img src="/readme_src/CPU_Exceptions.png"></p>

#### Divide-by-zero Error

Divide-by-zero Error는 DIV 또는 IDIV 명령을 사용하여 숫자를 0으로 나누거나 분할 결과가 너무 커서 대상에 표시할 수 없을 때 발생합니다.
결함이 있는 DIV 또는 IDIV 명령은 코드의 어디에든 추가하기 쉽기 때문에 많은 OS 개발자들은 예외 처리 코드가 작동하는지 테스트하기 위해 이 예외를 사용합니다.

저장된 명령 포인터는 예외를 발생시킨 DIV 또는 IDIV 명령을 가리킵니다.

#### Debug

Debug Exceptions는 다음과 같은 경우에 발생합니다.
<ul>
    <li>명령 가져오기 중단점(Fault)</li>
    <li>일반 감지 조건(Fault)</li>
    <li>데이터 읽기 또는 쓰기 중단점(Trap)</li>
    <li>I/O 읽기 또는 쓰기 중단점(Trap)</li>
    <li>단일 단계(Trap)</li>
    <li>작업 스위치(Trap)</li>
<ul>

#### Non-maskable Interrupt

OS DEV Non-maskable Interrupt[https://wiki.osdev.org/Non_Maskable_Interrupt]

#### Breakpoint

    중단점 예외는 INT3 명령을 실행할 때 발생합니다.
    일부 디버그 소프트웨어는 INT3 명령으로 명령을 대체하고 중단점이 Trap되면 INT3 명령을 원래 명령으로 대체하고 명령 포인터를 1씩 줄입니다.

    저장된 명령 포인터는 INT3 명령 뒤에 있는 바이트를 가리킵니다.

#### Overflow

    RFLAGS의 오버플로 비트가 1로 설정된 상태에서 INTO 명령이 실행될 때 오버플로 예외가 발생합니다.

    저장된 명령 포인터는 INTO 명령 뒤에 있는 명령을 가리킵니다.

#### Bound Range Exceeded

    이 예외는 BOUND 명령이 실행될 때 발생할 수 있습니다.
    BOUND 명령은 배열 인덱스를 배열의 하한 및 상한과 비교합니다.
    만약 인덱스가 범위를 벗어나면 Bound Range Exceededed 예외가 발생합니다.

    저장된 명령 포인터는 예외를 발생시킨 BOUND 명령을 가리킵니다.

#### Invalid Opcode

    이 예외는 현재 명령어가 유효하지 않을 때 발생합니다.
    (예: 지원하지 않는 이전 CPU에서 최신 SSE 명령어 를 사용하려고 할 때)

#### Device Not Available

    Device Not Available 예외는 FPU 명령이 시도되었지만 FPU가 없을 때 발생합니다.
    현대의 프로세서에는 내장형 FPU가 있기 때문에 그럴 가능성은 낮습니다.
    그러나 CR0 레지스터에는 FPU/MMX/SSE 명령을 비활성화하는 플래그가 있으므로 이 플래그를 시도할 때 예외가 발생합니다.
    이 기능은 운영 체제가 사용자 프로그램이 FPU 또는 XMM 레지스터를 사용하는 시기를 감지한 다음 멀티태스킹 시 적절히 저장/복원할 수 있기 때문에 유용합니다.

    저장된 명령 포인터는 예외를 발생시킨 명령을 가리킵니다.

#### Double Fault

    예외가 발생하면 CPU는 해당 핸들러 함수를 호출하려고 합니다.
    예외 핸들러를 호출하는 동안 다른 예외가 발생 하면 CPU는 이중 오류 예외를 발생시킵니다.
    이 예외는 예외에 대해 등록된 핸들러 함수가 없는 경우에도 발생합니다.

#### Coprocessor Segment Overrun

    FPU가 프로세서 외부에 있을 때 보호 모드에서 별도의 세그먼트 검사를 수행했습니다.
    486이 non-FPU 메모리 접근에서 이미 했던 것처럼 GPF에 의해 대신 처리되기 때문입니다.

#### Invalid TSS

    Invalid TSS 예외는 잘못된 세그먼트 선택기가 작업의 일부로 참조되거나 게이트 설명자를 통한 제어 전송의 결과로 참조될 때 발생하며,
    이로 인해 TSS에서 SS 선택기를 사용하여 잘못된 Stack-Segment 참조가 발생합니다.

    TSS에서 세그먼트 선택기를 로드하기 전에 예외가 발생하면 저장된 명령 포인터가 예외를 발생시킨 명령을 가리킵니다.
    그렇지 않으면 새 작업의 첫 번째 명령을 가리키는 것이 일반적입니다.

    오류 코드: 잘못된 TSS 예외는 선택기 색인 오류 코드를 설정합니다.

#### Segment Not Present

    Segment Not Present 예외는 현재 비트가 0으로 설정된 세그먼트 또는 게이트를 로드하려고 할 때 발생합니다.
    그러나 존재하지 않는 설명자를 참조하는 Stack-Segment 선택기를 로드할 때 Stack-Segment 결함이 발생합니다.

    하드웨어 작업 전환 중에 예외가 발생하는 경우 처리기가 세그먼트 값을 의존해서는 안 됩니다.
    즉, 처리기는 새 작업을 다시 시작하기 전에 이러한 작업을 확인해야 합니다.
    Intel 문서에 따르면 이 작업을 수행하는 세 가지 방법이 있습니다.

    저장된 명령 포인터는 예외를 발생시킨 명령을 가리킵니다.

    오류 코드: Segment Not Present 예외는 예외를 발생시킨 세그먼트 설명자의 세그먼트 선택기 색인 오류 코드를 설정합니다.

#### Stack-Segment Fault

    Stack-Segment 결함은 다음 경우에 발생합니다.

<ul>
    <li>존재하지 않는 세그먼트 설명자를 참조하는 Stack Segment Load일 때</li>
    <li>기본 레지스터로 ESP 또는 EBP를 사용하는 모든 PUSH 또는 POP 명령어의 스택 주소가 표준 형식이 아닌 동안 실행될 때</li>
    <li>스택 제한 검사가 실패했을 때</li>
<ul>

    하드웨어 작업 전환 중에 예외가 발생하는 경우 처리기가 세그먼트 값을 의존해서는 안 됩니다.
    즉, 처리기는 새 작업을 다시 시작하기 전에 이러한 작업을 확인해야 합니다.
    Intel 문서에 따르면 이 작업을 수행하는 세 가지 방법이 있습니다.
    저장된 명령 포인터는 하드웨어 작업 전환 중에 존재하지 않는 스택 세그먼트를 로드하여 오류가 발생하지 않는 한 예외를 발생시킨 명령을 가리키며, 이 경우 새 작업의 다음 명령을 가리킵니다.

    오류 코드: Stack-Segment Fault(스택-세그먼트 결함)는 하드웨어 작업 전환 중에 존재하지 않는 세그먼트 설명자가 참조되거나 제한 검사가 실패한 경우 스택 세그먼트 선택기 인덱스인 오류 코드를 설정합니다.
    그렇지 않은 경우(현재 세그먼트 및 이미 사용 중인 세그먼트의 경우), 오류 코드는 0입니다.

#### General Protection Fault

    가장 광범위한 원인을 가진 예외입니다.
    사용자 수준 코드에서 권한 있는 명령을 실행하거나 구성 레지스터에 예약된 필드를 쓰는 것과 같은 다양한 종류의 액세스 위반에서 발생합니다.

#### Page Fault

    잘못된 메모리 접근 시 Page Fault가 발생합니다.
    예를 들어, 현재 명령어가 매핑되지 않은 페이지에서 읽으려고 하거나 읽기 전용 페이지에 쓰려고 시도하는 경우입니다.

#### Reserved : 15 (0xF)

#### x87 Floating-Point Exception

    x87 부동소수점 예외는 FWAIT 또는 WAIT 명령 또는 대기 부동소수점 명령이 실행될 때 발생하며 다음 조건이 충족됩니다.

<ul>
    <li>CR0.NE는 1입니다.</li>
    <li>마스킹되지 않은 x87 부동소수점 예외가 보류 중입니다.(즉, x87 부동소수점 상태 워드 레지스터의 예외 비트가 1로 설정됨)</li>
<ul>

    저장된 명령 포인터는 예외가 발생했을 때 실행되려고 하는 명령을 가리킵니다.
    x87 명령 포인터 레지스터는 예외를 발생시킨 마지막 명령의 주소를 포함합니다.

    오류 코드: 예외는 오류 코드를 푸시하지 않습니다.
    그러나 예외 정보는 x87 상태 워드 레지스터에서 사용할 수 있습니다.

#### Alignment Check

    정렬 검사가 활성화되어 있고 정렬되지 않은 메모리 데이터 참조가 수행되는 경우 정렬 검사 예외가 발생합니다.
    정렬 점검은 CPL 3에서만 수행됩니다.

    선형 검사는 기본적으로 비활성화되어 있습니다.
    CR0을 활성화하려면 CR0.AM 및 RFlags.AC를 둘 다 1로 설정해야합니다.

    저장된 명령 포인터는 예외를 발생시킨 명령을 가리킵니다.

#### Machine Check

    Machine Check 예외는 모델에 따라 다르며 이를 지원하기 위해 프로세서 구현이 필요하지 않습니다.
    모델별 레지스터를 사용하여 오류 정보를 제공하고 기본적으로 비활성화되어 있습니다.
    CR4를 활성화하려면 CR4.MCE 비트를 1로 설정해야합니다.

    Machine Check 예외는 프로세서가 불량 메모리, 버스 오류, 캐시 오류 등과 같은 내부 오류를 감지할 때 발생합니다.

저장된 명령 포인터의 값은 구현 및 예외에 따라 달라집니다.

#### SIMD Floating-Point Exception

    SIMD Floating-Point Exception 예외는 마스킹되지 않은 128비트 미디어 부동 소수점 예외가 발생하고 CR4.OSXMMEXCPT 비트가 1로 설정된 경우에 발생합니다.
    OSXMMEXCEPT 플래그가 설정되지 않은 경우 SIMD 부동소수점 예외는 이 대신 정의되지 않은 Opcode 예외를 발생시킵니다.

    저장된 명령 포인터는 예외를 발생시킨 명령을 가리킵니다.

    오류 코드: 예외는 오류 코드를 푸시하지 않습니다.
    그러나 예외 정보는 MXCSR 레지스터에서 사용할 수 있습니다.

#### Virtualization Exception

#### Control Protection Exception

#### Reserved : 22-27 (0x16-0x1B)

#### Hypervisor Injection Exception

#### VMM Communication Exception

#### Security Exception

#### Reserved : 31 (0x1F)

#### Triple Fault

    CPU가 Double Fault 핸들러 기능을 호출하는 동안 예외가 발생하면 치명적인 Triple Fault가 발생합니다.
    대부분은 삼중 오류를 잡거나 처리할 수 없기 때문에 프로세서는 스스로 재설정하여 운영 체제를 재부팅합니다.

#### FPU Error Interrupt

    과거에는 부동소수점 유닛이 프로세서에 부착할 수 있는 전용 칩이었으나, 
    프로세서에 대한 FPU 오류의 직접적인 배선이 부족했기 때문에 대신 IRQ 13을 사용하여 CPU가 오류를 스스로 처리할 수 있게 되었습니다.
    486이 개발되고 멀티프로세서 지원이 추가되었을 때, FPU는 다이(die)에 내장되었고 FPU를 위한 글로벌 인터럽트는 바람직하지 않게 되었습니다.
    기본적으로 이 메서드는 이전 버전과의 호환성을 위해 부팅 시 사용할 수 없지만 OS는 그에 따라 설정을 업데이트해야 합니다.

### 오류에 따른 심각도

#### Faults

    해당 오류는 수정될 수 있으며 프로그램은 정상작동합니다.

#### Traps

    Traps는 트래핑 명령 실행 직후에 보고됩니다.

#### Aborts

    복구할 수 없는 심각한 오류입니다.

## 예외 발생 시 CPU가 대처하는 순서

    1.  명령어 포인터와 RFLAGS 레지스터 를 포함하여 스택의 일부 레지스터를 푸시합니다.
    2.  IDT(Interrupt Descriptor Table)에서 해당 항목을 읽습니다.
        예를 들어 CPU는 페이지 폴트가 발생하면 14번째 항목을 읽습니다.
    3.  항목이 있는지 확인하고, 그렇지 않은 경우 이중 오류를 발생시킵니다.
    4.  항목이 인터럽트 게이트인 경우 하드웨어 인터럽트를 비활성화합니다(비트 40이 설정되지 않음).
    5.  지정된 GDT 선택기를 CS 세그먼트에 로드합니다.
    6.  지정된 핸들러 함수로 이동합니다.

## 인터럽트 설명자 테이블

    예외를 포착하고 처리하려면 소위 IDT( Interrupt Descriptor Table )를 설정해야 합니다. 
    이 테이블에서 우리는 각 CPU 예외에 대한 핸들러 함수를 지정할 수 있습니다. 
    하드웨어는 이 테이블을 직접 사용하므로 사전 정의된 형식을 따라야 합니다. 
    각 항목은 다음과 같은 16바이트 구조를 가져야 합니다.