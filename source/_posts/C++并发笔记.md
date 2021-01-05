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



