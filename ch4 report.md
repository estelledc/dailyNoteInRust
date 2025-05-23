
## 编程作业：系统调用实现
### 未完成的系统调用
| 系统调用 | 文件位置 | 状态 |
|---------|---------|------|
| `sys_get_time` | `os/src/syscall/process.rs` | 未实现 |
| `sys_trace` | `os/src/syscall/process.rs` | 未实现 |
| `sys_mmap` | `os/src/syscall/process.rs` | 未实现 |
| `sys_munmap` | `os/src/syscall/process.rs` | 未实现 |
### 如何完成
#### map
把内存管理想象成 ​**​图书馆管理员的工作​**​：

- ​**​`mmap`​**​  
    像在图书馆登记新书架：  
    ✅ 划定书架位置（`start`到`end`）  
    ✅ 贴上标签（`flag`：只读/可写等）  
    ✅ 更新图书馆地图（页表）
    
- ​**​`munmap`​**​  
    像撤掉旧书架：  
    🔍 先找到目标书架位置  
    🗑️ 从地图上擦除信息  
    📋 从登记册中删除记录
    
- ​**​`is_overlap`​**​  
    类似检查："新书架的位置会和现有书架撞上吗？"
    
- ​**​`is_readable/writable`​**​  
    像检查："读者能看/修改这个书架的书吗？"

#### process
像给程序提供的"客服热线"——程序可以通过这些系统调用申请内存、查询时间、主动休息，或者干脆结束自己，而内核就是接听电话的客服，负责处理这些请求并返回结果

#### task
1. ​**​系统调用统计​**​
    - `count_syscall()`：记录每个系统调用的使用次数
    - `get_syscall_times()`：查询指定系统调用的调用次数
2. ​**​追踪系统调用(trace)​**​
    - 支持三种操作模式：
        - 模式0：安全读取用户内存数据（检查可读权限）
        - 模式1：安全写入用户内存数据（检查可写权限）
        - 模式2：获取系统调用统计次数
给操作系统装了个"监控摄像头+内存管家"——既能统计每个程序用了哪些功能，又能安全地帮程序分配和回收内存，还能防止程序乱动不该碰的内存区域。
## 问答作业
1. 请列举 SV39 页表页表项的组成，描述其中的标志位有何作用？
	页表项（PTE）是一个 ​**​64 位（8 字节）​**​ 的数据结构
	由 ​**​44位物理页号（PPN）​**​ 和 ​**​10位控制标志位​**​（如R/W/X权限、D/A/V状态位）组成，共同完成虚拟地址到物理地址的映射与访问控制。
	**标志位控制内存访问权限（R/W/X/U）、状态跟踪（V/A/D），并辅助操作系统管理内存（如缺页处理、页面置换）。​**
2. 缺页
    缺页指的是进程访问页面时页面不在页表中或在页表中无效的现象，此时 MMU 将会返回一个中断， 告知 os 进程内存访问出了问题。os 选择填补页表并重新执行异常指令或者杀死进程。
    
    - 请问哪些异常可能是缺页导致的？
        **Page Fault（缺页异常）​**​，表现为访问无效页（V=0）、权限不足（如用户访问内核页）或未映射的地址。
    - 发生缺页时，描述相关重要寄存器的值
        `stval` 存储出错的虚拟地址，`scause` 记录异常原因（如 12=指令缺页，13=加载缺页，15=存储缺页），`sepc` 保存触发异常的指令地址。
    
    缺页有两个常见的原因，其一是 Lazy 策略，也就是直到内存页面被访问才实际进行页表操作。 比如，一个程序被执行时，进程的代码段理论上需要从磁盘加载到内存。但是 os 并不会马上这样做， 而是会保存 .text 段在磁盘的位置信息，在这些代码第一次被执行时才完成从磁盘的加载操作。
    
    - 这样做有哪些好处？
        延迟内存分配和加载，​**​减少启动开销和内存浪费​**​（如只加载实际使用的代码段）。
    
    其实，我们的 mmap 也可以采取 Lazy 策略，比如：一个用户进程先后申请了 10G 的内存空间， 然后用了其中 1M 就直接退出了。按照现在的做法，我们显然亏大了，进行了很多没有意义的页表操作。
    
    - 处理 10G 连续的内存页面，对应的 SV39 页表大致占用多少内存 (估算数量级即可)？
        约 ​**​20MB​**​（三级页表每页 512 项，10G/4K=2.5M 页，每页表项8字节，2.5M×8≈20MB）。
    - 请简单思考如何才能实现 Lazy 策略，缺页时又如何处理？描述合理即可，不需要考虑实现。
        标记页表项为无效（V=0），缺页时根据保存的磁盘位置加载数据到物理页，更新 PTE 并重试指令
    
    缺页的另一个常见原因是 swap 策略，也就是内存页面可能被换到磁盘上了，导致对应页面失效。
    
    - 此时页面失效如何表现在页表项(PTE)上？
        页表项 ​**​V=0​**​，并利用保留位（如 RSW）或高位存储磁盘位置信息，换入时恢复 PTE 并映射物理页。
    
