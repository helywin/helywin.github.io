---
title: C++原子操作中的内存顺序
date: 2021-2-1 19:43:23
tags:
  - C++
excerpt: C++原子操作中的内存顺序
---

# C++原子操作中的内存顺序

头文件 `<atomic>`

C++11形式

```cpp
typedef enum memory_order {
    memory_order_relaxed,
    memory_order_consume,
    memory_order_acquire,
    memory_order_release,
    memory_order_acq_rel,
    memory_order_seq_cst
} memory_order;
```

C++20形式

```cpp
enum class memory_order : /*unspecified*/ {

    relaxed, consume, acquire, release, acq_rel, seq_cst
};
inline constexpr memory_order memory_order_relaxed = memory_order::relaxed;
inline constexpr memory_order memory_order_consume = memory_order::consume;
inline constexpr memory_order memory_order_acquire = memory_order::acquire;
inline constexpr memory_order memory_order_release = memory_order::release;
inline constexpr memory_order memory_order_acq_rel = memory_order::acq_rel;
inline constexpr memory_order memory_order_seq_cst = memory_order::seq_cst;
```



`std::memory_order`指定了怎样访问内存，包括常规的、非原子的内存访问、如何围绕原子操作排序。在多核系统中如果没有任何约束，当多个线程同时读写一些变量，一个线程可能以一种顺序观察值的改变不同于其他写入这些值的顺序。确实，这个很明显的顺序改变可能存在于在不同的读取线程之间。一些相似的影响甚至可能发生在单处理器系统，由于编译器内存模型允许的转换。

库里面提供的默认的原子操作行为是为顺序一致性的。该默认行为的可能会有性能损失，但是库中的原子操作可以给定额外的`std::memory_order`变量指定额外的约束条件。在原子性之外，编译器和处理器还必须强制该操作。

## 常量

| 值                   | 解释                                                         |
| -------------------- | ------------------------------------------------------------ |
| memory_order_relaxed | 宽松操作：没有对于其他读写强加的同步或者顺序，只有当前操作的原子性需要得到保证 |
| memory_order_consume | 一个使用这个内存顺序的加载操作在被影响的内存位置被称为消费操作：在该加载操作**之前**当前线程不允许有依赖于该值（要加载的）的读或者写的记录——当前线程中依赖于当前加载的该值的读或写不能被重排到此加载前。在其他释放相同原子变量的线程的对数据依赖变量的写操作对当前线程可见。在大多数情况下，这只会影响编译器的优化 |
| memory_order_acquire | 一个使用这个内存顺序的加载操作在被影响的内存位置被称为获得操作：在该加载操作**之前**当前线程不允许有其他的读或者写的记录——当前线程中读或写不能被重排到此加载前。在其他释放相同原子变量的线程的写操作对当前线程可见。 |
| memory_order_release | 一个使用这个内存顺序的存储操作被称为释放操作：在该存储操作**之后**当前线程不允许有其他的读或者写的记录——当前线程中的读或写不能被重排到此存储后。当前线程的所有写都对其他获取了相同原子变量的线程可见。 |
| memory_order_acq_rel | 一个使用这个内存顺序的读-修改-写操作同时是获得操作和释放操作：在该存储操作前后不允许内存的读写——当前线程的读或写内存不能被重排到此存储前或后。所有释放了相同原子变量的线程的所有的写在修改之前都可见，同时其他获取了相同原子变量的线程的修改都可见 |
| memory_order_seq_cst | 一个使用这个内存顺序的加载操作使用获得操作，存储使用释放操作，读-修改-写操作同时使用获得操作和释放操作，再加上存在一个单独全序，其中所有线程以同一顺序观测到所有修改 |

## 正式描述

线程间的同步和内存顺序界定了表达式的求值和副作用在不同线程中的执行顺序，术语定义如下：

### 先序于（Sequenced-before）

