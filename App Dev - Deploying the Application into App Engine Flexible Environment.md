`. prepare_environment.sh`

This script file

* Creates an App Engine application.
* Creates a Cloud Storage bucket.
* Exports environment variables GCLOUD_PROJECT and GCLOUD_BUCKET.
* Runs npm install.
* Creates entities in Datastore.
* Creates a Pub/Sub topic.
* Creates a Spanner Instance, Database, and Table.
* Creates a Cloud Function.
* Prints out the Google Cloud Platform Project ID.

Create the app.yaml file for the frontend
In the Cloud Shell code editor, open frontend/app.yaml.
Add two key: value pairs:
The first indicates that you want to use the nodejs runtime.

The second indicates that you want to use the flexible environment.

...frontend/app.yaml

runtime: nodejs
env: flex

Modify the frontend/config.json file to include a key GCLOUD_BUCKET, the value is the name of the -media bucket in your project, which is <GCP-Project-ID>-media.

...frontend/config.json

{
"GCLOUD_BUCKET" : "<REPLACE_WITH_BUCKET_NAME>"
}

`gcloud app deploy ./frontend/app.yaml`

In the App Engine window, click Versions. Use the checkboxes to select both versions of the application, and click Split traffic in the top-right of the Versions page.
Configure the traffic allocation to deliver 50% of traffic to the old version, and 50% to the new version.
Select the radio button to randomly split traffic to each versions.


