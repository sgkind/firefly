代码仓库地址：[David-Haim/concurrencpp at develop (github.com)](https://github.com/David-Haim/concurrencpp/tree/develop)

使用concurrencpp，c++开发者可以更容易、更安全的使用tasks、exeuctors和coroutines开发高并发应用程序。concurrencpp使需要异步运行的大型程序分割成同时运行的小任务，任务间协作获取想要的结果。
concurrencpp的优势：

* 不需要使用底层同步原语，如锁或者条件变量，来开发现代并发程序.(对于应用开发者，concurrencpp框架内部使用了同步原语)
* 能够写可以根据硬件资源自动扩展的高并发和并行程序(线程数根据cpu核数自动扩展+c++20的协程)
* 通过使用C++20协程和co_await可以容易的写非阻塞和看起来像同步程序的代码
* 可以减少竞争条件、数据竞争和死锁的发生
* 提供了几种完全继承协程的通用执行器
* 可以对执行器进行扩展

## 架构

### runtime

```plantuml
class details::executor_collection {
  - std::mutex m_lock
  - std::vector<std::shared_ptr<executor> m_executors

  + void register_executor(std::shared_ptr<executor> executor)
  + void shutdown_all()
}


struct runtime_options {
  + size_t max_cpu_threads
  + std::chrono::milliseconds max_thread_pool_executor_waiting_time

  + size_t max_background_threads
  + std::chrono::milliseconds max_background_executor_waiting_time

  + std::chrono::milliseconds max_timer_queue_waiting_time

  + runtime_options() noexcept
  + runtime_options(const runtime_options&) noexcept
  + runtime_options& operator=(const runtime_options&) noexcept
}


class runtime {
  - std::shared_ptr<inline_executor> m_inline_executor
  - std::shared_ptr<thread_pool_executor> m_thread_pool_executor
  - std::shared_ptr<thread_pool_executor> m_background_executor
  - std::shared_ptr<thread_executor> m_thread_executor

  - details::executor_collection m_registered_executors
  - std::shared_ptr<timer_queue> m_timer_queue

  + runtime()
  + runtime(const runtime_option& options)
  + std::shared_ptr<concurrencpp::timer_queue> timer_queue() const noexcept
  + std::shared_ptr<concurrencpp::inline_executor> inline_executor() const noexcept
  + std::shared_ptr<concurrencpp::thread_pool_executor> thread_pool_executor() const noexcept
  + std::shared_ptr<concurrencpp::thread_pool_executor> background_executor() const noexcept
  + std::shared_ptr<concurrencpp::thread_executor> thread_executor() const noexcept

  + std::shared_ptr<concurrencpp::worker_thread_executor> make_worker_thread_executor()
  + std::shared_ptr<concurrencpp::manual_executor> make_manual_executor()

  + static std::tuple<unsigned int, unsigned int, unsigned int> version() noexcept
  + template<class executor_type, class... argument_types>
        std::shared_ptr<executor_type> make_executor(argument_types&&... arguments)
}


runtime o-- details::executor_collection
runtime ..> runtime_options
```

其中runtime中有:

* m_timer_queue:  timer_queue

* m_inline_executor:  inline_executor

* m_thread_pool_executor:  thread_pool_executor

* m_background_executor: thread_pool_executor

* m_thread_executor: thread_executor

另外runtime也可以注册：

* worker_thread_executor

* manual_executor

## timer

定时器的实现，定时器自动执行器，定时器到期后任务在自带的执行器上执行。timer_queue检查定时器逻辑和现在time库中的检查逻辑差别不大。

类图

```plantuml
class timer_state_base {
  - const std::weak_ptr<timer_queue> m_timer_queue
  - const std::shared_ptr<executor> m_executor
  - const size_t m_due_time
  - std::atomic_size_t m_frequency
  - time_point m_deadline
  - std::atomic_bool m_cancelled
  - const bool m_is_oneshot

  - static time_point make_deadline(milliseconds diff) noexcept

  + timer_state_base(size_t due_time, size_t frequency, std::shared_ptr<executor> executor, std::weak_ptr<timer_queue> timer_queue, bool is_oneshot) noexcept
  + virtual ~timer_state_base() noexcept
  + virtual void execute() = 0
  + void fire()
  + bool expired(const time_point now) const noexcept
  + time_point get_deadline() const noexcept
  + size_t get_frequency() const noexcept
  + size_t get_due_time() const noexcept
  + bool is_oneshot() const noexcept
  + std::shared_ptr<executor> get_executor() const noexcept
  + std::weak_ptr<timer_queue> get_timer_queue() const noexcept
  + void set_new_frequency(size_t new_frequency) noexcept
  + void cancel() noexcept
  + bool cancelled() const noexcept
}

class timer_state {
  - callable_type m_callable

  + timer_state(size_t due_time, size_t frequency, std::shared_ptr<executor> executor, std::weak_ptr<timer_queue> timer_queue, bool is_oneshot, given_callable_type&& callable)
  + void execute() override
}

class timer {
  - std::shared_ptr<timer_state_base> m_state
  - void throw_if_empty(const char* error_message) const

  + timer(std::shared_ptr<timer_state_base> timer_impl) noexcept
  + void cancel()
  + std::chrono::milliseconds get_due_time() const
  + std::shared_ptr<executor> get_executor() const
  + std::weak_ptr<timer_queue> get_timer_queue() const
  + std::chrono::milliseconds get_frequency() const
  + void set_frequency(std::chrono::milliseconds new_frequency)

  + explicit operator bool() const noexcept
}

class timer_queue {
  - std::atomic_bool m_atomic_abort
  - std::mutex m_lock
  - std::vector<std::pair<timer_ptr, timer_request>> m_request_queue
  - details::thread m_worker
  - std::condition_variable m_condition
  - bool m_abort
  - bool m_idle
  - const std::chrono::milliseconds m_max_waiting_time

  - details::thread ensure_worker_thread(std::unique_lock<std::mutex>& lock)
  - void add_internal_timer(std::unique_lock<std::mutex>& lock, timer_ptr new_timer)
  - void remove_internal_timer(timer_ptr existing_timer)
  - void add_timer(std::unique_lock<std::mutex>& lock, timer_ptr new_timer)
  - lazy_result<void> make_delay_object_impl(std::chrono::milliseconds due_time,
        std::shared_ptr<concurrencpp::timer_queue> self,
       std::shared_ptr<concurrencpp::executor> executor)

  - timer_ptr make_timer_impl(size_t due_time,
                                  size_t frequency,
                                  std::shared_ptr<concurrencpp::executor> executor,
                                  bool is_oneshot,
                                  callable_type&& callable)
  - void work_loop()

  + timer_queue(std::chrono::milliseconds max_waiting_time)
  + ~timer_queue() noexcept
  + void shutdown()
  + bool shutdown_requested() const noexcept
  + timer make_timer(std::chrono::milliseconds due_time,
                         std::chrono::milliseconds frequency,
                         std::shared_ptr<concurrencpp::executor> executor,
                         callable_type&& callable,
                         argumet_types&&... arguments)
  + timer make_one_shot_timer(std::chrono::milliseconds due_time,
                                  std::shared_ptr<concurrencpp::executor> executor,
                                  callable_type&& callable,
                                  argumet_types&&... arguments)
  + lazy_result<void> make_delay_object(std::chrono::milliseconds due_time, std::shared_ptr<concurrencpp::executor> executor)
  + std::chrono::milliseconds max_worker_idle_time() const noexcept
}


timer_state_base <|-- timer_state

timer o-- timer_state_base

timer_queue o-- timer_state_base
```

类timer_state_base中部分字段的含义

* m_due_time: 

* m_frequency: 定时周期

* m_deadline: 下一次定时到期时间

* m_is_oneshot: 一次性定时器

类timer_queue中部分字段的含义

* m_request_queue: 保存添加删除timer请求的队列

* m_worker: 检查定时器是否到时的队列

timer自带执行器executor，timer_queue保存timer，并检查到期的定时器，然后调度timer->execute()执行定时器的callable

可参考点：

1. 定时器自带执行器

2. 定时检查线程等待指定最大等待时间内如果没有定时器会销毁，新添加timer后如果没有检查线程则创建

## thread

对std::thread的封装，可以设置线程名
类图

```plantuml
class thread {
  - std::thread m_thread
  - static void set_name(std::string_view name) noexcept

  + thread() noexcept = default
  + thread(thread&&) noexcept = default
  + thread(std::string name, callable_type&& callable)
  + thread& operator=(thread&& rhs) noexcept = default
  + std::thread::id get_id() const noexcept
  + static std::uintptr_t get_current_virtual_id() noexcept
  + bool joinable() const noexcept
  + void join()

  + static size_t hardware_concurrency() noexcept
}
```

## executor

类图

```plantuml
class executor {
  - static result<return_type> submit_bridge(executor_tag, executor_type&, callback_type callback, argument_types... arguments)
  - static result<return_type> bulk_submit_bridge(details::executor_bulk_tag, std::vector<task>& accumulator, callback_type callable)

  # static void do_post(executor_type& executor_ref, callable_type&& callable, argument_types&&... arguments)
  # static auto do_submit(executor_type& executor_ref, callable_type&& callable, argument_type&&... arguments)
  # static std::vector<concurrencpp::result<return_type>> do_bulk_submit(executor_type& executor_ref, std::span<callable_type> callable_list)

  + executor(std::string_view name)
  + const std::string name
  + virtual void enqueue(task task) = 0
  + virtual void enqueue(std::span<task> tasks) = 0
  + virtual int max_concurrency_level() const noexcept = 0
  + virtual bool shutdown_requested() const = 0
  + virtual bool shutdown() = 0

  + void post(callable_type&& callable, argument_type&&... arguments)
  + auto submit(callable_type&& callable, argument_type&&... arguments)
  + void bulk_post(std::span<callable_type> callable_list)
  + std::vector<result<return_type>> bulk_submit(std::span<callable_type> callable_list)
}


class derivable_executor {
  - concrete_executor_type& self() noexcept

  + derivable_executor(std::string_view name)
  + void post(callable_type&& callable, argument_types&&... arguments)
  + void submit(callable_type&& callable, argument_types&&... arguments)
  + void bulk_post(std::span<callable_type> callable_list)
  + std::vector<result<return_type>> bulk_submit(std::span<callable_type> callable_list)
}

executor <|-- derivable_executor
```

executor为执行器的基类，提供接口

derivable_executor 用户自定义执行器的基类。

* void post(callable_type& callable, argument_type&&... arguments)用于添加无返回值的任务

* void submit(callable_type& callable, argument_type&&... arguments)用于添加有返回值的任务

### inline_executor

inline_executor不创建单独的执行线程，任务在添加到执行器时在添加任务的线程上执行

类图

```plantuml
class inline_executor {
  - std::atomic_bool m_abort

  - void throw_if_aborted() const

  + inline_executor() noexcept
  + void enqueue(task task) override
  + void enqueue(std::span<task> tasks) override
  + int max_concurrency_level() const noexcept override
  + int shutdown() override
  + bool shutdown_requested() const override
}

class executor{}

executor <|-- inline_executor
```

inline_executor中没有单独的执行线程，任务在enquene时在线程上执行

```
void enqueue(task task) override {
  throw_if_aborted();
  task();
}

void enqueue(std::span<task> tasks) override {
  throw_if_aborted();
  for (auto& task : tasks) {
    task();
  }
}
```

### thread_executor

thread_executor每添加一个任务就创建一个线程执行此任务，任务执行完成后线程被销毁，因此线程不会被重用。用于执行长时运行的任务。

类图

```plantuml
class thread_executor {
  - std::mutex m_lock
  - std::list<details::thread> m_workers
  - std::condition_variable m_condition
  - std::list<thread> m_last_retired
  - bool m_abort
  - std::atomic_bool m_atomic_abort

  - void enqueue_impl(std::unique_lock<std::mutex>& lock, task& task)
  - void retire_worker(std::list<details::thread>::iterator it)

  + thread_executor()
  + ~thread_executor() noexcept

  + void enqueue(task task) override
  + void enqueue(std::span<task> tasks) override
  + int max_concurrency_level() const noexcept override
  + bool shutdown_requested() const override
  + void shutdown() override
}

class derivable_executor {}

class executor{}

executor <|-- derivable_executor
derivable_executor <|-- thread_executor
```

其中部分字段的含义

* m_lock: 锁
* m_workers: 工作线程
* m_last_retired: 已经结束的线程
* m_abort: 执行器调用shutdown，m_abort被设置为true，但是此时会等待线程执行结束
* m_atomic_abort: 执行器调用shutdown后m_atomic_abort设置为true

### manual_executor

manual_executor没有自带线程，任务添加后不会自动执行。如果需要执行任务，需要手动调用相关的接口同步执行若干任务。任务是在调用执行接口的线程上执行的。

```plantuml
class executor{}
class derivable_executor{}

executor <|-- derivable_executor

class manual_executor {
  - mutable std::mutex m_lock
  - std:deque<task> m_tasks
  - std::condition_variable m_condition
  - bool m_abort
  - std::atomic_bool m_atomic_abort

  - static std::chrono::system_clock::time_point to_system_time_point
  - static std::chrono::system_clock::time_point time_point_from_now

  - size_t loop_impl
  - size_t loop_until_impl
  - void wait_for_tasks_impl(size_t count)
  - size_t wait_for_tasks_impl(size_t count, std::chrono::time_point<std::chrono::system_clock> deadlie)

  + manual_executor()

  + void enqueue(task task) override
  + void enqueue(std::span<task> tasks) override
  + int max_concurrency_level() const noexcept override
  + void shutdown override
  + bool shutdown_requested() const override

  + size_t size() const
  + bool empty() const

  + size_t clear()

  + bool loop_once()
  + bool loop_once_for(std::chrono::milliseconds max_waiting_time)

  + bool loop_once_until(std::chrono::time_point<clock_type, duration_type> timeout_time)

  + size_t loop(size_t max_count)
  + size_t loop_for(size_t max_count, std::chrono::milliseconds max_waiting_time)

  + size_t loop_until(size_t max_count, std::chrono::time_point<clock_type, duration_type> timeout_time)

  + void wait_for_task()
  + bool wait_for_task_for(std::chrono::milliseconds max_waiting_time)

  + bool wait_for_task_until(std::chrono::time_point<clock_type, duration_type> timeout_time)

  + wait_for_tasks(size_t count)
  + size_t wait_for_tasks_for(size_t count, std::chrono::milliseconds max_waiting_time)

  + size_t wait_for_tasks_until(size_t count, std::chrono::time_point<clock_type, duration_type> timeout_time)
}

derivable_executor <|-- manual_executor
```

类中部分变量含义：

* m_tasks: 此executor中的task队列

部分接口作用：

* bool loop_once() 执行一个任务
* bool loop_once_for(std::chrono::milliseconds max_waiting_time) 执行一个任务，最长等待max_waiting_time时间
* size_t loop(size_t max_count) 执行max_count个任务
* size_t loop_for(size_t max_count, std::chrono::milliseconds max_waiting_time) 最多执行max_count个任务，最大等待时间为max_waiting_time
* size_t clear() 清除所以未执行的task
* void wait_for_task() 等待一个task被添加
* void wait_for_task_for(std::chrono::milliseconds max_waiting_time) 等待一个task被添加，最大等待时间为max_waiting_time
* void wait_for_tasks(size_t count) 等待count个任务被添加
* void wait_for_tasks_for(size_t count, std::chrono::milliseconds max_waiting_time) 等待count个任务被添加，最大等待时间为max_waiting_time
* void shutdown() 关闭excutor
* bool shutdown_requested() 是否shutdown被关闭

### worker_thread_executor

一个工作执行器和一个执行任务队列，可以向其添加任务，执行器会开一个线程一直执行被添加的任务。用于程序想在同一个线程上执行相关的任务。

类图

```plantuml
class executor{}
class derivable_executor{}

executor <|-- derivable_executor

class worker_thread_executor {
  - std::deque<task> m_private_queue
  - std::atomic_bool m_private_atomic_abort
  - details::thread m_thread
  - std::mutex m_lock
  - std::deque<task> m_public_queue
  - details::binary_semaphore m_semaphore
  - std::atomic_bool m_atomic_abort
  - bool m_abort

  - bool drain_queue_impl()
  - bool drain_queue()
  - void wait_for_task(std::unique_lock<std::mutex>& lock)
  - void work_loop()

  - void enqueue_local(task& task)
  - void enqueue_local(std::span<task> task)

  - void enqueue_foreign(task& task)
  - void enqueue_foreign(std::span<task> task)

  + worker_thread_executor()

  + void enqueue(task task) override
  + void enqueue(std::span<task> tasks) override

  + int max_concurrency_level() const noexcept override

  + bool shutdown_requested() const override
  + void shutdown() override
}

derivable_executor <|-- worker_thread_executor
```

类中部分变量含义：

* m_private_queue: 类中使用的task队列，执行任务时用
* m_private_atomic_abort: 没什么用
* m_thread: 执行线程
* m_public_queue: 保存外部添加task的队列，执行时将m_public_queue和m_private_queue交换

类中部分函数作用：

* bool drain_queue_impl() 执行m_private_queue中的所有任务
* void wait_for_task(std::unique_lock<std::mutex>& lock) 等待直到m_public_queue中有任务
* bool drain_queue() 执行m_public_queue中的所有任务
* void work_loop() 线程中的执行函数，一直执行任务
* void enqueue_local(task& task) 向m_private_queue中添加task
* void enqueue_local(std::span<task> tasks) 向m_private_queue中添加tasks
* void enqueue_foreign(task& task) 向m_public_queue中添加task
* void enqueue_foreign(std::span<task> tasks) 向m_public_queue中添加tasks
* void enqueue(task task) 向executor中添加task，如果向同一线程中添加则直接添加到m_private_queue中，如果向不同线程中添加，则添加到m_public_queue中

### thread_pool_executor

具有一个线程池的通用执行器,此执行器类在runtime中有两个对象：

* thread pool executor 适用于不会阻塞的短时运行的cpu密集型任务。用户在执行非阻塞任务时推荐使用此执行器。

* background executor 具有大量线程的线程池执行器，适用于短时阻塞任务，比如文件io和db查询等。和上面thread pool executor的区别，添加任务时调用不同的接口

类图

```plantuml
struct padded_flag {
  + std::atomic<status> flag {status::active}
}

class details::idle_worker_set {
  - std::atomic_intptr_t m_approx_size
  - const std::unique_ptr<padded_flag[]> m_idle_flags
  - const size_t m_size

  - bool try_acquire_flag(size_t index) noexcept

  + idle_worker_set(size_t size)

  + void set_idle(size_t idle_thread) noexcept
  + void set_active(size_t idle_thread) noexcept

  + size_t find_idle_worker(size_t caller_index) noexcept
  + void find_idle_workers(size_t caller_index, std::vector<size_t>& result_buffer, size_t max_count) noexcept
}

class details::thread_pool_worker {
  - std::deque<task> m_private_queue
  - std::vector<size_t> m_idle_worker_list
  - std::atomic_bool m_atomic_abort
  - thread_pool_executor& m_parent_pool
  - const size_t m_index
  - const size_t m_pool_size
  - const std::chrono::milliseconds m_max_idle_time
  - std::mutex m_lock
  - std::queue<task> m_public_queue
  - binary_semaphore m_semaphore
  - bool m_idle
  - bool m_abort
  - std::atmoic_bool m_task_found_or_abort
  - thread m_thread

  - void balance_work
  - bool wait_for_task(std::unique_lock<std::mutex>& lock)
  - bool drain_queue_impl()
  - bool drain_queue()
  - void work_loop()
  - void ensure_worker_active(bool first_enqueuer, std::unique_lock<std::mutex>& lock)

  + void enqueue_foreign(task& task)
  + void enqueue_foreign(std::span<task> tasks)
  + void enqueue_foreigh(std::deque<task>::iterator begin, std::deque<task>::iterator end)
  + void enqueue_foreigh(std::span<task>::iterator begin, std::span<task>::iterator end)

  + void enqueue_local(task& task)
  + void enqueue_local(std::span<task> tasks)

  + void shutdown()

  + std::chrono::milliseconds max_worker_idle_time() const noexcept
  + bool appears_empty() const noexcept
}

class executor {}
class derivable_executor {}

executor <|-- derivable_executor

class thread_pool_executor {
  - std::vector<details::thread_pool_worker> m_workers
  - std::atomic_size_t m_round_robin_cursor
  - details::idle_worker_set m_idle_workers
  - std::atomic_bool m_abort

  - void mark_worker_idle(size_t index) noexcept
  - void mark_worker_active(size_t index) noexcept
  - void find_idle_workers(size_t caller_index, std::vector<size_t>& buffer, size_t max_count) noexcept
  - details::thread_pool_workers& worker_at(size_t index) noexcept

  + void enqueue(task task) override
  + void enqueue(std::span<task> tasks) override

  + int max_concurrency_level() const noexcept override

  + bool shutdown_requested() const override
  + void shutdown() override

  + std::chrono::milliseconds max_worker_idle_time() const noexcept
}

derivable_executor <|-- thread_pool_executor
thread_pool_executor *-- details::idle_worker_set
thread_pool_executor o-- details::thread_pool_worker
```

#### idle_worker_set

类中部分变量的含义：

* m_approx_size: 空闲线程的数量
* m_idle_flags: 保存线程状态
* m_size: 总的线程数量

类中部分函数的含义：

* bool try_acquire_flag(size_t index) 将序号为index的线程状态设置为活动
* void set_idle(size_t idle_thread) 将序号为idle_thread的线程状态设置为空闲
* void set_active(size_t idle_thread) 将序号为idle_thread的线程状态设置为活动
* size_t find_idle_worker(size_t caller_index) 从caller_index下一个线程开始，找到第一个空闲线程，并将其置为活动
* void find_idle_workers(size_t caller_index, std::vector<size_t>& result_buffer, size_t max_count) 从caller_index开始，找到最多max_count个空闲线程，并将其设置为活动

#### details::thread_pool_worker

thread_pool_worker中的逻辑和worker_thread_executor逻辑基本相同，其进行的工作可以参考worker_thread_executor。区别点在于每一个thread_pool_executor中有多个details::thread_pool_worker，因此在每个details::thread_pool_worker中执行任务前会在thread_pool_executor内部的所有空闲details::thread_pool_worker间平衡任务。

#### worker_thread_executor

类中部分变量的含义：

* m_round_robin_cursor：添加任务时用来表示当前任务添加到哪个工作线程
* m_idle_workers: 对工作线程状态进行薄记和管理
* m_abort: 退出标记

类中部分函数作用：

* void mark_worker_idle(size_t index) 将第index个执行线程标记为空闲状态
* void mark_worker_active(size_t index) 将第index个执行线程标记为活动状态
* void find_idle_workers(size_t caller_index, std::vector<size_t>& buffer, size_t max_count) 找到最多max_count个空闲线程
* details::thread_pool_worker& worker_at(size_t index) 获取第index个工作线程
* void enqueue(task task) 向执行器添加单个任务
* void enqueue(std::span<task> tasks) 向执行器添加多个任务
* int max_concurrency_level() 并发数
* bool shutdown_requested() 执行器是否被关闭
* void shutdown() 关闭执行器
* std::chrono::milliseconds max_worker_idle_time() 工作线程最大工作时间

## results

### await_via_functor

```plantuml
class await_via_functor {
  - coroutine_handle<void> m_caller_handle
  - bool* m_interrupted

  + await_via_functor(coroutine_handle<void> caller_handle, bool* interrupted)
  + await_via_functor(await_via_functor&& rhs) noexcept
  + void operator()() noexcept
}

```
await_via_functor保存协程的句柄，执行协程
* void operator()() 恢复协程执行

### generator
```plantuml
class generator_state {
  - value_type* m_value
  - std::exception_ptr m_exception

  + generator<type> get_return_object()
  + suspend_always intial_suspend()
  + suspend_always final_suspend()
  + suspend_always yield_value(value_type& ref)
  + suspend_always yield_value(value_type&& ref)
  + void return void()
  + value_type& value()
  + void throw_if_exception()
}

class generator_iterator {
  - coroutine_handle<generator_state<type>> m_coro_handle

  + generator_iterator& operator++()
  + void operator++(int)
  + reference operator*()
  + pointer operator->()
}

class generator {
  - details::coroutine_handle<promise_type> m_coro_handle

  + generator(details::coroutine_handle<promise_type> handle)
  + generator(generator&& rhs)
  + ~generator()
  + explicit operator bool()
  + iterator begin()
}
```
其中generator_state是一个promise object，generator是一个协程对象

### consumer_context & producer_context

```plantuml
class wait_context {
  - std::mutex m_lock
  - std::condition_variable m_condition
  - bool m_ready

  + void wait()
  + bool wait_for(size_t milliseconds)
  + void notify()
}

class when_any_context {
  - std::atomic<const result_state_base*> m_status
  - coroutine_handle<void> m_coro_handle

  + bool any_result_finished() const noexcept
  + bool finish_processing() noexcept
  + const result_state_base* completed_result() const noexcept

  + void try_resume(result_state_base& completed_result) noexcept
  + bool result_inline(result_state_base& completed_result) noexcept
}

enum consumer_status {
  idle
  await
  wait_for
  when_any
}

class consumer_context {
  - consumer_status m_status
  - storage m_storage

  - void destroy() noexcept

  + void clear() noexcept
  + void resume_consumer(result_state_base& self) const
  
  + void set_await_handle(coroutine_handle<void> caller_handle) noexcept
  + void set_wait_for_context(const std::shared_ptr<wait_context>& wait_ctx) noexcept
  + void set_when_any_context(const std::shared_ptr<when_any_context>& when_any_context)
}

consumer_context o-- wait_context
consumer_context o-- when_any_context

```

#### wait_context
使用条件变量实现等待和通知的功能
* void wait() 等待m_ready变为true
* bool wait_for(size_t milliseconds) 等待m_ready变为true，最大等待时间为milliseconds
* void notify() 通知m_ready变为true

#### when_any_context
* bool any_result_finished() 结果是否处理完
* bool finish_processing() 将状态从处理中修改为处理完成
* void try_result(result_state_base& completed_result) 将状态修改为completed_result，然后恢复协程
* bool resume_inline(result_state_base& completed_result) 将状态修改为completed_result
* const result_state_base* completed_result() 获取状态

#### consumer_context
consumer_context可能包含下列三个类对象之一
+ coroutine_handle<void>
+ wait_context
+ when_any_context

* void set_await_handle(coroutine_handle<void> caller_handle) 通过协程句柄构造consumer_context
* void set_wait_for_context(const std::shared_ptr<wait_context>& wait_ctx) 通过wait_context构造consumer_context
* void set_when_any_context(const std::shared_ptr<when_any_context>& when_any_ctx) 通过when_any_context构造consumer_context

```plantuml
enum result_status {
  idle
  value
  exception
}

class producer_context {
  - storage m_storage
  - result_status m_status

  + void build_result(argument_types&&... arguments)
  + void build_exception(const std::exception_ptr& exception)
  + result_status status()
  + type get()

  type& get_ref()
}

producer_context ..> result_status
```

### lazy_result
```plantuml
class suspend_always{}

class lazy_final_awaiter {
  + coroutine_handle<void> await_suspend(coroutine_handle<promise_type> handle)
}

suspend_always <|-- lazy_final_awaiter

class lazy_result_state_base {
  # coroutine_handle<void> m_caller_handle

  + coroutine_handle<void> resume_caller()
  + coroutine_handle<void> await(coroutine_handle<void> caller_handle)
}

class lazy_result_state {
  - producer_context<type> m_producer
  
  + result_status status()
  + lazy_result<type> get_return_object()
  + void unhandled_exception()
  + suspend_always intial_suspend()
  + lazy_final_awaiter final_suspend()
  + void set_result(argument_type&&... arguments)
  + type get()
}

lazy_result_state <|-- lazy_result_state_base

class lazy_result {
  - details::coroutine_handle<details::lazy_result_state<type>> m_state

  - void throw_if_empty(const char* err_msg)
  - result<type> run_impl()

  + explicit operator bool()
  + auto operator co_await()
  + auto resolve()
  + result<type> run()
}
```

lazy_result_state是一个promise object

### result
```plantuml
enum pc_state {
  idle
  consumer_set
  comsumer_waiting
  consumer_done
  producer_done
}

class result_state_base {
  # std::atomic<pc_state> m_pc_state
  # consumer_context m_consumer
  # coroutine_handle<void> m_done_handle

  # void assert_done()

  + void wait()
  + void await(coroutine_handle<void> caller_handle)
  + pc_state when_any(const std::shared_ptr<when_any_context>& when_any_state)

  + void try_rewind_consumer()
}

class result_state {
  - producer_context<type> m_producer

  - void from_callable(std::true_type, callable_type&& callable)
  - void from_callable(std::false_type, callable_type&& callable)

  + void set_result(argument_type&&.. arguments)
  + void set_exception(const std::exception_ptr& error)
  + result_status status()
  + result_status wait_for(std::chrono::duration duration)
  + result_status wait_until(const std::chrono::time_point& timeout_time)

  + type get()
  + void from_callable(callable_type&& callable)
  + void complete_producer(coroutine_handle<void> done_handle)
  + void complete_consumer()
}

result_state_base *-- pc_state
result_state <|-- result_state_base
result_state o-- producer_context
```

#### result_state_base
* void wait() 等待producer任务执行完成
* bool await(coroutine_handle<void> caller_handle) 异步等待producer任务执行完成
* pc_state when_any(const std::shared_ptr<when_any_context>& when_any_state) ?
* void try_rewind_consumer() 回退consumer（将状态从consumer_set设置为idle）


#### 
## task

```plantuml
struct vtable {
  + move_destroy_fn: void (*)(void* src, void* dst) noexcept
  + execute_destroy_fn: void(*)(void* target)
  + destroy_fn: void(*)(void* target) noexcept

  + static constexpr bool trivially_copiable_destructible(decltype(move_destroy_fn) move_fn) noexcept
  + static constexpr bool trivially_destructable(decltype(destroy_fn) destroy_fn) noexcept
}

class callable_vtable {
  - static callable_type* inline_ptr(void* src) noexcept
  - static callable_type* allocated_ptr(void* src) noexcept
  - static callable_type*& allocated_ref_ptr(void* src) noexcept
  - static void move_destroy_inline(void* src, void* dst) noexcept
  - static void move_destroy_allocated(void* src, void* dst) noexcept
  - static void execute_destroy_inline(void* target)
  - static void execute_destroy_allocated(void* target)
  - static void destroy_inline(void* target) noexcept
  - static void destroy_allocated(void* target) noexcept
  - static constexpr vtable make_vtable() noexcept

  - static void build_inlinable(void* dst, passed_callable_type&& callable)
  - static void build_allocated(void* dst, passed_callable_type&& callable)

  + static constexpr bool is_inlinable() noexcept
  + static void build(void* dst, passed_callable_type&& callable)
  + static void move_destroy(void* src, void* dst) noexcept
  + static void execute_destroy(void* target)
  + static void destroy(void* target) noexcept
  + static constexpr callable_type* as(void* src) noexcept
  + static constexpr inline vtable svtable = make_vtable()
}

callable_vtable o-- vtable

class task {
  - std::byte m_buffer[details::task_constants::buffer_size]
  - const details::vtable* m_vtable

  - void build(task&& rhs) noexcept
  - void build(details::coroutine_handle<void> coro_handle) noexcept

  - static bool contains(const details::vtable* const vtable) noexcept

  + void operator()()
  + void clear()
  + explicit operator bool() const noexcept
  + bool contains() const noexcept
}
```

#### vtable
* bool trivially_copiable_destructible(decltype(move_destroy_fn) move_fn) 是否不能移动析构
* bool trivially_destructable(decltype(destroy_fn) destroy_fn) 是否不能析构

#### callable_vtable
这个类中需要注意，存在两种可执行对象
* 可执行对象之间保存在task的buf中
* 可执行对象不能保存到task的buf中，把可执行对象在堆上分配，然后把指针保存在task的buf中