3. 双页表与单页表
    
    为了防范侧信道攻击，我们的 os 使用了双页表。但是传统的设计一直是单页表的，也就是说， 用户线程和对应的内核线程共用同一张页表，只不过内核对应的地址只允许在内核态访问。 (备注：这里的单/双的说法仅为自创的通俗说法，并无这个名词概念，详情见 [KPTI](https://en.wikipedia.org/wiki/Kernel_page-table_isolation) )
    
    - 在单页表情况下，如何更换页表？
        通过修改 `satp` 寄存器切换页表基址，内核和用户态共享同一张页表，仅通过权限位（`U=0`）隔离内核空间。
    - 单页表情况下，如何控制用户态无法访问内核页面？（tips:看看上一题最后一问）
        内核页的页表项设置 `U=0`（用户态不可访问），用户态访问时触发权限异常（类似缺页处理）。
    - 单页表有何优势？（回答合理即可）
        **减少切换开销​**​（无需频繁换 `satp`），​**​TLB 效率更高​**​（内核/用户共享缓存），​**​实现简单​**​（仅需权限控制）。
    - 双页表实现下，何时需要更换页表？假设你写一个单页表操作系统，你会选择何时更换页表（回答合理即可）？
		**减少切换开销​**​（无需频繁换 `satp`），​**​TLB 效率更高​**​（内核/用户共享缓存），​**​实现简单​**​（仅需权限控制）。




## 荣誉准则
在完成本次实验的过程（含此前学习的过程）中，我曾分别与 以下各位 就（与本次实验相关的）以下方面做过交流，还在代码中对应的位置以注释形式记录了具体的交流对象及内容：

loukas

此外，我也参考了 以下资料 ，还在代码中对应的位置以注释形式记录了具体的参考来源及内容：

https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/6answer.html
[lab2 · LearningOS/2025s-rcore-NoahNieh@d20798c (github.com)](https://github.com/LearningOS/2025s-rcore-NoahNieh/commit/d20798c4b9a1d9b362acead930c11aff138f9604#diff-dca4f857dcfb98515dff473becd7e097351b82b6581462001251045106b2c0e8)

3. 我独立完成了本次实验除以上方面之外的所有工作，包括代码与文档。 我清楚地知道，从以上方面获得的信息在一定程度上降低了实验难度，可能会影响起评分。

4. 我从未使用过他人的代码，不管是原封不动地复制，还是经过了某些等价转换。 我未曾也不会向他人（含此后各届同学）复制或公开我的实验代码，我有义务妥善保管好它们。 我提交至本实验的评测系统的代码，均无意于破坏或妨碍任何计算机系统的正常运转。 我清楚地知道，以上情况均为本课程纪律所禁止，若违反，对应的实验成绩将按“-100”分计。