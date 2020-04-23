# 线程池
> 同一管理线程，手机空闲线程，节约创建线程所耗费的资源。
## 1.线程池核心参数
### 1.1 corePoolSize
> 核心线程数
### 1.2 maximumPoolSize
> 最大线程数
### 1.3 keepAliveTime
> 线程存活时间
### 1.4 workQueue
> 任务队列
## 1.4.1 常见工作队列
* SynchronousQueue
> 直接交接型队列
*  LinkedBlockingQueue
> 无界队列
*  ArrayBlockingQueue
> 有界队列
### 1.5 threadFactory
> 线程构建器
### 1.6 Handler
> 任务拒绝策略
## 2. 创建线程规则
1.	如果线程数小于corePoolSize ,即使其他工作线程处于空闲状态，也会创建一个新线程来运行新任务。
2.	如果线程数等于（或大于）corePoolSize但少于maximumPoolSize ,则将任务放入P人列。
3.	如果队列已满，并且线程个数小于maxPoolSize ,则创建一个新线程来运行任务。
4.	如果队列已满，并且线程数大于或等于maxPoolSize,则拒绝该任务。
## 3. 常见线程池
### 3.1 FixedThreadPool
> 固定线程数量的线程池
### 3.2 SingleThreadExecutor
> 只有一个线程处理任务
### 3.3 CachedThreadPool
> 最大线程数量取决于Integer.MAX_VALUE,可以自动回收空闲线程
### 3.4 ScheduledThreadPool
> 延期执行任务或周期执行任务
## 4. 如何确定线程数量
* CPU密集型
> 线程数为CPU核心数的1-2倍
* IO型
> 大于CPU核心数多倍，线程数 = CPU核心数*(1+平均等待时间、平均工作时间)
## 5. 线程池状态
* RUNNING : 能接受新提交的任务，并且也能处理阻塞队列中的任务
* SHUTDOWN : 不在接收新提交的任务，但可以处理队列中的任务
* STOP : 不在接收新的提交任务，也不处理队列任务
* TIDYING : 所有的任务都已终止
* TERMINATED : terminated()方法执行完成后进入该状态

![](./线程池状态.png)
## 6. 线程池组成部分
### 6.1 线程池管理器
> 协调管理线程池状态、线程的执行、任务队列的存储。
### 6.2 工作线程
> 用于处理外部的任务
### 6.3 任务队列
> 任务过多时，放入任务队列进行存储，有线程空闲时从队列中获取任务执行
### 6.4 任务接口
> 规范任务的类型
## 7. Executor
* Executor
> 顶层接口，只包含了一个执行任务的方法
* ExecutorService
> 继承了Executor并定义了终止、提交任务等方法
* AbstractExecutorService
> ExecutorService的实现，实现了submit，invokeAll，invokeAny 等
* ThreadPoolExecutor
> 线程池的最终实现
