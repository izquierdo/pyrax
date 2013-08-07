# Cloud Monitoring

## Basic Concepts
Rackspace Cloud Monitoring provides timely and accurate information about how your resources are performing. It supplies you with key information that can help you manage your business by enabling you to keep track of your cloud resources and receive instant notification when a resource needs your attention. You can quickly create multiple monitors with predefined checks, such as PING, HTTPS, SMTP, and many others.

## Monitoring in pyrax
Once you have authenticated, you can reference the monitoring service via `pyrax.cloud_monitoring`. This object is the client through which you interact with Cloud Monitoring.

For the sake of brevity and convenience, it is common to define abbreviated aliases for the modules. All the code in the document assumes that at the top of your script, you have added the following line:

    cm = pyrax.cloud_monitoring

Note that as of this writing, pyrax only supports **remote monitoring**. There is a second type of monitoring that is currently in Preview mode that uses a *Monitoring Agent* installed on your device.


## Key Terminology
### Entity
In Rackspace Cloud Monitoring, an entity is the object or resource that you want to monitor. It can be any object or device that you want to monitor. It's commonly a web server, but it might also be a website, a web page or a web service.

When you create an entity, you'll specify characteristics that describe what you are monitoring. At a minimum you must specify a name for the entity. The name is a user-friendly label or description that helps you identify the resource. You can also specify other attributes of the entity, such the entity's IP address, and any meta data that you'd like to associate with the entity.

### Check
Once you've created an entity, you can configure one or more checks for it. A check is the foundational building block of the monitoring system, and is always associated with an entity. The check specifies the parts or pieces of the entity that you want to monitor, the monitoring frequency, how many monitoring zones are launching the check, and so on. Basically it contains the specific details of how you are monitoring the entity.

You can associate one or more checks with an entity. An entity must have at least one check, but by creating multiple checks for an entity, you can monitor several different aspects of a single resource.

For each check you create within the monitoring system, you'll designate a check type. The check type tells the monitoring system which method to use, PING, HTTP, SMTP, and so on, when investigating the monitored resource. Rackspace Cloud Monitoring check types are fully described here.

Note that if something happens to your resource, the check does not trigger a notification action. Instead, notifications are triggered by alarms that you create separately and associate with the check.

### Monitoring Zones
When you create a check, you specify which monitoring zone(s) you want to launch the check from. A monitoring zone is the point of origin or "launch point" of the check. This concept of a monitoring zone is similar to that of a datacenter, however in the monitoring system, you can think of it more as a geographical region.

You can launch checks for a particular entity from multiple monitoring zones. This allows you to observe the performance of an entity from different regions of the world. It is also a way to prevent false alarms. For example, if the check from one monitoring zone reports that an entity is down, a second or third monitoring zone might report that the entity is up and running. This gives you a better picture of an entity's overall health.

### Collectors
A collector collects data from the monitoring zone and is mapped directly to an individual machine or a virtual machine. Monitoring zones contain many collectors, all of which will be within the IP address range listed in the response. Note that there may also be unallocated IP addresses or unrelated machines within that IP address range.

### Monitoring Agent
Note: The Monitoring Agent is a Preview feature.

The agent provides insight into the internals of your servers with checks for information such as load average and network usage. The agent runs as a single small service that runs scheduled checks and pushes metrics to the rest of Cloud Monitoring so the metrics can be analyzed, alerted on, and archived. These metrics are gathered via checks using agent check types, and can be used with the other Cloud Monitoring primatives such as alarms. See Section B.2, “Agent Check Types” for a list of agent check types.

### Alarms
An alarm contains a set of rules that determine when the monitoring system sends a notification. You can create multiple alarms for the different checks types associated with an entity. For example, if your entity is a web server that hosts your company's website, you can create one alarm to monitor the server itself, and another alarm to monitor the website.

The alarms language provides you with scoping parameters that let you pinpoint the value that will trigger the alarm. The scoping parameters are inherently flexible, so that you can set up multiple checks to trigger a single alarm. The alarm language supplies an adaptable triggering system that makes it easy for you to define different formulas for each alarm that monitors an entity's uptime. To learn how to use the alarm language to create robust monitors, see Alert Triggering and Alarms.

