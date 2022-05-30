# 操作系统的简介

## 对于使用者而言

* 用户角度，操作系统是一个控制软件
* 管理应用程序
* 为应用程序提供服务
* 杀死应用程序

## 对于其所管理的资源

* 资源管理
* 管理外设、分配资源

## 操作系统层次结构

* 硬件之上
* 应用程序之下

操作系统位于应用软件之下，为应用软件提供服务支撑

Linux、Windows、Android的界面属于外壳（Shell），而不是内核（kernel）kernel是我们研究重点，在shell之下

## OS kernel的特征

* 并发
  * 计算机系统中同时存在多个运行的程序，需要OS管理和调度
* 共享
  * 同时访问
  * 互斥共享
* 虚拟
  * 利用多道程序设计技术，让每个用户都觉得一个计算机专门为他服务
* 异步
  * 程序的执行不是一贯到底，而是走走停停，向前推进的速度不可预知
  * 但是只要运行环境，OS需要保证程序运行结果也要相同

# 学习方法

略