Clone source code in Cloud Shell
To clone the repository for the class, execute the following command:

`git clone https://github.com/GoogleCloudPlatform/training-data-analyst`

Create a soft link as a shortcut to the working directory:

`ln -s ~/training-data-analyst/courses/developingapps/v1.2/nodejs/stackdriver-trace-monitoring ~/stackdriver-trace-monitoring`

Configure and run the case study application
Navigate to the directory that contains the sample files for this lab:

`cd ~/stackdriver-trace-monitoring/start`

To configure the Quiz application, execute the following command:

`. prepare_environment.sh`

If prompted, enter Y to all unauthenticated invocations.

This script file

* Creates an App Engine application.
* Exports environment variables GCLOUD_PROJECT and GCLOUD_BUCKET.
* Runs npm install.
* Creates entities in Cloud Datastore.
* Creates a Cloud Spanner instance, database, and tables.
* Creates two Google Cloud Pub/Sub topics.
* Creates two Cloud Functions.
* Prints out the Google Cloud Platform Project ID.
To complete the preparation of the Cloud Shell environment to run the application, execute the following commands:

`gcloud iam service-accounts create quiz-account --display-name "Quiz Account"`

`gcloud iam service-accounts keys create key.json --iam-account=quiz-account@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com`

`export GOOGLE_APPLICATION_CREDENTIALS=key.json`

`gcloud projects get-iam-policy $DEVSHELL_PROJECT_ID --format json > iam.json`

`node setup/add_iam_policy_to_service_account.js`

`gcloud projects set-iam-policy $DEVSHELL_PROJECT_ID iam_modified.json`

These commands:

* Create a service account.
* Create and download the key file for the service account.
* Export an environment variable GOOGLE_APPLICATION_CREDENTIALS referencing the key file.
* Create the necessary IAM permissions to enable the service account to access all required APIs.

In production on Compute Engine or Container Engine, you would grant access to APIs either using a custom service account or via scopes. Typically, you would allow GCP to manage key rotation.

Enable the Cloud Trace API
In the Cloud Platform Console, on the Navigation menu, click APIs & services. Click Library.

For Search for APIs & services, type Google Trace. Click Stackdriver Trace API. If the API is not enabled, click Enable.

Return to Cloud Shell. To install the Node.js agent for Cloud Trace, execute the following command:

`npm install --save @google-cloud/trace-agent`

In Cloud Shell, click Open Editor to open in a new window.
Navigate to stackdriver-trace-monitoring/start.
Open ...frontend/app.js.

At the top of the app.js file, load the '@google-cloud/trace-agent' module and then start() it.

```javascript
frontend/app.js
// TODO: Load the trace-agent and start it
// Trace must be started before any other code in the
// application.

require('@google-cloud/trace-agent').start({
projectId: config.get('GCLOUD_PROJECT')
});

// END TODO
```

Save the file.

To start the application, execute the following command:

`npm start`

In Cloud Shell, click Web preview > Preview on port 8080 to preview the quiz application.
Return to the Cloud Platform Console. On the Navigation menu, click Trace. However, it may take a few minutes for the first request to populate.  After this happens, trace data is visible with just a few seconds' latency.

While waiting for the first Trace to populate, return to the Cloud Shell window and look for error messages.
Sometimes, due to a timing issue after enabling APIs, you might see errors about the Cloud Trace and/or Debug agent.

If you see errors, stop the Quiz application, restart after a minute, and then refresh the Quiz application home page.

Return to Trace in the Cloud Platform Console.
After your first Trace has populated, click Trace List.
You should see the Latency, HTTP Method, and URL for the request.

Return to the Cloud Trace window in the Cloud Platform Console and enable Auto Reload.
Look at the Trace list.

Eventually, two new requests are displayed:

GET /questions/add
POST /questions/add

Click on the POST request against the /questions/add URL.
You should see the total time to handle the request and the call made against the Cloud Datastore.

Return to the quiz application, and click Take Test. Click People. Complete the test, and submit feedback. Return to the Cloud Trace window in the Cloud Platform Console. Look at the Trace list again.

Eventually a set of new requests is displayed.

Find the following requests and click on them:

POST /api/quizzes/people
POST /api/quizzes/feedback/people
These represent the calls made from the client-side application against the quiz application.

Once again, you should see the total time taken to complete the request.

Notice that the calls are sequential. This is because the Cloud Pub/Sub request needs the correct answers from Question entities in Cloud Datastore to send answer data to Cloud Pub/Sub for Storage in Cloud Spanner.

In the quiz application, click Quite Interesting Quiz in the toolbar, and then click Leaderboard.
Return to the Cloud Trace window in the Cloud Platform Console and review the Trace data for the new request.
You should see that this request includes calls to Cloud Spanner.

Continue to explore the quiz application with Cloud Trace and identify and resolve a performance issue.

Return to the quiz application and click Take Test. Click Places. Complete the test, and submit feedback.
Return to the Cloud Trace window in the Cloud Platform Console. Look at the Trace list again.

