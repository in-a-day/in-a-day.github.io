---
title: 进程调度策略-多级反馈队列
tags:
  - os
  - 进程调度
categories: os
date: 2021-01-27 16:40:22
---

## MLFQ(多级反馈队列)
> Multi-level Feedback Queue, MLFQ使用历史预测未来

MLFQ需要解决两方面的问题: 
1. 优化周转时间, OS并不知道一个任务的运行时间, 而SJF, STCF都是基于任务运行时间的策略.
2. 优化响应时间, RR有较好的响应时间, 但是周转时间比较长.

### 基础规则
MLFQ拥有不同的队列, 每个队列都有不同的优先级. 在任意时间, 准备运行的任务在单独的队列中. MLFQ使用优先级决定将运行的任务.  
一个队列中可能存在多个任务, 这些任务拥有相同的优先级, 在这种情况下我们使用round-robin调度策略.  
综上, MLFQ基本规则:
- Rule1: If Priority(A) > Priority(B), A runs.
- Rule2: If priority(A) = Priority(B), A & B run in RR ({% post_link os/process_schedule %}).

MLFQ调度的关键在于如何设置优先级. MLFQ基于观察行为改变改变任务的优先级.MLFQ从任务运行历史中学习, 并根据历史预测任务的未来行为.

### 尝试1: 如何修改优先级




