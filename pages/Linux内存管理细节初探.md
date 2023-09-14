# 64 bits Canonical Address
	- **Low Canonical Address**：即用户地址空间，**[[$red]]==地址的高17位全为0==**
	- **High Canonical Address**：即内核地址空间，[[$red]]==**地址的高17位全为1**==
	- 通过判断高17位的情况可以快速判断地址的合法性：
		- **全是1**，内核空间，用户态无法直接访问
		- **全是0**，用户空间
		- **有0有1**，non-canonical address，非法地址，处于**canonical address空洞中**
	- ![image.png](https://cdn.xiaolincoding.com//mysql/other/532e6cdf4899588f8b873b6435cba2d8.png){:height 472, :width 471}
	- 在代码段和数据段之间有一段不可读写保护段，防止程序读写数据段时月结访问到代码段
	- 大多数操作系统中，**数值较小的地址[[$red]]==通常被认为是不合法的==**，因此可以给空指针赋0(NULL)
- # 用户态虚拟空间的管理
	- 内存管理的相关信息实际上页储存在用于管理任务的**``task_struct``**中
		- 因为任务管理和内存息息相关
	- ```
	  struct task_struct {
	          // 进程id
	  	    pid_t				pid;
	          // 用于标识线程所属的进程 pid
	  	    pid_t				tgid;
	          // 进程打开的文件信息
	          struct files_struct		*files;
	          // 内存描述符表示进程虚拟地址空间
	          struct mm_struct		*mm;
	  
	          //.......... 省略 .......
	  }
	  ```
	- 成员变量``mm``专门用于描述进程虚拟地址空间，这个结构体变量包含了之前介绍的进程虚拟空间的全部信息
	- 每个进程都有唯一的``mm_struct``变量，也即每个进程的虚拟地址空间都是独立的，互不干扰
	- 在调用``fork``函数创建进程时，``mm_struct``变量会随着``task_struct``的创建而创建
		- ```
		  long _do_fork(unsigned long clone_flags,
		  	      unsigned long stack_start,
		  	      unsigned long stack_size,
		  	      int __user *parent_tidptr,
		  	      int __user *child_tidptr,
		  	      unsigned long tls)
		  {
		          //......... 省略 ..........
		  	struct pid *pid;
		  	struct task_struct *p;
		  
		          //......... 省略 ..........
		      // 为进程创建 task_struct 结构，用父进程的资源填充 task_struct 信息
		  	p = copy_process(clone_flags, stack_start, stack_size,
		  			 child_tidptr, NULL, trace, tls, NUMA_NO_NODE);
		  
		           //......... 省略 ..........
		  }
		  ```
	- 使用``copy_process``直接用父进程的内容填充子进程
		- ```
		  static __latent_entropy struct task_struct *copy_process(
		  					unsigned long clone_flags,
		  					unsigned long stack_start,
		  					unsigned long stack_size,
		  					int __user *child_tidptr,
		  					struct pid *pid,
		  					int trace,
		  					unsigned long tls,
		  					int node)
		  {
		  
		      struct task_struct *p;
		      // 创建 task_struct 结构
		      p = dup_task_struct(current, node);
		  
		          //....... 初始化子进程 ...........
		  
		          //....... 开始继承拷贝父进程资源  .......      
		      // 继承父进程打开的文件描述符
		  	retval = copy_files(clone_flags, p);
		      // 继承父进程所属的文件系统
		  	retval = copy_fs(clone_flags, p);
		      // 继承父进程注册的信号以及信号处理函数
		  	retval = copy_sighand(clone_flags, p);
		  	retval = copy_signal(clone_flags, p);
		      // 继承父进程的虚拟内存空间
		  	retval = copy_mm(clone_flags, p);
		      // 继承父进程的 namespaces
		  	retval = copy_namespaces(clone_flags, p);
		      // 继承父进程的 IO 信息
		  	retval = copy_io(clone_flags, p);
		  
		        //...........省略.........
		      // 分配 CPU
		      retval = sched_fork(clone_flags, p);
		      // 分配 pid
		      pid = alloc_pid(p->nsproxy->pid_ns_for_children);
		  
		       //..........省略.........
		  }
		  ```
	- ``copy_mm``函数实际完成了子进程虚拟空间``mm_struct``结构的创建和初始化
		- ```
		  static int copy_mm(unsigned long clone_flags, struct task_struct *tsk)
		  {
		      // 子进程虚拟内存空间，父进程虚拟内存空间
		  	struct mm_struct *mm, *oldmm;
		  	int retval;
		  
		      //    ...... 省略 ......
		  
		  	tsk->mm = NULL;
		  	tsk->active_mm = NULL;
		      // 获取父进程虚拟内存空间
		  	oldmm = current->mm;
		  	if (!oldmm)
		  		return 0;
		  
		      //    ...... 省略 ......
		      // 通过 vfork 或者 clone 系统调用创建出的子进程（线程）和父进程共享虚拟内存空间
		  	if (clone_flags & CLONE_VM) {
		          // 增加父进程虚拟地址空间的引用计数
		  		mmget(oldmm);
		          // 直接将父进程的虚拟内存空间赋值给子进程（线程）
		          // 线程共享其所属进程的虚拟内存空间
		  		mm = oldmm;
		  		goto good_mm;
		  	}
		  
		  	retval = -ENOMEM;
		      // 如果是 fork 系统调用创建出的子进程，则将父进程的虚拟内存空间以及相关页表拷贝到子进程中的 mm_struct 结构中。
		  	mm = dup_mm(tsk);
		  	if (!mm)
		  		goto fail_nomem;
		  
		  good_mm:
		      // 将拷贝出来的父进程虚拟内存空间 mm_struct 赋值给子进程
		  	tsk->mm = mm;
		  	tsk->active_mm = mm;
		  	return 0;
		  
		      //    ...... 省略 ......
		  }
		  ```
			- 如果调用``vfork``或者``clone``函数创建子进程，``CLONE_VM``标识会被置为``true``
				- 此时创建的子进程**将共享父进程的地址空间**
				- [[$blue]]==**如此一来，创建的进程实际上就是线程**==
					- Linux实际上**[[$red]]==并不严格区分进程和线程==**，**是否共享地址空间是linux下进程和线程的[[$red]]==本质区别==**
					- 即，**线程对内核来说只是一个共享特定资源的进程**
	- [[$red]]==**内核线程没有用于描述地址空间的**==``mm_struct``，内核线程中的``mm``成员变量指向``NULL``
		- 在内核线程被调度时，会把上一个被调度的用户进程的mm直接复制给内核线程
		- 内核线程并不访问用户地址空间，这么做可以避免分配``mm_struct``的开销
	- **父进程和子进程的区别，进程和现成的区别，内核线程和用户进程的区别都与``mm_struct``相关**
	- ## 用户态和内核态虚拟地址空间的划分
		- ``mm_struct``中的``unsigned long task_size``变量定义了用户态和内核态地址空间之间的**[[$red]]==分界线==**
			- 在32位系统中，此变量的值为0xc0000000
			- 在64位系统中，此变量的值位0x0007ffffffff000(用户最大虚拟内存地址(0x0000800000000000-页大小(4KB(0x1000))
	- ## 进程虚拟内存空间的布局
		- ![image.png](https://cdn.xiaolincoding.com//mysql/other/2b2dbb2b6ea19871152a3bf6566df205.png)
		- ```
		  struct mm_struct {
		    unsigned long task_size;    /* size of task vm space */
		    unsigned long start_code, end_code, start_data, end_data;
		    unsigned long start_brk, brk, start_stack;
		    unsigned long arg_start, arg_end, env_start, env_end;
		    unsigned long mmap_base;  /* base of mmap area */
		    unsigned long total_vm;    /* Total pages mapped */
		    unsigned long locked_vm;  /* Pages that have PG_mlocked set */
		    unsigned long pinned_vm;  /* Refcount permanently increased */
		    unsigned long data_vm;    /* VM_WRITE & ~VM_SHARED & ~VM_STACK */
		    unsigned long exec_vm;    /* VM_EXEC & ~VM_WRITE & ~VM_STACK */
		    unsigned long stack_vm;    /* VM_STACK */
		   //...... 省略 ........
		  }
		  ```
			- ``arg_start``，``arg_end``是参数列表的位置，``env_start``，``env_end``是环境变量位置，都位于栈中最高地址处
			- ``total_pages``表示和物理内存地址建立了映射关系的虚拟页的总数 ，并不代表目前已经占用的物理页的总数
			- ``locked_vm``表示被锁定不能在内存吃紧时被换入换出的内存页数
			- ``pinned_vm``表示既不能换入换出，甚至不能被移动的内存总页数
	- ## 虚拟内存区域的管理
		- ``vm_area_struct``描述了**虚拟内存区域(VMA)**的情况
			- 如代码段，堆，数据段等在内核看来都是虚拟内存区域
		- ```
		  struct vm_area_struct {
		  
		  	unsigned long vm_start;		/* Our start address within vm_mm. */
		  	unsigned long vm_end;		/* The first byte after our end address
		  					   within vm_mm. */
		  	/*
		  	 * Access permissions of this VMA.
		  	 */
		  	pgprot_t vm_page_prot;
		  	unsigned long vm_flags;	
		  
		  	struct anon_vma *anon_vma;	/* Serialized by page_table_lock */
		      struct file * vm_file;		/* File we map to (can be NULL). */
		  	unsigned long vm_pgoff;		/* Offset (within vm_file) in PAGE_SIZE
		  					   units */	
		  	void * vm_private_data;		/* was vm_pte (shared mem) */
		  	/* Function pointers to deal with this struct. */
		  	const struct vm_operations_struct *vm_ops;
		  }
		  ```
		- 一块虚拟地址空间的范围是**``[vm_start, vm_end)``**
		- ![image.png](https://cdn.xiaolincoding.com//mysql/other/4956390184c51584798be1e5aaabedd1.png){:height 402, :width 271}
		- ### 虚拟内存区域的访问权限和行为规范
		  collapsed:: true
			- ``vm_page_prot``定义底层内存管理架构中页级别的**访问控制权限**
				- 可以直接被用在底层页表中，较为具体
				- 在进行从虚拟页到物理页的转换过程时，访问权限由此属性确定
			- ``vm_flags``
				- 定义整个虚拟内存区域的访问权限和行为规范，相较于``vm_page_prot``更高级，更抽象
				- 并不负责某个具体独立页
				- | vm_flags | 访问权限 |
				  | ---- | ---- | ---- |
				  | VM_READ | 可读 |
				  | VM_WRITE | 可写 |
				  | VM_EXEC | 可执行 |
				  | VM_SHARD | 可多进程之间共享 |
				  | VM_IO | 可映射至设备 IO 空间 |
				  | VM_RESERVED | 内存区域不可被换出 |
				  | VM_SEQ_READ | 内存区域可能被顺序访问 |
				  | VM_RAND_READ | 内存区域可能被随机访问 |
				- 通过``posix_fadvise``，``madvise``系统调用暗示操作系统相关内存区域的顺序读写和随机读写的偏向性
			- ``vma->vm_page_prot = vm_get_page_prot(vma->vm_flags)``可以从``vm_flags``获取具体页面访问权限
			- **各段一般默认所需权限**
				- ![image.png](https://cdn.xiaolincoding.com//mysql/other/600ef23c454d9f3653ece44debaaf3a7.png){:height 424, :width 285}
		- ### 关联内存中的映射关系
		  collapsed:: true
			- ``anon_vma``表明是匿名映射，映射到物理内存上，在磁盘上没有具体对应的持久化载体
				- 申请时超过128KB的内存区块分配在文件映射与匿名映射区，具体管理时使用``structanon_vma``管理
			- ``vm_file``表示文件映射，在磁盘上有具体对应的持久化载体
				- ``vm_pgoff``表示映射进虚拟内存中文件内容相较于文件头部的偏移量
			- ``vm_private_data``用于存储VMA中的私有数据
	- ## 针对虚拟内存区域的相关操作
- # [[Linux 物理内存分配算法]]