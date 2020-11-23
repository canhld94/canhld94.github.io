---
layout: post
title: "Blocking queue with a feed-back scheme in C++"
date: 2020-09-11
tags: "concurrency C++"
---

Recently, I\'m developing a deep learning server that runs some object detection models, then consume an image from clients and return the object detection result on that image. My server should run the models on heterogeneous hardware, say CPU, GPU, and FPGA. That is a very hard task if we do everything from scratch. Therefore, I decided to utilize the vendor-provided deep learning framework to build the inference engine, and then I will build the server on top of it. The hardest part is settled, but still, there are many interesting problems that I face and learn from them during the project. One of them is when building an inference engine with FPGA, I found that the FPGA device that I have will break the program if more than two threads are trying to access it at the same time. In this article, I will write about how I made a blocking queue with a feed-back loop to run the FPGA inference engine in one thread, then other threads can submit an image to the queue, and wait for the result. My solution uses nothing but only the STL primitives.

<div align="center">
  <figure>
    <img style="text-align: center; width:80%" src="/assets/contents/parallel_server_impl.jpg">
    <figcaption>The design of my server to run inference engine on FPGA</figcaption>
  </figure>
</div>

# The concurrent queue

I need a task queue for queueing the jobs. The functionality is no different from an ordinary queue: some threads will push messages to the queue, and some will read messages from the queue and proceed with the task. To make the queue thread-safe, there are two requirements:

- Only one thread can push or pop from the queue at a time.
- When the queue is full, the pushing thread should wait or be blocked; similarly, when the queue is empty, the popping thread should wait or be blocked.

The first requirement can be resolve with a `mutex`. For the second requirement, the naive idea is spin waiting: after acquiring the `mutex`, the thread pushes or pops to the queue only if some condition is true, otherwise, it should release the lock and keep waiting in the spinning loop until it can perform its operation. This method is very inefficient, as we waste the CPU resource just for waiting in a loop. The second idea is using a condition variable to block the thread when the condition is not meet. This way, the thread goes to a sleep state and therefore doesn\'t consume CPU resources while waiting for the condition. The full code for the concurrent queue is as follows.

```cpp
using namespace std;
template<class T>
class blockingQueue {
public:
  blockingQueue() {
    ready = 0;
  };
  using value_type = T;
  using ptr = std::shared_ptr<blockingQueue<T>>;

  void push(value_type & val) {
    std::unique_lock<std::mutex> lk{mtx};
    actual.push(val);
    if (ready == 0) ready = 1;
    lk.unlock();
    cv.notify_one();
  }

  value_type pop() {
    std::unique_lock<std::mutex> lk{mtx};
    cv.wait(lk, [&]() {
      return ready == 1;
    });
    auto val = actual.front();
    actual.pop();
    if (actual.size() == 0) ready = 0;
    return val;
  }

  size_t size() {
    std::unique_lock<std::mutex> lk{mtx};
    return actual.size();
  }

private:
  std::queue<value_type> actual;
  int ready = 0;
  std::mutex mtx;
  std::condition_variable cv;
};

```

In the above code, I assume that the queue length is infinity and only handle the problem when popping from an empty queue. I use an STL queue as the container, a mutex `mtx` to make all operation is mutually exclusive, and a condition variable `cv` and a state value `ready` to block threads from popping an empty queue. In the method `pop`, the thread will wail for the condition variable `cv` and only wake-up when `ready == 1`. Here, the `ready` is set to `1` when there is at least one element in the queue and is set to `0` when the last element is popped from the queue. The mutex `mtx` will do three tasks:

- It makes all operation on the queue mutually exclusive
- It is used as an associated lock with the condition variable
- It makes all operation on the variable `ready` mutually exclusive

So far so good. Now we can have producer and consumer to write and read to the queue just like an STL queue:

```cpp
void producer(blockingQueue<int>::ptr taskq) {
  while(1) {
    // get val
    taskq->push(val);
  }
}

void consumer(blockingQueue<int>::ptr taskq) {
  while(1) { 
    auto val = taskq->pop();
    // do the job
  }
```

