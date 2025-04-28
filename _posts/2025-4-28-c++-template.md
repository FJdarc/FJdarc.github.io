---
layout: mypost
title: C++模板（template）详解
categories: [keyword, C++]
---


---

### **1. 函数模板**
**作用**：定义可处理任意类型的通用函数。
- **语法**：
  ```cpp
  template<typename T>
  T max(T a, T b) {
      return (a > b) ? a : b;
  }
  ```
- **使用**：
  ```cpp
  int a = 10, b = 20;
  cout << max(a, b); // 隐式推导为 int 类型
  cout << max<double>(5.5, 6); // 显式指定类型
  ```

---

### **2. 类模板**
**作用**：创建可存储或操作任意类型的类。
- **语法**：
  ```cpp
  template<class T>
  class Stack {
  private:
      T elements[100];
      int top;
  public:
      void push(T value);
      T pop();
  };
  
  // 成员函数定义
  template<class T>
  void Stack<T>::push(T value) {
      elements[top++] = value;
  }
  ```
- **实例化**：
  ```cpp
  Stack<int> intStack; // 存储 int 的栈
  Stack<string> strStack; // 存储 string 的栈
  ```

---

### **3. 非类型模板参数**
**作用**：允许传递常量值（如整数、指针）作为模板参数。
- **示例**：
  ```cpp
  template<int N>
  class Array {
      int data[N];
  public:
      int size() { return N; }
  };
  
  Array<10> arr; // 大小为 10 的数组
  ```

---

### **4. 模板特化**
**作用**：为特定类型提供定制实现。
- **全特化**：
  ```cpp
  template<>
  class Stack<bool> { // 针对 bool 的特化
      // 特殊实现（如位压缩存储）
  };
  ```
- **偏特化**（部分特化）：
  ```cpp
  template<class T>
  class Stack<T*> { // 针对指针类型的特化
      // 处理指针的特殊逻辑
  };
  ```

---

### **5. 可变参数模板（C++11）**
**作用**：处理任意数量和类型的参数。
- **递归展开**：
  ```cpp
  template<typename T>
  void print(T t) {
      cout << t << endl;
  }
  
  template<typename T, typename... Args>
  void print(T t, Args... args) {
      cout << t << " ";
      print(args...); // 递归调用
  }
  
  print(1, "Hello", 3.14); // 输出：1 Hello 3.14
  ```
- **折叠表达式（C++17）**：
  ```cpp
  template<typename... Args>
  auto sum(Args... args) {
      return (args + ...); // 展开为 args1 + args2 + ...
  }
  ```

---

### **6. 类型推导与 `auto`（C++11）**
- **`decltype` 推导返回类型**：
  ```cpp
  template<typename T1, typename T2>
  auto add(T1 a, T2 b) -> decltype(a + b) {
      return a + b;
  }
  ```

---

### **7. 模板元编程**
**作用**：在编译时进行计算。
- **编译时阶乘**：
  ```cpp
  template<int N>
  struct Factorial {
      static const int value = N * Factorial<N-1>::value;
  };
  
  template<>
  struct Factorial<0> {
      static const int value = 1;
  };
  
  cout << Factorial<5>::value; // 输出 120
  ```

---

### **8. 概念约束（C++20）**
**作用**：明确限制模板参数类型。
- **语法**：
  ```cpp
  template<typename T>
  requires std::integral<T> // 约束 T 为整数类型
  T square(T n) {
      return n * n;
  }
  ```

---

### **常见问题与解决**
- **链接错误**：模板定义必须放在头文件中。
- **类型不支持操作**：使用 `static_assert` 或概念约束确保类型合规。
  ```cpp
  template<typename T>
  void print(T val) {
      static_assert(std::is_arithmetic<T>::value, "T 必须是算术类型");
      cout << val << endl;
  }
  ```
