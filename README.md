# How to Use Celery and RabbitMQ and Flower with Django?

## What is celery?
Celery is an open-source Python library which is used to run the tasks asynchronously. It is a task queue that holds the tasks and distributes them to the workers in a proper manner. It is primarily focused on real-time operation but also supports scheduling (run regular interval tasks).

## Uses of celery:
1. Offloading work (using Celery workers)
2. Scheduling task execution (using Celery beat)

## What is RabbitMQ?
RabbitMQ is an open-source message-broker software that originally implemented the Advanced Message Queuing Protocol. We will understand more about it later in this tutorial.

## What is Flower?
Flower is a web based tool for monitoring and administrating Celery clusters. Flower helps us to check the following things:
1. Task progress and history
2. Ability to show task details (arguments, start time, runtime, and more)
3. Graphs and statistics (and many more things)

## Few points you should know as a pre-requisite:
1. <b>Queue</b> is an abstract data structure, somewhat similar to Stacks. Unlike stacks, a queue is open at both its ends. One end is always used to insert data (enqueue) and the other is used to remove data (dequeue). Queue follows First-In-First-Out methodology, i.e., the data item stored first will be accessed first.
2. <b>Task queues</b> let applications perform work, called tasks, asynchronously outside of a user request. If an app needs to execute work in the background, it adds tasks to task queues. The tasks are executed later, by worker services. The Task Queue service is designed for asynchronous work.
3. Task, work, message or job are used to refer the same thing i.e. a piece of work to be done or undertaken.

## Before starting
There might be few problems working with celery on windows. However, we will try to make it work on windows as well. So don't lose your mind, while setting up celery!

## Understanding the architecture 
1. Celery is a <b>distributed task queue</b> that can collect, record, schedule, and perform tasks outside of your main program.
2. Celery requires a <b>message broker</b> for communication.
3. Redis and RabbitMQ are two message brokers that developers often use together with Celery.

<b>What is message broker?</b><br>
A message broker is an intermediary computer program module that translates a message from the formal messaging protocol of the sender to the formal messaging protocol of the receiver.

