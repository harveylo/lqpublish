# 问题收集：
collapsed:: true
	- **多个线程阻塞在同一个mutex(信号量，条件变量等)，若等待资源可用，这些线程谁先被唤醒**
		- 取决于系统的线程调度策略
		- 在linux下，会根据当前线程调度策略(``SCHED_RR``, ``SCHED_FIFO``)和线程之间的优先级决定
- # 工具类
	- ## ``conditional_variable``
		- **条件变量**常和``unique_lock``一起使用以提升性能
		- 条件变量必须和一个锁(``unique_lock``)一起使用
		- 其使用流程简单来说就是：
			- 先获取一个锁
			- 如果条件变量没有被通知，那么原子开锁并且yield，直到被条件变量通知
		- ### 方法
			- ``wait``
				- 等待条件变量可用，如果不可用则原子开锁并阻塞当前线程
				- 可以传入一个额外的谓词，则条件变量会在谓词会假时一直等待
				- 避免了假唤醒导致的逻辑错误
			- ``wait_for``
				- 和`wait`相同，但是可以设置等待超时的时间
			- ``notify_one``
				- 通知(唤醒)一个正在等待的线程
			- ``notify_all``
				- 通知所有正在等待的线程
		- ### 注意事项
			- **虚假唤醒** 是指并没有其他线程调用``notify``函数的时候，因为``wait``而
	- ## ``counting_semaphore``, ``binary_semaphore``
		- 在C++20从才引入，如果在C++20之前要使用信号量，那么建议使用POSIX信号量
		- ``counting_semaphore``在声明时指定其最大值，如：``counting_semaphore<20> s(10)``，表示最大值20，初始值为10
		- ``binary_semaphore``等价于``counting_semaphore<1>``
		- 主要操作为``release``和``acquire``，另有``try_acquire``，``try_acquire_for``和``try_acquire_until``，人如其名，很好理解
	- ## ``semaphore``中的``sem_t``
		- POSIX中的信号量
		- 声明：``sem_t s``
		- 初始化：``sem_init(&s, 0, 0)``
			- 第一个参数是信号量的地址
			- 第二个参数若为0表示此信号量在一个进程的线程间共享，不为0表示进程间共享
			- 第三个参数是初始值
		- p操作：``sem_post(&s)``，将信号量的值增加1
		- v操作：``sem_wait(&v)``，如果信号量的值为0则会阻塞直到信号量的值大于0，返回时会将信号量的值减1
- # 示例
	- ## 生产者消费者模型
		- 根据生产者和消费者数量的不同有很多特化版本
			- 但生产者单消费者有最高性能，最轻量化的实现
			- 多生产者多消费者的版本优化不太好做但是实现出来之后的适用性最强
		- ### 使用条件变量的版本
			- ```c++
			  template<typename T>
			  class BlockQueue{
			  public:
			      BlockQueue(int cap):mtx(),_full_cond(),_empty_cond(),_count(0),cap(cap){}
			      void push(T val){
			          unique_lock<mutex> lock(mtx);
			          while(cap<=_count){
			              _full_cond.notify_all();
			              _empty_cond.wait(lock);
			          }
			          _que.push_back(val);
			          _count++;
			          printf("%d has been pushed in the que, size now: %d\n ",val,_count);
			          _full_cond.notify_one();
			      }
			      T pop(){
			          unique_lock<mutex> lock(mtx);
			          while(_count==0){
			              _empty_cond.notify_all();
			              _full_cond.wait(lock);
			          }
			          auto cur = _que.front();
			          _que.pop_front();
			          _count--;
			          printf("Value poped: %d, size now: %d\n",cur,_count);
			          _empty_cond.notify_one();
			          return cur;
			      }
			  private:
			  
			      mutex mtx;
			      condition_variable _full_cond;
			      condition_variable _empty_cond;
			      deque<T> _que;
			      int _count;
			      int cap;
			  };
			  ```
	- ### 使用信号量的版本
		- 待补全，不过只要理解了原理，使用信号量实现非常简单优雅
		- 详情可以去看leetcode上的并发题：[1188: 有限阻塞队列](https://leetcode.cn/problems/design-bounded-blocking-queue/)
		-