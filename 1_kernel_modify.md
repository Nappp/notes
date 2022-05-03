### 一些kernel级别的修改，以本人手上的oneplus3T为例
### 主要参考[安卓反调试](https://bbs.pediy.com/thread-223324.htm)实现

- `kernel/oneplus/msm8996/fs/notify/inotify/inotify_user.c`
    - 找到`inotify_find_inode(pathname, &path, flags);`
    - 在调用`inotify_find_inode`前添加以下判断，用于对抗`inotify`监测dump ( 第16条 ).
    -
 ````cpp
 if ( strstr(pathname, "/mem") || strstr(pathname, "/pagemap") ) {
         // nop
         return 0;
 }
 ````
 ````
 部分国酒app似乎会针对notify返回值做处理，这样修改会导致Kernel Panic，
 在inotify_add_watch中覆盖标志位应该可以绕过这部分检测
 
 if ( strstr(pathname, "/mem") || strstr(pathname, "/pagemap") ) {
        /* override mask to meaning-less state */
        mask = IN_DELETE;
}
 ````
- `kernel/oneplus/msm8996/fs/proc/array.c`
  - `task_state_array`数组，将`t (tracing stop)`修改为`S (sleeping)` ( 第15条 )
  - 找到`static inline void task_state`函数，添加以下判断（ 第11条 ）
````cpp
if (tracer) {
    tpid = task_pid_nr_ns(tracer, ns);
    if( ! (tpid == pid_nr_ns(pid, ns) || tpid == ppid ) ) {
            tpid = 0;
  }
}
````
- `kernel/oneplus/msm8996/fs/proc/base.c`
  - [参考](https://bbs.pediy.com/thread-213481.htm)
  - 找到`static int proc_pid_wchan`, `if`部分修改为
````cpp

if (lookup_symbol_name(wchan, symname) < 0)
        if (!ptrace_may_access(task, PTRACE_MODE_READ_FSCREDS))
                return 0;
        else
                return seq_printf(m, "%lu", wchan);
else {
        if (strstr(symname, "trace")) {
                return seq_printf(m, "%s", "sys_epoll_wait");
        }
        return seq_printf(m, "%s", symname);
}
````

- `kernel/oneplus/msm8996/kernel/ptrace.c`
  - 找到`static int ptrace_traceme`，返回值修改 ( 第9条 )
````cpp
if( ret == -EPERM ) {
    return 0;
}
````
