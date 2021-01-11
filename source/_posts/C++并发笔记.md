---
title: C++并发笔记
date: 2021-1-4 10:27:45
tags:
  - C++
excerpt: 《C++ Concurrency in Action》笔记
---

# C++ Concurrency in Action

## Managing threads

### Basic thread

`std::thread`可以使用函数和callable对象创建

<details>
  <summary>code</summary>

```cpp
#include <iostream>
#include <thread>
#include <mutex>

using namespace std;
mutex print_lock;

void hello_func()
{
    lock_guard<mutex> lg{print_lock};
    cout << "hello function: " << std::this_thread::get_id() << endl;
}

class hello_class
{
public:
    void operator()()
    {
        lock_guard<mutex> lg{print_lock};
        cout << "hello callable: " << std::this_thread::get_id() << endl;
    }
};

int main()
{
    std::thread thread_func(hello_func);
    hello_class c;
    std::thread thread_class(c);
    std::thread thread_lambda([] {
        lock_guard<mutex> lg{print_lock};
        cout << "hello lambda: " << std::this_thread::get_id() << endl;
    });
    {
        lock_guard<mutex> lg{print_lock};
        cout << "main: " << std::this_thread::get_id() << endl;
    }
    thread_func.join();
    thread_class.join();
    thread_lambda.join();
    return 0;
}
```

</details>

使用`detach`分离线程和当前线程, 使用`join`等待线程完成, 如果在调用`join`时线程是非`joinable`的就会出现异常, 首先要判断是否`joinable`

**RAII(Resource Acquisition Is Initialization)**方式管理线程

C++构造函数获取资源, 析构函数释放资源, 利用C++对象必定会析构的特征

<details>
  <summary>code</summary>

```cpp
#include <thread>
#include <chrono>
#include <iostream>

//using namespace std;
using namespace std::chrono_literals;

class thread_guard
{
    std::thread &t;
public:
    explicit thread_guard(std::thread &t_) : t(t_) {}
    ~thread_guard() {
        if (t.joinable()) {
            t.join();
        }
    }
    thread_guard(thread_guard const &) = delete;
    thread_guard &operator=(thread_guard const &)=delete;
};

void func(int n) {
    for (int i = 0; i < n; ++i) {
        std::this_thread::sleep_for(1s);
        std::cout << "sleep for 1s" << std::endl;
    }
}

int main()
{
    std::thread t1(func, 3);
    thread_guard guard(t1);
    return 0;
}
```

</details>

`std::thread`对象在`detach`后就进入后台运行, 没有任何手段获取该对象的引用, 也不能`join`, 该对象从此由C++ Runtime Library管理. 在Unix中分离的线程成为*daemon threads*, 同样的进程称为*daemon process*, 代表在后台运行没有显示的用户界面. 通常这种后台线程用于监视文件系统, 清理缓存中无用的记录, 优化数据结构. 同时也用于验证在fire and forget(收发隔离系统)任务中

只有在`joinable()`为真时才能`detach()`

例子:

<details>
  <summary>code</summary>

```cpp
void edit_document(std::string const& filename)
{
	open_document_and_display_gui(filename);
	while(!done_editing())
	{
		user_command cmd=get_user_input();
		if(cmd.type==open_new_document)
		{
			std::string const new_name=get_filename_from_user();
			std::thread t(edit_document,new_name);
			t.detach();
		}
		else
		{
			process_user_input(cmd);
		}
	}
}
```
</details>

在新建文件时创建新的线程, 后台处理文本编辑命令, command pattern

### Passing arguments to a thread function

传入线程的参数是引用和指针时要考虑变量的生存周期, 但有时候线程需要更新数据就要传输引用进去

就算函数声明的是引用, 在构造线程传参时依旧会变成拷贝, 例子:

<details>
  <summary>code</summary>

```cpp
#include <thread>
#include <string>
#include <iostream>

using namespace std;

struct people
{
    int age;
    string name;
};

void happy_birthday(people &p)
{
    cout  << p.name << " is " << p.age++
    << " years old, next year will be " << p.age << endl;
}

int main()
{
    people p{18, "Bob"};
    std::thread t(happy_birthday, p);
    t.join();
    cout << "age :" << p.age << endl;
}

//结果
Bob is 18 years old, next year will be 19
age :18
```

</details>

### Transferring ownership of a thread

**传递引用**

<mark>为了传引用必须使用`std::ref(p)`才能成功</mark>

**绑定成员函数**

```cpp
struct cat
{ void sleep() {}; }
cat c;
std::thread t(&cat::sleep, &c);
```

第一个参数是类的指针

**传递右值(变量转移)**

```cpp
void process_big_object(std::unique_ptr<big_object>);
std::unique_ptr<big_object> p(new big_object);
p->prepare_data(42);
std::thread t(process_big_object,std::move(p));
```

thread是moveable的, unique_ptr, 和ifstream也是, 可以用std::move, 可以用thread::move

<details>
  <summary>code</summary>

