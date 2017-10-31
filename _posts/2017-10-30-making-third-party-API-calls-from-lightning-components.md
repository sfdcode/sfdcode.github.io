---
title: Making Third Party API Calls From Lightning Components
categories:
  - Integration
tags:
  - apex
  - lightning
  - API call
  - webservice
---
There are many APIs, public or available in the enterprise service catalog. The traditional way is to use server side capabilities in our application to make requests, process and deliver to the browser. In this article we're going to see this approach and a different one, using the Content Security Policy (CSP) available in Lightning components to make this calls directly from the browser.

## Making API calls to third party API from Lightning component in client-side
Browsers apply a same-origin restriction to content requests. This restriction prevent the browser running from one domain (origin) from getting content retrieved from a different one. This article describes the mechanism to make client-side cross-origin requests in from Salesforce Lightning components, following the specifications defined in <a href="https://www.w3.org/TR/cors/" target="_blank">Cross-Origin Resource Sharing (CORS)</a>.
<p align="center">
  <img src="/assets/images/lightning_cors.png"/>
</p>

First we're going to analyze the behavior, for that we build a simple Lightning component to make third party API calls providing an Url, here we go:
#### ```lightning_thirdparty_callout.cmp```
{% highlight html %}
<aura:component implements="flexipage:availableForRecordHome,force:hasRecordId" access="global" >
    <lightning:input aura:id="url" label="url" name="url" placeholder="url" type="url"/><br/>
    <lightning:button label="Send Request" onclick="{!c.thirdpartyClientCall}"/>
</aura:component>
{% endhighlight %}

#### ```lightning_thirdparty_calloutController.js```
{% highlight javascript %}
({ 
    thirdpartyClientCall : function(cmp, event, helper) { 
        helper.thirdpartyClientCaller(cmp, event, helper); 
    } 
})
{% endhighlight %}

#### ```lightning_thirdparty_calloutHelper.js```
{% highlight javascript %}
({ 
    thirdpartyClientCaller : function(cmp, event, helper) { 
    	var xmlHttp = new XMLHttpRequest();
        // Test url: https://heroku-csp-test-server.herokuapp.com
    	var url = cmp.find("url").get("v.value");
    	console.log(url);
    	xmlHttp.open( "GET", url, true );
	    xmlHttp.setRequestHeader('Content-Type', 'application/json');
		xmlHttp.responseType = 'text';
		xmlHttp.onload = function () {
    	    console.log("onload");
        	console.log(xmlHttp.readyState);
        	console.log(xmlHttp.status);
    		if (xmlHttp.readyState === 4) {
    	    	if (xmlHttp.status === 200) {
	            	console.log(xmlHttp.response);
            		console.log(xmlHttp.responseText);
		        }
	    	}
		};
	    xmlHttp.send( null );
	    console.log("Request sent");
    }
})
{% endhighlight %}

Now we have this form and can use it to test all the possible scenarios, to check the errors retrieved by the browser in each case, and we can see how to configure the things to make this kind of calls:
<p align="center">
  <img src="/assets/images/lightning_form_callout.png"/>
</p>

### Errror 1: Url not allowed in CSP
For example if we provide a url not allowed, as https://www.yahoo.com, the code tries to invoke the XMLHttpRequest, but the error that we're going to get is this one. In fact, the browser blocks the callout.
{% highlight plaintext %}
Refused to connect to 'https://www.yahoo.com/' because it violates the following
Content Security Policy directive: "connect-src 'self'
http://www.google.com https://staging.bluetail.salesforce.com
https://na73.salesforce.com https://api.bluetail.salesforce.com 
https://preprod.bluetail.salesforce.com *.na73.visual.force.com
https://heroku-csp-test-server.herokuapp.com".
{% endhighlight %}

