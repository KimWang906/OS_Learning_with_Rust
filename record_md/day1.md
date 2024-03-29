# Chapter 1:

## Source code

```rust
/*
    운영체제 커널을 만드는 첫 단계는 표준 라이브러리(standard library)를 링크하지 않는 Rust 실행파일을 만드는 것입니다. 
    이러한 실행파일은 운영체제가 없는 bare metal 시스템에서 동작할 수 있습니다.

    운영체제에 의존하지 않으려면 Rust 표준 라이브러리의 많은 부분을 사용할 수 없습니다. 
    그래도 우리가 이용할 수 있는 Rust 언어 자체의 기능들은 많이 남아 있습니다. 
    예를 들어 반복자, 클로저, 패턴 매칭, option / result, 문자열 포맷 설정, 그리고 소유권 시스템 등이 있습니다. 
    이러한 기능들은 우리가 커널을 작성할 때 undefined behavior나 
    메모리 안전성에 대한 걱정 없이 큰 흐름 단위의 코드를 작성하는 데에 집중할 수 있도록 해줍니다.
*/
#![no_std]
#![no_main]

use core::panic::PanicInfo;

/// 패닉이 일어날 경우, 이 함수가 호출됩니다.
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}

#[no_mangle]
pub extern "C" fn _start() -> ! {
    loop {}
}
/*
    #[no_mangle] 속성을 통해 name mangling을 해제하여 
    Rust 컴파일러가 _start 라는 이름 그대로 함수를 만들도록 합니다. 
    이 속성이 없다면, 컴파일러가 각 함수의 이름을 고유하게 만드는 과정에서 
    이 함수의 실제 이름을 _ZN3blog_os4_start7hb173fedf945531caE 라는 이상한 이름으로 바꿔 생성합니다. 
    우리가 원하는 실제 시작 지점 함수의 이름을 정확히 알고 있어야
    링커 (linker)에도 그 이름을 정확히 전달할 수 있기에 (후속 단계에서 진행) #[no_mangle] 속성이 필요합니다.

    또한 우리는 이 함수에 extern "C"라는 표시를 추가하여 
    이 함수가 Rust 함수 호출 규약 대신에 C 함수 호출 규약을 사용하도록 합니다. 
    함수의 이름을 _start로 지정한 이유는 그저 런타임 시스템들의 실행 시작 함수 이름이 대부분 _start이기 때문입니다.

    ! 반환 타입은 이 함수가 발산 함수라는 것을 의미합니다. 
    시작 지점 함수는 오직 운영체제나 부트로더에 의해서만 직접 호출됩니다.
    따라서 시작 지점 함수는 반환하는 대신 운영체제의 exit 시스템콜을 이용해 종료됩니다. 
    우리의 “freestanding 실행 파일” 은 실행 종료 후 더 이상 실행할 작업이 없기에, 
    시작 지점 함수가 작업을 마친 후 기기를 종료하는 것이 합리적입니다.
    여기서는 일단 ! 타입의 조건을 만족시키기 위해 무한루프를 넣어 줍니다.
    */
```