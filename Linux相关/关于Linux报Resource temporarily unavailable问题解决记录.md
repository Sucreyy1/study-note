**索引**
[TOC]
# 踩坑记录
## 事件

由于今天需要在测试环境把应用从root用户迁移到inmgr用户，当时创建了inmgr用户就直接进行了迁移。5分钟后应用直接出现OOM异常,经过上网查实，新建用户Linux系统会限制用户的最大进程数。应用程序占满进程数过后，执行任何命令都会报```/bin/bash: Resource temporarily unavailable```。
## 解决方案

1. 首先查看当前用户inmgr的各项限制参数：  
切换到inmgr用户，然后执行：```ulimit -a```
    ```shell
    [root@VM_3_220_centos ~]# su inmgr
    [inmgr@VM_3_220_centos root]$ ulimit -a
    core file size          (blocks, -c) 0
    data seg size           (kbytes, -d) unlimited
    scheduling priority             (-e) 0
    file size               (blocks, -f) unlimited
    pending signals                 (-i) 7271
    max locked memory       (kbytes, -l) 64
    max memory size         (kbytes, -m) unlimited
    open files                      (-n) 100001
    pipe size            (512 bytes, -p) 8
    POSIX message queues     (bytes, -q) 819200
    real-time priority              (-r) 0
    stack size              (kbytes, -s) 8192
    cpu time               (seconds, -t) unlimited
    max user processes              (-u) 2048
    virtual memory          (kbytes, -v) unlimited
    file locks                      (-x) unlimited
    ```
    其中重要参数：  
   
    **max user processes**：允许用户创建的最大子进程数。  
   
    现在这台服务器分别是2048。很明显```max user processes```不够用。
2. 现在以这台服务器为例：
    1. 切换到root用户，进入到```/etc/security/limits.d/```:  
        ```shell
        [root@VM_3_220_centos ~]$ cd /etc/security/limits.d/
        [root@VM_3_220_centos limits.d]$ ll
        total 8
        -rw-r--r-- 1 root root 195 Aug 29  2017 20-nproc.conf
        -rw-r--r-- 1 root root  60 Apr 21  2016 80-nofile.conf
        ```
    2. 编辑20-nproc.conf：
        ```shell
        [root@VM_3_220_centos limits.d]$ vim 20-nproc.conf 
        ```
    3. 添加以下信息：
        ```bash
        inmgr      soft    nproc     60000
        ```
        添加后的文件内容应该为：
        ```bash
        # Default limit for number of user's processes to prevent
        # accidental fork bombs.
        # See rhbz #432903 for reasoning.
        
        *          soft    nproc     4096
        root       soft    nproc     unlimited
        inmgr      soft    nproc     60000
        ```
    4. 编辑```/etc/security/limits.conf```，添加一下内容：  
        ```bash
        inmgr soft nproc 60000
        inmgr hard nproc 65535
        inmgr soft nofile 60000
        inmgr hard bofile 65535
        ```
        编辑后的文件内容为：
        ```bash
        # End of file
        * soft nofile 100001
        * hard nofile 100002
        root soft nofile 100001
        root hard nofile 100002
        inmgr soft nproc 65535
        inmgr hard nproc 65535
        inmgr soft nofile 60000
        inmgr hard nofile 65535
        ```
        **nproc**：表示max number of processes  
        **nofile**：表示max number of open file descriptors  
        **hard/soft**：soft是一个警告值，而hard则是一个真正意义的阀值，超过就会报错。
    5. 再次切换到inmgr用户，执行```ulimit -a```：
        ```
        [inmgr@VM_3_220_centos ~]$ ulimit -a
        core file size          (blocks, -c) 0
        data seg size           (kbytes, -d) unlimited
        scheduling priority             (-e) 0
        file size               (blocks, -f) unlimited
        pending signals                 (-i) 7271
        max locked memory       (kbytes, -l) 64
        max memory size         (kbytes, -m) unlimited
        open files                      (-n) 65535
        pipe size            (512 bytes, -p) 8
        POSIX message queues     (bytes, -q) 819200
        real-time priority              (-r) 0
        stack size              (kbytes, -s) 8192
        cpu time               (seconds, -t) unlimited
        max user processes              (-u) 65535
        virtual memory          (kbytes, -v) unlimited
        file locks                      (-x) unlimited
        ```
        可以看到最大进程数已经修改为65535了。重启应用后没有出现OOM异常。