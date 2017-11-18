---
title: "Integration: Event Driven Architecture in Salesforce (1 of 4)"
categories:
  - Integration
tags:
  - Event Driven Architecture
  - Platform Events
  - WebSockets
  - Streaming API
  - Apex
  - Heroku
---
The next entry in our blog is the first one of a serie about different ways to integrate Salesforce with external systems following the designs of an Event Driven Architecture (EDA). We are going to cover three different technologies to make this integration. In the next post we will cover the Platform Events, this are the new way that Salesforce offers to the customer to deploy an EDA solution, quickly and reliable. In a further post we will cover Streaming API, the old way to do it before the launching of Platform Events, and in a last post we will cover a custom solution based in WebSocket to integrate with the Salesforce UI, the biggest advantage of this method is that it doesn't reach any governor limit. So it can be a good approach for solutions with a very big amount of data.

Before we start talking about Salesforce it would be good to understand what is Event Driven Architecture. Traditionally we have point-to-point integration, where a system call the other when it needs something from him. Nowadays this is still a valid pattern, but in many cases it is hard to mantain because normally we need to have some code in the client and in the server and if you change something the integration could be broke down. So with this in mind it appears the Enterprise Service Bus commonly named ESB. An ESB is a piece of software that centralice all the services you need to expose in your organization and deals with transformations and routing rules to keep the integrations in better managed in a single place. Ok, but we where going to speak about EDA, right??? Yes, EDA is actually a change of philosophy in the integrations world. We said before that in point-to-point and in ESB integrations the client system calls to the server when it needs something from him, but in an EDA the process is quite different. Event Driven Architecture works with a `publishers` and `subscribers` entities. It means that when something important happends in a system it will publish an event. And then one or several consumers can consume this event and react properly to it. For example we could think in a newspaper and several news agencies working with this newspaper. Everitime the newspaper publishes a news it also publish an event with the most relevant information about this new. Then the consumers or subscribers receive the new as soon as it happend and they can inform to them customer. Well the example is not the best but maybe with this diagram the things are more clear... ;) Normally an Event Queue is used to transport the events. Apache Kafka is one of the most used and also give to you solution many important aspects as failover, scalabillity,...

<p align="center">
    <img src="/assets/images/eda_1.jpg"/>
</p>

As you can imagine, Event Driven Architectures allows us to implement very dynamic system integrations; loosely coupled integrations, asyconcronous and in real-time or near-to-real-time. 

## How to use this in Salesforce?

Well, as we said before, we are going to use this concepts in next post in combination with Salesforce. Just to say that with three technologies (Platform Events, Streaming API and WebSocket) we can use Salesforce as a publisher or as a subscriber. And the same happends with the 3th party system, it could be the producer or the consumer.

### Platform Events

In the next post we are going to publish an basic example where an Application in Heroku will be the producer and Salesforce will act as Consumer. We will use Salesforce Process Builder to create a chatter post when we receive an event from the Heroku Application.

### Streaming API

In this post we are going to create an near-to-real time example where Salesforce will act as publisher and subscriber. We will use Streaming API to inform to the Salesforce UI and see an update just when the thing is happening in the backend.

### WebSockets

Finally we will release a last post where Salesforce will communicate with other Heroku Application. Here the technology implemented will be websockets and the integration will go thru the Salesforce UI, which will be the consumer and Heroku App that will be the producer.