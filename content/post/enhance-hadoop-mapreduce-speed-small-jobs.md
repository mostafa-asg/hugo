---
title: "Enhance Hadoop MapReduce Speed for small jobs"
date: 2018-02-06T07:15:50+03:30
draft: false
tags: [Hadoop,MapReduce,Performance]
---
## Introduction
There are some circumstances when input of Hadoop's MapReduce is relatively small. Consequently the overhead 
of allocating and running tasks in new containers outweighs the gain to be had in running them in parallel, 
compared to running them sequentially on one node. Such a job is said to be **uberized**, or run as an **uber** task.

## Enable uber optimization
To enable uberized job, simply set **mapreduce.job.ubertask.enable** to true. But that is not sufficient. To run in uber mode,
 you must also define what qualifies as a *small* job. This conditions must be met:  
 
 * mapreduce.job.ubertask.maxmaps (default: 9) < job's total maps
 * number of job's reducers must be 1 or zero
 * Total length for all input splits <= mapreduce.job.ubertask.maxbytes (default: hdfs block size)
 * max(mapreduce.map.memory.mb, mapreduce.reduce.memory.mb) <= yarn.app.mapreduce.am.resource.mb
 * max(mapreduce.map.cpu.vcores, mapreduce.reduce.cpu.vcores) <= yarn.app.mapreduce.am.resource.cpu-vcores
 * map class must not extend **ChainMapper**
 * reduce class must not extend **ChainReducer**
