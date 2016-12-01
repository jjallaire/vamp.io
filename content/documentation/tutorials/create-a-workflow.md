---
date: 2016-09-13T09:00:00+00:00
title: Create a workflow to automate events
menu:
  main:
    parent: "Tutorials"
    name: "Create a workflow to automate events"
    weight: 80
draft: true
---


This tutorial will explain how to work with the Vamp events stream. We will create and retrieve some (custom) events using the Vamp REST API, then create a simple workflow to automate this interaction. Let's get started!

### Requirements

* A running version of Vamp 0.9.x (this tutorial has been tested on the [Vamp hello world set up](documentation/installation/hello-world) using Vamp 0.9.1)



## Vamp events
Vamp is a distributed system tied together by a central events stream. Every action in Vamp creates events, which in turn can be measured and used as triggers for new actions. For example, gateway updates are triggered by deployments (synchronisation events), while canary releases and autoscaling are based on health and metrics events. Each event created by Vamp has a type and one or more tags. Event tags are a unique combination of key and one (optional) value, for example `['event_tag1', 'event_tag2:value', 'event_tag3:value'] event_type`. Vamp works with a number of default event types, custom event types can also be created using the `events` API endpoint (more on that later).

Default event types |  Description
----------|--------
Archive     |    Store/update of static Vamp artifacts (breeds, blueprints etc.)
Synchronization | Successful matching of a desired state with an observed state (for example, a successful update from 3 to 5 running instances)
Event | Hooks (for example, succesful deploy or undeploy of a blueprint)
Health | Generated by the health workflow and used by the Vamp UI
Metrics | Generated by the metrics workflow and used by the Vamp UI
Workflow | Generated by workflows used in Vamp Runner


### Storing events

Vamp events are strored by default in Elasticsearch. Elasticsearch indexing is based on the event type and is updated for each event received by the API. Custom event types will be indexed individually.  
[Read more about events](documentation/using-vamp/events/)

  
## Use the REST API to create events

The `events` endpoint of the Vamp API can be used to create and retrieve events stored in Elasticsearch. We are going to use this endpoint to create an event with the custom type `hello_api`.

Use postman or curl to `POST` the below JSON to the `/api/v1/events` endpoint.

```
{
  "tags": [
    "test_key",
    "test_key:test_value"
  ],
  "type": "hello_api"
}
```  
If all runs to plan, you will receive a response from the API with the accepted JSON (including an added timestamp) in the body. The created event will also show up in the EVENTS stream at the bottom of the Vamp UI:

![](images/screens/v091/events_vampui_hello_api.png)

Best practice when creating events is to include both the tag with no value (in this example ` "test_key" `) and the tag with its specific value (in this example ` "test_key:test_value" `). This allows the published event to be picked up by everything listening on a tag key, not just those listening on a specific tag value. For example ` "deployment" ` not just ` "deployment:xyz" `.

## Create a simple workflow
Now we know how to generate events with the Vamp API, we can take this a step further and create a simple workflow to automate the process. You can automate Vamp with any application (for example something developed in python), as long as it can count or trigger an action against the Vamp API. For this tutorial, we will create a Vamp workflow. 

Vamp workflows are a convenient way of creating Node JS based scripts that run as containers and access the Vamp API. 


Explain what a workflow is, why we use them and how they are created...

*  
* 
* Workflows can be scheduled to run as a deamon, be triggered by events or run at specified timestamp.
* The workflow driver is used for time based and event based workflows
* Create a workflow, referencing the breed - this is best practice.

### Create a breed 
Here we're going to add our JavaScript. There are few items that we need to include so we can interact with Vamp. First, we require the `vamp-node-client` library to be able to interact with the Vamp API. We also need to create a a new Vamp API object and set the run interval.


  1. Go the BREEDS tab in the Vamp UI
  * Click ADD
  * Paste in the below breed YAML and click SAVE

```
name: hello_workflow
kind: breed
deployable:
  type: application/javascript
  definition: |
    'use strict';

    var vamp = require('vamp-node-client'); // required to interact with the Vamp API

    var api = new vamp.Api(); // a new Vamp API object

    var period = 5;  // milliseconds

    var run = function () {   // create a Vamp event - tag_key:tag_value, event_type
         api.event(['test_key', 'test_key:test_value'], 'hello_workflow');
     };

    run();

    setInterval(run, period * 1000); // run this workflow every x seconds
ports: {}
environment_variables: {}
constants: {}
arguments: []
dependencies: {}
```
![](images/screens/v091/events_vampui_breeds.png)

### Deploy the hello_workflow breed as a workflow
We can now create a workflow that references the `hello_workflow` breed. Once the workflow is created, the breed will be deployed in a Vamp workflow agent container. We will schedule our workflow to run as a daemon, so it will immediately start running at the set interval (5 seconds).

1. Go to the WORKFLOWS tab in the Vamp UI 
* Click ADD (top right)
* Paste in the below blueprint YAML and click SAVE

```
name: hello_workflow
kind: workflow
breed: hello_workflow
schedule: daemon
```
The workflow will be deployed and you will see the created events appearing in the Vamp UI EVENTS stream. 

![](images/screens/v091/events_vampui_hello_workflow_5.png)

### Update the running workflow
In Vamp 0.9.1, updates made to a breed will not be carried over to an associated workflow. Vamp breeds are static artifacts, so updating the breed alone won't have any effect on the running workflow . To update a running workflow you need to re-deploy the workflow (a restart workflow feature is coming very soon, which makes this much simpler). Let's update our running workflow so it runs every second.
 
1. Go the BREEDS tab in the Vamp UI
* Open the `hello_workflow` breed and update the script to set `var period = 1` (this will cause the events to be generated every second)
* Click SAVE
* Check the events stream in the Vamp UI 
  * `hello_workflow` events will continue to appear every 5 seconds
* Go to the WORKFLOWS tab
* Open the `hello_workflow` workflow and update ???
* Click SAVE - the workflow will be redployed, including the changes made to the `hello_workflow` breed
* Check the events stream again 
  * `hello_workflow` should now be generating events every second - that's a lot of events! Feel free to delete the workflow and stop it running.
 
![](images/screens/v091/events_vampui_hello_workflow_10.png)

## Retrieve and filter events using the REST API
Now we have a lot of events stored we can use the same REST API events endpoint to retrieve them. We can search for specifc events and filter for specific types and tags. You can try this out in postman or curl by sending a GET request to the `/api/v1/events` endpoint or you could do this in brower. you can use `?type=` or `?tag=` to search through events.






## Summing up
We created our first workflow and showed how the Vamp events stream is central to Vamp. 

## Looking for more of a challenge?
Just for fun, you could try these:

* 
* 

{{< note title="What next?" >}}
* What would you like to see for our next tutorial? [let us know](mailto:info@magnetic.io)
* Find our more about [using Vamp](documentation/using-vamp/artifacts)
* Read more about the [Vamp API](documentation/api/api-reference)
{{< /note >}}
