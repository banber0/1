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