### Notifications
A notification is an informational message that you receive from the monitoring system when an alarm is triggered. You can set up notifications to alert a single individual or an entire team. Rackspace Cloud Monitoring currently supports webhooks and email for sending notifications.

### Notification Plans
A notification plan contains a set of notification rules to execute when an alarm is triggered. A notification plan can contain multiple notifications for each of the following states:

* Critical
* Warning
* Ok


## How Cloud Monitoring Works
Cloud Monitoring helps you keep a keen eye on all of your resources, from web sites to web servers, routers, load balancers, and more. Here is an overview of the Monitoring workflow:

* You create an entity to represent the item you want to monitor. For example, the entity might represent a web site.
* You attach a predefined check to the entity. For example, you could use the PING check to monitor your web site's public IP address.
* You can run your checks from multiple monitoring zones to provide redundant monitoring as well as voting logic to avoid false alarms.
* You create a notification which lets you define an action which Cloud Monitoring uses to communicate with you when a problem occurs. For example, you might define a notification that specifies an email that Cloud Monitoring will send when a condition is met.
* You create notification plans allow you to organize a set of several notifications, or actions, that are taken for different severities.
* You define one or more alarms for each check. An alarm lets you specify trigger conditions for the various metrics returned by the check. When a specific condition is met, the alarm is triggered and your notification plan is put into action. For example, your alarm may indicate a PING response time. If this time elapses, the alarm could send you an email or a webhook to a URL.

## Create an Entity
The first step in working with Cloud Monitoring is to create an `entity`, which represents the device to be monitored. To do so, you specify the characteristics of the device, which include one or more IP addresses. The parameter `ip_addresses` is a dictionary, with the keys being a string that can be used to identify the address, and the value the IPv4 or IPv6 address for the entity. You can include as many addresses as you need. You can also include optional metadata to help you identify what the entity represents in your system.

    ent = cm.create_entity(name="sample_entity", ip_addresses={"example": "1.2.34"},
            metadata={"description": "Just a test entity"})

