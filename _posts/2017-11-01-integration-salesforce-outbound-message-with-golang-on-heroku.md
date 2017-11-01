---
title: Integration of Salesforce Outbound Message with GoLang on Heroku
categories:
  - Integration
tags:
  - Workflow
  - Outbound Message
  - GoLang
  - Heroku
---
Salesforce offers many ways to call your backend systems. This ways are commonly call integration patterns an some examples are those, callout a web service (REST or SOAP), use an ETL (if you are interested in this pattern, please read our previous post <a href="/integration/integration-salesforce-talend-etl-heroku/" target="_blank">integration-salesforce-talend-etl-heroku</a>), and Outbound Messages in a workflow. This one is probably the simplest way to integrate withing third party system and the most declarative. We don't need to codify anything in Salesforce to use the patterns. This are enaugh reasons to create a post in our blog. So, here we go

Before we start we need to have a system to integrate with, pretty obvious, isn't it? ;) So, for this post we are going to deploy a Web Service Server on Heroku. This service will be call by the Outbound Message when the workflow would be trigged in the Salesforce org. As we see in previous posts Heroku allows us to use several programing languages to write our application. In this post we are going to use Go, commonly called GoLang, Go is a modern language created by Google and is getting more and more popular each day. For further information about Go I recommend you to follow this simple Go tutorial that covers all the functionalities of this funny language. <a href="https://
tour.golang.org/welcome/1" target="_blank">A Tour of Go</a>


### Deploying on Heroku

Deploy an app on Heroku is a simple process. There is a Heroku CLI with all the operations to create and mantain an application. But there is even a simpler way to deploy a full app on Heroku only by clicking a button. This a very nice way to distribute applications that can be used for your live demos. 

In this GitHub repository <a href="https://github.com/sfdcode/basic-go-postgres" target="_blank">basic-go-postgres</a> you have a project ready to be deploy by a button click. In order to do that you have to create a `app.json` file where you configure the settings of your app. In our case we are only going to add the postgres addon.

#### ```app.json```
{% highlight json %}
{
    "name": "Go Sample",
    "description": "A simple Go server with Postgres",
    "repository": "https://github.com/sfdcode/simple-node-postgress",    
    "addons": ["heroku-postgresql:hobby-dev"],    
    "keywords": ["golang"],
    "buildpacks": [{"url": "heroku/go"}]
}
{% endhighlight %}

Now we are ready to deploy by a single click our app. Go to the <a href="https://github.com/sfdcode/basic-go-postgres" target="_blank">basic-go-postgres</a> repository and click in the `Deploy to Heroku` button


<img src="https://www.herokucdn.com/deploy/button.png"/>
<b><span style="font-size: 11px">This is just an image, you need to click the button from the GitHub repository.</span></b>

Once you click on the button a form will appear asking us the name of the application. In our case we will call it `my-simple-golang-app`

<p align="center">
    <img src="/assets/images/outbound-message-1.png"/>
</p>

We click on Deploy and then we save the URL of the new application. That will be `https://my-simple-golang-app.herokuapp.com`

### Creating the Outbound Message

Create an workflow that triggers an Outbound message in Salesforce is a declarative process. We don't need to write a single line of code.

To create the workflow rule. Go to Setup > Workflow Rules > Add New > Pick Account

Fill the workflow config like in the below image

<p align="center">
    <img src="/assets/images/outbound-message-2.png"/>
</p>

Click Next and create a new action of type Outbound Message. And fill the setting like in this image:

<p align="center">
    <img src="/assets/images/outbound-message-3.png"/>
</p>

In this example we are sending only the `Id` and `Name` of the account, but of course, we could pick as many fields as we need in our requirement.

Click Save and Activate the Workflow rule.

Thats all!!!! In two simple screens we have integrate our Salesforce with our backend.

Please check this <a href="https://youtu.be/D6Aeb7SucdE" target="_blank">YouTube</a> video in our channel to see the whole process step by step