### Error 2: Url allowed in CSP but without permissions in thirdparty server
{% highlight plaintext %}
Failed to load https://heroku-csp-test-server.herokuapp.com/noaccess: Response to
preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin'
header is present on the requested resource. Origin
'https://mbdev1-dev-ed.lightning.force.com' is therefore not allowed access.
{% endhighlight %}

To avoid these problems in the configuration, the first thing is to create the entry for the thirdparty url in Content Private Policy (CSP). Go to setup and in the quick find box type "CSP". Create a new entry and provide the url, name and description, once you have it configured, you are allowing to perform callout requests over that domain:
<p align="center">
  <img src="/assets/images/new_csp.png"/>
</p>

Plus to this the thirdparty service have to authorize this kind of callouts, to understand that we are going to see the type of the call that is doing the Lightning component, note that the used method in the request is ```OPTIONS```, the response manage the authorization using the headers Access-Control-Allow-Origin (for the domain), Access-Control-Allow-Methods (HTTP methods) and Access-Control-Allow-Headers (other headers like Content-Type for example)

__REQUEST:__
{% highlight properties %}
OPTIONS /domainaccess HTTP/1.1
Host: heroku-csp-test-server.herokuapp.com
Connection: keep-alive
Access-Control-Request-Method: GET
Origin: https://mbdev1-dev-ed.lightning.force.com
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36
Access-Control-Request-Headers: content-type
Accept: */*
Referer: https://mbdev1-dev-ed.lightning.force.com/
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.8,es;q=0.6,gl;q=0.4
{% endhighlight %}

__RESPONSE:__
{% highlight properties %}
HTTP/1.1 200 OK
Server: Cowboy
Connection: keep-alive
X-Powered-By: Express
Access-Control-Allow-Origin: https://mbdev1-dev-ed.lightning.force.com
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: Content-Type
Content-Type: application/json; charset=utf-8
Content-Length: 66
Etag: W/"42-uRgKIC1HYTS0DskZrCgIynTJI8A"
Date: Mon, 30 Oct 2017 19:42:19 GMT
Via: 1.1 vegur
{% endhighlight %}



## Making API calls to third party API from Lightning component using an AuraEnabled proxy class
Sometimes it is not possible to modify the behavior of the thirdparty API provider to authorize requests from our salesforce organization, then the alternative is to use the AuraEnabled Lightning Controller using Apex on the server side where this kind of browser restrictions does not apply and then provide to the browser the result, acting as a proxy. For this article we developed a simple Apex proxy class that can be extended from your Lightning AuraEnabled controllers to provide this proxy functionality.
<p align="center">
  <img src="/assets/images/apex_proxy.png"/>
</p>
Here we have a simple Lightning component who generates a form to collect all the HttpRequest parameters and call to a generic proxy, available in our git hub repo GitHub repository [apex-http-proxy-for-lightning](https://github.com/sfdcode/apex-http-proxy-for-lightning.git)
#### ```lightning_thirdparty_callout_apex_proxy.cmp```
{% highlight html %}
<aura:component implements="flexipage:availableForAllPageTypes" access="global" controller="SampleController">
    <lightning:input aura:id="method" label="method" name="method" placeholder="method" type="text"/><br/>
    <lightning:input aura:id="endPoint" label="endPoint" name="endPoint" placeholder="url" type="url"/><br/>
    <lightning:input aura:id="clientCertificateName" label="clientCertificateName" name="clientCertificateName" placeholder="clientCertificateName" type="text"/><br/>
    <lightning:input aura:id="timeout" label="timeout" name="timeout" placeholder="timeout" type="number"/><br/>
    <lightning:textarea aura:id="body" label="body" name="body"/><br/>
	  <ui:inputCheckbox aura:id="compressed" label="compressed" value="false"/>
    <lightning:textarea aura:id="headers" label="headers" name="headers"/><br/>
    <lightning:button label="Send Request" onclick="{!c.thirdpartyProxyCall}"/><br/><br/>
</aura:component>
{% endhighlight %}

#### ```lightning_thirdparty_callout_apex_proxyController.js```
{% highlight javascript %}
({ 
    thirdpartyProxyCall : function(cmp, event, helper) { 
        helper.thirdpartyProxyCaller(cmp, event, helper); 
    } 
})
{% endhighlight %}

#### ```lightning_thirdparty_callout_apex_proxyHelper.js```
{% highlight javascript %}
({ 
    thirdpartyProxyCaller : function(cmp, event, helper) { 
        
        var action = cmp.get("c.send");
        var endPoint = cmp.find("endPoint").get("v.value");
        var method = cmp.find("method").get("v.value");
        var timeout = null;
        if (!$A.util.isUndefined(cmp.find("timeout")) && cmp.find("timeout") != null) {
            timeout = parseInt(cmp.find("timeout").get("v.value"));
        }
        var compressed = cmp.find("compressed").get("v.value");
        var clientCertificateName = "";
        if (!$A.util.isUndefined(cmp.find("clientCertificateName")) && cmp.find("clientCertificateName") != null) {
        	cmp.find("clientCertificateName").get("v.value");
        }
        var headersStr = "";
        if (!$A.util.isUndefined(cmp.find("headers")) && cmp.find("headers") != null) {
            headersStr = '{' + cmp.find("headers").get("v.value") + '}';
        } else {
            headersStr = '{}';
        }
        var headers = JSON.parse(headersStr);
        var body = "";
        if (!$A.util.isUndefined(cmp.find("body")) && cmp.find("body") != null) { 
            body = cmp.find("body").get("v.value").replace('\"','\\"');
        }
        var jsonReq =  {"endPoint": endPoint, 
                        "method": method,
                        "timeout": timeout,
        			          "compressed": compressed,
                        "clientCertificateName": clientCertificateName,
        			          "body": body,
        			          "headers": headers
            		      };
        action.setParams({ "request": JSON.stringify(jsonReq) });
        // Create a callback that is executed after 
        // the server-side action returns
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {
                var jsonResp = JSON.parse(response.getReturnValue());
                console.log("json string: " + response.getReturnValue());
                console.log(jsonResp);
            }
            else if (state === "INCOMPLETE") {
                // do something
            }
            else if (state === "ERROR") {
                var errors = response.getError();
                if (errors) {
                    if (errors[0] && errors[0].message) {
                        console.log("Error message: " + 
                                 errors[0].message);
                    }
                } else {
                    console.log("Unknown error");
                }
            }
        });

        $A.enqueueAction(action);
    }
})
{% endhighlight %}

The proxy can be deployed from github directly to your org, and you only need to create your own @AuraEnabled controller, here an example.

And provide a simple JSON structure from the Lighting Component:
{% highlight javascript%}
{"endPoint": "https://mydomain.org", 
"method": "POST",
"timeout": 15000,
"compressed": false,
"clientCertificateName": "",
"body": "a=1",
"headers": {"Content-Type": "application/json"}
}
{% endhighlight %}

It will return a simple JSON structure too, to manage in your Lightning Component. Don't forget to add an entry in > Setup > Remote sites to provide the entry for the domain, it is needed to allow the Salesforce org to connect to external domains.
{% highlight json%}
{"status": "OK",
"statusCode": 200,
"body": {"content": "content1"},
"headers": {"Content-Type": "application/json"}
}
{% endhighlight %}

The sample controller only needs to call the HttpService.send method and expose the method with @AuraEnabled to provide access to the Lightning Component:
{% highlight java %}
public with sharing class SampleController {
	@AuraEnabled
  public static String send(String request) {
    System.debug(request);
    HttpService.Request req = (HttpService.Request)JSON.deserialize(request, HttpService.Request.class);
  return JSON.serialize(HttpService.send(req));
  }
} 
{% endhighlight %}

All the source code of this post is in this GitHub repository[apex-http-proxy-for-lightning](https://github.com/sfdcode/apex-http-proxy-for-lightning.git)