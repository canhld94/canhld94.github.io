---
layout: post
title: "C++ factory design pattern in action"
date: 2020-10-12
tags: "C++ oop design-pattern"
---

Polymorphism is the first thing to know when we learn object-oriented design. Non-formally, polymorphism alow developer to create a single interface to entities of different classes (or types). In the code, polymorphism is usually implement by defining an abstract class, and the concrete class will inherit some methods from the abstract class and implement the proper behavior of those methods (in the context of the concrete class). 

If we have many different classes that inherit from one abstract class, how to provide a convinient way to create an instance of a specific type? That is when the factory design pattern plays its role. In factory design pattern, each abstract class comes with an factory to create concrete types. Usually, the factory class will have a `create` method that client will supply the parameter and the method should return the proper instance.

Back to my project that I describe in [Blocking queue with a feed-back scheme in C++](/2020/09/11/blocking-queue.html), I've found many places where the polymorphism and factory pattern is applicable. In this article, I will write about how I adopt polymorphism and factory method in different modules to make my code concise, reuseable, and extendable. 

> You can check out the [project repo](https://github.com/canhld94/HeteroServing) and the [brief architecture](https://github.com/canhld94/HeteroServing/blob/master/docs/dev/server.md) of the system.

# Factory method to create inference engines

I need to create some inference engines (IE for short) with different back-end devices and frameworks, but the IEs should look identical to other modules in the system. Therefore, I can apply the polymorphism and the factory pattern to solve the problem. The interface of an IE is:

```cpp
class inference_engine {
    public:
    /****************************************************************/
    /*              Inference engine public interface               */
    /****************************************************************/
    vector<bbox> run_detection(const char*, size_t sz) = 0;
}
```

When client send a request to my server, it will contain an image in compressed format. I want an IE receive the bytes stream, and run the detection model, and return to me a vector of bounding boxes. Then, for each framework, I will have a concrete IE as follow.

```cpp
class openvino_inference_engine : public inference_engine {
public:
  /****************************************************************/
  /*  Inference engine public interface implementation            */
  /****************************************************************/
  std::vector<bbox> run_detection(const char* data, size_t sz) final {
    auto net_out = do_infer(data, sz);
    return detection_parser(net_out);
  }
  virtual std::vector<bbox> detection_parser(network_output& net_out) {
      return {};
  }
private:
  detection_output do_infer(const char* data, size_t sz) {
      // impl of do inference
  }
}
```

Here OpenVino is the Intel deep learning framework that I used to build the IEs for FPGA and CPU. The framework requires some complex steps to build the IE. However, from the perspective of other components, they don\'t care about what happens inside, as long as they can call the public interface `run_detection` and the engine return the correct answer. 

On the other hand, there are many object detection models, and they are very different even within a framework. When working with them for a while, I found the pattern: running the computational graph is same for all models, but interpret the output is different. Therefore, I seprate `run_detection` into two small methods: `do_infer` to run the graph and `detection_parser` to interpret the output of the graph. And each model will implement it own `detection_parser`.

```cpp
class openvino_ssd final: public openvino_inference_engine {
public:
  detection_output detection_parser(network_output& net_out) override {
    // implement ssd parser
  }
}
class openvino_yolov3 final : public openvino_inference_engine {
public:
  detection_output detection_parser(network_output& net_out) override {
    // implement yolov3 parser
  }
}
class openvino_frcnn final : public openvino_inference_engine {
public:
  detection_output detection_parser(network_output& net_out) override {
    // implement faster rcnn parser
  }
}
```

That's it so far for CPU and FPGA. I did the same thing for GPU with NVIDIA TensorRT. In total, I have 6 types of IEs. 


# Factory method to create memory buffers 