# The feedback mechanism

When a thread submits the job to the queue, sometimes it wants to get back the result of the task. In my project, the result of the task is a vector of bounding boxes on the image with class name and probability. This vector should be serialized and send back to the client, and we don't want to do it in the inference thread. Deep learning is a compute-intensive application, so we have a dedicated thread that runs on dedicated hardware to do that job. We want this thread to only focus on computing and leave the post-processing job to others. There are two potential solutions:

- Create another thread groups, that will do the post-processing 
- The owner of the task will do the post-processing

I tried both, and I found the second way to work better for me. In the first approach, we need to pass the socket around, which is quite messy because we pass the socket to the inference thread for nothing but only to forward it to another thread. It takes me several days to make a solution for approach two. And that is an interesting story. 

To understand my approach, let take a real-life example. Imagine you go to some government office to get a seal on your document. There are many people like you here, but there is only one officer to process all papers. The sealing process takes time, so all of you don\'t want to stand in line and wait for hours. Instead, you put documents in a box (box A) in FIFO manner, and the officer will process the document in the box bottom-up, and throw them to another box (box B) when sealing is done. The question is: how do you know when your document is done?

The first approach: whenever a new document arrives in box B, all of you come and check is this my document. This is fine, but you don\'t want to move back and forth too much. So, there is the second approach: When a document is done, the officer will shout the name on that paper, and all of you know when is my turn to come. 

That is exactly what I\'ve done. I want to pass to the inference thread an empty vector, and I want the thread to fill that vector, and give it back to me. Here, we already have the box (that is the queue we create above). But we need an object to notify people that their data is ready. We need something to make noise. We need a bell! And this is the concept of the bell:

```cpp
template <class Key, class LiteralKey, LiteralKey reset_state,
          class CondVar = std::condition_variable, 
          class Mutex = std::mutex,
          class Lock = std::unique_lock<std::mutex>>
class simple_bell {
 private:
  CondVar cv;             //!< Conditional variable that producer will wait for
  Mutex mtx;              //!< Associated mutex
  Key key = reset_state;  //!< A key to preventing suspicious wake-up.
 public:

  simple_bell() = default;

  ~simple_bell(){};

  void wait(Key&& desired_state) {
    auto lk = lock();
    cv.wait(lk, [&]() { return (key == desired_state); });
    key = reset_state;
  }

  Lock lock() { return Lock(mtx); }

  void ring(Key&& set_state) {
    auto lk = lock();
    key = set_state;
    lk.unlock();
    cv.notify_all();
  }
  using ptr = std::shared_ptr<simple_bell>;
};
```

A bell is just a condition variable with a state variable `key`. After submitting a task to the queue, the thread will wait for a bell. When the task is done, the inference thread will ring a bell. In the first approach, all threads will share the same bell. Therefore, `set_state` should be identical to each thread, so it can be the threadid. In the second approach, each thread will have its own bell, so `key` can be just `0` or `1`. Both work fines.

# Afterthoughts

Queue is more interesting than it looks. The queue I made above is the simplest version of a concurrent queue. There is other implementations in [Intel TBB](https://software.intel.com/content/www/us/en/develop/tools/threading-building-blocks.html) or [Microsoft PPL](https://docs.microsoft.com/en-us/cpp/parallel/concrt/parallel-patterns-library-ppl?redirectedfrom=MSDN&view=msvc-160). There are other lock-free implements of concurrent queue on Github. Queue is used everywhere, from browser to cloud. A good implementation of concurrent queue will boost the performance of the system, but it also requires much effort.

Even I\'m excited about the `bell` concept that I\'ve made, tbh I feel it is so naive and amateur. There must be a better solution out there but I couldn\'t find or understand. The most related technology I found is the [JavaScrip event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop). We can make an event loop just by using C++ STL. I cannot make it in my project because I\'ve not yet fully understand the asynchronous code. If you take a look back at my design, everything is synchronous. So far, my brain still thinks synchronously. That is fine, but not enough. Therefore, I\'m learning to think asynchronously, and hopefully, I will have better design in the future.


<!-- https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop -->