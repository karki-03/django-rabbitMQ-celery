# How to Use Celery and RabbitMQ with Django?

# What is celery?
Celery is an open-source Python library which is used to run the tasks asynchronously. It is a task queue that holds the tasks and distributes them to the workers in a proper manner. It is primarily focused on real-time operation but also supports scheduling (run regular interval tasks).

# Uses of celery:
1. Offloading work (using Celery workers)
2. Scheduling task execution (using Celery beat)

# Few points you should know as a pre-requisite:
1. <b>Queue</b> is an abstract data structure, somewhat similar to Stacks. Unlike stacks, a queue is open at both its ends. One end is always used to insert data (enqueue) and the other is used to remove data (dequeue). Queue follows First-In-First-Out methodology, i.e., the data item stored first will be accessed first.
2. <b>Task queues</b> let applications perform work, called tasks, asynchronously outside of a user request. If an app needs to execute work in the background, it adds tasks to task queues. The tasks are executed later, by worker services. The Task Queue service is designed for asynchronous work.
3. Task, work, message or job are used to refer the same thing i.e. a piece of work to be done or undertaken.

# Before starting
There might be few problems working with celery on windows. However, we will try to make it work on windows as well. So don't lose your mind, while setting up celery!

# Understanding the architecture 
1. Celery is a distributed task queue that can collect, record, schedule, and perform tasks outside of your main program.
2. Celery requires a message broker for communication.
3. Redis and RabbitMQ are two message brokers that developers often use together with Celery.
4. 
