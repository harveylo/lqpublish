# 工具类
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
- # 示例
	- ## 生产者消费者模型
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
-