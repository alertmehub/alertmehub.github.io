---
layout: page
title: Getting Started
permalink: /getstarted/
---

# Getting Started
Getting started with the alertme service involves three simple steps:
1.	[Signing up](#signing-up) and [creating topics](#creating-topics).
2.	[Deploying the alertme component](#deploying-the-component) into your application.
3.	[Publishing alerts](#publishing-alerts)

## Signing Up
In order to use the Alertme service, you must first [sign up for an account](https://admin.alertmehub.com/register/)

Pick a publisher id for your organization – typically the domain where the Alertme component will be placed.  Pick a password, and provide the name and email of the administrator for this publisher id.  The email will only be used to contact the administrator with important information about the Alertme service.

## Creating Topics
After signing into the Alertme Administration portal, the first step is to define the Topics to which subscribers can subscribe to receive alerts.  For example, the bank contoso.com might define the following topics:
* Low Balance Alert
* Large Transaction Alert
* Scheduled Transfer Reminder

The topics you define will be driven by your business model.
The following information is required to set up a topic.

### Basic Info
* Name – This is the internal name for the topic.  It is used by the API (see Using the API below) to publish an alert for this topic.  It should be a short name without any spaces.
* Label – The name of the topic as displayed to the subscriber when signing up for alerts.
* Description – A short (one or two sentence) description of the topic shown to the subscriber.

### Parameters
Topics can define parameters that further control whether a subscriber will receive an alert for this topic.  For example, the “low-balance” topic could have a parameter called “min-balance” where the subscriber will specify at what threshold they would like to receive this alert.
* Name – This is the internal name for the parameter.  It is used by the API (see Using the API below) when publishing an alert for this topic.  It should be a short name without any spaces and must start with a letter.
* Label – The name of the parameter as displayed to the subscriber when signing up for alerts.
* Type  – The type of parameter (number, text, lookup, etc).
* Lookup – If the parameter type is “lookup”, then pick a lookup list to be displayed.

### Lookups
For the lookup parameter type, you can provide a list of lookup values in either JSON, or CSV format.
The following fields are provided for the lookup values:
* Id – Can be any string
* Item Name – Displayed to the user.
* Group – Optional.  If a group is provided, then the items in the dropdown list will be grouped under that heading.

### Alert Text
Finally, provide the alert text to be sent to the subscriber when the alert is triggered.
You can provide separate messages for emails and SMS text.  You can also provide simple value substitutions using double curly braces.  See the section on the API for how to pass in substitution data.
The subject of the email will be the Topic Name, and the Topic Name will also be prepended to the SMS text.  The email will come from [publisherId]@mail.alertmehub.com.

## Deploying the component
The alertme component is the way that your users can subscribe to the alert topics that you’ve defined.  It must be placed on a page of your existing portal.  There are several different ways to accomplish this, depending on your portal technology stack.

ASP.NET
If your portal is built on top of Microsoft ASP.NET MVC, then the process of deploying the component looks like this:
1.	Download the latest JavaScript from https://github.com/alertmehub/alertme-component-javascript/tree/master/lib.  
2.	Place the html tag in your View html. 

``` html
<alertme-preference-center publisher="test.com" token="@ViewBag.CustomerToken"></alertme-preference-center>
<script type="text/javascript" src="alertme-1.0.4.js"></script>
````

3.	In the controller, make an API call to get the token.

``` cs
        public async Task<IActionResult> Alerts()
        {
            string tokenUrl = "https://api.alertmehub.com/api/v1/subscriber/token/" + User.Identity.Name;
            using (var httpClient = new HttpClient())
            {
                httpClient.DefaultRequestHeaders.Add("Authorization", "4f0c8355d0107f3e4b4705b85085506a4567debfb0842d4b2ab1ee38dcd62ac5");

                try
                {
                    string response = await httpClient.GetStringAsync(tokenUrl);
                    ViewBag.CustomerToken = response.Replace("\"", "");
                }
                catch(HttpRequestException ex)
                {
                    ViewBag.Error = ex.Message;

                }
            }
            return View();
        }
```

## Publishing Alerts

Publishing alerts is done through our API.  

To publish an alert, make a POST request to https://api.alertmehub.com/api/v1/alert/
Set the authorization header to your API key.
And set the document body to a JSON object like so:
``` JSON
{
  "topic": "topic1",
  "parameters":{"parameter1": 100},
  "data":{"name": "Eric"},
  "options":{"smsProvider": "manual", "emailProvider": "manual"}
}
```
* topic - The name of the topic as defined in the admin tool
* parameters - One or more parameter values as an object
* data - Substitution data e.g. {{name}} in the message template would be replaced by Eric.
* options - smsProvider and emailProvider.  If set to manual, then the message is not sent, and only the email addresses and mobile phone numbers are returned - so that you can "manually" send them on your side.  If left blank or set to "alertme" then the alertme service will send the text/email.

