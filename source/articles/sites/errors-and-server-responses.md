---
title: Errors and server responses
description: Understand server responses and error messages.
category:
  - supporting
  - debugging

---


## Pantheon Error Messages

Sometimes, there are problems in the cloud and one of Pantheon's services is unable to fulfill a request. In those rare and unfortunate circumstances, Pantheon will serve an error message instead of expected site content.  













### Pantheon 401 Unauthorized

![](https://pantheon-systems.desk.com/customer/portal/attachments/184676)  





### Pantheon 403 Forbidden

![](https://pantheon-systems.desk.com/customer/portal/attachments/184677)  
"Access denied to uploaded PHP files." This message is shown when a PHP file is attempted to be accessed in Valhalla, Pantheon's network file system.  




### Pantheon - 404 Unknown Site

![](https://pantheon-systems.desk.com/customer/portal/attachments/184679)  


### Pantheon - 502 Bad Gateway
 ![](https://pantheon-systems.desk.com/customer/portal/attachments/184849)

"There was an error connecting to the PHP backend." If the php-fpm process hangs or cannot start, Nginx, the web server will report this problem.

### Pantheon - 502 Routing failure
 ![](https://pantheon-systems.desk.com/customer/portal/attachments/184850)

"Page Could Not Be Loaded. The request could not be completed due to a networking failure. Contact support if this issue persists." An internal networking issue has occurred with Styx, Pantheon's routing mesh.

### Pantheon - 503 Target in maintenance
 ![](https://pantheon-systems.desk.com/customer/portal/attachments/184852)

"The web site you were looking for is currently undergoing maintenance." This is  **not** Drupal's maintenance mode; this is a manually toggled emergency message reserved for unusual circumstances when a site is known to be not available.

### Pantheon - 503 Target not responding
 ![](https://pantheon-systems.desk.com/customer/portal/attachments/184854)

"The web page you were looking for could not be delivered." No DROPs are available to complete the request. These errors occur when PHP rendering resources for your site are full. Each DROP has a fixed limit of requests it can concurrently process. When this limit gets hit, nginx will queue up to 100 requests in the hope that PHP resources will free up to service those requests. Once nginx's queue fills up, the DROP cannot accept any more requests.

We could increase the nginx queue above 100, but it would only mask the problem for longer. It would be like a retail store with a grand opening line longer than it can serve in the business hours of a single day. At some point, it's better to turn away further people and serve those already in line.

This can be caused by sustained spikes in traffic (often caused by search engine crawlers) and by having PHP processes that run too slowly or have long waiting times for external resources which occupy the DROP for long periods. If you have too much traffic for your site's resources, consider [upgrading your site plan](/articles/howto/selecting-a-plan/).

### Pantheon - 503 Database not responding
 ![](https://pantheon-systems.desk.com/customer/portal/attachments/184855)

"The web page you were looking for could not be delivered." The MySQL database is not responding, possible from being suspended and not resuming.

### Error 503 - Service Unavailable
 ![](https://pantheon-systems.desk.com/customer/portal/attachments/231974)

"The service is temporarily unavailable. Please try again later."

This error generally occurs when a request is going through our Rackspace Cloud load balancer, which imposes a timed limit on requests. If end-user pages take longer than this threshold, there is a performance issue with the site. More information about [Timeouts on Pantheon](/articles/advanced-topics/timeouts-on-pantheon/) is available in our helpdesk.

If you get a generic Service Unavailable that is not styled like the above and you're using AJAX when HTTP basic auth (the Security username/password), then that's a misleading message - best workaround is to disable the security option for the environment for testing.

### Pantheon - 504 Gateway Timeout

![](https://pantheon-systems.desk.com/customer/portal/attachments/185064)  


Typically the request timeout is much shorter than the hard timeout for PHP. While you may be able to let an operation run for several minutes in your local development environment, this isn't possible on Pantheon. Luckily there are ways to solve the problem.

There are many things which could cause your site to exceed the request timeout limit. The first step to fixing any problem is to identify the root cause.

## Administrative Pages

It is unfortunately possible for some normal operations in Drupal to outlast the request timeout. Submitting the modules page, manually running cron, running update.php, or flushing caches can be extremely slow operations on sites with large numbers of modules and/or a lot of data and activity.

If you are seeing request timeouts for administrative operations, you may be able to address this by optimizing your application, or by using a work-around (see below).

## Slow Queries / High Query Volume

Pages which leverage a large number of Drupal views can often bog down because of the slow speed of the queries. It can also happen that a sufficiently high query volume (1,000+ queries on one page) can push things over the edge.

Individually slow queries should be refactored if possible. However, often cacheing can help mitigate slow queries or high query volumes quickly. There will still be slow page loads when the cache needs to be populated, but subsequent page-loads should be speedier.

## External Web Service Calls

It is not uncommon for API or web-service integration modules to make `curl()` or `drupal_http_request()` calls, which will halt the execution of your application until a response is received. Obviously, a slow response from the external service could lead to a timeout on Pantheon.

Even the most reliable web services will occasionally experience slowness, and it is also inevitable that there are network disruptions which could slow down external calls. That's why modules and custom code should set a relatively low timeout threshold for the external call itself. If the external web service doesn't respond in a few seconds, it should fail gracefully and move on.

If you are seeing frequent problems with external web services, it's a good idea to evaluate the code making the call, if not the service provider themselves.

## Overloaded Workers

If your PHP workers are overloaded, it's possible that pages will timeout before they are ever even picked up by the back-end. This can happen if you are suddenly hit with a flood of un-cachable/authenticated traffic.

<script src="https://gist.github.com/timani/412437aa8c1e5e6b5abe.js"></script>
##### 1. Fix errors

If your site is throwing a lot of warning and notices, there is a performance penalty are resources are used to log errors to disk and slow down your site by performing additional database write operations. In this case the solution is not to disable watchdog bug fix the errors.

Even with the watchdog off these errors will still be written to the PHP error logs so they should be addressed as soon as possible.

##### 2. Optimize the site

Long running processes like batch jobs, background tasks and heavy operations cron jobs can also lead to backend resources being maxed out on your site. [Use New Relic](/articles/howto/new-relic-performance-analysis-on-pantheon/-new-relic-performance-analysis-on-pantheon) to identify performance bottlenecks, fix errors and make changes to enhance performance.

##### 3. Upgrade your plan

If the all the errors have been resolved and the views, batches and tasks have been optimized, the next step is to look into [upgrading your plan](https://www.getpantheon.com/pricing) for more resources. The recommendation here is to select the most appropriate plan for the resource usage of your site.

### Unexpected Timeouts

There's no accounting for buggy code. We've seen bugs ranging from Drupal running cron on every page-load, to the advanced\_help spidering the entire code tree looking for help files cause sufficiently slow page load times to trigger timeouts.

If you are seeing timeouts in unexpected places, debugging with New Relic or looking at your php slow logs can be informative.

## Admin Work-Arounds

In the best of all possible worlds, there are no slow queries, all external calls are fast, and the application is a finely-tuned highly-optimized cheetah of the web. In reality, sometimes we just need to get around a pesky timeout in order to get the job done.

Drush is a great workaround for most administrative bottlenecks. Because it runs via the PHP command-line, there are no time limits. Enabling/disabling modules, running update.php, clearing caches; these are all supported by Drush.

## Handle More Traffic

See our article on [debugging performance bottlenecks](/articles/advanced-topics/debugging-slow-performance/) for details on how to streamline your site to handle additional traffic.