## Create a Check
There are numerous types of checks, and each requires its own parameters, and offers its own combination of metrics. The list of all [available check types](http://docs.rackspace.com/cm/api/v1.0/cm-devguide/content/appendix-check-types.html) shows how extensive your monitoring options are.

### Check Types
As an example, create a check on the HTTP server for the entity. This check will try to connect to the server and retrieve the specified URL using the specified method, optionally with the password and user for authentication, using SSL, and checking the body with a regex. This can be used to test that a web application running on a server is responding without generating error messages. It can also test if the SSL certificate is valid.

From the table in the Available Check Types link above, you can find that the ID of the desired check type is `remote.http`. You can also get a list of all check types through pyrax by calling:

    chk_types = cm.list_check_types()

This returns a list of `CloudMonitorCheckType` objects. Each has object has an attribute named `fields` that lists the parameters for that type, with each indicating whether the field is required or optional, its name and a brief description. As an example, here is the `fields` attribute for the `remote.http` check type:

    [{u'description': u'Target URL',
          u'name': u'url',
          u'optional': False},
     {u'description': u'Body match regular expression (body is limited to 100k)',
          u'name': u'body',
          u'optional': True},
     {u'description': u'Arbitrary headers which are sent with the request.',
          u'name': u'headers',
          u'optional': True},
     {u'description': u'Body match regular expressions (body is limited to 100k, matches are truncated to 80 characters)',
          u'name': u'body_matches',
          u'optional': True},
     {u'description': u'HTTP method (default: GET)',
          u'name': u'method',
          u'optional': True},
     {u'description': u'Optional auth user',
          u'name': u'auth_user',
          u'optional': True},
     {u'description': u'Optional auth password',
          u'name': u'auth_password',
          u'optional': True},
     {u'description': u'Follow redirects (default: true)',
          u'name': u'follow_redirects',
          u'optional': True},
     {u'description': u'Specify a request body (limited to 1024 characters). If following a redirect, payload will only be sent to first location',
          u'name': u'payload',
          u'optional': True}]

Note that most of the parameters are optional; the only required parameter is **url**. If you only include that, the monitor will simply check that a plain GET on that URL gets some sort of response. By adding additional parameters to the check, you can make the tests that the check carries out much more specific.


### Monitoring Zones
To list the available Monitoring Zones, call:

    cm.list_monitoring_zones()

This returns a list of `CloudMonitorZone` objects:

    [<CloudMonitorZone country_code=US, id=mzdfw, label=Dallas Fort Worth (DFW), source_ips=[u'2001:4800:7902:0001::/64', u'50.56.142.128/26']>,
     <CloudMonitorZone country_code=HK, id=mzhkg, label=Hong Kong (HKG), source_ips=[u'180.150.149.64/26', u'2401:1800:7902:1:0:0:0:0/64']>,
     <CloudMonitorZone country_code=US, id=mziad, label=Washington Dulles (IAD), source_ips=[u'2001:4802:7902:0001::/64', u'69.20.52.192/26']>,
     <CloudMonitorZone country_code=GB, id=mzlon, label=London (LON), source_ips=[u'2a00:1a48:7902:0001::/64', u'78.136.44.0/26']>,
     <CloudMonitorZone country_code=US, id=mzord, label=Chicago (ORD), source_ips=[u'2001:4801:7902:0001::/64', u'50.57.61.0/26']>,
     <CloudMonitorZone country_code=AU, id=mzsyd, label=Sydney (SYD), source_ips=[u'119.9.5.0/26', u'2401:1801:7902:1::/64']>]


## Create the Check
To create the check, run the following:

    chk = cm.create_check(ent, label="sample_check", type="remote.http",
            details={"url": "http://example.com/some_page"}, period=900,
            timeout=20, monitoring_zones_poll=["mzdfw", "mzlon", "mzsyd"],
            target_hostname="http://example.com")

This will create an HTTP check on the entity `ent` for the page `http://example.com/some_page` that will run every 15 minutes from the Dallas, London, and Sydney monitoring zones.

There are several parameters for `create_check()`:

Parameter | Required? | Default | Description
------ | ------ | ------ | ------ 
**label** | no | -blank- | An optional label for this check
**name** | no | -blank- | Synonym for 'label'
**check_type** | yes | | The type of check to create. Can be either a `CloudMonitorCheckType` instance, or its ID.
**details** | no | None | A dictionary for the parameters needed for this type of check.
**disabled** | no | False | Passing `disabled=True` creates the check, but it will not be run until the check is enabled.
**metadata** | no | None | Arbitrary key/value pairs you can associate with this check
**monitoring_zones_poll** | yes | | Either a list or a single monitoring zone. Can be either `CloudMonitoringZone` instances, or their IDs.
**period** | no | (account setting) | How often to run the check, in seconds. Can range between 30 and 1800.
**timeout** | no | None | How long to wait before failing the check. Must be less than the period.
**target_hostname** | Mutually exclusive with `target_alias` | None | Either the IP address or the fully qualified domain name of the target of the check. 
**target_alias** | Mutually exclusive with `target_hostname` | None | A key in the 'ip_addresses' dictionary of the entity for this check.

Note that you must supply either a `target_hostname` or a `target_alias`, but not both.

## Create a Notification

There are two supported notification types, `email` and `webhook`, by which you can be notified of alarms. The `create_notification()` method contains several parameters:

Parameter | Required? | Default | Description
------ | ------ | ------ | ------ 
**notification_type** | yes | | Either "email" or "webhook"
**label** | no | None | Friendly name for the notification
**details** | no | None | A dictionary of details for your `notification_type`

### Notification Details

When the `notification_type` is "email", the `details` parameter should be a dictionary with a key "address", and a value specifying an email address.

When the `notification_type` is "webhook", the `details` parameter should be a dictionary with a key "url", and a value specifying a url to POST.

## Create the Notification

To create the notification, run the following:

    not = cm.create_notification("email", label="my_email_notification",
            details={"address": "me@example.com")

This will create an email notification, which can then be added to a Notification Plan.

## Create a Notification Plan

Notification Plans outline the notifications to contact under three states: ok, warning, and critical.



## Create an Alarm

To create the alarm, run the following: