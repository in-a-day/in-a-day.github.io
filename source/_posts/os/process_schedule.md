---
title: 进程调度-基础
tags:
  - os
  - 进程调度
categories: os
date: 2021-01-27 13:52:49
---

## 调度指标
1. 周转时间(turnaroud time): 任务到达系统到执行结束的时间
$$ T_{周转时间} = T_{完成时间} - T_{到达时间} $$
2. 响应时间(response time): 任务到达系统到首次运行的时间 
$$ T_{响应时间} = T_{首次运行} - T_{达到时间} $$

## 调度策略
### FIFO(先进先出)
> First In First Out, 也称先到先服务(First Come Fist Served), 最先进入的任务先执行.

### SJF(最短任务优先)
> Shortest Job First
- 非抢占式, 任务时长最短的任务先执行, 前提是知道任务运行时间, 如果任务同时到达, 且以周转时间作为指标来说, 该策略最优.

### STCF(最短完成时间优先)
> Shortest Time-to-Completion First, 又称抢占式最短作业优先(Preemptive Shortest Job First, PSJF). 完成时间最短的任务先执行.
- 任务在不同时间达到, 且以周转时间作为指标来说, 该策略最优.

### Round-Robin(RR, 时间片轮转)
> 在一个时间片(time slice, aka. shceduling quantum)内运行一个任务, 时间片结束后切换到下一个任务, 而不是运行一个任务直到结束.

时间片长度对RR至关重要. 时间片越短, RR的响应时间越短, 但是上下文切换的成本将影响整体的性能. 利用amortizatioin(摊销)技术来摊销上下文切换的成本(设置适当的时间片长度).  

RR对于响应时间十分友好, 而对于周转时间来说RR是一个是分糟糕的策略.

