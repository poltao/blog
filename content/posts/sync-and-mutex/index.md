+++
title = "Send & Mutex: Misconceptions about Send"
description = "【翻译】在 Rust 中，想要实现一个类型的值在不同的线程或者异步任务中正常使用（确切的来说应该是无data race的读写），该类型必须实现两个标记 Trait: Send+Sync。而作者这篇文章旨在通过一系列的例子，来指明在某种程度上我们对 Send Trait 存在的误解，即 Send 应该指的：是不同的线程在不同的时间安全的使用；而不是我们下意识认为的：将一个类型的值从一个线程发送到另外一个线程。借此回答了作者在 Reddit 上看到的一个有趣的问题：为什么 Mutex<T> 想要保证线程安全就必须确保类型 T 实现了 Send Trait。"
date = 2024-06-04
[taxonomies]
tags = ["Rust", "Mutex", "Trait", "Translate"]
+++

> 原文：[Send & Mutex - Misconceptions about Send](https://cryptical.xyz/rust/send-mutex)
>
> Author: [Özgün Özerk](https://github.com/ozgunozerk)

## 1. 背景介绍

&#x2003;在 Rust 中，想要实现一个类型的值在不同的线程或者异步任务中正常使用（确切的来说应该是无 data race 的读写），该类型必须实现两个标记
Trait: Send+Sync。而作者这篇文章旨在通过一系列的例子，来指明在某种程度上我们对 Send Trait 存在的误解，即 Send
应该指的：是不同的线程在不同的时间安全的使用；而不是我们下意识认为的：将一个类型的值从一个线程发送到另外一个线程。借此回答了作者在
Reddit 上看到的一个有趣的问题：<font color=red>为什么 Mutex\<T\> 想要保证线程安全就必须确保类型 T 实现了 Send Trait？</font>

首先可以通过两个简单的例子，来窥探一下，实现了 Send+Sync 的类型和未实现的类型，在 Rust 异步任务中的编译情况：

<div style="display: flex; justify-content: space-between; align-items: center;">
    <img src="no_sync_demo.png" alt="No Sync Demo" style="max-width: 52%; height: auto;">
    <img src="sync_demo.png" alt="Sync Demo" style="max-width: 48%; height: auto;">
</div>

如上图所示，no_sync_demo 展示了在异步任务中使用了未实现 Sync 的数据结构 Rc，从而导致程序无法通过编译。而在 sync_demo
中我们切换为了线程安全的 Arc，从而使得代码可以正常运行。究其原因是因为 Arc 实现了 Send + Sync 两个
Trait，如下所示（11、13 行）：

```rust, linenos
pub struct Arc<
    T: ?Sized,
    #[unstable(feature = "allocator_api", issue = "32838")] A: Allocator = Global,
> {
    ptr: NonNull<ArcInner<T>>,
    phantom: PhantomData<ArcInner<T>>,
    alloc: A,
}

#[stable(feature = "rust1", since = "1.0.0")]
unsafe impl<T: ?Sized + Sync + Send, A: Allocator + Send> Send for Arc<T, A> {}

#[stable(feature = "rust1", since = "1.0.0")]
unsafe impl<T: ?Sized + Sync + Send, A: Allocator + Sync> Sync for Arc<T, A> {}
```

## 2. Send & Sync (intro)

这是由编译器自动实现的两个标记 Trait，他们用来指定一个类型可以在并发编程模型下安全使用而不会产生 Data
Race。我们很容易找到两者的官方文档定义：[Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html#send-and-sync)
，但仍旧有些晦涩难懂。相对应的，[Jonathan Giddy](https://stackoverflow.com/users/2644842/jonathan-giddy)
在 stackoverflow 上的[这个回答](https://stackoverflow.com/a/60109068/14075443)要更加的直观：

- Sync allows an object to be used by two threads A and B at the same time.

- Send allows an object to be used by two threads A and B at different times.

同时老哥还详细介绍了一下 Rc 和 Arc 的区别，以及为什么 Rc 不是线程安全的。

> 大多数类型都是实现了 Send 的，但 Rc 是一个例外，因为 Rc 允许一个值有多个所有者，然而其底层的引用计数又是非原子操作的，在多线程环境下同步操作引用计数可能会导致不符合预期的结果，所以 Rc 不是线程安全的。
>
> Arc 是使用了原子类型实现了引用计数的 Rc，所以是线程安全的。此外如果 Arc 指向的数据是 Sync 的，那么整个对象也应该是 Sync 的。而如果某个对象不是 Sync 的，我们可以尝试使用 Mutex 包裹起来，从而使其变得线程安全。因此在异步 Rust 编程中，我们会看到大量的 Arc<Mutex\<T\>>类型的变量。

作者之所以在这篇文章中特意引入了这个非官方的定义，而不是官方的：<font color='red'>A type is Send if it is safe to send it to another thread，</font>是因为他认为这个解释会带来一些误解。而事实也确实如此，我此前一直认为 Send 意味着将一个类型的值从一个线程发送到另外一个线程，但实际上这并不是 Send 的真正含义。

### 2.1 Sync Intro

- [x] Say, you have a string.
- [x] And you want to share this string with multiple threads.
- [x] If more than one thread tries to modify the same string at the same time, you will get a data race.

根据上面这个例子可以看到，Sync 的工作机制和 Rust 的 borrow checker 有些类似，它确保了一个类型可以被多个线程同时访问，但是不允许多个线程同时修改（在无任何同步机制的情况下）。显然对于 String 我们可以在多个线程之间共享读而不会出现问题，因为 borrow checker 不会允许在存在不可变引用的情况下去获取到可变引用。所以 String 是 Sync 的，并且 Rust 中几乎所有的类型都是 Sync 的。

那什么类型是非 Sync 的呢？官方文档的[回答](https://doc.rust-lang.org/std/marker/trait.Sync.html#:~:text=Types%20that%20are%20not%20Sync%20are)如下：

> Types that are not Sync are those that have “interior mutability” in a non-thread-safe form, such as Cell and RefCell. These types allow for mutation of their contents even through an immutable, shared reference. For example the set method on Cell<T> takes &self, so it requires only a shared reference &Cell<T>. The method performs no synchronization, thus Cell cannot be Sync.

可以看到拥有内部可变性的类型，比如 Cell 和 RefCell，由于允许通过不可变引用来修改其内容，可以绕过 borrow checker 的所有权检查规则，让多个线程同时修改一个值，但 Cell 和 RefCell 本身又没有同步机制来避免 Data Race，所以它们是非 Sync 的。

### 2.2 Send Intro

如 Sync，Rust 中几乎所有的类型都是 Send 的，但是有一些例外，比如 Rc。

**Single-threaded example:**

- [x] We have an Rc<String> variable.
- [x] We clone it multiple times.
- [x] At this point, we cannot get a mutable reference to our String via Rc<String> clones, because there are multiple Rc<String> pointers present.
- [x] If we drop some Rc<String> clones, so that there is only one reference remaining, then we can get a mutable reference to our String via Rc<String>.

**Multi-threaded example:**

- [x] We have an Rc<String> variable.
- [x] We clone it multiple times.
- [x] We send these clones to different threads (actually we can’t do that, because Rust compiler forbids us to, but let’s say we can for the sake of this example).
- [x] At this point, we cannot get a mutable reference to our String, because there are multiple Rc<String> pointers present. So, let’s drop some of them.
- [x] If two threads tries to drop their Rc<String> clones at the same time, a data race on the reference counting mechanism will occur. And we will never be able to get a mutable reference to our String.

### 2.3 Sync and Send relationship

The Rustonomicon [notices that](https://doc.rust-lang.org/nomicon/send-and-sync.html#send-and-sync): <font color="green">A type is Sync if it is safe to share between threads (T is Sync if and only if &T is Send).</font>

实话说，看到这句话我的第一反应是（T is Sync if and only if &T is Send）是什么玩意，是人话吗？但看到作者的解释，倒也感觉有些道理：Sync 的本质是允许多个线程同时访问一个值，为了做到这一点我们需要把这个值的引用传递给其他线程，而这个引用就是 &T，所以 Sync 的定义就是可以被解释为<font color="red">和其他的线程安全地共享对象的不可变引用。</font>

## 3. Mutex Deep Dive

显然在多线程环境下，几乎所有类型都可以在不同的线程中安全的使用不可变引用，但是对于可变引用，我们需要使用 Mutex 或 RwLock 来保证线程安全。Mutex 和 RwLock 是最常见的两种允许多个线程同时修改临界数据的同步机制，同时它们也拥有内部可变性。

所以，现在我们可以再一次来回顾前面提出的问题，<font color='red'>“Why T needs to be Send in order to Mutex\<T\> to be Sync?”</font>

其实现在的问题可以转换为：<font color='red'>如果类型 T 是非 Send 的，那么 Mutex\<T\> 为什么不能 Sync 呢？</font>按照逻辑必定存在这样一个类型 T，它是非 Send 的，从而导致 Mutex\<T\> 也不能在多线程环境下安全的使用。

### 3.1 Does T needs to be Sync?

为了确保 Mutex\<T\>是 Sync 的，需要确保 T 实现了 Send。那么 T 需要实现 Sync 吗？答案是否定的，具体可以通过 Cell\<T\> 这个未实现 Sync 的类型来验证。具体的例子如下：

```rust
#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cell = Arc::new(Mutex::new(Cell::new(String::from("bbb"))));
    let mut handles = vec![];
    for idx in 0..10 {
        let elem = format!("ccc:{}", idx);
        let cell_ref = Arc::clone(&cell);
        let h = tokio::spawn(async move {
            let changed = cell_ref.lock().unwrap();
            changed.set(elem);
        });
        handles.push(h);
    }
    futures::future::join_all(handles).await;
    println!("{}", cell.lock().unwrap().get_mut()); // Maybe output: ccc:5
    Ok(())
}
```

这段代码可以正常编译运行通过，说明 Mutex\<T\> 想要是 Sync 的，只需要 T 满足 Send 即可，不管 T 是否满足 Sync。究其原因是因为 Mutex 限制了对 T 的访问，使得 T 的内部状态不会被多个线程同时访问，从而避免了 Data Race。

### 3.2 Are we send T when sending &Mutex\<T\> across threads?

标题的意思旨在提出在不同的线程中使用 Mutex\<T\> 的引用是否等同于将 T 发送到了另一个线程，答案是否定的。Mutex\<T\> 的引用只是一个指向 Mutex\<T\> 的指针，而 Mutex\<T\> 本身是一个包含了 T 的结构体，所以 Mutex\<T\> 的引用并不等同于将 T 发送到了另一个线程。最直接的办法就是查看 Mutex\<T\> 修改底层 T 类型对象的方法 lock()：

```rust
pub fn lock(&self) -> LockResult<MutexGuard<'_, T>>
```

继续查看 [MutexGuard](https://doc.rust-lang.org/std/sync/struct.MutexGuard.html) 的定义：

```rust
/// An RAII implementation of a "scoped lock" of a mutex. When this structure is
/// dropped (falls out of scope), the lock will be unlocked.

/// The data protected by the mutex can be accessed through this guard via its
/// [`Deref`] and [`DerefMut`] implementations.
pub struct MutexGuard<'a, T: ?Sized + 'a>
```

我们注意到他本身是一个 RAII 的结构体，同时通过实现 Deref 和 DerefMut 两个 Trait，使得我们可以通过 MutexGuard 来访问 Mutex\<T\> 中的数据。其中 DerefMut 的[定义](https://doc.rust-lang.org/std/ops/trait.DerefMut.html)如下：

```bash
Used for mutable dereferencing operations, like in *v = 1;
```

此时再次回想起前面提到的问题：Why T needs to be Send for Mutex\<T\> to be Sync?

我们很容易就能够理解为什么会提出这个问题，这个困惑是由 Send 命名造成的，让读者误以为 Send 是将对象自身发送到另一个线程，而实际上 Send 只是一个标记 Trait，用来指定一个类型可以在不同的时间被不同的线程安全地使用。因此，作者还特意选择了上文[Jonathan Giddy](https://stackoverflow.com/users/2644842/jonathan-giddy)提出的这个定义。

### 3.3 Example: Mutex\<Rc\>

现在是时候来分析一下 Mutex\<Rc\<String\>\> 这个数据结构了，我们都知道 Rc\<String\> 是没有 Sync+Send 的，

```rust
impl<T: ?Sized, A: Allocator> !Send for Rc<T, A> {}
impl<T: ?Sized, A: Allocator> !Sync for Rc<T, A> {}
```

那么使用 Mutex\<Rc\<String\>\> 会发生什么呢？

- [x] 创建一个 Mutex\<Rc\<String>> 对象 obj。
- [x] 在多个线程中使用 obj 的引用。
- [x] Mutex 的 lock 机制会保证在任意时刻只有一个线程可以访问 obj 内部的 Rc\<String\>。
- [x] 此时在多个线程中你可以直接使用 Rc\<String\> 的 clone 方法来创建 Rc\<String\> 的克隆，并且它们在 Mutex 的临界区之外也可以仍旧有效。如此相当于就有了多个不在 Mutex 的临界区内的 Rc\<String\> 的引用。
- [x] 如此就会导致有多个线程在尝试 drop Rc\<String\> 的时候，由于引用计数是非原子的，就会导致出现 Data Race。

## 4. Conclusion

“Safe to be used by two threads A and B at different times” 相对比较清楚的诠释了 [Send 的定义](https://doc.rust-lang.org/nomicon/send-and-sync.html#send-and-sync)，同时它也没有忽略原来的内容，而只是将其封装起来。
通过转移所有权的方式来实现线程对数据的安全访问是一种很好的方式，也是 Rust 在并发编程里面的基石。需要注意的是，纵使是 Arc\<Mutex\<T\>\> 仍旧工作在这个原则之下，虽然看上去好像有多个值的所有者，并且都可以去修改这个值，但实际上同时只有一个线程可以操作这个值，它仍旧满足 Rust 所有权的基本原则。
