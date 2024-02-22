---
layout: post
title: Difference between a method and a function
date: 2018-04-12 14:59:40
categories: 其他
---

## Founction

A function is a piece of code that is called by name. It can be passed data to operate on (i.e. the parameters) and can optionally return data (the return value). All data that is passed to a function is explicitly passed.

## Method

A method is a piece of code that is called by a name that is associated with an object.

## Diff

In most respects it is identical to a function except for two key differences:  
A method is implicitly passed the object on which it was called.
A method is able to operate on data that is contained within the class (remembering that an object is an instance of a class - the class is the definition, the object is an instance of that data).
(this is a simplified explanation, ignoring issues of scope etc.)

## Another

A method is on an object. A function is independent of an object. For Java, there are only methods. For C, there are only functions. For C++ it would depend on whether or not you're in a class.

> [Difference between a method and a function](https://stackoverflow.com/questions/155609/difference-between-a-method-and-a-function)
