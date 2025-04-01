## 什么是泛型？
泛型（Generics）是 Rust 中的一种特性，允许我们编写更加通用和灵活的代码。通过泛型，我们可以在定义函数、结构体、枚举或方法时使用占位符类型，而不必指定具体的类型，从而实现代码的复用性和类型安全。

### 泛型的优势
- **代码复用**：避免为不同类型编写重复的函数或结构体
- **类型安全**：编译时进行类型检查，避免运行时错误
- **零成本抽象**：Rust 的泛型在编译时会被单态化，不会产生运行时开销

### 泛型函数示例
```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

// 使用
let numbers = vec![34, 50, 25, 100, 65];
let result = largest(&numbers);

let chars = vec!['y', 'm', 'a', 'q'];
let result = largest(&chars);
```

### 结构体与方法的泛型写法

```rust
struct Wrapper<T> {
    value: T,
}

impl<T> Wrapper<T> {
    pub fn new(value: T) -> Self {
        Wrapper { value }
    }
    
    pub fn get_value(&self) -> &T {
        &self.value
    }
}

// 可以为特定类型提供特殊实现
impl<T: Display> Wrapper<T> {
    pub fn display(&self) {
        println!("Value: {}", self.value);
    }
}
```

### 泛型约束
```rust
// 限制 T 必须实现 Display 和 PartialOrd trait
fn print_and_compare<T: Display + PartialOrd>(t: &T, u: &T) -> bool {
    println!("{} and {}", t, u);
    t > u
}

// 使用 where 子句（更清晰）
fn complex_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
    // 函数体
}
```

## trait
trait 是 Rust 中定义共享行为的方式。它类似于其他语言中的接口。
作用如下：
- 定义通用接口，支持多种类型实现相同的行为。
- 作为泛型约束，确保类型具备特定能力。
- 实现运行时多态（动态分发）。
- 提供默认实现，减少重复代码。
- 组合多个 trait，定义复杂行为。
- 扩展现有类型，添加自定义功能。

### trait 定义与实现示例
```rust
// 定义 trait
pub trait Summary {
    fn summarize(&self) -> String;
    
    // 带默认实现的方法
    fn summarize_default(&self) -> String {
        String::from("(Read more...)")
    }
}

// 为类型实现 trait
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
    // 没有实现 summarize_default，将使用默认实现
}
```

### trait 作为参数
```rust
// 接受任何实现了 Summary trait 的类型
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// 使用 trait bound 语法（等价写法）
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

### trait 对象与动态分发
```rust
// 创建 trait 对象数组，允许存储不同类型，但都实现了 Summary
let articles: Vec<Box<dyn Summary>> = vec![
    Box::new(NewsArticle { /* ... */ }),
    Box::new(Tweet { /* ... */ }),
];

// 遍历和调用方法
for article in articles {
    println!("{}", article.summarize());
}
```

### know1
在 Rust 中，trait 可以为其方法提供默认实现。
如果某个类型实现了该 trait，但没有显式实现某个方法，则会使用默认实现。

### trait 继承与组合
```rust
trait Animal {
    fn name(&self) -> String;
}

// OutbackCreature 继承了 Animal trait
trait OutbackCreature: Animal {
    fn noise(&self) -> String;
}

// 必须同时实现 Animal 和 OutbackCreature
struct Kangaroo {
    name: String,
}

impl Animal for Kangaroo {
    fn name(&self) -> String {
        self.name.clone()
    }
}

impl OutbackCreature for Kangaroo {
    fn noise(&self) -> String {
        String::from("Boing, boing!")
    }
}
```

## lifetime
生命周期（Lifetime）是 Rust 最独特的特性之一，用于确保引用在使用时有效。

### 生命周期的核心概念
- 生命周期标注并不改变引用的实际生存时间
- 它们告诉编译器引用之间的关系
- 大多数情况下，生命周期是隐式的，由编译器推断
- 当引用关系变复杂时，需要显式标注

### 生命周期语法
```rust
// <'a> 在函数名后声明了一个生命周期参数
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

这段代码表示：
- 参数 x 和 y 至少存活时间 'a
- 返回值的生命周期与参数相同
- 确保返回的引用不会比输入引用活得更久

### 结构体中的生命周期
```rust
// 结构体包含引用时必须标注生命周期
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    // 方法可能也需要生命周期标注
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

### 生命周期省略规则
Rust 有三条生命周期省略规则：
1. 每个引用参数被赋予自己的生命周期参数
2. 如果只有一个输入生命周期参数，该生命周期被赋给所有输出生命周期参数
3. 如果有多个输入生命周期参数，但其中一个是 &self 或 &mut self，self 的生命周期被赋给所有输出生命周期参数

### 静态生命周期
```rust
// 'static 表示引用在整个程序运行期间有效
let s: &'static str = "I have a static lifetime.";
```

生命周期是 Rust 内存安全保证的重要组成部分，它使编译器能够在编译时检测潜在的内存安全问题，而不需要运行时垃圾收集或手动内存管理。

## 迭代器
在 Rust 中，​迭代器（Iterator）​ 是一种强大的抽象，用于以统一的方式遍历集合类型（如数组、Vec、HashMap 等）或生成序列。Rust 的迭代器设计遵循零成本抽象（Zero-Cost Abstraction）原则，即用高级语法编写的迭代器代码，在编译后通常与手写的底层循环性能相当。

### Iterator trait
Rust 的迭代器是一个实现了 Iterator trait 的类型，定义在标准库中：
```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // 其他默认方法...
}
```

### 创建和使用迭代器
```rust
let v = vec![1, 2, 3];

