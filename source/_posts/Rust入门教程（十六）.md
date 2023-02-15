---
title: Rust入门教程（十六）：最后的项目
mathjax: true
tags:
  - Rust
categories:
  - 编程语言
  - Rust
abbrlink: 9e622d63
date: 2022-09-29 22:55:08
---

>前言

<!-- more -->

## 一、单线程 Web 服务器

### 1.1 构建单线程 Web 服务器

- 在 socket 上监听 TCP 连接
- 解析少量的 HTTP 请求
- 创建一个合适的HTTP响应
- 使用线程池改进服务器的吞吐量
- 注意：并不是最佳实践

直接放代码：

```rust
use std::{net::{TcpListener, TcpStream}, io::{Read, Write}};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:9999").unwrap();
    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();
    println!("Request: {}\n", String::from_utf8_lossy(&buffer));

    let contents = std::fs::read_to_string("hello.html").unwrap();
    let response = format!("HTTP/1.1 200 OK\r\nContent-Length:{}\r\n\r\n{}", contents.len(), contents);
    println!("{}", response);
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

这里 `TcpListener::bind()` 方法表示监听所绑定的 IP 及端口，返回一个 Result 枚举，`incoming` 方法能够将所监听到的流转化成一个个迭代器，然后一依次处理这些流。

在 `handle_connection()` 函数中，先构造了一个 `buffer` 用于存放每个流请求的具体内容（请求头），然后写了一个 `hello.html` 页面，构造一个响应头同时写回请求的流中。在 `response` 字段中，要注意添加 `Content-Length:{}`，这样执行该程序，在浏览器中访问 `127.0.0.1:9999`，便可以返回刚刚写的页面。

控制台输出如下：

```rust
➜  mweb git:(master) ✗ cargo run
   Compiling mweb v0.1.0 (/Users/cherry/Code/Rust/learning/mweb)
    Finished dev [unoptimized + debuginfo] target(s) in 0.14s
     Running `target/debug/mweb`
Request: GET / HTTP/1.1
Host: 127.0.0.1:9999
Connection: keep-alive
Cache-Control: max-age=0
sec-ch-ua: "Google Chrome";v="105", "Not)A;Brand";v="8", "Chromium";v="105"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=

HTTP/1.1 200 OK
Content-Length:170

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <title>Hello!</title>
</head>

<body>
    <h1>Hello!</h1>
    <p>Hi From Rust</p>
</body>

</html>
```

浏览器显示的页面如下：

![浏览器正常显示页面](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220929233943.png)

下面我们新建一个 404 页面，用于处理访问其他页面时显示的结果，我们主要修改 `handle_connection()` 函数：

```rust
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();
    // println!("Request: {}\n", String::from_utf8_lossy(&buffer));

    let get = b"GET / HTTP/1.1\r\n";
    let mut response = String::from("");
    if buffer.starts_with(get) {
        let contents = std::fs::read_to_string("hello.html").unwrap();
        response = format!(
            "HTTP/1.1 200 OK\r\nContent-Length:{}\r\n\r\n{}",
            contents.len(),
            contents
        );
        // println!("{}", response);
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        response = format!(
            "{}\r\nContent-Length:{}\r\n\r\n{}",
            status_line,
            contents.len(),
            contents
        );
        // println!("{}", response);
    }
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

首先判断请求头是不是以 `GET / HTTP/1.1\r\n` 开头，这表明我们请求的是根目录的资源，这里 `let get = b"GET / HTTP/1.1\r\n";` 的 `b` 表示字节字符串，可以将字符串转化成字节，这样就可以用 `start_with()` 方法进行比较。

在浏览器中访问一个非根目录的资源，控制台输出如下：

```rust
➜  mweb git:(master) ✗ cargo run
   Compiling mweb v0.1.0 (/Users/cherry/Code/Rust/learning/mweb)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32s
     Running `target/debug/mweb`
Request: GET /undefined HTTP/1.1
Host: 127.0.0.1:9999
Connection: keep-alive
sec-ch-ua: "Google Chrome";v="105", "Not)A;Brand";v="8", "Chromium";v="105"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "macOS"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/105.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fet

HTTP/1.1 404 NOT FOUND
Content-Length:201

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <title>Hello!</title>
</head>

<body>
    <h1>Oops!</h1>
    <p>Sorry,I don't know what you' re asking for. </p>
</body>

</html>
```