Eventually a set of new requests will be displayed.

Find the following requests and click on them:

POST /api/quizzes/places
POST /api/quizzes/feedback/places
These represent the calls made from the client-side application against the quiz application.

Once again, you should see the total time taken to complete the request.

Notice that the calls are sequential. This is because the Cloud Pub/Sub request needs data from Cloud Datastore to send the answer data to Cloud Pub/Sub.

However, you should also see that each call to Cloud Pub/Sub is also sequential; this is not OK!

Return to the quiz application and click GCP.
Complete the test, and submit feedback.
Return to the Cloud Trace window in the Cloud Platform Console.
Look at the Trace list once again.

Eventually a set of new requests will be displayed.

Find the following requests and click on them:

POST /api/quizzes/gcp
POST /api/quizzes/feedback/gcp
These represent the calls made from the client-side application against the quiz application.

Once again, you should see the total time taken to complete the request.

Notice that the calls are sequential. This is because the Cloud Pub/Sub request needed data from Cloud Datastore to send the answer data to Cloud Pub/Sub.

However, you should also see that once again, each call to Cloud Pub/Sub is also sequential.

With four questions in the quiz, it's taking four times as long to send the Cloud Pub/Sub messages!

This is definitely not OK!

Modify application code to resolve the performance problem
Return to the Cloud Shell and launch the Code editor if it is not opened already.
Navigate to stackdriver-trace-monitoring/start/.
Open ...frontend/api/index.js.
Find the statement that publishes answer messages in sequence, and modify it to perform the operations in parallel.
Fortunately, this is a very easy change to make! The code change is shown below:

```javascript
api/index.js before the change
// TODO: Sends the answers to Pub/Sub in parallel
// Sends the answers to Pub/Sub in sequence
// Change sequence to parallel in the next statement

sequence(answersWithCorrect.map(answer => () => publisher.publishAnswer(answer))).then(() => {
  // Waits until all the Pub/Sub messages have been acknowledged before returning to the client
  const score = answersWithCorrect.filter(a => a.answer == a.correct).length;
  res.status(200).json({ correct: score, total: questions.length });
});

// END TODO
```

```javascript
api/index.js after the change
// TODO: Sends the answers to Pub/Sub in parallel
// Changed to parallel

 parallel(answersWithCorrect.map(answer => () => publisher.publishAnswer(answer))).then(() => {
  // Waits until all the Pub/Sub messages have been acknowledged before returning to the client
  const score = answersWithCorrect.filter(a => a.answer == a.correct).length;           
  res.status(200).json({ correct: score, total: questions.length });
});
 // END TODO
```

Confirm problem resolution with Cloud Trace
Return to the Cloud Shell, stop the application by pressing Ctrl+C, and then start it again.
Return to the Quiz application, and take the People, Places, and GCP tests again.
Return to the Cloud Trace window in the Cloud Platform Console.
Look at the Trace list once again.

Eventually a set of new requests will be displayed.

Find the following requests and click on them:

POST /api/quizzes/people|places|gcp
POST /api/quizzes/feedback/people|places|gcp
These represent the calls made from the client-side application against the quiz application.

Once again, you should see the total time taken to complete the request.

This time, you should see that all of the Pub/Sub messages have been dispatched in parallel.

The time taken to complete processing of the requests is significantly lower.

Visualizing Application Metrics with Cloud Monitoring
In this section, you continue to explore the quiz application with Cloud Monitoring by exploring metrics and create dashboards.

You will now setup a Monitoring workspace that's tied to your Qwiklabs GCP Project. The following steps create a new account that has a free trial of Monitoring.

In the Google Cloud Platform Console, click on Navigation menu > Monitoring.

Wait for your workspace to be provisioned.

When the Monitoring dashboard opens, your workspace is ready.

In the left-hand pane, click Dashboards.Click +Create Dashboard.
For New Dashboard Name, type Quiz Application Metrics.Click Line Chart.
Under Resource type, click inside the VM Instance textbox.Click Only show active.

Examples of resources that are active:

* cloud_function
* datastore_request
* gae_app (this is because of Cloud Datastore)
* gcs_bucket
* global
* pubsub_subscription
* pubsub_topic
* spanner_instance

Click on each resource in turn, and select a few metrics that are of interest to you.


Here are some example metrics to explore:

* cloudfunctions/function/execution_count
* datastore/api/request_count
* storage/api/request_count
* logging/log_entry_count
* pubsub/subscription/num_outstanding_messages
* pubsub/topic/message_sizes
* spanner/api/request_count

Create several charts, including your own selection of resource and metrics.
Review

Which Google Cloud product should be loaded first, before any other code?

* Cloud Trace
* Cloud Debugger
* Cloud Error Reporting
* Cloud Logging

Which GCP products are integrated into Google Cloud Trace?

* Cloud Datastore
* Cloud Spanner
* Cloud Pub/Sub
* Cloud Storage

With Cloud Monitoring, which feature allows charts to be assembled to view performance of your application?

* Alerting
* Uptime Checks
* Dashboards
* Groups
