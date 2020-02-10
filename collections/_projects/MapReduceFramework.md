---
title: MapReduce Framework in Python
---

## 1. Motivation
There are a lot of tasks that issue large scale queries. Currently the solution is to manually split the work and copy jobs to different servers. The problems with this approach are:
- not able to handle failures
- cannot monitor the work progress
- code is not reusable

To facilitate large scale queries, we decided to develop a simplified MapReduce framework in python.

## 2. Goals
The framework should be able to:
- automatically split the work and assign jobs to different servers.
- monitor each job's progress and automatically reassign failed jobs to available workers.
- support customized policies / configurations for different task requirements.

## 3. Architecture
This is a typical **Master-Worker** structure, where master is responsible for assigning tasks and keeping track of jobs, and workers for executing each task.

![MapReduce Framework](/assets/images/projects/mapreduce.png)
### 3.1 Master
The master should be able to:
- divide the work into multiple jobs
- keep track of each assigned job, and reassign failed ones if necessary
- report work progress on demand

When the data scale is not too large, master itself can split the work. Sometimes the way to split work is also important and varies from task to tasks. As a result, we provide a default task partitioner, and also allow users to use their own.

As for task assignments, we currently do not provide a way for users to specify the mapping from jobs to workers. Instead every job is assigned randomly to one of the available workers. This is fine for most of the cases. For the assigned jobs, master will keep track of how long each job has taken, and if it times out (`timeout` is a configuration parameter), depending on the configuration, master will reassign it to another worker, or simply record that job as failed.

There are two main reasons why we sometimes do not reassign a failed job. The first one is that, `timeout` parameter may not be well-configured so some jobs will always timeout, or some jobs simply cannot finish because of the way to partition the work. We do not want a single job to block the whole work so we provide a configuration parameter `maxRetries`, which specify the maximum number of retries for each job. The second reason, is that sometimes a job *cannot be assigned to multiple workers*. For example, the job is to write data to database, and the write is not idempotent, so you do not want possibly multiple workers to work on the same job. In that circumstance you can set `maxRetries` to 0.

The reporting mechanism is a good-to-have. Sometimes even you distribute the work across multiple machines, the work still takes a long time to finish. In that case you may want a way to monitor the work progress and the health of the system.
### 3.2 Workers
Workers are to actually execute each job. Currently users are asked to implement a `JobExecutor` and pass it to the `Worker`. `JobExecutor` will specify how a job should be done.

One thing worth mentioning is that, every worker should be configured with a `maxLiving` time. This is to ensure that even if it lost connection with the master, the process can still end.

## 4. Communications
One of the design choices to make is how the master and workers should communicate with each other. There are two possible ways,
- build an http server on every machine
- copy files directly to pass jobs and results

Http servers are the more common way to implement MapReduce framework. Since `Python` does not have a very good `rpc` package, we may have to implement an http server ourselves. The http server on master is to receive results, and the ones on workers are to receive jobs. And the communication should better be async because a job can take a long time to finish.

Copying files requires setting up a receiving directory for each server, which will be periodically checked by master or worker to know if there is new job or result. The problems with this approach are:
- polling can waste CPU time
- lacking acknowledgement may cause expired files to get processed
