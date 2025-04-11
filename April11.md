## 裸机程序设置

- `#![no_std]`：不使用 Rust 标准库，只使用核心库（core）
- `#![no_main]`：不使用标准 main 函数入口点，需自定义系统入口

## 汇编集成

- `global_asm!(include_str!("文件名"))`：在 Rust 代码中嵌入全局汇编
- `use core::arch::global_asm` 和 `use core::arch::asm`：引入内联汇编宏

## 函数标记

- `#[no_mangle]`：禁止函数名混淆，保留原始函数名供汇编代码调用
- `-> !`：发散类型，表示函数永不返回

## 内存管理

- `extern "C" { fn symbol(); }`：声明由链接器提供的外部符号
- BSS 段清零：初始化全局未初始化变量的内存区域
- 裸指针操作：`(address as *mut u8).write_volatile(0)`

## 安全边界

- `unsafe` 块：执行不安全操作，如裸指针操作和内联汇编
- 使用不安全代码的责任在于程序员确保其正确性

## 反汇编分析

### 使用 rust-objdump 工具

```bash
# 安装工具
rustup component add llvm-tools-preview
cargo install cargo-binutils

# 反汇编整个程序
cargo objdump --bin os -- -d > disassembly.txt

# 查找特定函数
cargo objdump --bin os -- -d --demangle | grep -A 20 "函数名"
```

### RISC-V 汇编理解

- 栈设置：`addi sp, sp, -16` 分配栈空间
- 寄存器保存：`sd ra, 8(sp)` 保存返回地址
- 地址计算：`auipc` 和 `addi` 组合加载符号地址
- 内存操作：`sb zero, 0(a1)` 写入零字节
- 循环控制：`bgeu`/`bne` 条件跳转指令

## 链接和内存布局

- 链接器脚本定义符号：如 `sbss`（BSS 段起始）和 `ebss`（BSS 段结束）
- 内存段划分：代码段(.text)、数据段(.data)、BSS 段(.bss)
- 程序加载：了解程序如何从存储加载到内存并开始执行


```
xun@LAPTOPofzx:~/rust/test1/os$ cargo asm --bin os rust_main
warning: `/home/xun/rust/test1/os/.cargo/config` is deprecated in favor of `config.toml`
note: if you need to support cargo 1.38 or earlier, you can symlink `config` to `config.toml`
warning: `/home/xun/rust/test1/os/.cargo/config` is deprecated in favor of `config.toml`
note: if you need to support cargo 1.38 or earlier, you can symlink `config` to `config.toml`
   Compiling os v0.1.0 (/home/xun/rust/test1/os)
    Finished `release` profile [optimized] target(s) in 0.05s

.section .text.rust_main,"ax",@progbits
        .globl  rust_main
        .p2align        1
.type   rust_main,@function
rust_main:
        .cfi_startproc
        addi sp, sp, -16
        .cfi_def_cfa_offset 16
        sd ra, 8(sp)
        sd s0, 0(sp)
        .cfi_offset ra, -8
        .cfi_offset s0, -16
        addi s0, sp, 16
        .cfi_def_cfa s0, 0
.Lpcrel_hi1:
        auipc a0, %pcrel_hi(ebss)
        addi a0, a0, %pcrel_lo(.Lpcrel_hi1)
.Lpcrel_hi2:
        auipc a1, %pcrel_hi(sbss)
        addi a1, a1, %pcrel_lo(.Lpcrel_hi2)
        bgeu a1, a0, .LBB1_2
.LBB1_1:
        addi a2, a1, 1
        sb zero, 0(a1)
        mv a1, a2
        bne a0, a2, .LBB1_1
.LBB1_2:
        j .LBB1_2
```
