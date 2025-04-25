---
layout: mypost
title: 工厂模式详解与C++实现
categories: [设计模式, C++]
---


---

### **1. 简单工厂模式（Simple Factory）**
#### 实现逻辑
- **核心思想**：通过一个**中心化的工厂类**，根据传入的参数决定创建哪种具体产品。
- **结构组成**：
  
  1. **产品基类**：定义所有产品的公共接口（`Product`）。
  2. **具体产品**：继承基类，实现具体逻辑（`ConcreteProductA/B`）。
  3. **简单工厂类**：包含一个静态方法，通过 `switch-case` 或 `if-else` 根据参数创建对象。
- **代码逻辑**：
  
  ```cpp
  #include <iostream>
  
  // 产品基类
  class Product {
  public:
      virtual void operation() = 0;
      virtual ~Product() = default;
  };
  
  // 具体产品A
  class ConcreteProductA : public Product {
  public:
      void operation() override {
          std::cout << "ConcreteProductA operation\n";
      }
  };
  
  // 具体产品B
  class ConcreteProductB : public Product {
  public:
      void operation() override {
          std::cout << "ConcreteProductB operation\n";
      }
  };
  
  // 简单工厂类
  class SimpleFactory {
  public:
      enum ProductType { A, B };
      
      static Product* createProduct(ProductType type) {
          switch(type) {
              case A: return new ConcreteProductA();
              case B: return new ConcreteProductB();
              default: return nullptr;
          }
      }
  };
  
  // 使用示例
  int main() {
      Product* productA = SimpleFactory::createProduct(SimpleFactory::A);
      Product* productB = SimpleFactory::createProduct(SimpleFactory::B);
      
      productA->operation(); // 输出: ConcreteProductA operation
      productB->operation(); // 输出: ConcreteProductB operation
      
      delete productA;
      delete productB;
  }
  ```
  - 用户直接调用工厂类的静态方法，传入类型参数（如 `A` 或 `B`）。
  - 工厂根据参数实例化对应的具体产品，返回基类指针。
- **特点**：
  - **优点**：客户端无需关心对象创建细节。
  - **缺点**：违反开闭原则（新增产品需修改工厂类的 `switch-case`）。
---

### **2. 工厂方法模式（Factory Method）**
#### 实现逻辑
- **核心思想**：将对象的创建延迟到**子类**，每个具体产品对应一个**独立的工厂类**。
- **结构组成**：
  1. **产品基类**：定义接口（`Product`）。
  2. **具体产品**：实现产品逻辑（`ConcreteProductA/B`）。
  3. **工厂基类**：声明创建产品的抽象方法（`Creator::createProduct()`）。
  4. **具体工厂**：继承工厂基类，实现具体产品的创建（`ConcreteCreatorA/B`）。
- **代码逻辑**：
  ```cpp
  #include <iostream>
  
  // 产品基类
  class Product {
  public:
      virtual void operation() = 0;
      virtual ~Product() = default;
  };
  
  // 具体产品A
  class ConcreteProductA : public Product {
  public:
      void operation() override {
          std::cout << "ConcreteProductA operation\n";
      }
  };
  
  // 具体产品B
  class ConcreteProductB : public Product {
  public:
      void operation() override {
          std::cout << "ConcreteProductB operation\n";
      }
  };
  
  // 工厂基类
  class Creator {
  public:
      virtual Product* createProduct() = 0;
      virtual ~Creator() = default;
  };
  
  // 具体工厂A
  class ConcreteCreatorA : public Creator {
  public:
      Product* createProduct() override {
          return new ConcreteProductA();
      }
  };
  
  // 具体工厂B
  class ConcreteCreatorB : public Creator {
  public:
      Product* createProduct() override {
          return new ConcreteProductB();
      }
  };
  
  // 使用示例
  int main() {
      Creator* creatorA = new ConcreteCreatorA();
      Creator* creatorB = new ConcreteCreatorB();
      
      Product* productA = creatorA->createProduct();
      Product* productB = creatorB->createProduct();
      
      productA->operation(); // 输出: ConcreteProductA operation
      productB->operation(); // 输出: ConcreteProductB operation
      
      delete creatorA;
      delete creatorB;
      delete productA;
      delete productB;
  }
  ```
  - 客户端选择具体工厂（如 `ConcreteCreatorA`）。
  - 调用工厂的 `createProduct()` 方法创建对应的产品。