在同一线程中，求值A可以先序于求值B，如[求值顺序](https://zh.cppreference.com/w/cpp/language/eval_order)中所描述。

>“按顺序早于 (sequenced-before)”是同一线程中的求值之间的非对称的、传递的对偶关系。
>
>-  若 A 按顺序早于 B，则 A 的求值将在 B 的求值开始前完成。
>-  若 A 不按顺序早于 B 而 B 按顺序早于 A，则 B 的求值将在 A 的求值开始前完成。
>-  若 A 不按顺序早于 B 而 B 不按顺序早于 A，则存在两种可能：
>  -  A 与 B 的求值是无顺序 (unsequenced) 的：它们能以任何顺序进行，并可能重叠（在同一执行线程内，编译器可以将组成 A 与 B 的 CPU 指令交错）
>  -  A 与 B 的求值是顺序不确定 (indeterminately sequenced) 的：它们可以任意顺序进行但不可重叠，A 在 B 前完成，或 B 在 A 前完成。下次求值相同表达式时顺序可以相反。

### 携带依赖（Carries dependency）

在同一线程中，求值A先序于求值B可能也把依赖带进了B（就是说B依赖于A），如果满足以下条件：

1. A用作为B的操作数，除了：

   a. 如果B调用`std::kill_dependency`

   b. 如果A是内建`&&`, `||`, `?:`或`,`操作符的左操作数，因为前两个操作符右边的求值可能会省去，后面的一个是条件一个是逗号操作符，只是单纯的连接多个运算

2. A写入标量对象M，B从M读取
3. A携带依赖到X，X携带依赖到B（依赖链条）

`std::kill_dependency`用于告诉编译器使用`std::memory_order_consume`原子加载的变量的依赖树不在`std::kill_dependency`的返回值扩展，就是变量不携带依赖到返回值

用于避免当依赖链条离开函数作用域（同时函数没有加上`[[carries_dependency]]`属性）不必要的`std::memory_order_acquire`围栏

```cpp
struct foo { int* a; int* b; };
std::atomic<struct foo*> foo_head[10];
int foo_array[10][10];
 
// consume operation starts a dependency chain, which escapes this function
[[carries_dependency]] struct foo* f(int i) {
    return foo_head[i].load(memory_order_consume);
}
 
// the dependency chain enters this function through the right parameter
// and is killed before the function ends (so no extra acquire operation takes place)
int g(int* x, int* y [[carries_dependency]]) {
    return std::kill_dependency(foo_array[*x][*y]);
}
```

### 修改顺序（Modification order）

对于特定的原子变量的所有修改，发生于对于特定原子变量的全序

下列四种需求对于所有原子操作都得到保证：

1. **写-写连贯**：若修改某原子对象 M 的求值 A （写操作）*先发生于*修改 M 的求值 B ，则 A 在 M 的*修改顺序*中早于 B 出现。
2. **读-读连贯**：若某原子对象 M 的值计算 A （读操作）*先发生于*对 M 的值计算 B ，且 A 的值来自对 M 的写操作 X ，则 B 的值要么是 X 所存储的值，要么是在 M 的*修改顺序*中后于 X 出现的 M 上的副效应 Y 所存储的值。
3. **读-写连贯**：若某原子对象 M 的值计算 A （读操作）*先发生于* M 上的操作 B （写操作），则 A 的值来自 M 的*修改顺序*中早于 B 出现的副效应 X （写操作）。
4. **写-读连贯**：若原子对象 M 上的副效应 X （写操作）*先发生于* M 的值计算 B （读操作），则求值 B 应从 X 或从 M 的*修改顺序*中后随 X 的副效应 Y 取得其值。

### 释放序列（Release sequence）

在原子对象 M 上执行一次*释放操作* A 之后， M 的修改顺序的最长连续子序列由下列内容组成

1. 由执行 A 的同一线程所执行的写操作
2. 任何线程对 M 的原子的读-修改-写操作

被称为*以 A 为首的释放序列*。

### 依赖先序于（Dependency-ordered before）

线程之间，如果下列任一条件满足则求值A依赖先序于求值B：

1. A对原子变量M执行释放操作，在另外线程中B对相同的原子变量M执行消费操作，同时B读取A（所引领的释放序列的任何部分 (C++20 前)）写的值。
2. A依赖先序于X同时X携带依赖到B（依赖链条）

###  线程间先发生于（Inter-thread happens-before）

线程之间，如果下列任一条件满足则求值A线程间发生于求值B：

1. A同步于B
2. A依赖先序于B
3. A同步于X，X先序于B
4. A先序于X，X线程间发生于B
5. A线程间发生于X，X线程间发生于B

### 先发生于（Happens-before）

不管线程，如果下列任一条件满足则求值A先发生于B：

1. A先序于B
2. A线程间先发生于B

实现需要确保先发生于关系是无环的，有必要需要通过额外的同步确保（仅当消费操作参与才是必要的）。

如果一个求值修改了一块内存，其他求值读取或修改同一块内存，如果至少有一个求值不是原子操作，那么程序的行为就是未定义的（程序存在数据竞争），除非在两个求值之间存在先发生于的关系。

### 简单先发生于（C++20）

不管线程，如果下列任一条件满足则求值A先发生于B：

1. A先序于B
2. A同步于B
3. A先发生于X，X先发生于B

Note: 如果没有消费操作，简单先发生于和先发生于关系是一样的

### 强先发生于（C++20）

不管线程，如果下列任一条件满足则求值A先发生于B：

C++20之前：

1. A先序于B
2. A同步于B
3. A强先发生于X，X强先发生于B

C++20之后：

1. A先序于B
2. A同步于B，A和B均为序列一致的原子操作
3. A先序于X，X简单先发生于Y，Y先序于B
4. A强先发生于X，X强先发生于B

Note: 非正式的，若 A *强先发生于* B ，则在所有环境中 A 均显得在 B 之前得到求值。

Note: 强先发生于排除消费操作

### 可见副效应（Visible side-effects）

如果以下都为真，那么标量M上的可见副效应A关于标量M上的值计算B是可见的：

1. A先发生于B
2. 对于标量M上不存在可见副效应X，其中A先发生于X，X先发生于B

若副效应 A 相对于值计算 B 可见，则*修改顺序*中，满足 B 不*先发生于*它的对 M 的副效应的最长相接子集，被称为*副效应的可见序列*。（ B 所确定的 M 的值，将是这些副效应之一所存储的值）

注意：线程间同步可归结为避免数据竞争（通过建立先发生于关系），及定义在何种条件下哪些副效应成为可见。

### std::atomic_thread_fence

对于非原子操作或者宽松原子操作建立内存同步顺序，由于通过顺序指导，不需要关联原子操作

例子1：

```cpp
#include <atomic>
#include <string>
#include <thread>
#include <iostream>

//Global
std::string computation(int v) {
    return std::to_string(v);
}

void print(const std::string& str)
{
    std::cout << str << std::endl;
}

std::atomic<int> arr[3] = {-1, -1, -1};
std::string data[1000]; //non-atomic data

// Thread A, compute 3 values
void ThreadA(int v0, int v1, int v2)
{
//assert( 0 <= v0, v1, v2 < 1000 );
    data[v0] = computation(v0);
    data[v1] = computation(v1);
    data[v2] = computation(v2);
    std::atomic_thread_fence(std::memory_order_release);
    std::atomic_store_explicit(&arr[0], v0, std::memory_order_relaxed);
    std::atomic_store_explicit(&arr[1], v1, std::memory_order_relaxed);
    std::atomic_store_explicit(&arr[2], v2, std::memory_order_relaxed);
}

// Thread B, prints between 0 and 3 values already computed.
void ThreadB()
{
    int v0 = std::atomic_load_explicit(&arr[0], std::memory_order_relaxed);
    int v1 = std::atomic_load_explicit(&arr[1], std::memory_order_relaxed);
    int v2 = std::atomic_load_explicit(&arr[2], std::memory_order_relaxed);
    std::atomic_thread_fence(std::memory_order_acquire);
// v0, v1, v2 might turn out to be -1, some or all of them.
// otherwise it is safe to read the non-atomic data because of the fences:
    if (v0 != -1) { print(data[v0]); }
    if (v1 != -1) { print(data[v1]); }
    if (v2 != -1) { print(data[v2]); }
}

int main()
{
    std::thread thread1(ThreadA, 1, 2, 3);
    std::thread thread2(ThreadB);
    thread1.join();
    thread2.join();
    return 0;
}
```

该代码最后的结果是要么三个变量都为-1，要么都为赋值后的值并打印出来

还要一个扫描数组的例子

```cpp
const int num_mailboxes = 32;
std::atomic<int> mailbox_receiver[num_mailboxes];
std::string mailbox_data[num_mailboxes];
 
// The writer threads update non-atomic shared data 
// and then update mailbox_receiver[i] as follows
mailbox_data[i] = ...;
std::atomic_store_explicit(&mailbox_receiver[i], receiver_id, std::memory_order_release);
 
// Reader thread needs to check all mailbox[i], but only needs to sync with one
for (int i = 0; i < num_mailboxes; ++i) {
    if (std::atomic_load_explicit(&mailbox_receiver[i], std::memory_order_relaxed) == my_id) {
        std::atomic_thread_fence(std::memory_order_acquire); // synchronize with just one writer
        do_work( mailbox_data[i] ); // guaranteed to observe everything done in the writer thread before
                    // the atomic_store_explicit()
    }
 }
```

`std::memory_order_release`顺序会确保writer线程中的两个写入是同步的，然后利用`std::atomic_thread_fence`确保遍历的时候两个变量都更新过或都没更新过，而不会出现中间状态

### 消费操作（Consume operation）

带`memory_order_consume`或者更强的原子性加载为消费操作。注意`std::atomic_thread_fence`会比消费操作强加更强的同步需求。

### 获取操作（Acquire operation)

带`memory_order_acquire`或者更强的原子性加载为获取操作。Mutex的`lock()`方法也是一个获取操作。注意`std::atomic_thread_fence`会比消费操作强加更强的同步需求。

### 释放操作（Release operation）

带`memory_order_release`或者更强的原子性存储为释放操作。Mutex的`unlock()`方法也是一个释放操作。`std::atomic_thread_fence`会比消费操作强加更强的同步需求。

## 解释

### 宽松顺序（Relaxed ordering）

标记为`memory_order_relaxed`的原子操作不是同步操作；对于并发内存的访问没有强加顺序。只保证原子性和修改一致性。

例子：

```cpp
// Thread 1:
r1 = y.load(std::memory_order_relaxed); // A
x.store(r1, std::memory_order_relaxed); // B
// Thread 2:
r2 = x.load(std::memory_order_relaxed); // C 
y.store(42, std::memory_order_relaxed); // D
```

允许产生`r1 == 42 && r2 == 42`的结果，按照下面顺序排列

```cpp
y.store(42, std::memory_order_relaxed); // D
r1 = y.load(std::memory_order_relaxed); // A
x.store(r1, std::memory_order_relaxed); // B
r2 = x.load(std::memory_order_relaxed); // C 
```

因为即使A在线程1先序于B，C在线程2先序于D，并没有阻止在修改y的顺序的时候D发生在A前面，修改x的顺序的时候B发生在C前面。D在y上的副效应对于线程1中A加载是可见的，同样B在x上的副效应对于线程2中的C加载是可见的。尤其在线程2中D在C前完成这更加可能发生，或者是由于编译器重新排序或者在运行时。

即使是使用宽松顺序，凭空（out-of-thin-air）值也不允许循环依赖自己的计算，例如

```cpp
// Thread 1;
r1 = y.load(std::memory_order_relaxed);
if (r1 == 42) x.store(r1, std::memory_order_relaxed);
// Thread 2;
r2 = x.load(std::memory_order_relaxed);
if (r2 == 42) y.store(42, std::memory_order_relaxed);

```

它不允许产生`r1 == r2 == 42`因为42存储到y只有在x存储了42，循环依赖了存储42到y。直到C++14，这个都被允许，但是不推荐真的去用

通常宽松顺序用在计数变量，比如`std::shared_ptr`里面的引用计数器，因为它只要求原子性，但不需要顺序和同步（**但是减少计数器值需要在析构函数用到获取释放(acquire-release)同步机制**）

```cpp
#include <vector>
#include <iostream>
#include <thread>
#include <atomic>
 
std::atomic<int> cnt = {0};
 
void f()
{
    for (int n = 0; n < 1000; ++n) {
        cnt.fetch_add(1, std::memory_order_relaxed);
    }
}
 
int main()
{
    std::vector<std::thread> v;
    for (int n = 0; n < 10; ++n) {
        v.emplace_back(f);
    }
    for (auto& t : v) {
        t.join();
    }
    std::cout << "Final counter value is " << cnt << '\n';
}
```

输出：

```
Final counter value is 10000
```

### 释放获得顺序（Release-Acquire ordering）

如果在线程A存储的原子变量被标记为`memory_order_release`同时同一个原子变量在线程B中被使用`memory_order_acquire`加载，所有在线程A视角看来的先发生于原子存储的写入（非原子的或宽松原子的），都成为线程B的可见副作用。一旦原子加载完成，线程B能够保证观察到线程A中的所有写入。

If an atomic store in thread A is tagged memory_order_release and an atomic load in thread B from the same variable is tagged memory_order_acquire, all memory writes (non-atomic and relaxed atomic) that happened-before the atomic store from the point of view of thread A, become visible side-effects in thread B. That is, once the atomic load is completed, thread B is guaranteed to see everything thread A wrote to memory. 

若线程 A 中的一个原子存储带标签 memory_order_release ，而线程 B 中来自同一变量的原子加载带标签 memory_order_acquire ，则从线程 A 的视角先发生于原子存储的所有内存写入（非原子及宽松原子的），在线程 B 中成为可见副效应，即一旦原子加载完成，则保证线程 B 能观察到线程 A 写入内存的所有内容。 

该同步只发生在相同原子变量的释放和获取。其他线程能见到与被同步线程的一者或两者相异的内存访问顺序。 

在强顺序系统（ x86 、 SPARC TSO 、 IBM 主框架）上，释放获得顺序对于多数操作是自动进行的。无需为此同步模式添加额外的 CPU 指令，只有某些编译器优化受影响（例如，编译器被禁止将非原子存储移到原子存储-释放后，或将非原子加载移到原子加载-获得前）。在弱顺序系统（ ARM 、 Itanium 、 Power PC ）上，必须使用特别的 CPU 加载或内存栅栏指令。 

互斥锁（例如 std::mutex 或原子自旋锁）是释放获得同步的例子：线程 A 释放锁而线程 B 获得它时，发生于线程 A 环境的临界区（释放之前）中的所有事件，必须对于执行同一临界区的线程 B （获得之后）可见。 

```cpp
#include <thread>
#include <atomic>
#include <cassert>
#include <string>

std::atomic<std::string *> ptr;
int data;

void producer()
{
    std::string *p = new std::string("Hello");
    data = 42;
    ptr.store(p, std::memory_order_release);
}

void consumer()
{
    std::string *p2;
    while (!(p2 = ptr.load(std::memory_order_acquire)));
    assert(*p2 == "Hello"); // never fires
    assert(data == 42); // never fires
}

int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join();
    t2.join();
}
```

下例演示三个线程间传递性的释放获得顺序

```cpp
#include <thread>
#include <atomic>
#include <cassert>
#include <vector>

std::vector<int> data;
std::atomic<int> flag = {0};

void thread_1()
{
    data.push_back(42);
    flag.store(1, std::memory_order_release);
}

void thread_2()
{
    int expected = 1;
    while (!flag.compare_exchange_strong(expected, 2, std::memory_order_acq_rel)) {
        expected = 1;
    }
}

void thread_3()
{
    while (flag.load(std::memory_order_acquire) < 2);
    assert(data.at(0) == 42); // 决不出错
}

int main()
{
    std::thread a(thread_1);
    std::thread b(thread_2);
    std::thread c(thread_3);
    a.join();
    b.join();
    c.join();
}
```

**该顺序保证了在原子操作之前（代码前面）的其他变量（非原子或者宽松操作）的写入一定会发生在该原子操作的写入之前。**

### 释放消费顺序（Release-Consume ordering）

如果原子变量存储在线程A中且标为`memory_order_release`，同时另外一个同样的变量在线程B中读取存储的值标为`memory_order_consume`，所有的先发生于线程A视角的原子存储的写（非原子或者宽松原子），变成了线程B中这些操作的可见副作用，进入加载操作的携带依赖，一旦原子加载完成，线程B中从加载获取的值的这些操作和函数会被保证可以观察到线程A中所有的内存的写入。

If an atomic store in thread A is tagged memory_order_release and an atomic load in thread B from the same variable is tagged memory_order_acquire, all memory writes (non-atomic and relaxed atomic) that *happened-before* the atomic store from the point of view of thread A, become *visible side-effects* in thread B. That is, once the atomic load is completed, thread B is guaranteed to see everything thread A wrote to memory.

仅仅当线程之间释放和消费相同的原子变量同步才会发生。其他能够相较于这两个同步线程获取到不同的内存顺序。

在所有的除去DEC Alpha的主流CPU中，依赖顺序是原子性的，为了这个同步模式没有额外的CPU指令需要发出，仅仅特定的编译器优化受到影响（例如，编译器在涉及依赖链的对象时被禁止随机性加载）。

通常的对于这个顺序的用例参与读访问而很少写的并发数据结构（路由表，配置，安全策略，防火墙规则等）和使用指针介导（pointer-mediated）发布-订阅场景，当发布者发布一个指针通过消费者能够获取的信息：没有必要让生产者写的所有内存对于消费者可见（可能在弱顺序架构可能是昂贵的操作），这种场景的一个例子是rcu_dereference（Linux 中的RCU）机制。

可以参考`std::kill_dependency`和`[[carries_dependency]]`来细粒度的控制依赖链

这个指针介入的依赖顺序同步的例子：整数的数据和字符串指针没有数据依赖关系，所以它的值在消费者中是未定义的。

```cpp
#include <thread>
#include <atomic>
#include <cassert>
#include <string>
 
std::atomic<std::string*> ptr;
int data;
 
void producer()
{
    std::string* p  = new std::string("Hello");
    data = 42;
    ptr.store(p, std::memory_order_release);
}
 
void consumer()
{
    std::string* p2;
    while (!(p2 = ptr.load(std::memory_order_consume)))
        ;
    assert(*p2 == "Hello"); // 绝无出错： *p2 从 ptr 携带依赖
    assert(data == 42); // 可能也可能不会出错： data 不从 ptr 携带依赖
}
 
int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join(); t2.join();
}
```

**该顺序保证了在原子操作之前（代码前面）的其他变量（非原子或者宽松操作）其中携带该原子变量依赖变量的写入会发生在该原子操作的写入之前。**



### 序列一致顺序（Sequentially-consistent ordering）

标记为`memory_order_seq_cst`不仅仅和释放获取顺序一样（所有在一个线程中先发生于存储操作的在另外进行加载的操作的线程成为可见副作用），还对所有带此标签的内存操作建立单独全序。

意思是当仅用来读时和`memory_order_acquire`一致，写时和`memory_order_release`一致，读写都有时和`memory_order_acq_rel`一致。

单独全序，也就是所有的线程会观察到一致的内存修改

正式的**（C++20前）**，

每个从原子变量M的内存加载操作`memory_order_seq_cst`的操作B，观测到如下之一：

- 修改M的上一个操作A，A在单独全序中出现在B之前
- 或者，如果存在这种操作A，则操作B可能观测到非`memory_order_seq_cst`且非先发生于A的对于M的一些修改结果
- 或者，如果不存在这种操作A，则操作B可能观测到非`memory_order_seq_cst`的M的一些不相关的修改

如果是`memory_order_seq_cst`的`std::atomic_thread_fence`操作X先序于B，那么B观测到如下之一：

- 在单独全序中修改M的上一个`memory_order_seq_cst`先出现于X操作
- 在M的修改顺序中后出现于它的某些 M 的无关联修改

对于M的一对原子操作A和B，A写B读M的值，如果有两个`memory_order_seq_cst`的`std::atomic_thread_fences`X和Y，如果A先序于X，Y先序于B，X在单独全序中先出现于Y，则B观察到二者之一：

- A的效应
- 某些在 M 的修改顺序中后出现于 A 的无关联修改

对于M的一对原子操作A和B，若符合下列条件之一，则M的修改顺序中B后发生于A

- 存在一个`memory_order_seq_cst`的`std::atomic_thread_fence`X，它满足A先序于X，且X在单独全序中先出现于B
- 或者，存在一个`memory_order_seq_cst`的`std::atomic_thread_fence`Y，满足Y先序于B且A在单独全序中先出现于B
- 或者，存在两个`memory_order_seq_cst`的`std::atomic_thread_fence`X和Y，A先序于X，Y先序于B，X在单独全序中先出现于Y

注意这表明：

1. 只要不带`memory_order_seq_cst`标签的原子操作进入局面，则立即丧失序列一致性
2. 序列一致栅栏（sequentially-consistent fences）仅为栅栏自身建立全序，而不为通常情况下的原子操作建立（先序于不是跨线程关系，不同于先发生于）

正式的**（C++20）**，

某原子对象 M 上的原子操作连贯先序于 M 上的另一原子操作 B ，若下列任一为真： 

1. A是修改，B读取A存储的值
2. 在M的修改顺序中A先于B
3. A读取原子修改操作M存储的值，X在修改顺序中先于B，同时A和B不是相同的读-修改-写操作
4. A 连贯先序于X，而X连贯先序于B

所有的`memory_order_seq_cst`操作，包括围栏（fences）上，有单独全序S，满足以下制约条件：

1. 如果A和B都是`memory_order_seq_cst`操作，A强先发生于B，那么A在S中前于B
2. 对于对象M上的每对原子操作A和B，其中A连贯先序于B：
    a. 如果A和B都是`memory_order_seq_cst`操作，那么A在S中前于B
    b. 如果A是`memory_order_seq_cst`操作，B先发生于`memory_order_seq_cst`围栏Y，那么A在S中前于Y
    c. 如果X是`memory_order_seq_cst`围栏且先发生于A，B是`memory_order_seq_cst`操作，那么X在S中前于B
    d. 如果X是`memory_order_seq_cst`围栏且先发生于A，B先发生于`memory_order_seq_cst`围栏Y，那么X在S中前于Y

这些正式的定义确保了：

1. 单独全序与任何原子对象的修改顺序一致
2. 一个`memory_order_seq_cst`加载要么从上一个`memory_order_seq_cst`修改获取值，要么从一些不是先发生于`memory_order_seq_cst`修改的非`memory_order_seq_cst`修改中获取值

单独全序可能与先发生于不一致。这允许在CPU上更加有效率的实现`memory_order_acquire`和`memory_order_release`。当`memory_order_acquire`和`memory_order_release`和`memory_order_seq_cst`混合时能产生惊异的结果。

例如，x，y的初始值为0，

```cpp
x = 0;
y = 0;
// Thread 1:
x.store(1, std::memory_order_seq_cst); // A
y.store(1, std::memory_order_release); // B
// Thread 2:
r1 = y.fetch_add(1, std::memory_order_seq_cst); // C
r2 = y.load(std::memory_order_relaxed); // D
// Thread 3:
y.store(3, std::memory_order_seq_cst); // E
r3 = x.load(std::memory_order_seq_cst); // F
```

允许产生`r1 == 1 && r2 == 3 && r3 == 0`的结果，当A先发生于C，但是在`memory_order_seq_cst`的单独全序C-E-F-A中C前于的A。

注意：

1. 一旦不带 memory_order_seq_cst 标签的原子操作进入局面，程序的序列一致保证就会立即丧失
2. 多数情况下， memory_order_seq_cst 原子操作相对于同一线程所进行的其他原子操作可重排 

在多生产者多消费者的情形中序列顺序是必要的，当所有的消费者必须以相同的顺序观察生产者中发生的动作。

全序顺序在所有的多核系统中需要完全的内存栅栏CPU指令。这可能会成为性能瓶颈因为它强制受影响的内存访问传播到每个核。

这个例子演示了序列顺序是必要的情形。其他任何顺序都会触发断言因为这可能会导致线程c和d用相反的顺序观察原子变量x和y的改变

```cpp
#include <thread>
#include <atomic>
#include <cassert>
#include <iostream>


std::atomic<bool> x = {false};
std::atomic<bool> y = {false};
std::atomic<int> z = {0};

void write_x()
{
    x.store(true, std::memory_order_seq_cst);
}

void write_y()
{
    y.store(true, std::memory_order_seq_cst);
}

void read_x_then_y()
{
    while (!x.load(std::memory_order_seq_cst));
    if (y.load(std::memory_order_seq_cst)) {
        ++z;
    }
}

void read_y_then_x()
{
    while (!y.load(std::memory_order_seq_cst));
    if (x.load(std::memory_order_seq_cst)) {
        ++z;
    }
}

int main()
{
    std::thread a(write_x);
    std::thread b(write_y);
    std::thread c(read_x_then_y);
    std::thread d(read_y_then_x);
    a.join(); b.join(); c.join(); d.join();
    assert(z.load() != 0);  // 决不发生
}
```



# 参考链接

> https://en.cppreference.com/w/cpp/atomic/memory_order
> https://zh.cppreference.com/w/cpp/atomic/memory_order
> https://en.cppreference.com/w/cpp/atomic/atomic_thread_fence

注：cppreference中文翻译中有很多错误，建议对照英文看