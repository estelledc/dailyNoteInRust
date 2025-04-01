## 智能指针

智能指针是Rust中一种特殊的数据结构，它们的行为类似于指针，但拥有额外的元数据和功能。与普通引用不同，智能指针通常拥有它们指向的数据，并在超出作用域时执行特定操作（如释放内存）。

### Box\<T\> - 堆分配

Box是最简单的智能指针，它允许将数据存储在堆上而非栈上。

**特点**：
- 单一所有权
- 固定大小（一个指针的大小）
- 自动释放所指向的堆内存

**主要用途**：
- 解决递归类型的大小不确定问题
- 存储大型数据，避免栈溢出
- 实现特征对象

```rust
// 递归数据结构
enum List {
    Cons(i32, Box<List>),
    Nil,
}

// 创建列表
let list = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
```

### Rc\<T\> - 引用计数

Rc（Reference Counted）允许数据有多个所有者，适用于单线程环境。

**特点**：
- 跟踪指向数据的引用数量
- 允许多个所有者（共享所有权）
- 只提供不可变访问
- 仅在单线程环境中安全

**主要用途**：
- 需要在程序多个部分共享数据
- 无法确定哪一部分最后使用数据
- 构建图状数据结构

```rust
use std::rc::Rc;

let sun = Rc::new(Sun {});
let earth = Planet::Earth(Rc::clone(&sun));
let mars = Planet::Mars(Rc::clone(&sun));

// 查看引用计数
println!("引用计数: {}", Rc::strong_count(&sun)); // 输出: 3
```

### Arc\<T\> - 原子引用计数

Arc（Atomic Reference Counted）是Rc的线程安全版本，可在多线程间安全共享数据。

**特点**：
- 与Rc功能相似，但使用原子操作更新引用计数
- 线程安全，性能略低于Rc
- 只提供不可变访问

**主要用途**：
- 多线程间共享不可变数据
- 并行计算中的数据分发

```rust
use std::sync::Arc;
use std::thread;

let numbers = Arc::new(vec![1, 2, 3, 4]);

for i in 0..3 {
    let numbers_clone = Arc::clone(&numbers);
    thread::spawn(move || {
        println!("线程 {} 看到的数字: {:?}", i, *numbers_clone);
    });
}
```

### Cow\<T\> - 写时克隆

Cow（Clone on Write）提供对借用数据的不可变访问，仅在需要修改时才创建拷贝。

**特点**：
- 可以包含借用数据或拥有的数据
- 只在需要修改或获取所有权时才克隆数据
- 优化内存使用，避免不必要的复制

**主要用途**：
- 字符串处理
- 配置数据修改
- API返回值优化

```rust
use std::borrow::Cow;

fn capitalize(input: &str) -> Cow<str> {
    if input.chars().next().map_or(false, |c| c.is_uppercase()) {
        // 已经是大写，直接返回引用
        Cow::Borrowed(input)
    } else {
        // 需要修改，创建拥有的数据
        let mut result = input.to_owned();
        if let Some(r) = result.get_mut(0..1) {
            r.make_ascii_uppercase();
        }
        Cow::Owned(result)
    }
}
```

### 内部可变性模式

某些智能指针（如RefCell和Mutex）允许在遵循借用规则的前提下修改不可变引用指向的内容。

**主要类型**：
- `RefCell<T>` - 单线程内部可变性
- `Mutex<T>` - 多线程互斥锁
- `RwLock<T>` - 多线程读写锁

```rust
use std::cell::RefCell;

// RefCell示例
let data = RefCell::new(vec![1, 2, 3]);
{
    let mut v = data.borrow_mut();
    v.push(4);
}
println!("数据: {:?}", data.borrow()); // 输出: [1, 2, 3, 4]
```

### 智能指针选择指南

- **单一所有权，需要在堆上分配**：使用 `Box<T>`
- **多个所有者，单线程**：使用 `Rc<T>`
- **多个所有者，多线程**：使用 `Arc<T>`
- **可能需要修改借用数据**：使用 `Cow<T>`
- **需要内部可变性（单线程）**：使用 `RefCell<T>`
- **需要内部可变性（多线程）**：使用 `Mutex<T>` 或 `RwLock<T>`
- **共享可变数据（多线程）**：使用 `Arc<Mutex<T>>` 或 `Arc<RwLock<T>>`

智能指针是Rust内存管理的核心工具，通过它们可以构建各种复杂的数据结构，同时保持Rust的内存安全保证。
