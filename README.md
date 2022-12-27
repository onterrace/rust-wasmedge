# WasmEdge 

일단, 윈도우즈에서서는 설치가 되지 않습니다. Ubuntu에서 설치하고 테스트했습니다. 

Rust는 WebAssembly 생태계의 '일급 시민' 프로그래밍 언어 중 하나입니다. WebAssembly에 대한 모든 WasmEdge 확장은 개발자를 위한 Rust API와 함께 제공됩니다. 이 장에서는 Rust 애플리케이션을 WebAssembly로 컴파일하고 WasmEdge 런타임에서 실행하는 방법을 보여줍니다.


이 글의 내용은 [Write a WebAssembly Application](https://wasmedge.org/book/en/write_wasm/rust.html)을 참고하여 작성했습니다.


## 준비물 
시작하려면 Rust와 WasmEdge를 설치해야 합니다. Rust 툴체인의 wasm32-wasi target도 설치해야 합니다.

```shell
rustup target add wasm32-wasi
```

> wasm32-wasi 타겟은 새롭고 여전히(2019년 4월 현재) 실험적인 타겟입니다. 이 파일의 정의는 시간이 지남에 따라 조정될 수 있으므로 너무 많이 의존해서는 안 됩니다.
> wasi 대상은 WebAssembly 파일이 상호 운용할 수 있는 표준화된 syscall 세트를 정의하기 위한 제안입니다. 이 syscall 세트는 파일 시스템 액세스, 네트워크 액세스 등과 같은 기본 기능으로 WebAssembly 바이너리를 강화하기 위한 것입니다.
> 여기서 Rust 대상 정의는 몇 가지 면에서 흥미롭습니다. 우리는 이 대상과 함께 두 가지 사용 사례를 제공하고자 합니다.    
> * 첫째, 우리는 대상의 Rust 사용이 가능한 한 번거롭지 않기를 원합니다. 이상적으로는 로컬 wasm32-wasi 툴체인을 구성하고 설치할 필요가 없습니다.
> * 둘째, LLVM의 새로운 wasm 백엔드 및 LLD의 wasm 지원의 주요 사용 사례 중 하나는 모든 컴파일된 언어가 다른 언어와 상호 운용될 수 있다는 것입니다. 따라서 wasm32-wasi 대상은 실행 가능한 C 표준 라이브러리와 sysroot 공통 정의가 있는 첫 번째 대상이므로 wasm32-unknown-unknown으로 컴파일될 때 Rust 및 C/C++ 코드가 상호 운용되기를 원합니다.
> [Module rustc_target::spec::wasm32_wasi](https://doc.rust-lang.org/beta/nightly-rustc/rustc_target/spec/wasm32_wasi/index.html) 참조.


## 프로그램 작성

다음을 실행하여 프로젝트를 생성한다. 

```shell
cargo new hello
```
main.rs에 다음을 추가한다.

**src/main.rs**    
```rust
use std::env;

fn main() {
  println!("hello");
  for argument in env::args().skip(1) {
    println!("{}", argument);
  }
}
```

### WASM bytecode 빌드 

```shell
cargo build --target wasm32-wasi
```

### 프로그램 실행 
커맨드라인에서 애플리케이션을 실행한다.
```shell
cargo build --target wasm32-wasi
```



## 간단한 함수 
다음을 실행하여 새로운 패키지를 생성한다. 
```shell
cargo new func
```

lib.rs에 다음을 추가한다.

**src/lib.rs**    
```rust
#[no_mangle]
pub fn add(a: i32, b: i32) -> i32 {
  return a + b;
}
```

### WASM bytecode 빌드 

Cargo.toml에 다음을 추가한다.
```toml
[lib]
name = "add"
path = "src/lib.rs"
crate-type =["cdylib"]
```
빌드한다. 
```shell
cargo build --target wasm32-wasi
```

다음과 같이 생성된다. 
```shell
📂project_root
  📂func
    📂target
      📂wasm32-wasi
        📂debug
          📄add.wasm
```



### 프로그램 실행 
커맨드라인에서 애플리케이션을 실행한다.
```shell
wasmedge --reactor target/wasm32-wasi/debug/add.wasm add 2 2
4
```


**물론 대부분의 경우 CLI 인수를 사용하여 함수를 호출하지 않습니다. 대신 함수를 호출하고 호출 매개변수를 전달하고 반환 값을 수신하려면 WasmEdge의 언어 SDK를 사용해야 할 것입니다.** 

[WasmEdge는 WasmEdge Rust SDK](https://wasmedge.org/book/en/sdk/rust.html) 를 통해 Rust 애플리케이션에 삽입하는 것을 지원합니다. WasmEdge Rust SDK를 사용하는 방법에 대해서는 다른 문서에서 정리합니다. 






