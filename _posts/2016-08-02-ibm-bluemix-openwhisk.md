---
layout: post
title: IBM Bluemix OpenWhisk
---

[IBM Bluemix](https://www.ibm.com/cloud-computing/bluemix/) is a cloud platform as a service developed by IBM. Think of it as an alternative to Amazon Web Services or Google Cloud Platform. Bluemix is developed using [Cloud Foundry](https://en.wikipedia.org/wiki/Cloud_Foundry) which is basically a software stack to build your own PaaS (another alternative to Cloud Foundry is [OpenStack](https://en.wikipedia.org/wiki/OpenStack)).

[OpenWhisk](https://github.com/openwhisk/openwhisk) is a distributed event-driven programming service. It provides programming model to upload event handlers to a cloud service, and register the handlers to respond to various events. The events can come from various Bluemix services like Cloudant, external sources, or direct invocations from web or mobile apps over HTTP.

With OpenWhisk, developers only pay for what they use and they don’t have to manage a server. However, OpenWhisk is still an experimental service from IBM that might change in ways that are incompatible with earlier versions.

#### Getting Started

Install the `wsk` command line tool: `pip install --upgrade https://new-console.ng.bluemix.net/openwhisk/cli/download`. Set the OpenWhisk namespace and authorisation key: `wsk property set --apihost openwhisk.ng.bluemix.net --auth XXXX:XXXX --namespace "namespace_dev"` where the required information can be found on the OpenWhisk web console. Verify the setup: `wsk action invoke /whisk.system/samples/echo -p message hello --blocking --result` which performs a synchronous blocking invocation of `echo` with `hello` as an argument.

#### Key Concepts

There are several key concepts and terminologies in OpenWhisk:

- *Trigger*: a class of events emitted by event sources.
- *Action*: the actual code to be executed whenever a trigger fires. It can be a snippet of JavaScript or Swift code, or a custom binary in a Docker container.
- *Rule*: an association between a trigger and an action.
- *Feed*: a code that configures an external event source to fire trigger events.
- *Package*: a bundle of feeds and actions that describe a service in a uniform manner.

#### OpenWhisk Actions

OpenWhisk supports action code written in Swift that will be executed in a Linux environment. However, be aware that the version of Swift on Linux that is used with OpenWhisk might be different with versions of Swift from stable releases of Xcode. Create the following `greetSomeone.swift`:

{% highlight shell %}
	func main(args: [String:Any]) -> [String:Any] {
	  if let name = args["name"] as? String {
	    return [ "greeting" : "Hello \(name)!" ]
	  } else {
	    return [ "greeting" : "Howdy stranger!" ]
	  }
	}
{% endhighlight %}

Note that Swift actions always consume a dictionary and produce a dictionary. You can create the OpenWhisk action by executing: `wsk action create greetSomeone greetSomeone.swift` and invoking it by executing: `wsk action invoke --blocking greetSomeone --param name Tim`

{% highlight json %}
 {
	  "result": {
	      "greeting": "Hello Tim!"
	  },
	  "status": "success",
	  "success": true
	}
{% endhighlight %}

The `--result` flag makes the invocation shows only the result (it requires `--blocking` flag). If you don’t need the result right away, you can make a non-blocking invocation:

{% highlight shell %}
	$ wsk action invoke greetSomeone --param name Tim
  ok: invoked greetSomeone with id e106be8bf30d4eb598c8bb5141a1b00c
	$ wsk activation result e106be8bf30d4eb598c8bb5141a1b00c
	{
	    "greeting": "Hello Tim!"
	}
{% endhighlight %}

##### Default Parameters

OpenWhisk allows us to set default parameters for action invocations. As an example, we can set default parameters for the previous example: `wsk action update greetSomeone --param name Steve` and invoking it:

{% highlight shell %}
$ wsk action invoke --blocking --result greetSomeone
{
	"greeting": "Hello Steve!"
}
{% endhighlight %}

Note that it achieves the same thing as the Swift default parameters in functions although it is at a different abstraction level.

##### Action Sequences

OpenWhisk allows us to create an action from a sequence of actions. Think of it like a pipeline of OpenWhisk actions. This allows us to create abstractions of common functions and compose them to build the desired actions. For example, write the following code as `sort.swift` and create a new OpenWhisk action: `wsk action create sort sort.swift`:

{% highlight swift %}
	func main(args: [String:Any]) -> [String:Any] {
	  if let payload = args["payload"] as? [Any] {
	    let strings = payload.map { String($0) }
	    return ["payload": strings.sort { $0 < $1 }]
	  } else if let payload = args["payload"] as? String {
	    return ["payload": payload]
	  } else {
	    return ["payload": [String]()]
	  }
	}
{% endhighlight %}

Write the following code as `uppercase.swift` and create a new OpenWhisk action: `wsk action create uppercase uppercase.swift`:

{% highlight swift %}
	func main(args: [String:Any]) -> [String:Any] {
	  if let payload = args["payload"] as? [Any] {
	    let strings = payload.map { String($0).uppercaseString }
	    return ["payload": strings]
	  } else if let payload = args["payload"] as? String {
	    return ["payload": payload.uppercaseString]
	  } else {
	    return ["payload": [String]()]
	  }
	}
{% endhighlight %}

We could test the above two actions:

{% highlight shell %}
	$ wsk action invoke --blocking --result sort --param payload '["a", "c", "b"]'
	{
	    "payload": [ "a", "b", "c" ]
	}
	$ wsk action invoke --blocking --result uppercase --param payload '["a", "c", "b"]'
	{
	     "payload": [ "A", "C", "B" ]
	}
{% endhighlight %}

And we could create a new action by composing the previous two actions:

{% highlight shell %}
	$ wsk action create uppercaseSort --sequence uppercase,sort
	$ wsk action invoke --blocking --result uppercaseSort --param payload '["a", "c", "b"]'
	{
	     "payload": [ "A", "B", "C" ]
	}
{% endhighlight %}

Note that the parameter name `payload` is the same for input as well as for output. This simplifies action composition.

##### Miscellaneous

We could delete unused actions by executing `wsk action delete` with the action name. For example:

{% highlight shell %}
	$ wsk action list
	actions
	/worqbench_dev/uppercaseSort                                      private
	/worqbench_dev/greetSomeone                                      private
	/worqbench_dev/uppercase                                             private
	/worqbench_dev/sort                                                        private
	$ wsk action delete greetSomeone
{% endhighlight %}

#### Triggers and Rules

A trigger can be thought of as a named channel for a class of events. A trigger can be activated or fired by an event which is a dictionary of key-value pairs. Each firing of a trigger results in an activation ID. A trigger can be fired by a user or by an external event source. A feed is a way to configure an external event source to fire a trigger.

A rule associates one trigger with one action. Every firing of the trigger causes the action to be executed with the event as the input. If a trigger is fired without an accompanying rule to match against then it has no visible effect.

As an example, create the following action `helloAction.swift`:

{% highlight swift %}
	func main(args: [String:Any]) -> [String:Any] {
	      if let name = args["name"] as? String {
	          return [ "payload" : "\(name)" ]
	      } else {
	          return [ "payload" : "anonymous" ]
	      }
	}
{% endhighlight %}

{% highlight shell %}
	$ wsk action create helloAction helloAction.swift
{% endhighlight %}

Create a trigger and confirm that it has been created:

{% highlight shell %}
	$ wsk trigger create eventUpdate
	$ wsk trigger list
	triggers
	/worqbench_dev/eventUpdate                                      private
{% endhighlight %}

The above command creates a named channel to which events can be fired. We need to create a rule to observe the effect: `wsk rule create --enable myRule eventUpdate helloAction`

In a new terminal window, we could poll for the rule invocation: `wsk activation poll helloAction`.  When we fire a trigger event, e.g. `wsk trigger fire eventUpdate --param name "Rob"`. We could see the activation ID and payload as follows:

{% highlight shell %}
	$ wsk activation poll helloAction
	Hit Ctrl-C to exit.
	Polling for logs
	Activation: helloAction (4542fc011e214fe19ad94d2de583b5c4)
	$ wsk activation result 4542fc011e214fe19ad94d2de583b5c4
	{
	    "payload": "Rob"
	}
{% endhighlight %}

We can create multiple rules that associate one trigger with different actions. The trigger and action that define a rule must be in the same namespace and cannot belong to a package. If an action belongs to a package, we need to copy it into our namespace first, e.g. `wsk action create echo --copy /whisk.system/samples/echo`.

#### API Calls

OpenWhisk actions that we have created can be called directly without using the OpenWhisk CLI. It requires the auth key and token that can be obtained by executing: `wsk property get --auth`. The strings before and after the colon are the key and token, respectively. We could take a look at the HTTP request and response by using the `-v` flag. For example:

{% highlight shell %}
	$ wsk -v action invoke --blocking --result helloAction --param name 'Donny'
	{'apihost': 'openwhisk.ng.bluemix.net', 'namespace': 'worqbench_dev', 'clibuild': '2016-03-14T11:48:47-05:00', 'apiversion': 'v1'}
	========
	REQUEST:
	POST https://openwhisk.ng.bluemix.net/api/v1/namespaces/worqbench_dev/actions/helloAction?blocking=true
	Headers sent:
	{
	    "Authorization": "Basic OxLc4NzA3YjYtNTUyMC00YWY3LTgyMzctMDM5YmIxN2QwNzY3OnQ2RjVmdXBiZ1dLVE9kRWxhTWZoZTdxeXVJR3FTTEpQcUs0WUQ0bEF0MDg0MUtibTBHeGk3NFFUSkhSVVhpN1k=",
	    "Content-Type": "application/json"
	}
	Body sent:
	{"name": "Donny"}
	--------
	RESPONSE:
	Got response with code 200
	Body received:
	{
	  "name": "helloAction",
	  "subject": "donny.kurniawan@gmail.com",
	  "activationId": "6874839ad6734954a1f00e8d06df3764",
	  "publish": false,
	  "annotations": [],
	  "version": "0.0.1",
	  "response": {
	    "result": {
	      "payload": "Donny"
	    },
	    "success": true,
	    "status": "success"
	  },
	  "end": 1470054288984,
	  "logs": [],
	  "start": 1470054288959,
	  "namespace": "donny.kurniawan@gmail.com"
	}
	========
	{
	    "payload": "Donny"
	}
{% endhighlight %}

The above execution is equivalent to the following `curl` invocation:

{% highlight shell %}
	$ curl -X POST -H "Content-Type: application/json" --data '{"name": "Donny"}' -u 0x78707b6-5520-4af7-8237-039bb17d0767:t6F5fupbgWKTOdElaMfhe7qyuIGqSLJPqK4YD4lAt0841Kbm0Gxi74QTJHRUXi7Y https://openwhisk.ng.bluemix.net/api/v1/namespaces/worqbench_dev/actions/helloAction?blocking=true
	{
	  "name": "helloAction",
	  "subject": "donny.kurniawan@gmail.com",
	  "activationId": "3d450ffb41cf407ca0f3109eeece4f6e",
	  "publish": false,
	  "annotations": [],
	  "version": "0.0.1",
	  "response": {
	    "result": {
	      "payload": "Donny"
	    },
	    "success": true,
	    "status": "success"
	  },
	  "end": 1470058872478,
	  "logs": [],
	  "start": 1470058868947,
	  "namespace": "donny.kurniawan@gmail.com"
{% endhighlight %}

All OpenWhisk APIs are protected with HTTP Basic authentication. At the moment, OpenWhisk supports only one key per account. Be aware that your key would need to be embedded in client-side code making it visible to the public.
