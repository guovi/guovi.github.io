+++
authors = ["lostone"]
title = "手写shared_ptr"
date = "2023-10-05"
description = "shared_ptr简单实现以及相关面试问题"
tags = [
    "C++",
    "手写",
    "智能指针",
    "shared_ptr"
]
categories = [
    "code",
    "手写实现"
]
series = ["C++面试系列"]
+++

### 代码实现
```cpp
#include <iostream>

using namespace std;

template<typename T> 
class SharedPtr {
private:
    size_t* _count;
    T* _ptr;
public:
    //  构造函数
    SharedPtr(): _ptr(nullptr), _count(new size_t(0)) {}
    SharedPtr(T* ptr): _ptr(ptr), _count(new size_t) {
        *_count = 1;
    }
    //  析构函数
    ~SharedPtr() {
        if (_count) {
            -- (*_count);
        }
        if (!_count || *_count == 0) {  // _count为nullptr或0时delete
            delete _ptr;
            delete _count;
            _ptr = nullptr;
            _count = nullptr;
        }
    }
    //  拷贝构造
    SharedPtr(const SharedPtr& ptr) {
        cout << "copy " << endl;
        _count = ptr._count;
        _ptr = ptr._ptr;
        ++(*_count);
    }
    //  拷贝赋值
    SharedPtr& operator=(const SharedPtr& ptr) {
        cout << "copy =" << endl;
        if (_ptr == ptr._ptr) {  // 检查自我拷贝
            return *this;
        }
        (*_count)--;  // 处理旧对象
        if (*_count == 0) {
            delete _ptr;
            delete _count;
            _ptr = nullptr;
            _count = nullptr;
        }
        _ptr = ptr._ptr;
        _count = ptr._count;
        (*ptr._count)++;
        return *this;
    }
    //  移动构造
    SharedPtr(SharedPtr&& rhs): _ptr(std::move(rhs._ptr)), _count(rhs._count) {
        cout << "move " << endl;
        rhs._ptr = nullptr;
        rhs._count = nullptr;
    }
    //  移动赋值
    SharedPtr& operator=(SharedPtr&& rhs) {
        cout << "move =" << endl;
        std::swap(_ptr, rhs._ptr);
        std::swap(_count, rhs._count);
        return *this;
    }
    T& operator*() const {
        return *_ptr;
    }
    T* operator->() const {
        return _ptr;
    }
    operator bool() {
        return _ptr != nullptr;
    }
    T* get() {
        return _ptr;
    }
    size_t use_count() {
        cout << "use_count: ";
        return *_count;
    }
    bool unique() {
        return *_count == 1;
    }
    void swap(SharedPtr& rhs) {
        std::swap(*this, rhs); 
    }

};

class Test {
    int a = 23;
    string b = "dfghjkl";
public:
    Test() { std::cout << "Test::Test()\n"; }
    // Note: non-virtual destructor is OK here
    ~Test() { std::cout << "Test::~Test()\n"; }
};

int main() {
    SharedPtr<Test> sp0;
    cout << "sp0: " << sp0.use_count() << endl;
    {
        SharedPtr<Test> sp1(new Test());
        cout << "sp1: " << sp1.use_count() << endl;
        SharedPtr<Test> sp2(sp1);
        cout << "sp2: " << sp2.use_count() << endl;
        sp0 = sp2;
        cout << "sp0: " << sp0.use_count() << ", sp2: " << sp2.use_count() << endl;

        SharedPtr<Test> sp3(new Test());
        cout << "s3: " << sp3.use_count() << endl;
        SharedPtr<Test> sp4(move(sp3));  // 移动构造
        cout << "s4: " << sp4.use_count() << endl;
        SharedPtr<Test> sp5;
        sp5 = move(sp4);  // 移动赋值
        cout << "s5: " << sp5.use_count() << endl;
    }

    cout << "sp0: " << sp0.use_count() << endl;

    return 0;
}
```

### 面试问题
