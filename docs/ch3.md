#ch3
    ##运行本章代码时报错，原因是代理问题
        在寻找是因为git下载链接被篡改了
        git config --global --list | grep ghproxy检查git全局配置，通过命令git config --global --remove-section url."https://mirror.ghproxy.com/https://github.com/"将其删除

        #首先取消终端里的环境变量
            unset http_proxy
            unset https_proxy
            unset all_proxy
        #取消 Git 的代理配置
            git config --global --unset http.proxy
            git config --global --unset https.proxy


    ##修改task.rs
        在任务控制块中添加一个数组，用来记录系统被调用的次数
    ##修改mod.rs
        添加更新当前任务的某个系统调用次数函数以及获取当前任务的某个系统调用次数
    ##修改process.rs
        实现读内存，写内存，查计数
        0为读取id地址处的一个字节
        1为将 data 写入 id 地址
        2为查询系统调用 id 的调用次数
    ##最后修改syscall下的mod.rs
        引入 task 模块的更新函数use crate::task::current_add_syscall_times;


    ##make run BASE=0后有报错
    运行问题1：是关于参数下划线的问题，需要把下划线去掉
    前缀 _ 通常用来标记“未使用的变量”以避免编译器警告，而使用后需要去掉下划线
    运行问题2：#再次运行后出现没有写///注释的问题，在项目中#![deny(missing_docs)]为严格的代码检查，任何公开的（pub）结构体字段或函数，上面需要写文档注释，否则编译器就会报错

简答
2.
#1
sp为内核栈的栈顶
__restore 的两种使用情景
系统调用/中断返回： 当一个用户程序陷入内核，内核处理完系统调用或中断后，调用 __restore 将 CPU 状态恢复，返回用户态继续执行。
新进程首次启动： 当内核构造一个新的进程时，会在该任务的内核栈上手动构造一个 TrapContext，并将栈指针 sp 指向它。然后“伪造”一个从内核返回用户态的过程，通过 __restore 启动该进程的第一条指令。
#2
特殊处理了sstatus, sepc, sscratch寄存器
sstatus控制cpu的特权级状态
sepc记录trap发生时的指令地址
sscratch用于交换栈指针
3.
x2 是栈指针寄存器。此时我们需要依赖 sp 来读取栈上的 TrapContext 数据。如果现在就恢复了 sp，我们就丢失了指向 TrapContext 的指针，无法继续读取剩下的寄存器数据了。sp 必须在最后一步通过 csrrw 恢复
x4是线程指针寄存器，用户程序通常不使用线程指针寄存器 tp，或者操作系统不需要在 Trap 期间保存/恢复它
4.
执行前：sp: 指向 内核栈，scratch: 保存了 用户栈 的地址
执行后：sp: 恢复为 用户栈 的地址，sscratch: 保存了 内核栈 的地址
5.
sret指令
它会读取 sstatus 寄存器中的 SPP 字段，当指令执行时cpu根据SSP的值将当前特权级设置为 用户模式，并将 PC 跳转到 sepc 中的地址，从而真正进入用户态。
6.
执行前，sp: 用户栈指针，sscratch: 内核栈指针。
执行后，sp: 内核栈 指针，sscratch: 用户栈 指针。
确立了内核栈的使用权，保证后续的寄存器保存操作（压栈）是写入到内核内存区域，而不是用户内存区域。
7.
ecall系统调用指令
