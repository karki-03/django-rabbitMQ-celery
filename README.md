# How to Use Celery and RabbitMQ and Flower with Django?

### What you will get from this tutorial?
1. Basics & pre-requisites for learning Celery.
2. Architecture.
3. Workflow of task queue.
4. About AMQP.
5. Types of exchanges.
6. Messaging models.
7. Installing RabbitMQ, Celery and Flower.
8. Practically making a simple django project to send mail using task queue.
9. Code snippet for getting started with celery. (<b><a href="https://github.com/karki-03/celery_django_app">Click here</a></b> to directly go to code.)


## What is celery?
Celery is an open-source Python library which is used to <b>run the tasks asynchronously</b>. It is a task queue that holds the tasks and distributes them to the workers in a proper manner. It is primarily focused on real-time operation but also supports scheduling (run regular interval tasks).

## Uses of celery:
1. Offloading work (using Celery workers)
2. Scheduling task execution (using Celery beat)

## What is RabbitMQ?
RabbitMQ is an open-source <b>message-broker</b> software that originally implemented the <b>Advanced Message Queuing Protocol</b>. We will understand more about it later in this tutorial.

## What is Flower?
Flower is a web based tool for <b>monitoring and administrating Celery</b> clusters. Flower helps us to check the following things:
1. Task progress and history
2. Ability to show task details (arguments, start time, runtime, and more)
3. Graphs and statistics (and many more things)

## Few points you should know as a pre-requisite:
1. <b>Queue</b> is an abstract data structure, somewhat similar to Stacks. Unlike stacks, a queue is open at both its ends. One end is always used to insert data (enqueue) and the other is used to remove data (dequeue). Queue follows <b>First-In-First-Out</b> methodology, i.e., the data item stored first will be accessed first.
<br>
2. <b>Task queues</b> let applications perform work, called tasks, asynchronously outside of a user request. If an app needs to execute work in the background, it adds tasks to task queues. The tasks are executed later, by worker services. The Task Queue service is designed for <b>asynchronous work</b>.
<br>
3. <b>Task, work, message or job</b> are used to refer the same thing i.e. a piece of work to be done or undertaken.

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
9. Make a new file: urls.py inside the folder `send_mail`. To make it accessible , add `path("", include("send_mail.urls")),` in the urpatterns list in celerytutorial/urls.py file. Also, dont forget to import `include` from `django.urls`
10. Make a directory `templates`, inside of which 3 html files has to be created: base.html, index.html and success.html. The code for all these template files are as follows. Also, add ('DIRS': [Path(BASE_DIR,'templates')],) in your TEMPLATES settings.

base.html
![image](https://user-images.githubusercontent.com/57327185/187139181-b02d3c58-4bfe-4d3a-801e-91fdeb508af3.png)

index.html
![image](https://user-images.githubusercontent.com/57327185/187139233-9dab7337-e645-4a0c-a069-0e807a717281.png)

success.html
![image](https://user-images.githubusercontent.com/57327185/187139278-4b1e320f-e186-48e2-b24f-f65bd1077167.png)

send_mail/urls.py
![image](https://user-images.githubusercontent.com/57327185/187139368-9296bef9-e5e3-4ff7-91a0-bebe8d3becd1.png)

send_mail/views.py
![image](https://user-images.githubusercontent.com/57327185/187139439-ef980b82-1220-4fe4-8cb5-0ba376046d7c.png)

### Let us test the django project on our browser
1. Run `python manage.py makemigrations` and `python manage.py migrate`
2. Run `python manage.py runserver` and open the url on your browser. (http://127.0.0.1:8000/ in my case, please make the required changes in the code if your port is different.)
3. You should be able to see a simple form. ![image](https://user-images.githubusercontent.com/57327185/187140030-aa89cf83-8c46-4c28-aefa-094ee4083a38.png)
On clicking submit, success page is rendered and we can click the send mail again button. Till now we have just made the skeleton of the project. Let us now write the logic for sending mail in the views.py file inside SendMailView class. 

## Install celery
1. Inside your virtual environment run `python -m pip install celery`
2. Create a new file `celery.py` in the `celerytutorial` directory. ![image](https://user-images.githubusercontent.com/57327185/187141129-19b920d0-44e2-4ec4-8445-3f614fe49a25.png)
3. Let us create a task now. So, create `tasks.py` in the `send_mail` directory. ![image](https://user-images.githubusercontent.com/57327185/187142089-201f8f0f-aa5d-4d64-bca9-6ce3c78631fc.png)
4. Let us call this function (send_email_task) from our API view. ![image](https://user-images.githubusercontent.com/57327185/187142760-3f706499-0526-4f21-aa7e-2299e0035ef5.png)
5. Celery needs a message broker to communicate with programs that send tasks to the task queue. Without a broker, Celery isn’t able to receive instructions.
6. So, we need to start the rabbitMQ server by running `rabbitmq-server` in the RabbitMQ Command Prompt.
7. Run `pip install eventlet`. Then run `celery -A celerytutorial worker -l info -P eventlet`
8. Let us add our message broker's server url in the settings file. (CELERY_BROKER_URL = "amqp://guest:guest@localhost:5672/"). 
9. To actually send the mail you need to set up everything. To sum up you have to add these settings: 
   EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
   EMAIL_HOST = 'smtpxxxxxxxxxxx'
   EMAIL_USE_TLS = True
   EMAIL_PORT = xxx
   EMAIL_HOST_USER = 'xxxx@xxxx.com'
   EMAIL_HOST_PASSWORD = 'xxxxxxx'
   
   Alternatively, to save up time, you can print the mail on console by adding `EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"`
   
10. Once you are sure, celery worker, rabbitMQ server and django server all are running. Then hit the localhost url for home view. Fill up the form and send the email. You can see how fast the response is given. You can verify this by adding sleep command in the tasks.py file's send_email_task function, which is called in the views file. (This sleep represents the complex logic which requires time to execute.)
11. With this, you have successfully send a task to be executed asynchronously by celery worker. But, we have not discussed about Flower till now.

## Install Flower
1. Run `pip install flower` inside the virtual environment.
2. Run `celery -A celerytutorial flower  --address=127.0.0.6 --port=5566` in another prompt.
3. Open `http://127.0.0.6:5566/` on the browser. Here you can visualize the activities done by celery.

With this we have completed this tutorial. However, it is just the begining, we can schedule our tasks through celery and there are many use cases. The motive of this tutorial was to somehow make you understand the core concept behind task queues and implementing them as quickly as possible. I will attach the github repo for the project we made in this tutorial. I will also attach the cheatsheet for all commands we used. 

You might have thought that each and every line of code was not explained properly in this tutorial. However, I will be explaing each and everything after this point. So, give it a break and comeback after to learn more about the code you've just written.












