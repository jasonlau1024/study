
# Linux
#### 0001 rm 删除文件空间就释放了吗？

1）为什么 rm 删除一个文件空间没有释放？
因为该文件的应用计数不为0。
1. rm 看似删除了文件，实际上只是将文件引用计数减1，即删除一个文件名到 inode 的链接。
2. 当一个文件的引用计数为0(包括硬链接数)时，才可能调用unlink删除。
3. 当一个程序打开一个文件的时候（获取到文件描述符），它的引用计数会被+1。


2）如何释放已经被删除文件占用的空间？
停止占用该文件的进程即可。
查看`deleted`状态的文件：`lsof |grep deleted` 获取占用进程PID

*补充：*恢复已打开并被删除的文件
```SHELL
# 1.获取占用该文件的进程 PID
lsof |grep deleted |grep 'fortest'
... ...
# 2. 获取改删除文件的fd
ls -al /proc/${PID}/fd
... ...
... 3 -> /root/fortest (deleted)
# 3. 查看文件内容
cat /proc/${PID}/fd/3
# 4. 恢复文件
dd if=/proc/${PID}/fd/3 of=/fortest
```

