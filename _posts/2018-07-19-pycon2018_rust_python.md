# 2018 파이콘에서 설명할 러스트 설명



![image](https://user-images.githubusercontent.com/16227780/42924867-02b0ec24-8b67-11e8-9bcb-7f52891fd2d5.png)



#### 들어가기전에

> 저는 파이썬을 5년 이상 사용하였습니다. 
>
> 하지만 아직까지도 저는 파이선을 2%만 사용하는 것 같습니다. 
>
> 빠른 연산을 필요로 할때에는 Cython을 사용하는데, 이번에는 요즘 핫한 Rust 언어와  Python을 함께 사용해서
>
> 블록체인의 proof of work 부분을 구현해보려고 합니다.

------





####  러스트언어란?

> 
>
> 러스트 프로그래밍 언어는 여러분이 더 빠르고, 더 안정적인 소프트웨어를 작성하도록 해줍니다. 프로그래밍 언어 디자인에서 고수준의 인간공학과 저수준의 제어는 종종 조화롭지 못합니다; 러스트는 이러한 갈등에 도전합니다. 강력한 기술적 능력과 훌륭한 개발자 경험을 조화롭게 하는 것을 통해, 러스트는 (메모리 사용 같은) 저수준 디테일을 그러한 제어를 하는데 동반되는 전통적으로 귀찮은 것들 없이도 제어하는 옵션을 제공합니다
>
> 

------



자 그럼 본격적으로 Rust에 대해서 잠깐 알아보자!

#### Hello World!

```rust
fn main() {
    // The statements here will be executed when the compiled binary is called
	println!("Hello World!");
}

```



에디터를 닫고 나서 아래의 명령어를 입력해보자

```bash
$ rustc hello.rs
```



정상적으로 컴파일이 되었다면 실행가능한 파일이 생길 것이다.

```bash
$ ./hello

 Hello World!
```



#### Mutability

```rust
fn main() {
    let _immutable_binding = 1;
    let mut mutable_binding = 1;

	println!("Before mutation: {}", mutable_binding);

	// Ok
	mutable_binding += 1;

	println!("After mutation: {}", mutable_binding);

    // Error!
	_immutable_binding += 1;
	// FIXME ^ Comment out this line
}
```



#### For and iterators

```rust
fn main() {
    let names = vec!["Bob", "Frank", "Ferris"];

	for name in names.iter() {
   		match name {
        	&"Ferris" => println!("There is a rustacean among us!"),
       		 _ => println!("Hello {}", name),
   	 	}
	}
}
```



------



#### 간단한 블록체인 인증 알고리즘



> 블록체인을 파이선으로 구현하려고 했었습니다. 1주일전에... 
>
> 거기에서 자격증명하는 부분이 컴퓨팅파워가 많이 드는 부분인데, 그 부분을 러스트를 사용해서 재미있게 해결해보고 싶었습니다.
>
> 
>
> 여기 간단한 파이선 코드를 보실까요?
>
> 자격증명이라는 부분인데, 이부분은 블록체인에서 상당히 중요한 부분입니다.



코드를 한번 보시죠

```python
def proof_of_work(self, block):
    """ Docstring for proof_of_work.
    Function that tries different values of nonce to get a hash
    that satisfies our difficulty criteria.
    """
    block.nonce = 0

    computed_hash = block.compute_hash()

    while not computed_hash.startswith('0' * BlockChain.difficulty):
        block.nonce += 1
        computed_hash = block.compute_hash()
        print(computed_hash, block.nonce)

    return computed_hash
```



한눈에 보기 어렵다면 이렇게 생각하면 좋습니다. (참고로 우리는 블록체인을 완벽하게 이해할 필요는 없습니다!)

1. 임의의 문자열을 계속 생성을 합니다. 
2. 우리가 원하는 결과값을 "000..." 이런식으로 지정을 해놓고. 해당 문자열이 나올때 까지 임임의 문자열을 계속 매치 시켜보는 겁니다. 
   임의의 문자열을 계속 생성시키는 과정에서 숫자 1을 계속 더해줌으로써, 중복되지 않는 문자열을 생성을 할 수가있습니다.
3. 자 한번 보실까요?



좀더 쉬운 코드입니다. 

simple_poof_job함수를 보시면 diffuculty 라는 integer를 받아서

"0" * 2 -> "00"

"0" * 3 -> "000"

"0" * 5 -> "00000"

. . . . . . 

"0" * difficulty -> "0"이 difficulty 만큼 반복

![image](https://user-images.githubusercontent.com/16227780/42924473-53252ffa-8b65-11e8-878b-3322dfc60946.png)



한눈에 보더라도 파이선으로 이 코드를 실행한다면, 매우 매우 느릴 것입니다.

그렇기 때문에, poof을 하는 함수를 rust를 통해서 사용 할 것입니다.

------





Cargo.toml

```toml

[package]

name = "pyrust-proof-example"

version = "0.1.0"

authors = ["elastic7327 <elastic7327@gmail.com>"]

[lib]

name = "example"

crate-type = ["dylib"]

[dependencies]

cpython = {git = "https://github.com/dgrunwald/rust-cpython.git"}

rust-crypto = "^0.2"
```





lib.rs

```rust
#[macro_use]
extern crate cpython;

#[macro_use]
extern crate crypto;

use cpython::{Python, PyResult};
use self::crypto::sha2::Sha256;
use self::crypto::digest::Digest;


 pub fn valid_proof(last_proof: u32, proof: u32, diff: usize) -> bool {
    let mut hasher = Sha256::new();
    let guess = &format!("{}{}", last_proof, proof);
    hasher.input_str(guess);
    let output = hasher.result_str();
    &output[..diff] == "0".repeat(diff)
}

 fn example(_py: Python, diff: u32) -> PyResult<u32>{

    let mut proof = 0;
    let last_proof = 0;

    while valid_proof(last_proof, proof, diff as usize) == false {
         proof = proof + 1
    }

    Ok(proof)
}
py_module_initializer!(while_with_sha, initexample, PyInit_example, |py, m| {

    m.add(py, "example", py_fn!(py, example(diff: u32)))?;
    Ok(())
});
```





```
cargo build --release 
```



src/target/release에 가보면  libexample.dylib 파일이 있습니다.

```bash
drwxr-xr-x@ 11 dan  staff   352B  7 19 16:22 **.**
drwxr-xr-x   6 dan  staff   192B  7 19 14:44 **..**
-rw-r--r--   1 dan  staff     0B  7 19 14:41 .cargo-lock
drwxr-xr-x  33 dan  staff   1.0K  7 19 14:41 **.fingerprint**
drwxr-xr-x  12 dan  staff   384B  7 19 14:41 **build**
drwxr-xr-x  44 dan  staff   1.4K  7 19 16:16 **deps**
drwxr-xr-x   2 dan  staff    64B  7 19 14:41 **examples**
drwxr-xr-x   2 dan  staff    64B  7 19 14:41 **incremental**
-rw-r--r--   1 dan  staff   180B  7 19 16:05 libexample.d
-rwxr-xr-x   2 dan  staff   2.7M  7 19 16:16 libexample.dylib         <<--- what we need!
drwxr-xr-x   2 dan  staff    64B  7 19 14:41 **native**
```



이 파일을 *.so 형식으로 변환해줍니다. 

```
mv libexample.dylib example.so
```



바로 그자리에서 ipython으로  적절하게 import 가 되는지 확인합니다.

```bash
**pycon2018_with_rust** **git:(****master****)** **✗** ipython

Python 3.6.6 (v3.6.6:4cf1f54eb7, Jun 26 2018, 19:50:54)

Type 'copyright', 'credits' or 'license' for more information

IPython 6.4.0 -- An enhanced Interactive Python. Type '?' for help.

In [**1**]: **import** **example**

In [**2**]: example.example(2)

Out[**2**]: 563
```



성공적으로 improt 가 되었습니다!.



이후 benchmark를 위해서 테스트 코드를 적절하게 작성해서 테스트를 해봅니다.

```python

import pytest
import example # rust implemented!

from hashlib import sha256


def valid_proof(last_proof, proof, difficulty):
    guess = "{}{}".format(last_proof, proof)
    res = sha256(guess.encode()).hexdigest()
    return "0"*difficulty == res[:difficulty]


def simple_proof_job(difficulty):

    proof = 0

    while (valid_proof(0, proof, difficulty) == False):
        proof += 1

def test_rust_implemented_proof_job_benchmark(benchmark):
    benchmark(example.example, 4)


def test_pure_python_proof_job(benchmark):
    benchmark(simple_proof_job, 4)
```



자 이제 테스트를 해봅니다. 

```bash
pytest -s -v 
```



결과화면 입니다.

![image](https://user-images.githubusercontent.com/16227780/42927934-e7ad6254-8b70-11e8-904a-fd131ad6d2cf.png)

rust를 사용한 파이선이, 순수 파이선 코드보다 3배 빠르다는 것을 확인 할 수 있었습니다.!



감사합니다.