// 创建迭代器
let v_iter = v.iter();

// 使用 for 循环（隐式调用 next()）
for val in v_iter {
    println!("Got: {}", val);
}

// 显式调用 next()
let mut v_iter = v.iter();
assert_eq!(v_iter.next(), Some(&1));
assert_eq!(v_iter.next(), Some(&2));
assert_eq!(v_iter.next(), Some(&3));
assert_eq!(v_iter.next(), None);
```

### 迭代器适配器
迭代器适配器是一些方法，它们消费一个迭代器并产生另一个迭代器：

```rust
let v = vec![1, 2, 3, 4, 5];

// map 转换元素
let v2: Vec<_> = v.iter().map(|x| x * 2).collect();
assert_eq!(v2, vec![2, 4, 6, 8, 10]);

// filter 过滤元素
let v3: Vec<_> = v.iter().filter(|x| *x % 2 == 0).collect();
assert_eq!(v3, vec![&2, &4]);

// chain 连接迭代器
let v4 = vec![6, 7, 8];
let chained: Vec<_> = v.iter().chain(v4.iter()).collect();
assert_eq!(chained, vec![&1, &2, &3, &4, &5, &6, &7, &8]);

// zip 并行迭代
let names = vec!["Alice", "Bob", "Charlie"];
let ages = vec![30, 25, 35];
let people: Vec<_> = names.iter().zip(ages.iter()).collect();
assert_eq!(people, vec![(&"Alice", &30), (&"Bob", &25), (&"Charlie", &35)]);
```

### 消费者适配器
消费者适配器消耗迭代器并产生一个最终结果：

```rust
let v = vec![1, 2, 3, 4, 5];

// sum 求和
let total: i32 = v.iter().sum();
assert_eq!(total, 15);

// fold 折叠/归约操作
let product = v.iter().fold(1, |acc, x| acc * x);
assert_eq!(product, 120);

// any 检查是否有满足条件的元素
let has_even = v.iter().any(|x| x % 2 == 0);
assert_eq!(has_even, true);

// all 检查是否所有元素都满足条件
let all_positive = v.iter().all(|x| *x > 0);
assert_eq!(all_positive, true);
```

### 不同类型的迭代方法
```rust
let mut v = vec![1, 2, 3];

// iter() - 不可变引用迭代器
let sum: i32 = v.iter().sum(); // v 仍可使用

// iter_mut() - 可变引用迭代器
for item in v.iter_mut() {
    *item *= 2;
}
assert_eq!(v, vec![2, 4, 6]);

// into_iter() - 获取所有权的迭代器
let total: i32 = v.into_iter().sum();
// v 不再可用，所有权已转移
```

​核心逻辑：迭代器通过 next() 方法逐个产生值，直到返回 None 表示结束。
​惰性求值：迭代器默认不会立即执行，只有在主动消费（如 collect()、for 循环）时才会触发操作。
​零成本抽象：迭代器的实现不会引入运行时开销，最终会被编译为与手写循环等效的代码。

## Box
Box<T> 是 Rust 标准库提供的最简单的智能指针，它允许将数据存储在堆上而非栈上。

### Box 的主要用途

1. **处理编译时大小未知的数据类型**

Rust 在编译时需要知道每个类型的确切大小。对于递归类型（如链表），这会导致问题：
```rust
// 不能编译：编译器无法确定类型大小
pub enum List {
    Cons(i32, List), // 问题在这里！编译器无法确定 List 的大小
    Nil,
}

// 使用 Box 修复递归类型定义
#[derive(PartialEq, Debug)]
pub enum List {
    Cons(i32, Box<List>), // 使用 Box 包装递归部分
    Nil,
}

// 使用示例
fn main() {
    let list = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
    println!("{:?}", list);
}
```

2. **存储大量数据但不转移所有权**

```rust
// 直接在栈上存储大数组可能导致栈溢出
let large_array = Box::new([0; 1000000]); // 在堆上分配
```

3. **实现 trait 对象**

```rust
// trait 对象允许不同类型的值放在同一容器中
let shapes: Vec<Box<dyn Draw>> = vec![
    Box::new(Circle { /* ... */ }),
    Box::new(Rectangle { /* ... */ }),
];
```

### Box 与所有权

```rust
let a = Box::new(5);
let b = a; // 所有权转移给 b，a 不再有效
// println!("{}", a); // 错误：a 的值已被移动

