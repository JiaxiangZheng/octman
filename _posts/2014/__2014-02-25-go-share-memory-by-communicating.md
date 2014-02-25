---
layout: post
title: Go的通信共享内存
categories:
- 翻译与转载
tags:
- Golang
- Goroutine
---

本文翻译了Go官方博客的一篇文章[Share Memory By Communicating](http://blog.golang.org/share-memory-by-communicating)，同时也正好加强自己对Goroutine的理解。

* * *

传统线程模型（在编写Java、C++、Python等程序时经常使用）要求程序员通过共享内存达到线程通信的目的。通常情况下，共享的数据结构需要通过加锁进行保护，而且同时线程将通过轮询锁结构以访问共享的数据。在某些情况下，可以通过使用诸如Python的队列等线程安全的数据结构简化其使用。

Go的并发原语（goroutine和channel）则提供了一个不一样且优雅的方法来组织并发软件。
