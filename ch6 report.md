## 编程作业


本次实验实现了硬链接相关的系统调用，包括 `sys_linkat`、`sys_unlinkat` 和 `sys_fstat`，涉及的主要知识点如下：

1. **文件系统基本原理**
  
  * inode（索引节点）与目录项的关系
  * 多个目录项指向同一个 inode 实现硬链接
  * 文件删除时 inode 和数据块的回收机制
2. **系统调用实现**
  
  * 系统调用参数的解析与用户空间字符串的内核空间转换
  * 系统调用的注册与分发机制
  * 错误码的处理与返回
3. **内核数据结构设计与操作**
  
  * 文件描述符表的管理
  * Stat 结构体与文件类型（StatMode）的定义和使用
  * 文件状态信息的获取与填充
4. **Easy-FS 文件系统实现细节**
  
  * 目录项的遍历与修改
  * inode 的引用计数（nlink 字段）管理
  * 目录项的添加、删除与查找
  * 数据块和 inode 的分配与回收

## Question

在 easy-fs 文件系统中，root inode（根索引节点）是整个文件系统的入口，代表根目录（即 / 目录）。它的作用主要有：

作为文件系统的起点：所有文件和目录的查找、遍历、创建等操作，都是从 root inode 开始递归进行的。管理根目录内容：root inode 记录了根目录下所有文件和子目录的目录项（DirEntry），这些目录项包含了文件/目录名和对应的 inode 编号。保证文件系统结构的完整性：root inode 的存在和正确性，是整个文件系统能被正常挂载和访问的前提。如果 root inode 的内容损坏了，会发生以下问题：

无法挂载文件系统：文件系统初始化时需要读取 root inode，如果损坏，无法正确解析目录结构，文件系统可能直接挂载失败。无法访问任何文件或目录：所有路径查找都以 root inode 为起点，损坏后无法找到任何有效的目录项，导致所有文件和目录都不可见或不可访问。数据丢失风险：虽然数据块本身可能未损坏，但由于无法通过 root inode 访问，等同于数据“丢失”。可能导致进一步的数据一致性问题：如果 root inode 的元数据（如 nlinks、size、block 指针等）异常，可能导致错误的块分配、回收，甚至破坏其他文件的数据。总结：root inode 是 easy-fs 的“根”，其损坏会导致整个文件系统瘫痪，无法访问任何数据。实际系统中通常会对 root inode 做特殊保护或冗余备份。

## 荣誉准则

在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

none

此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

https://github.com/LearningOS/2025s-rcore-liu0fanyi/

1. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。
  
2. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。