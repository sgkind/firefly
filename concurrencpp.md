代码仓库地址：[David-Haim/concurrencpp at develop (github.com)](https://github.com/David-Haim/concurrencpp/tree/develop)



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

2. 定时检查线程在没用定时器时会销毁，新添加timer后如果没有检查线程则创建

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
thread_executor每创建一个任务就创建一个线程执行此任务

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

manual_executor没有自带线程，任务添加后不会自动执行。如果需要执行任务，需要调用相关的接口同步执行若干任务。任务是在调用执行接口的线程上执行的。

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
一个工作执行器，可以向其添加任务，执行器会开一个线程一直执行被添加的任务

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
```