- **特点**：
  - **优点**：符合开闭原则（新增产品只需添加新工厂，无需修改已有代码）。
  - **缺点**：类的数量膨胀（每个产品需要一个工厂类）。
---

### **3. 抽象工厂模式（Abstract Factory）**
#### 实现逻辑
- **核心思想**：创建**多个相关产品家族**（如 GUI 中的按钮、文本框），确保产品之间的兼容性。
- **结构组成**：
  1. **抽象产品族**：定义多个产品接口（如 `AbstractProductA` 和 `AbstractProductB`）。
  2. **具体产品**：实现同一家族的不同产品（如 `ProductA1/ProductB1` 属于同一风格）。
  3. **抽象工厂**：声明创建多个产品的方法（`createProductA()` 和 `createProductB()`）。
  4. **具体工厂**：实现同一产品家族的创建逻辑（如 `ConcreteFactory1` 生产 `A1/B1`）。
- **代码逻辑**：
  ```cpp
  #include <iostream>
  
  // 抽象产品A
  class AbstractProductA {
  public:
      virtual void operationA() = 0;
      virtual ~AbstractProductA() = default;
  };
  
  // 具体产品A1
  class ProductA1 : public AbstractProductA {
  public:
      void operationA() override {
          std::cout << "ProductA1 operation\n";
      }
  };
  
  // 具体产品A2
  class ProductA2 : public AbstractProductA {
  public:
      void operationA() override {
          std::cout << "ProductA2 operation\n";
      }
  };
  
  // 抽象产品B
  class AbstractProductB {
  public:
      virtual void operationB() = 0;
      virtual ~AbstractProductB() = default;
  };
  
  // 具体产品B1
  class ProductB1 : public AbstractProductB {
  public:
      void operationB() override {
          std::cout << "ProductB1 operation\n";
      }
  };
  
  // 具体产品B2
  class ProductB2 : public AbstractProductB {
  public:
      void operationB() override {
          std::cout << "ProductB2 operation\n";
      }
  };
  
  // 抽象工厂
  class AbstractFactory {
  public:
      virtual AbstractProductA* createProductA() = 0;
      virtual AbstractProductB* createProductB() = 0;
      virtual ~AbstractFactory() = default;
  };
  
  // 具体工厂1
  class ConcreteFactory1 : public AbstractFactory {
  public:
      AbstractProductA* createProductA() override {
          return new ProductA1();
      }
      
      AbstractProductB* createProductB() override {
          return new ProductB1();
      }
  };
  
  // 具体工厂2
  class ConcreteFactory2 : public AbstractFactory {
  public:
      AbstractProductA* createProductA() override {
          return new ProductA2();
      }
      
      AbstractProductB* createProductB() override {
          return new ProductB2();
      }
  };
  
  // 使用示例
  int main() {
      AbstractFactory* factory1 = new ConcreteFactory1();
      AbstractFactory* factory2 = new ConcreteFactory2();
  
      AbstractProductA* a1 = factory1->createProductA();
      AbstractProductB* b1 = factory1->createProductB();
      
      AbstractProductA* a2 = factory2->createProductA();
      AbstractProductB* b2 = factory2->createProductB();
  
      a1->operationA(); // 输出: ProductA1 operation
      b1->operationB(); // 输出: ProductB1 operation
      a2->operationA(); // 输出: ProductA2 operation
      b2->operationB(); // 输出: ProductB2 operation
  
      delete factory1;
      delete factory2;
      delete a1;
      delete b1;
      delete a2;
      delete b2;
  }
  ```
  - 客户端选择一个具体工厂（如 `ConcreteFactory1`）。
  - 通过工厂创建同一家族的所有产品（如 `A1` 和 `B1`）。
- **特点**：
  - **优点**：保证产品族的兼容性，切换产品族只需更换工厂。
  - **缺点**：扩展新产品类型困难（需修改所有工厂类）。

---

