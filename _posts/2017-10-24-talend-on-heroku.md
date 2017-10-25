---
title: "Integration: Talend on Heroku"
categories:
  - Integration
tags:
  - ETL
  - Heroku
  - Talend
  - Java
---
Several times in my daily work within Salesforce environments I face the requirement of integrate a batch job with Salesforce. For this purpose there are many ETL such us, Informatica, Oracle Data Integrator, Talend,... But with this we cover half of the equation. Where should I deploy my batch process. Informatica cloud has in own infrastructure but sometimes is not easy (or cheap) to have this infrastructure. So in this post we are going to deploy our process in a cloud Paas, Heroku in our case, and we will use Talend Open Studio for Data Integration to build the process because if free. So we will have quicky our process ready to run and for very few money ;)

## Heroku

Heroku is a PaaS which is the acronym of Platform as a Service and enables you to run your own applications. In this post we are going to use the Java build pack due to the hability of Talend for export the batch processes as a jar file. But many other languages cound be used in Heroku. Please refer to this link if you would like to know more about [Heroku](https://devcenter.heroku.com/start)

The fundamental part of Heroku are the Dynos which are system processes ables to run webservers and batch processes. In our example we are going to create a `worker` dyno to run the Talend process and a `web` dyno to hold a REST web service that when is invoked will launch the Talend job, in other words, it will call to the worker dyno.

As I have mentioned before we are going to use the java `buildpack`, this buildpack relies on Maven to deploy into heroku the dynos. So we need to be familiarized with maven and java concept before we start. The Maven main file is the `pom.xml` where we will define several things like dependencies, repositories, build cycle,... For more information please refer to the [Maven documentation](https://maven.apache.org/)

### Coding in Heroku

Before we start to coding our application we need some dependencies. The piece we are going to use to communicate the dynos among them is the `rabbitmq` queue where the web dyno will publish messages and the worker dyno will be suscribed to starting launch the batch when a message arrives. Other of the depencencies that we will use in this post is and embed `Tomcat` to serve the `Jersey` web services. In order to add this dependencies into our project we will need to add them to the pom.xml file.

{% highlight xml %}
...
<dependencies>
  ...
  <dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>3.3.4</version>
  </dependency>
  ...
</dependencies>
...
{% endhighlight %}

The full pom.xml is in the GitHub repository.

The web service is very simple an it will exponse a simple method that will publish a message in the rabbitMQ queue. Here is the code that will publish in the rabbitMQ queue.

{% highlight java %}
@GET
@Produces(MediaType.TEXT_PLAIN)
public String getIt() {

    try{
        //GET THE QUEUE URL FROM ENV PROPERTY
        String uri = System.getenv("CLOUDAMQP_URL");
        if (uri == null) uri = "amqp://guest:guest@localhost";

        //CREATE THE CONNECTION TO THE QUEUE
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri(uri);
        factory.setRequestedHeartbeat(30);
        factory.setConnectionTimeout(30);
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        //PUBLISH IN AN QUEUE THE MESSAGE
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        String message = "Hello CloudAMQP!";
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());            
        System.out.println(" [x] Sent '" + message + "'");

        channel.close();
        connection.close();

    }catch(Exception e){
        System.err.println(e.getMessage());
    }

    return "Hello, Heroku!!!";
}
{% endhighlight %}

Finally we will need to consume the published messages in the queue by the worker and when this will happend the Talend job will be launched. Here is the code snipplet in charge of the consumption part in the worker.

{% highlight java %}
public static void main(String[] argv) throws Exception {
  String uri = System.getenv("CLOUDAMQP_URL");
  if (uri == null) uri = "amqp://guest:guest@localhost";

  ConnectionFactory factory = new ConnectionFactory();
  factory.setUri(uri);
  factory.setRequestedHeartbeat(30);
  factory.setConnectionTimeout(30);
  Connection connection = factory.newConnection();
  Channel channel = connection.createChannel();

  channel.queueDeclare(QUEUE_NAME, false, false, false, null);
  System.out.println(" [*] Waiting for messages");

  QueueingConsumer consumer = new QueueingConsumer(channel);
  channel.basicConsume(QUEUE_NAME, true, consumer);

  while (true) {
    //Every time a message is consumed by the worker the talend job is launched
    QueueingConsumer.Delivery delivery = consumer.nextDelivery();
    String message = new String(delivery.getBody());
    System.out.println(" [x] Received '" + message + "'");
    
    String[] args = {""};
    TestHeroku.main(args);      
  }
}
{% endhighlight %}

The glue that put all pieces together is the `Procfile` file where we will define the workers and its type. So before we need two runnables processes one for the web dyno and other for worker dyno. We can configure in the `pom.xml` to generate executable files from a java class.

{% highlight xml %}
...
<program>
    <mainClass>launch.Main</mainClass>
    <name>webapp</name>
</program>
<program>
  <mainClass>com.jsuarez.WorkerProcess</mainClass>
  <name>worker</name>
</program>
...
{% endhighlight %}

So finally our `Procfile` looks like this:
{% highlight txt %}
web: sh target/bin/webapp
worker: sh target/bin/worker
{% endhighlight %}

## Talend Job

Talend Open Studio for Data Integration is an ETL (Extract, Transform and Load) and has a free version. In this tool we can create batch processes to perform load of data in Salesforce. In this post we are not going to go very deep into the Talend process. Is enaught to know that it takes a CSV and loads the data of the CSV into Salesforce. An interesting feature of Talend is that in can be exported as a runnable jar. So this jar will be imported into the Heroku project to do the job when will be requested.

A Talend Job look like this:

![alt text](/assets/images/talend-on-heroku1.png "Logo Title Text 1")

Maven has a central repository where you can find and reference the most common jars, as for instance the RabbitMQ that we have used before, but as you can understand our personal jars won't be found in this central repository. So for upload the Jar with the talend job, and other jars that the talend uses as reference, we can create a locar repo. Again we must define this local repo in the `pom.xml` with this snipplet of code

{% highlight xml %}
...
<repositories>        
    <repository>
        <id>project.local</id>
        <name>project</name>
        <url>file:${project.basedir}/repo</url>
    </repository>
</repositories>
...
{% endhighlight %}

And then we can reference the dependency in the local repository

{% highlight xml %}
...
<dependency>
    <artifactId>salesforceCRMManagement</artifactId>
    <groupId>com.javier.local</groupId>
    <version>1.0.0</version>
</dependency>
...
{% endhighlight %}

### Source

All the source code of this post is in this [GitHub repository](https://github.com/sfdcode/talend-on-heroku.git)