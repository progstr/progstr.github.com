---
title: JavaScript
layout: default
permalink: javascript.html
---
Table of Contents
=================
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Getting Started](#getting_started)
* [Supported Platforms](#supported_platforms)

Introduction
=====================
Progstr.log is a service that collects and manages programmer log entries in the cloud. Most applications log errors and special external events generated by users or third party systems. Logging all can be both simple or complex and might grow into a full-blown application. Progstr.log tries to do that for you. It is simple to use and gets you going in minutes, but it also takes care of the heavy lifting of involved in handling the log data your appliation generates. Our service takes away the pain of complex logging system configurations, provides a centralized location for all your log entries and helps you analyze the data you’ve collected over time. **progstr-js** is the JavaScript client library that collects log entries and transports them to the Progstr data store.

Installation
============

The easiest way to get the *progstr-js* library into your project is to include the `progstr.js` script on your page by refering it from the `ajax.progstr.com` server:

{% highlight html %}
<script type="text/javascript" src="http://ajax.progstr.com/progstr-js/1.0.0/progstr.js"></script>
{% endhighlight %}

Configuration
=============

[Sign up](https://app.progstr.com/signup) for the progstr.log service and get your API token from the account *Settings* page. That token is used to identify you against the service.

Configure the API token by setting the `Progstr.apiToken` property in a script block:

{% highlight html %}
<script type="text/javascript">
    Progstr.apiToken = "6f413b64-a8e1-4e25-b9e6-d83acf26ccba"
</script>
{% endhighlight %}

Getting Started
-------------------------
There are four log severity levels that you can use to log events of different importance:

* *Info*: used to log general information events that can be used for reference or troubleshooting if needed.
* *Warning*: something odd happened and somebody needs to know about it.
* *Error*: something failed and needs to be fixed.
* *Fatal*: the entire application or a critical part of it is not working at all.

To log an event you need to obtain a logger object and call some of its `info`/`warning`/`error`/`fatal` methods.

{% highlight javascript %}
var logger = Progstr.logger("js.demo.source")
...
logger.info("Loading page: " + window.location.href)
...
logger.warning("User not found.")
...
logger.error("Operation failed.")
...
logger.fatal("Could not connect to server.")
{% endhighlight %}

Supported browsers
------------------------
* All major browsers out there.

Feedback
--------
* The library source code is hosted on [GitHub](https://github.com/progstr/progstr-js).
* If you have a feature suggestion, enhancement idea, or a bug report for the client library, please open a ticket on the project [issue tracker](https://github.com/progstr/progstr-js/issues).
* For general problems or inquiries regarding integrating the library in your project or the *progstr.log* service, please contact [support](mailto:support@progstr.com).