浏览器页面如下：

![404页面](https://raw.githubusercontent.com/CherryYang05/PicGo-image/master/images/20220929235635.png)

重构一下 `handle_connection()` 函数，用元组来重构：

```rust
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();
    // println!("Request: {}\n", String::from_utf8_lossy(&buffer));

    let get = b"GET / HTTP/1.1\r\n";
    let (status_line, file_name) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n", "404.html")
    };

    let contents = std::fs::read_to_string(file_name).unwrap();
    let response = format!(
        "{}Content-Length:{}\r\n\r\n{}",
        status_line,
        contents.len(),
        contents
    );

    println!("{}", response);
    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

## 二、多线程 Web 服务器

### 2.1 阻塞的场景

当前我们对于流的处理都是单线程，一旦有某个请求耗费的时间长了，那么其他请求就必须被阻塞等待。我们构造下面这样的场景：当访问 `sleep` 页面的时候，线程睡眠 5 秒钟再执行处理，运行程序后访问其他页面可以正常被处理，当访问 `sleep` 页面的时候会卡住 5 秒钟，这时访问其他页面就需要等待这 5 秒钟结束。

```rust
let sleep = b"GET /sleep HTTP/1.1\r\n";

let (status_line, file_name) = if buffer.starts_with(get) {
    ("HTTP/1.1 200 OK\r\n", "hello.html")
} else if buffer.starts_with(sleep) {
    thread::sleep(Duration::from_secs(5));
    ("HTTP/1.1 200 OK\r\n", "hello.html")
} else {
    ("HTTP/1.1 404 NOT FOUND\r\n", "404.html")
};
```

在代码中增加了对 `sleep` 页面的访问，运行效果与预期一致。

我们用线程池来解决这一问题。

### 2.2 线程池

线程池是一组预先被分配的线程，他们用于等待并随时处理可能的任务。线程池可以实现并发处理请求，当前线程执行完之后，将其放回线程池。首先考虑到使用 `thread::spawn()` 为每个请求创建线程：

```rust
thread::spawn(||{
    handle_connection(stream);
});
```

但是这样显然会导致一个问题：当有黑客对服务器进行洪泛攻击时，服务器的资源将会很快被耗尽，因此我们还需要对线程数量进行限制。

在该项目中，采用编译器驱动开发的方式（笑），先将可能用到的结构体或方法等先写好，再逐步完善，修改 main 如下：

```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:9999").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

很明显 `ThreadPool` 和 `execute` 是未定义的。

新建 `lib.rs`，实现未定义的部分：

```rust
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        
    }
}
```

对于 `execute` 方法，我们参考 `thread::spawn()`，后者实现了 `FnOnce() + Send + 'static`。

然后我们给结构体 `ThreadPool` 添加一个字段 `threads`。

```rust
pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}
```

这个是怎么来的呢，我们看 `spawn` 的实现：

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
where
    F: FnOnce() -> T,
    F: Send + 'static,
    T: Send + 'static,
{
    Builder::new().spawn(f).expect("failed to spawn thread")
}
```

它返回的是 `JoinHandle`，其中有一个范型 `T`，这个 `T` 就是传进去的闭包的返回值，但是我们实现的方法闭包没有返回值，因此返回单元类型 `()` 即可。

再次修改 `new()` 函数：

```rust
pub fn new(size: usize) -> ThreadPool {
    assert!(size > 0);
    let mut threads = Vec::with_capacity(size);

    // 创建线程并存储到 vec 中
    for _ in 0..size {
        todo!()    
    }
    ThreadPool {threads}
}
```

我们再看`spawn` 的实现，他会创建一个线程，立即执行接收到的代码。但是我们希望线程创建之后进入等待状态，当有代码传给他们的时候再执行线程。这里我们创建一个新的结构体，叫 `Worker`，用来管理和实现上述所说的行为。

实现 `Worker` 相关结构体和函数：

```rust
pub struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});
        Worker {id, thread}
    }
}
```

同时将 `ThreadPool` 中的字段名改成 `workers`，更改 `ThreadPool` 的 `new()` 函数。

```rust
pub fn new(size: usize) -> ThreadPool {
    assert!(size > 0);
    let mut workers = Vec::with_capacity(size);

    // 创建线程并存储到 vec 中
    for id in 0..size {
        workers.push(Worker::new(id));
    }
    ThreadPool {workers}
}
```

目前为止，`lib.rs` 代码如下：

```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {

    /// 创建一个线程池
    /// 
    /// size 表示线程池中线程数量
    /// 
    /// # Panics
    /// 
    /// Panic: size is zero
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        let mut workers = Vec::with_capacity(size);

        // 创建线程并存储到 vec 中
        for id in 0..size {
            workers.push(Worker::new(id));
        }
        ThreadPool {workers}
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        
    }
}

