


> WIKI：<https://zh.wikipedia.org/wiki//dev/random>

Linux提供两个特殊设备文件`/dev/random`和`/dev/urandom`来生成随机数


# 熵池

**熵池的概念：**

Linux是根据系统的熵池来产生随机数的。熵池就是系统当前的环境噪音，环境噪音的来源包括键盘的输入、鼠标的移动、内存的使用、文件的使用量、进程数量等等。

**查看系统熵池：**

查看熵池容量：`cat /proc/sys/kernel/random/poolsize`

查看当前熵数：`cat /proc/sys/kernel/random/entropy_avail`

熵池阻塞阀值：`cat /proc/sys/kernel/random/read_wakeup_threshold`

- 熵池的熵数小于该值，读取`/dev/random `会被阻塞

**自动增加熵数：**

1. 安装`haveged`服务：`sudo yum install haveged -y`
2. 启动`haveged`服务：`systemctl enable haveged && systemctl start haveged `

# random与urandom比较

`/dev/random`是真随机数生成器，它会消耗熵值来产生随机数，同时在熵耗尽的情况下会阻塞，直到有新的熵生成。

`/dev/urandom` 是伪随机数生成器，它根据一个初始的随机种子(这个种子来源就是熵池中的熵)来产生一系列的伪随机数，而并不会在熵耗尽的情况下阻塞。

# 总结

- 在无需高强度密码的情况下，使用`/dev/urandom`
- 在需要生成高强度长期的密码时，需要确保现有熵数是否足够，或者结合haveged服务使用，避免造成系统阻塞。