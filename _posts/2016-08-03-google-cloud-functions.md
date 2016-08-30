---
layout: post
title: Google Cloud Functions
---

### Google Cloud Functions

[Google Cloud Functions](https://cloud.google.com/functions) is a compute solution offering from Google that is described as lightweight and event-based. It allows you to create small, single-purpose functions that respond to cloud events without the need to manage a runtime environment or a server. Cloud Functions however is an alpha release that might be changed in backward-incompatible way and it requires the corresponding Google Cloud Platform account to be whitelisted.

Cloud Functions code are written in JavaScript and executed on Google Cloud Platform in a managed Node.js environment. The code can be triggered asynchronously by events from Google Cloud Storage and Google Cloud Pub/Sub. The code can also be triggered synchronously by HTTP invocation.

#### Getting Started

Install the Google Cloud SDK: `brew cask install google-cloud-sdk`. This includes the `gcloud` [tool](https://cloud.google.com/sdk/gcloud/) that provides the primary command-line interface to the Google Cloud Platform. Authenticate the `gcloud` tool with your Google Cloud Platform account: `gcloud init`.  Lastly, install the alpha commands component: `gcloud components install alpha`.

Using the web console, create a new project, e.g. `DevWorld`. And set it as the default project:

{% highlight shell %}
$ gcloud projects list
PROJECT_ID              NAME      PROJECT_NUMBER
devworld-1470605832118  DevWorld  386816325421
$ gcloud config set project devworld-1470605832118
{% endhighlight %}

Lastly, go to the Cloud Functions web console to enable the API.

#### Git Repository

When we create a new project, we get a Git repository that we can use to store our files; so that we don’t need to use a Cloud Storage bucket. Note that if you install the SDK with Homebrew Cask, you need to rename the `git-credential-gcloud` file or create a soft link to `git-credential-gcloud.sh`, e.g. `cd ~/.homebrew/bin/ && ln -s git-credential-gcloud git-credential-gcloud.sh`

{% highlight shell %}
$ mkdir cloudfunctions && cd cloudfunctions
$ git init
$ touch README.md
$ git add .
$ git commit -m 'Initial commit'
$ git config credential.helper gcloud.sh
$ git remote add origin https://source.developers.google.com/p/devworld-1470605832118/r/default
$ git push --all origin
{% endhighlight %}

On another computer, we could just clone the repository: `gcloud source repos clone default cloudfunctions --project=devworld-1470605832118`

We will use this repository to store our Cloud Functions files.

#### Example

As an example, let’s create a simple function that can be called by using `curl`. Create the following `index.js` file in a new `helloMessage` folder:

{% highlight javascript %}
exports.helloMessage = function helloMessage(req, res) {
    var name;
    switch (req.get('content-type')) {
        case 'application/json': // '{"name":"John"}'
            name = req.body.name;
            break;
        case 'text/plain': // 'John'
            name = req.body;
            break;
        case 'application/x-www-form-urlencoded': // 'name=John'
            name = req.body.name;
            break;
    }

    res.status(200).send('Hello ' + (name || 'World') + '!');
};
{% endhighlight %}

Commit the code, push it to the remote repository, and deploy the code by executing: `gcloud alpha functions deploy helloMessageFunction --source-url https://source.developers.google.com/p/devworld-1470605832118/r/default --source /helloMessage --trigger-http --source-branch master --entry-point helloMessage`. This creates a new function with the name `helloMessageFunction` that uses the code in `helloMessage` module in the `index.js` file in the `/helloMessage` folder. The function can be invoked synchronously via HTTP methods.

We can test the function by directly triggering the function with the `gcloud` tool: `gcloud alpha functions call helloMessageFunction --data '{"name":"Bill"}'`. Or, by using `curl`: `curl -X POST https://us-central1-devworld-1470605832118.cloudfunctions.net/helloMessageFunction -H "Content-Type:application/json" --data '{"name":"Steve"}'`.

#### Cloud Functions Types

There are two distinct Cloud Functions types: HTTP (foreground) functions and background functions.

##### HTTP Functions

HTTP functions are used when we want to invoke them via standard HTTP requests with various methods like: `GET`, `PUT`, `POST`, `DELETE`, and `OPTIONS`. The function signature takes HTTP-specific arguments: request and response. The arguments have properties of [ExpressJS](https://expressjs.com) [Request](http://expressjs.com/en/4x/api.html#req) and [Response](http://expressjs.com/en/4x/api.html#res) objects that are used to extract and return data. Note that when the function has completed, we need to call a termination method such as [send()](http://expressjs.com/en/api.html#res.send), [json()](http://expressjs.com/en/api.html#res.json), or [end()](http://expressjs.com/en/api.html#res.end). Otherwise, the function may continue to run and be terminated by the system platform.

##### Background Functions

Background functions are used when we want the Cloud Functions code to be invoked because of a change in a Cloud Storage bucket or a message on a Pub/Sub topic.The function signature takes two arguments: context and data. As in the case with HTTP function, when the background function has completed, we need to call a termination method such as `success()`, `failure()`, or `done()`. Otherwise, the function may continue to run and be terminated by the system platform. The context parameter holds the information about the execution environment and it also includes callback functions to signal completion of the background function. And the data parameter contains the data associated with the event that triggered the function execution.