pub struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});
        Worker {id, thread}
    }
}
```

### 2.3 使用通道

下面我们需要考虑如何让 `Worker` 从线程池中接收任务并执行任务，这里就要使用到通道。

在 `ThreadPool` 中添加一个字段 `sender`，表示通道的发送端。线程池持有通道的发送端，而接收者应该是 `worker`。在通道中，可以有多个发送者，但是只能有一个接收者，我们希望所有线程共享同一个 `receiver`，从而能够在线程间分发任务。同时从通道队列中取出 `receiver` 也意味着这是可变的。我们可以用 “智能指针” 那一小节中的 `Arc` 和 `Mutex` 来实现线程间多所有权的可变引用。

```rust
pub fn new(size: usize) -> ThreadPool {
    assert!(size > 0);
    let mut workers = Vec::with_capacity(size);
    let (sender, receiver) = mpsc::channel();
    let receiver = Arc::new(Mutex::new(receiver));

    // 创建线程并存储到 vec 中
    for id in 0..size {
        workers.push(Worker::new(id, Arc::clone(&receiver)));
    }
    ThreadPool {workers, sender}
}
```

同时修改 `Worker` 结构体的 `new` 的函数签名：

```rust
fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {}
```

### 2.4 实现 execute 方法

新建一个 `job`，然后通过通道的发送端将 `job` 发送出去，接收端 `worker` 的 `new` 函数接收该 `job`。

```rust
pub fn execute<F>(&self, f: F)
where
    F: FnOnce() + Send + 'static,
{
    let job = Box::new(f);
    self.sender.send(job).unwrap();
}

fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
    let thread = thread::spawn(move || {
        let job = receiver.lock().unwrap().recv().unwrap();
        println!("Worker {} get a job; executing", id);
        (*job)(); 
    });

    Worker {id, thread}
}
```

我们知道 `job` 是一个 `Box` 类型，那么实现 `FnOnce()` 的闭包要想调用就要先将其从 `Box` 取出来，但是 Rust 不允许这样做，因为不知道 `Box` 中的类型具体有多大。`FnOnce()` 中有一个 `call_once()` 方法，其中的参数便是 `self`，为了获得其所有权，但是现在不允许。我们可以将 `self` 改成 `Box<Self>`，这样方法就可以在类型的 `Box` 上来调用。

然后为实现了 `FnOnce()` 的类型实现 `FnBox`，`self` 的类型就是 `Box<F>`，而 `F` 就是实现了 `FnOnce()` 的类型，这样就可以获得 `Box` 里的所有权，再进行闭包调用，同时更改 `Job` 的类型。

```rust
trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<Self>) {
        (*self)()
    }
}

type Job = Box<dyn FnBox + Send + 'static>;
```

最后在 `Worker` 的 `new` 函数中加入 loop，使得释放锁后还能继续使用该线程。

最终 `lib.rs` 文件如下：

```rust
use std::{thread, sync::{mpsc, Arc, Mutex}};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

// pub struct Job {}

impl ThreadPool {

    /// 创建一个线程池
    /// 
    /// size 表示线程池中线程数量
    /// 
    /// # Panics
    /// 
    /// Panic: size is zero
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);
        let mut workers = Vec::with_capacity(size);
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));

        // 创建线程并存储到 vec 中
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        ThreadPool {workers, sender}
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

pub struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<Self>) {
        (*self)()
    }
}

type Job = Box<dyn FnBox + Send + 'static>;

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();
            println!("Worker {} get a job; executing", id);
            job.call_box(); 
        });

        Worker {id, thread}
    }
}
```








