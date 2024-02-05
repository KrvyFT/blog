---
title: Tokio Rutime
date: 2024-02-05 01:45:02
tags: Tokio
categories: 笔记
---

Runtime 异步运行时。

## 创建
可以使用 Runtime Builder 来创建和配置运行时环境。
```rust
// 创建带有线程池的runtime

let rt = tokio::runtime::Builder::new_multi_thread()

.worker_threads(8) // 8个工作线程

.enable_io() // 可在runtime中使用异步IO

.enable_time() // 可在runtime中使用异步计时器(timer)

.build() // 创建runtime

.unwrap();
```

tokio 提供两种运行时模式。
- 单线程适合密集计算的场景。
- 多线程模式适合多IO的场景。

可以这样来创建不同的模式：
```rust
// 创建单一线程的runtime
let rt = tokio::runtime::Builder::new_current_thread().build().unwrap();

// 创建多线程的runtime
let rt = tokio::runtime::Runtime::new().unwrap();
```

也可以使用简单的方式创建运行时:
```rust
#[tokio::main]
async fn main() {}
```

具体来说有这几种:
```rust
#[tokio::main(flavor = "multi_thread")] // 等价于#[tokio::main]
#[tokio::main((flavor = "multi_thread", worker_threads = 10))]
#[tokio::main((worker_threads = 10))]
```

## 在 Runtime 中执行任务
使用 tokio 的 sleep 来解释。


> `tokio::time` 中的 sleep 是放弃它所在任务的 CPU 并等待唤醒，它不会阻塞当前线程，它所在的线程可以继续执行其他任务。


### block_on

```rust
fn main() {
    let rt = Runtime::new().unwrap();
    rt.block_on(async {
        println!("before sleep: {}", Local::now().format("%F %T.%3f"));
        tokio::time::sleep(tokio::time::Duration::from_secs(2)).await;
        println!("after sleep: {}", Local::now().format("%F %T.%3f"));
    });
}
```
block_on 方法，接受一个 Future 参数，这个方法将阻塞当前线程直到指定的任务全部完成。

block_on 返回异步任务的返回值。

### spawn
有时候 要执行异步任务却不在 Runtime 中时，可以使用 spawn 来生成异步任务。
```rust
use std::thread;

use chrono::Local;
use tokio::{self, runtime::Runtime, time};

fn now() -> String {
    Local::now().format("%F %T").to_string()
}

// 在runtime外部定义一个异步任务，且该函数返回值不是Future类型
fn async_task() {
  println!("create an async task: {}", now());
  tokio::spawn(async {
    time::sleep(time::Duration::from_secs(10)).await;
    println!("async task over: {}", now());
  });
}

fn main() {
    let rt1 = Runtime::new().unwrap();
    rt1.block_on(async {
      // 调用函数，该函数内创建了一个异步任务，将在当前runtime内执行
      async_task();
    });
}
```

## 进入 Runtime
使用 `entry()` 进入 Runtime 时，不会阻塞当前线程，他会返回一个 `EnterGuard` 仅用于表示所有任务都在 Runtime 中执行 知道被删除。

在删除`EnterGuard`后可以进入到另一个``EnterGuard`` 。
```rust
use std::time::Duration;
use chrono::Local;
use tokio::{self, time::sleep};

fn main() {
let rt = tokio::runtime::Builder::new_multi_thread()
	.worker_threads(8)
	.enable_io()
	.enable_time()
	.build()
    .unwrap();

let gruad1 = rt.enter();

rt.spawn(async {
	println!("Gruad 1 start: {}", now());
	sleep(Duration::from_secs(1)).await;
	println!("Gruad 1 end: {}\n", now());
});

drop(gruad1); // 注释掉这一行将 panic!

let _gruad2 = rt.enter();
rt.spawn(async {
	println!("Gruad 2 start: {}", now());
	sleep(Duration::from_secs(1)).await;
	println!("Gruad 2 end: {}\n", now());
});

std::thread::sleep(Duration::from_secs(10));
}

fn now() -> String {
	Local::now().format("%F %T%.3f").to_string()
}
```

## tokio 的两种线程
首先是用于异步的工作线程，他用于不会阻塞的任务。每一个工作线程都绑定一个系统核心。
其次是阻塞线程，他用于会长时间阻塞或者同步的任务。
阻塞线程默认是不存在的，只有在调用了`spawn_blocking()`时才会创建一个对应的阻塞线程。