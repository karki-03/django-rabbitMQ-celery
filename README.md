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
1. Celery is a <b>distributed task queue</b> that can collect, record, schedule, and perform tasks outside of your main program.
2. Celery requires a <b>message broker</b> for communication.
3. Redis and RabbitMQ are two message brokers that developers often use together with Celery.

<b>What is message broker?</b><br>
A message broker is an intermediary computer program module that translates a message from the formal messaging protocol of the sender to the formal messaging protocol of the receiver.

You can checkout the below diagram (made by me) to know how task queue works!
![Untitled Diagram](https://user-images.githubusercontent.com/57327185/186875182-d055f417-9d29-463d-abdd-2272eb2f9681.png)

1. <b>Producers</b> or celery clients are basically the APIs that will send messages to the message broker.
2. <b>Message broker</b> is responsible for the connection between producer and consumers.
3. <b>Consumers</b> or celery workers are responsible for executing the tasks.
4. The queues are stored in message broker (its server)

# Workflow:
1. Producers (or celery client) i.e. our API will pass a message (Let us suppose a simple mail to be sent) to a message broker (Rabbit MQ in our case). 
2. The message will first be received by the exchange. (You can think exchange as a post office from where letters are distributed.)
3. Exchange then distributes the tasks to the queues.
4. This exchange & task queue is together called as message broker.
5. The task queue then gives out the tasks to the workers or consumers (celery workers in our case)
6. Celery workers can tackle computations as a background task and allow your users to continue browsing.
7. There are various techniques through which this tasks can be provided by task queues to the workers. (like fanout, direct, header, defualt, topic)
8. The task queue (stored in Rabbit MQ server) follows acknowledgement rule, i.e. until and unless the task is completed by the worker and an acknowledgement is not sent, the queue will store the task that are pending. 