// 解引用访问内容
let x = Box::new(42);
assert_eq!(*x, 42); // 使用解引用运算符 *
```

### Box::leak - 将 Box 转为静态引用

```rust
let s = Box::new(String::from("hello, world"));
// 消耗 Box 并返回静态引用，延长生命周期至程序结束
let static_str: &'static mut String = Box::leak(s);
static_str.push_str("!");
println!("{}", static_str); // 输出 "hello, world!"
```

Box<T> 是一个智能指针，它将数据存储在堆上，而在栈上只保存一个固定大小的指针。这解决了递归类型的大小问题。

**数据结构：Cons 列表**
- 源自 Lisp 编程语言
- 每个元素包含一个值和指向下一个元素的链接
- 函数式编程中常用的不可变链表实现

## Rc<T>
Rc<T> 是 "Reference Counted" 的缩写，它允许数据有多个所有者。

### 工作原理

1. **共享所有权**：当需要在多个地方使用同一个数据，但无法确定哪个部分最后使用这些数据时使用

2. **引用计数**：
- `Rc::new(value)` 创建一个新的引用计数指针，计数为 1
- `Rc::clone(&rc)` 创建一个新的指针指向相同数据，计数加 1
- 当一个 Rc 指针被丢弃时，计数减 1
- 当计数变为 0 时，数据被释放

3. **只读访问**：Rc<T> 只允许不可变借用，不支持可变借用（需要 RefCell<T> 等内部可变性）

### 使用示例

```rust
use std::rc::Rc;

enum List {
    Cons(i32, Rc<List>),
    Nil,
}

fn main() {
    use List::{Cons, Nil};
    
    // 创建一个列表：5 -> 10 -> Nil
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    
    // 计数：1
    println!("count after creating a = {}", Rc::strong_count(&a));
    
    // 创建 b 共享 a 的部分数据
    let b = Cons(3, Rc::clone(&a));
    
    // 计数：2
    println!("count after creating b = {}", Rc::strong_count(&a));
    
    {
        // 创建 c 也共享同样的数据
        let c = Cons(4, Rc::clone(&a));
        
        // 计数：3
        println!("count after creating c = {}", Rc::strong_count(&a));
    } // c 离开作用域，计数减 1
    
    // 计数：2
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
```

### 结合 RefCell 实现内部可变性

```rust
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));
    
    let a = Rc::clone(&value);
    let b = Rc::clone(&value);
    
    // 通过 a 修改共享的值
    *a.borrow_mut() += 10;
    
    // 通过 b 也能看到变化
    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("value after = {:?}", value);
}
```

需要注意 Rc<T> 只能在单线程环境中使用，多线程情况应使用 Arc<T>（原子引用计数）。

## Arc<T>
Arc<T> 是 "Atomic Reference Counted" 的缩写，它是 Rust 标准库中的一个智能指针，用于在多线程环境中安全地共享数据。

### 为什么需要 Arc<T>？
Rust 的所有权系统通常只允许一个变量拥有一个值。然而，有时我们需要在多个地方（特别是多个线程）共享同一数据，这时就需要 Arc<T>。

### Arc<T> 与 Rc<T> 的区别
- Arc<T> 使用原子操作计数，线程安全但有性能开销
- Rc<T> 使用非原子操作计数，仅限单线程使用但性能更好

### 使用示例

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    // 创建一个 Arc 指针指向数据
    let data = Arc::new(vec![1, 2, 3, 4, 5]);
    
    // 创建多个线程，每个线程共享同一数据
    let mut handles = vec![];
    
    for i in 0..3 {
        // 克隆 Arc 指针（增加引用计数）
        let data_clone = Arc::clone(&data);
        
        // 移动 data_clone 到新线程
        let handle = thread::spawn(move || {
            println!("Thread {}: {:?}", i, *data_clone);
        });
        
        handles.push(handle);
    }
    
    // 等待所有线程结束
    for handle in handles {
        handle.join().unwrap();
    }
    
    // 主线程仍然拥有数据的访问权
    println!("Original data still accessible: {:?}", *data);
}
```

### 结合 Mutex 实现线程安全的可变数据

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // 创建一个包装在 Mutex 和 Arc 中的数据
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            // 锁定互斥锁并增加计数
            let mut num = counter.lock().unwrap();
            *num += 1;
            // 锁在这里被自动释放
        });
        handles.push(handle);
    }
    
    // 等待所有线程完成
    for handle in handles {
        handle.join().unwrap();
    }
    
    // 打印最终结果
    println!("Result: {}", *counter.lock().unwrap());
}
```

### Arc<T> 的内存和性能考虑
- Arc<T> 在堆上分配数据，加上引用计数的额外内存
- 原子操作比普通操作慢，但通常不会成为性能瓶颈
- 精确的线程同步（如 Mutex）通常比引用计数的开销更大

Arc<T> 是多线程 Rust 程序中安全共享数据的基础工具，经常与其他同步原语（如 Mutex、RwLock）结合使用。