# 开发例程

复现开发文档中【开发例程】

## 主机环境
* Windows10_x86-64
* **WSL2**
* Debian
* Ubuntu 20.04

需要注意，如果在 Windows + WSL 的环境，一定要用 **WSL2**，WSL2 包含完整的 Linux 内核，如果是 WSL（WSL1），部分系统调用不识别，例程无法编译。

WSL2 安装方式：<https://learn.microsoft.com/zh-cn/windows/wsl/install-manual>

比较 WSL1 和 WSL2：<https://learn.microsoft.com/zh-cn/windows/wsl/compare-versions>

> **注意：**
>
> 根据官方开发文档上的说明，最终测试会放在 Ubuntu 20.04 上进行，所以建议使用该版本，避免不必要的问题

## 编译例程

### 0. 安装 Rust

参考官网教程 <https://www.rust-lang.org/zh-CN/tools/install>

*可以更换 Cargo 源，提高开发效率。*

### 1. 下载

<https://h5.vivo.com.cn/blueos-atom/c2rust/开发范例.zip> \
<https://h5.vivo.com.cn/blueos-atom/c2rust/测试用例.zip>

解压后，资源都是中文名文件夹，为了后续方便可以改为较短的英文名：
```bash
$ mkdir vivo-c2rust
$ wget https://h5.vivo.com.cn/blueos-atom/c2rust/开发范例.zip
$ unzip 开发范例.zip
$ mv 开发范例 dev-demo
$ cd dev-demo
$ mv 基于ChatGLM的命令行工具 cl-base-ChatGLM
$ cd cl-base-ChatGLM
Cargo.lock  Cargo.toml  LICENSE  src
```

### 2. 编译
`cl-base-ChatGLM` 是一个典型 `cargo` 生成的目录结构，直接运行：
```bash
$ cargo build
...
   Compiling mio v0.8.10
error: failed to run custom build command for `openssl-sys v0.9.101`

Caused by:
...
  run pkg_config fail: Could not run `PKG_CONFIG_ALLOW_SYSTEM_CFLAGS=1 pkg-config --libs --cflags openssl`
  The pkg-config command could not be found.
  
  Most likely, you need to install a pkg-config package for your OS.
  Try `apt install pkg-config`, or `yum install pkg-config`,
  or `pkg install pkg-config`, or `apk add pkgconfig` depending on your distribution.
...
  Make sure you also have the development packages of openssl installed.
  For example, `libssl-dev` on Ubuntu or `openssl-devel` on Fedora.
...
```

根据提示需要安装 `libssl-dev`、`pkg-config`：
```bash
$ sudo apt install libss-dev pkg-config
```

继续编译：
```bash
$ cargo build
...
warning: `C2RustGLM` (bin "C2RustGLM") generated 16 warnings (run `cargo fix --bin "C2RustGLM"` to apply 11 suggestions)
    Finished dev [unoptimized + debuginfo] target(s) in 23.95s
```
编译成功，但有很多警告，多数都是定义但未用到的变量，暂时不是处理。

### 2. 运行
```bash
$ cargo run
...
    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
     Running `target/debug/C2RustGLM`
Usage: target/debug/C2RustGLM <cpp_file_path>
```
根据提示，需要提供C文件，【测试用例.zip】中只有pdf文件，按照 `多线程安全代码转换.pdf` 编写一份C文件：
```c
// c-src/thread-safe.c

#include <pthread.h>
#include <stdlib.h>

int data = 0;
void *increment(void *v) {
    for (int i = 0; i < 1000000; i++) {
        data++;
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
}
```
执行时传入文件：
```bash
$ mkdir c-src
$ vim thread-safe.c
$ cargo run -- c-src/thread-safe.c
    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
     Running `target/debug/C2RustGLM c-src/thread_safe.c`
    ```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn increment(data: Arc<Mutex<i32>>) {
    for _ in 0..1_000_000 {
        let mut data = data.lock().unwrap();
        *data += 1;
    }
}

fn main() {
    let data = Arc::new(Mutex::new(0));
    let data_clone = Arc::clone(&data);

    let handle1 = thread::spawn(move || increment(data));
    let handle2 = thread::spawn(move || increment(data_clone));

    handle1.join().unwrap();
    handle2.join().unwrap();
}
    ```
```
结果和测试有区别，但逻辑没有错误，对输出结果进行验证，编译、运行均没有错误。


### 3. 更换 API key
在不做任何修改的情况下，已经可以正常运行，但建议按开发文档上的说明，将工程中用到的智谱 AI 开放平台的 API key 更换成自己的。

<https://maas.aminer.cn/usercenter/apikeys>

登录后首页即可查看到自己的 API key，复制、修改工程中的 `src/sse_invoke_method/sse_invoke/constant_value.rs:8:28`。

重新编译、运行。

**后记**

在第二遍执行后，多数情况为以下结果，即没有 `main` 函数翻译：
```bash
    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
     Running `target/debug/C2RustGLM c-src/thread-safe.c`
    ```rust
use std::sync::{Arc, Mutex};
use std::thread;

let data = Arc::new(Mutex::new(0));

fn increment(data: Arc<Mutex<i32>>) {
    for _ in 0..1_000_000 {
        let mut data = data.lock().unwrap();
        *data += 1;
    }
}

let data_clone = Arc::clone(&data);
let handle1 = thread::spawn(move || increment(data));
let handle2 = thread::spawn(move || increment(data_clone));

handle1.join().unwrap();
handle2.join().unwrap();
    ```
```