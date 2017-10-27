---
title: "Lightning: Using reCAPTCHA in Lightning Components"
categories:
  - Lightning
tags:
  - UI
  - Lightning
  - Javascript
  - Salesforce
---
reCAPTCHA is a Google project that has been created to make your forms in public websites more safe. The basic concept of this project is to stop the bots that are trying to fill web forms. By introducing some task that are very easy for the Humans but almost impossible for a bot. For instance reading some deformated text. The Google guys are very smart and they are usign the reCAPTCHA project for more goals than protect the web form. They are using it also to train them Artificial Intelligence systems. Well done guys!!!. Here you have all the details for the <a href="https://www.google.com/recaptcha/intro/android.html" target="_blank">reCAPTCHA project</a>

## Configure reCAPTCHA

Before we start with this short tutorial we need to create a site key in the reCAPTCHA admin site. This site key will allow to the Google API for reCAPTCHA to validate our site. To get a valid site key you need to have a Google account. Then, once login, navigate to this <a href="https://www.google.com/recaptcha/admin" target="_blank">site</a>. Set the domain, in our case, `force.com`. And select the `reCAPTCHA V2` checkbox. You can see the needed configuration in this image

<p align="center">
  <img src="/assets/images/lightning-recaptcha1.png"/>
</p>

Click in the `Register` button.

Now we have a site key that we need to store in our javascript code.

## Salesforce coding

For this Lightning Component we are going to need several pieces. Of course, the main will be to create a Lightning Component, that will show the reCAPTCHA and a form that will be enabled when the reCAPTCHA has validated we are not a bot. But we also will need to create a VisualForce page. A VisualForce page?!?!? why??. Well this is because our friend the `Locker Service`. If you don't know the Lightning Locker Service I will recommend you to read this <a href="https://developer.salesforce.com/blogs/developer-relations/2016/04/introducing-lockerservice-lightning-components.html" target="_blank">article</a>. As a short summary I will say that the Locker Service is a piece that Salesforce created to enhance the security in the Lightning Components. His basic fuctions are two. Protect the DOM of the Lightning Components to avoid forbidden accesses, a Custom Component can access his DOM and the DOM of his children but it can't access to other components DOM. And the second function of the Locker Service, is to protect the resource for security reasons, this is based on CSP and it will only allow to have the sources in some location. One of the forbidden accesses is to call XHR request from other domain directly, and as in our tutorial we need to call to google API from Salesforce, the Locker Service will cut this request. Here is where the VisualForce page come to help us to solve this restriction.

### Lightning code

Where, the lightning code we are going to develop in this example is pretty basic. Is just a simple button and when the user will press this button an alert message will be show. But the button will be disabled until the user validates in the reCAPTCHA that he is not a Bot. Simple not? The last part is an iframe, a `lightning:container` could be used aswell, to the VisualForce that actually prints the reCAPTCHA.

The source code of the Lightning component is this:

{% highlight xml %}
<aura:component implements="flexipage:availableForAllPageTypes" access="global" >
    <aura:handler name="init" value="{!this}" action="{!c.doInit}" />
    <iframe src="/apex/sfdcode_recaptcha" height="74px" style="border:0px"/>
    <br/>
    <lightning:button aura:id="myButton" label="Submit" onclick="{!c.doSubmit}" disabled="true" />    
</aura:component>
{% endhighlight %}

and the client-side controller is this:

{% highlight javascript %}
({
	doInit: function (cmp, evt, helper){
		let vfOrigin = "https://jadm--c.eu5.visual.force.com";
        window.addEventListener("message", function(event) {
            console.log(event.data);
            if (event.origin !== vfOrigin) {
                // Not the expected origin: Reject the message!
                return;
            } 
            if (event.data==="Unlock"){            	
            	let myButton = cmp.find("myButton");
                myButton.set('v.disabled', false);
            }            
        }, false);                
	},
    doSubmit: function (cmp, evt, helper){
        alert("Do Submit");
    }
    
})
{% endhighlight %}

In this client-side controller the more complex part is the `window.addEventListener`. Is created to the communication with the VisualForce page. That code basically enables the button when it receives the message `Unlock` from the VisualForce.

And this is the look&feel of this component once deployed

<p align="center">
  <img src="/assets/images/lightning-recaptcha2.png"/>
</p>

### VisualForce code

The VisualForce is what actually will display the reCAPTCHA, and when the reCAPTCHA validates that you are not a Bot then it will post a message to the Lightning Component to unlock the button.

Here is the code:

{% highlight xml %}
<apex:page >
    <html>
      <head>
        <title>reCAPTCHA demo: Explicit render after an onload callback</title>
        <script type="text/javascript">
          var verifyCallback = function(response) {
              parent.postMessage("Unlock", "https://jadm.lightning.force.com");
          };
          var onloadCallback = function() {
              grecaptcha.render('html_element', {
                  'sitekey' : '<your_site_key>',
                  'callback' : verifyCallback,
              });
          };
        </script>
      </head>
      <body>
        <form action="?" method="POST">
          <div id="html_element"></div>
            <br/>
            <input type="submit" value="Submit" style="display:none"/>
        </form>
        <script src="https://www.google.com/recaptcha/api.js?onload=onloadCallback&render=explicit" async="" defer="">
        </script>
      </body>
    </html>
</apex:page>
{% endhighlight %}

Some things that need a little explanation. First, the `script` that loads the reCAPTCHA uses a function `onloadCallback`. This callback is executed when the script finish it load. This function is very important for two reason. First it defines the `site key` that we get in the first step of this tutorial. Second it will define the callback to be invoked when the reCAPTCHA validates that the user is a human. Second, the `form` with a simple submit button which is hidden. This is a requirement of the reCAPTCHA, is there is no form then the reCAPTCHA doesn't work. 

### YouTube

<<Video Embebido o Link to YouTube>>

### Source code

All the source code of this post is in this <a href="https://github.com/sfdcode/talend-on-heroku.git" target="_blank">GitHub repository</a>