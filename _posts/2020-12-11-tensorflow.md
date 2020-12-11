---
layout: post
title: "Tensorflow Deep Dive"
date: 2020-12-10
tags: "Interview deep-learning tensorflow"
---


# Tensorflow graph 

- A deep learning model is a computational graph with nodes are operators, and the data streams on edges
- Tensorflow allow user to create the computational graph and execute the graph
- The graph in tensorflow (`TF.graph`) have two kind of information:
  - Graph structure: the nodes and the edges of the graph
  - Graph collections: metadata of the graph (variables...)

## Variable

- Local variables: variables that must be initialized before using
- Calling tf.Variable() adds several ops to the graph:
  - A variable op that holds the variable value.
  - An initializer op that sets the variable to its initial value. This is actually a tf.assign op.
  - The ops for the initial value, such as the zeros op for the biases variable in the example are also added to the graph.


## Building a graph

- Tensorflow program start with constructing a graph
- Invoke TF api to construct new operation (`tf.Operation`) and edges (`tf.Tensor`) and add them to `tf.Graph` instance.
- Most of program only have one default global graph
- Calling the tf api will add operation and tensors to default graph, but does not perform any actual computation (in other words, we just *compose* a graph)

## Naming operations

- Every `tf.Operation` objects in a graph must have an unique name; if we don't define it, the framework will create a default unique name for it
- Name may have a name scope (`tf.name_scope`), many operation can share a name scope

## Operation execution policy

- We can specify which device the operation should run on with `tf.device`

# Session

- `tf.Session` - A context to run a graph
- Once a session is created, it will acquire all resource needed to run the graph, including the physical device
- The session should be destroy to free resource


# Tensor

- Muti-dimensional matrix
- Automatic conversion with tensor-like object such as numpy array, variable, scala value
- A `tf.Tensor` object represents a partially defined computation taht will eventually produce a value.
- Generization of vector
- In tensorflow program, tensors are the defaul objects to pass around
- Tensor has two attributes:
  - a type 
  - a shape
- The type must be specify when we build the graph, but the shape might be only partial know
- Tensors are immutable except for `tf.Variable`

## Rank of a tensor

The rank of a tensor object is its number of dimensions. 

## Shape

# Optimizer

# Tape