```cpp
#include <thread>
#include <iostream>
#include <chrono>

using namespace std;
using namespace std::chrono_literals;

void some_function1()
{
    for (int i = 0; i < 100; ++i) {
        this_thread::sleep_for(1s);
        cout << "func 1, this id: " << this_thread::get_id() << endl;
    }
}

void some_function2()
{
    for (int i = 0; i < 100; ++i) {
        this_thread::sleep_for(1s);
        cout << "func 2, this id: " << this_thread::get_id() << endl;
    }
}

int main()
{
    thread t1(some_function1);
    this_thread::sleep_for(3s);
    thread t2 = std::move(t1);
    this_thread::sleep_for(3s);
    t1 = thread(some_function2);
    this_thread::sleep_for(3s);
    thread t3 = std::move(t2);
    this_thread::sleep_for(3s);
    t1 = std::move(t3);     // std::terminate()会发生
}
```

</details>

在已经运行函数的线程中用另外一个线程赋值会崩溃, move线程不会改变线程id

### Choosing the number of threads at runtime

使用`std::thread::hardware_concurrency()`可以得到cpu并发能力

`std::accumulate`累加算法, `std::distance`迭代器直接的距离

### Identifying threads

使用`std::thread::id`来标识线程, 对于当前线程使用`std::this_thread::get_id()`

## Sharing data between threads

涉及线程间共享数据, 用锁保护数据, 其他保护共享数据的方法

### Problems with sharing data between threads

双向链表的移除, 先修改两边的指针, 然后再删除, 然而修改指针分两步, 左边和右边, 与此同时如果线程进行读取将会出问题

解决办法有两种:

- 采用数据结构保护机制
- 采用无锁编程, memory model需要设计
- 采用STM(software transactional memory), 参考[事务内存](https://zhuanlan.zhihu.com/p/151425608)

### Protecting shared data with mutexes

不推荐直接使用`std::mutex`, 而是使用`std::lock_guard`

可以把mutex写在结构体里面, 但是要确定结构体里面的正确的数据被保护起来了

原则: **不要在mutex保护范围外传递被保护数据的引用和指针, 或者通过函数返回, 或者在可见的额外内存存储, 或者把他们当变量传递给其他用户提供的函数**

设计接口问题:

对于总体需要进行保护的数据, 比如双向链表, 单保护被删除的对象不行, 其左右两边的对象也需要加锁;

对于类似stack这种数据, 其`empty()`,`top()`和`size()`方法虽然调用的时候是正确的, 但是在实际使用的时候, 一旦返回完这两个函数的结果, 另一个线程对数据进行改变, 那么这两个结果可能就失去了正确性, 就算在数据结构内部增加mutex防止读写问题也没用

对于大量数据的操作, 比如`stack<vector<int>>`, 如果stack在pop的时候由于申请新的内存失败, 然后没能把数据拷贝出去就删除了数据, 那么就会造成数据的永久丢失, 所以stack的接口设计了`top`和`pop`, 把数据拷贝和数据删除分开, 防止出现该问题

解决方案1:

传递一个引用到pop里面去, 确保内存申请操作在pop函数里面得到确认

```cpp
std::vector<int> result;
some_stack.pop(result);
```

缺点是需要额外的构造变量, 而且有些对象没有默认构造只有拷贝构造或者构造的时候就初始化, 不支持默认构造

解决方案2:

使用移动构造, 或者无异常构造, 缺点就是有些用户定义的对象不可避免在构造函数抛出异常

解决方案3:

返回被`pop`的对象的指针, 缺点就是对于int这种数据返回指针不划算, 同时返回的指针容易产生内存泄露, 也可以用共享指针来管理内存

实现代码:

<details>
  <summary>code</summary>

```cpp
class empty_stack : public std::exception
{
public:
    empty_stack() noexcept
            : exception("empty stack", 1)
    {}
};

template<typename T>
class threadsafe_stack
{
private:
    std::stack<T> data;
    mutable std::mutex m;
public:
    threadsafe_stack() = default;

    threadsafe_stack(const threadsafe_stack &other)
    {
        std::lock_guard<std::mutex> lock(other.m);
        data = other.data;
    }

    void push(T new_value)
    {
        std::lock_guard<std::mutex> lock(m);
        data.push(new_value);
    }

    std::shared_ptr<T> pop()
    {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty()) throw empty_stack();
        std::shared_ptr<T> const res(std::make_shared<T>(data.top()));
        data.pop();
        return res;
    }

    void pop(T &value)
    {
        std::lock_guard<std::mutex> lock(m);
        if (data.empty()) throw empty_stack();
        value = data.top();
        data.pop();
    }

    bool empty()
    {
        std::lock_guard<std::mutex> lock(m);
        return data.empty();
    }
};
```

</details>

fine-grained locking scheme: 细粒度锁方案

如果采用细粒度锁会增加复杂度, 而且多个锁会有可能造成**死锁**

死锁的解决办法是每次都按照相同的顺序给两个mutex上锁, C++标准库中有`std::lock`可以同时对两个锁加锁而不会产生死锁

<details>
  <summary>code</summary>

```cpp
class some_big_object;
void swap(some_big_object &lhs, some_big_object &rhs);

class X
{
private:
    some_big_object some_details;
    std::mutex m;
public:
    X(some_big_object const &d) : some_detail(sd) {}
    friend void swap(X &lhs, X&rhs)
    {
        if (&lhs == &rhs) return;
        std::lock (lhs.m, rhs.m);
        std::lock_guard<std::mutex> lock_a(lhs.m, std::adopt_lock);
        std::lock_guard<std::mutex> lock_b(rhs.m, std::adopt_lock);
        swap(lhs.some_detail, rhs.some_detail);
    }
};
```

</details>

声明`std::adopt_lock`的含义是表示构造函数第一个参数中的锁已经锁上了

