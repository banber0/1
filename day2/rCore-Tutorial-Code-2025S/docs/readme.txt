事件1: 应用程序与基本执行环境
今天跟着教程把 rCore 的第一章过了一遍，从最基础的 Rust Hello, world! 开始，意识到应用程序依赖标准库 std 和操作系统（如 Linux）提供的系统调用支持。
尝试将目标平台切换为 riscv64gc-unknown-none-elf，遇到了标准库缺失的问题。通过引入 core 库，并移除对 std 和 main 函数的依赖（#![no_std], #![no_main]），手动实现了语义项 panic_handler，成功构建了一个合法的但没有任何功能的空 ELF 程序。
用户态最小化环境：在 main.rs 中使用内联汇编编写了 _start 入口，并通过 ecall 指令封装了 sys_exit 和 sys_write 系统调用。实现了基于 Write Trait 的 Stdout 和格式化宏 print!，成功在 QEMU User Mode 下输出字符并正常退出。
内核态裸机环境：将目标转向 QEMU System Mode，利用 RustSBI 作为 BootLoader。
SBI 调用：将退出机制改为调用 RustSBI 提供的 SBI_SHUTDOWN 服务。
内存布局：编写了 linker.ld 链接脚本，指定程序入口为 _start，并将基址设为 0x80200000，合理安排了 .text, .rodata, .data, .bss 段。
栈设置：在 entry.asm 中预留了 64KB 的栈空间，并初始化栈指针 sp，解决了 QEMU 无法正常运行的问题。
BSS 清零：实现了 clear_bss 函数，确保未初始化全局变量被正确清零。
问题
对于 linker.ld 链接脚本里的语法细节还有点懵，虽然照抄代码能跑通，但对于 .bss 段清零的具体时机（是在汇编里还是 Rust 代码里）感觉还需要再梳理一下。
计划
接下来准备进入第二章，学习批处理系统，研究如何加载并运行其他应用程序。