You can checkout the below diagram (made by me) to know how task queue works!
<br>
![Untitled Diagram](https://user-images.githubusercontent.com/57327185/186875182-d055f417-9d29-463d-abdd-2272eb2f9681.png)
<br>

1. <b>Producers</b> or celery clients are basically the APIs that will send messages to the message broker.
2. <b>Message broker</b> is responsible for the connection between producer and consumers.
3. <b>Consumers</b> or celery workers are responsible for executing the tasks.
4. The queues are stored in message broker (its server)
5. Exchanges are connected to the task queues with bindings which can be referenced by the binding key.

## Workflow:
1. Producers (or celery client) i.e. our API will pass a message (Let us suppose a simple mail to be sent) to a message broker (Rabbit MQ in our case). 
2. The message will first be received by the exchange. (You can think exchange as a post office from where letters are distributed.)
3. Exchange then distributes the tasks to the queues.
4. This exchange & task queue is together called as message broker.
5. The task queue then gives out the tasks to the workers or consumers (celery workers in our case)
6. Celery workers can tackle computations as a background task and allow your users to continue browsing.
7. There are various techniques through which this tasks can be provided by task queues to the workers. (like fanout, direct, header, defualt, topic. Will learn more about this later.)
8. The task queue (stored in Rabbit MQ server) follows acknowledgement rule, i.e. until and unless the task is completed by the worker and an acknowledgement is not sent, the queue will store the task that are pending. 

*By now we have learn about task queue, message broker, producer, consumer and learnt a bit about what rabbit MQ and celery does.*
<br>
<b>Now, we will dive deep into how tasks are distributed internally and other related things. In the end, we will be implementing the setup. So please be patient with it.</b>

### About AMQP
1. AMQP <b>(Advanced Message Queuing Protocol)</b> is a messaging protocol that enables conforming client applications to communicate with conforming messaging middleware brokers. 
2. AMQP is the core protocol for RabbitMQ.

### About different type of exchanges
1. <b>Fanout Exchange:</b> Exchange will duplicate the message and send it to every single queue it knows about.
2. <b>Direct Exchange:</b> Message will have a routing key, which will be compare to the binding key. If matched, message will move through.
3. <b>Topic Exchange:</b> Partial match between routing and binding key. (Topic should be same, eg. ship.shoes is the routing key & ship.any is the binding key)
4. <b>Header Exchange:</b> Routing key is ignored completely, message is moved according to header.
5. <b>Default/Nameless Exchange:</b> Only the part of RabbitMQ and not of AMQP. Here, routing key name is same as the queue name.

### About message acknowledgement in RabbitMQ
When a message is in the queue and it goes to the consumer, the message stays in the queue until the consumer lets the broker know that it has received the message. That prevents system from losing any messages.


### About messaging models
<b>1. Point-to-point:</b>
Point to Point means message(s) is sent from one application(producer or sender) to another application(consumer/receiver) via a queue. There can be more than one consumer listening on a queue but only one of them will be get the message. Hence it is Point to Point or One to One.
<br>
<b>2. Pub/Sub:</b>
Pub/Sub or Publisher/Subscriber is another messaging model where a message is sent to multiple consumers(or subscribers) through a topic. The topic is the link between publisher and subscriber. (Topic here means the binding).
<br>
<br>
*Note: To be precise Pub/Sub is the name of the `messaging pattern`. And producers and consumers are a part of it. We are using pub/sub messaging model in this tutorial.*
<br>
<br>

# Let's get our hands dirty!

## Install Rabbit MQ
Follow the instructions given on the official download page of rabbitMQ to install rabbitMQ on your machine.
Download page (RabbitMQ): https://www.rabbitmq.com/download.html

In this tutorial I will assume we are using Windows operating system. So, for windows users, click this link: https://www.rabbitmq.com/install-windows.html and download the excutable file and follow the instructions to successfully install rabbit MQ on your machine. 

You also need to install Erlang. Otherwise, RabbitMQ might not work.

From start menu, you can search for `RabbitMQ Command Prompt`, a command prompt will be opened where you can start the rabbitMQ server by typing `rabbitmq-server`
<br><br>
![4](https://user-images.githubusercontent.com/57327185/187129827-28fa0734-4f93-41ff-b4a3-2717d93d13be.JPG)
<br>
By default you can find rabbitMQ server running on: http://localhost:15672/
Enter username and password both as `guest`
<br><br>
![5](https://user-images.githubusercontent.com/57327185/187130233-8513d771-a21e-40cd-947b-b1cbafca2a15.JPG)
<br>
After logging in the dashboard of RabbitMQ server will open up.
<br><br>
![6](https://user-images.githubusercontent.com/57327185/187130474-5680accc-5218-463e-a5c9-807d73e08a5c.JPG)
<br>
Congrats! You have successfully installed rabbitMQ on your system.

## Setting up a Django project

1. Open command prompt, go to your projects directory where you will be creating the django project.
2. Execute `django-admin startproject celerytutorial`
3. Then change the directory by executing: `cd celerytutorial` ![7](https://user-images.githubusercontent.com/57327185/187131427-0ac49f39-63d1-447b-bcd9-e563561f69f9.JPG)
4. Execute `code .` to open up VS code. Alternatively, open the project in any code editor of your choice. You file directory will look something like this. ![image](https://user-images.githubusercontent.com/57327185/187131712-511e7f49-76e3-40d1-a446-74afbd160524.png)
5. Create a virtual environment to isolate our package dependencies by executing: `python -m venv env` ('env' can be replaced with the name of virual environment of your choice.)
6. On windows type: `env\Scripts\activate` to activate the virtual environment. Alternatively, type: `source env/bin/activate` on linux.
7. Now inside the virual environment run the command: `pip install djangorestframework`. Also, include 'rest_framework' in the INSTALLED_APPS definition in settings.py file. ![image](https://user-images.githubusercontent.com/57327185/187134092-ca6a6eb9-0761-4a78-9788-930489789c22.png)
 (Installing django rest framework is not mandatory, but gives us a lot of convenience and makes our work easier compared to plain django.)
8. Create a new app, send_mail by running: `python manage.py startapp send_mail`. By now your file directory should look like this. ![image](https://user-images.githubusercontent.com/57327185/187133942-a547d5be-f2e6-451d-b60d-9419465637c0.png)
9. 










