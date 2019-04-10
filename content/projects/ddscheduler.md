+++
date = "2019-04-09"
title = "FreeRTOS Deadline Driven Scheduler"
slug = "ddscheduler"
+++

ECE 455: Real-Time Operating System Design Project

System validation was completed using a toy application utilizing on-board LEDs and and the user button to create aperiodic tasks on the STM32F407G Discovery board

{{< figure src="/images/rtos/test1.gif" caption="">}}

*Table 1. Test Task Set*

| Task     | Execution time         | Relative Deadline  | LED Colour   |
| ------------- |:-------------:| -----:| -----:|
| Periodic Task 1 | 95 | 500 | Amber   |
| Periodic Task 2 | 150 |   500 |  Green |
| Periodic Task 3 | 250     |    750 |  Blue  |
| Aperiodic Task 1 | 50     | 200   | Red |

The implemented deadline driven scheduler on top of the FreeRTOS priority driven scheduler was a success with excellent performance. The scheduler successfully realized EDF (earliest deadline first) dynamic scheduling with very little overhead. Using a 1 kHz Tickrate, the system was able to schedule a task set with a calculated CPU utilization of 100%. This project utilizes many concepts of real-time systems such as the utilization of queues, task notification, memory management and dynamic scheduling algorithms.