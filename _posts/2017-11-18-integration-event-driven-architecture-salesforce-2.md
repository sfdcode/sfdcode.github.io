---
title: "Integration: Event Driven Architecture in Salesforce (2 of 4)"
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
As we inform in the last <a href="https://sfdcode.github.io/integration/integration-event-driven-architecture-salesforce/" target="blanck">post</a>, we are doing a series with EDA (Event Driven Architecture) in Salesforce. Here is the second post of the serie and here we will treat the Platform Events approach. For those who doesn't know, `Platform Events` is a new way to do integrations in Salesforce. Is a new feature released as GA in `winter'18` and will be enhanced in future releases of Salesforce. Nowadays we can create publisher and subscriber to Platform Events in Salesforce and also in external systems. In this post we are going to use the Platform Events `API` to publish the event into Salesforce and an Apex Trigger and a Process Builder to consume it.

## Platform Events

Platform Events are well explained in the Salesforce official documentation <a href="https://developer.salesforce.com/docs/atlas.en-us.platform_events.meta/platform_events/platform_events_intro.htm" target="_blank">here</a>. As a very brief sumary we will say that `Events` are similar to custom objects, but we can only insert them, not update and delete operations are available for platform events. We will define fields that would be the payload of the event. And then we can publish events within the following technologies:
<ul>
<li>Process Builder</li>
<li>Flows</li>
<li>Apex</li>
<li>API</li>
</ul>
And to subscribe to events we can use this ones:
<ul>
<li>Process Builder</li>
<li>Flows</li>
<li>Apex Triggers</li>
<li>CometD</li>
</ul>

In our example we are going to use the `API` to publish the Event in the Heroku App and then we will define an `Apex Trigger` and a `Process Builder` to consume the Event.

## Salesforce configuration

### Create Platform Event

First step in this tutorial, we need to create an Event. So let's go to login into our Salesforce organization. An then we will go to the Setup > Platform Events. Once there we press in `New Platform Event` button. We fill the Label and Plural Label to `SfdcTest` and leave the rest of the parameters with the default values. 

<p align="center">
    <img src="/assets/images/eda_2_1.png"/>
</p>

Click on Save to create the platform event.

Now we are going to create a field in the event to send as its payload. Find the section name `Custom Fields & Relationships` and then click in `New`. Select `Text` as the field type and click in `Next`. Now let's going to introduce the Field Label as `Payload` and the Length as `255`. As we see in this picture.

<p align="center">
    <img src="/assets/images/eda_2_2.png"/>
</p>

Click on Save to create the event field.

### Consume the Platform Event in Salesforce

Now we need to create a consumer or subscriber to handle the Platform Event. Previous in this post we said that we will use Process Builder to do that. Well, we will use Process Builder but we also will need to create an Apex Trigger. The reason for that is that Process Builder is still not ready to use the payload of the Platform Event in the action side. Now the unique thing we can do with the Platform Event payload is to correlate with an Salesforce Object which can be used in the actions of the Process Builder. So the trick we are going to use consist in create a Trigger, this trigger will create an auxiliar Salesforce Object with the payload and then a Process Builder will use this payload to post a Chatter message. Is a bit tricky but it will work.

So first of all we are going to create a Salesforce Object with a field of type text to track the payload received in the Platform Event.

<p align="center">
    <img src="/assets/images/eda_2_3.png"/>
</p>

And now we need to create a simple Trigger for the Platform Event we have created before. The code of this trigger is the next:

{% highlight java %}
trigger Consume_Sfdcode_Event on SfdCode__e (after insert) {
    
    List<SfdcEventAux__c> laux = new List<SfdcEventAux__c>();
    
    //This for loop is to bulkified on the creation of events
    for (SfdCode__e event : Trigger.New) {
        SfdcEventAux__c saux = new SfdcEventAux__c();
        saux.Payload__c = event.Payload__c;
        laux.add(saux);
    }
    
    insert laux;

}
{% endhighlight %}

This code is very simple. It is bulkified to it can consume in a single Salesforce transaction the creation of several events. The code create an auxiliar SObject for each Platform Event that it receives. And it copies the Payload of the event into the SObject. Then we will create a Process Builder that will create a chatter post on the creation of the SObject with the Payload received.

So, let's go to create the Process Builder to finish the Salesforce implementation.

<p align="center">
    <img src="/assets/images/eda_2_4.png"/>
</p>

## Publishing a Platform Event

As we had explained before there is many ways to publish an event in Salesforce Platform Events, we are going to use one mechanism that would be one of the most used. Salesforce provides an `API` to publish events. We can use this API in our applications to publish events, but in this case, we just will use the `workbench` application that is a very simple way to call REST services

### Salesforce workbench

Open your browser and go to `https://workbench.developerforce.com/restExplorer.php`

If you had used the same name proposed in this post you will be able to find the endpoint: `/services/data/v41.0/sobjects/SfdCode__e`. Let's go to send this simple request:

{% highlight json %}
{
  "Payload__c": "This is a simple test"
}
{% endhighlight %}

Now press on the `Execute` button and you will see something like this:

<p align="center">
    <img src="/assets/images/eda_2_5.png"/>
</p>

If in the response is successful that means that the Platform event has been created successfully. Now let's go to see in our user chatter page to see if we have received the post.

<p align="center">
    <img src="/assets/images/eda_2_6.png"/>
</p>

Yeah!!! Everything has worked fine. In this post we see a very simple way to communicate our Salesforce with and external system. 

## Next steps

In the next post we will see how enhance the use case by using Streaming API. With Streaming API we will be able to inform our UI, so as soon as the event happend we will be able to see our page updated with the new information. This is commonly called `server push` but let's wait till next post to see it ;)