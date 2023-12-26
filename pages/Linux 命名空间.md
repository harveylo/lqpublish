- Linux下的**命名空间(namespace)**技术是一种用于隔离内核资源的方式
- 如字面意思，不同的命名空间下资源的标识互不影响，以此做到资源的隔离
- Linux下有**八种命名空间**，分别可以做到隔离八大类的资源
	- | namespace名称 | 使用的标识 - Flag | 控制内容 |
	  | ---- | ---- | ---- |
	  | Cgroup | CLONE_NEWCGROUP | Cgroup root directory cgroup 根目录 |
	  | IPC | CLONE_NEWIPC | System V IPC, POSIX message queues信号量，消息队列 |
	  | Network | CLONE_NEWNET | Network devices, stacks, ports, etc.网络设备，协议栈，端口等等 |
	  | Mount | CLONE_NEWNS | Mount points挂载点 |
	  | PID | CLONE_NEWPID | Process IDs进程号 |
	  | Time | CLONE_NEWTIME | 时钟 |
	  | User | CLONE_NEWUSER | 用户和组 ID |
	  | UTS | CLONE_NEWUTS | 系统主机名和 NIS(Network Information Service) 主机名（有时称为域名） |
- 使用系统调用``clone``可以创建新的命名空间