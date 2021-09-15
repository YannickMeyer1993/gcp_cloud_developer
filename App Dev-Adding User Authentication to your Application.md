`git clone https://github.com/GoogleCloudPlatform/training-data-analyst`

`cd ~/training-data-analyst/courses/developingapps/java/firebase/start`

`prepare_environment.sh`

This script file:

* Creates an App Engine application.
* Creates a Cloud Storage bucket named gs://[Project-ID]-media.
* Exports two environment variables: GCLOUD_PROJECT and GCLOUD_BUCKET.
* Runs mvn clean install.
* Creates entities in Cloud Datastore.
* Prints out the Project ID.

From the Google Cloud console, open a new browser tab and navigate to https://console.firebase.google.com/.

Be sure to open the new tab from the Google Cloud console window to stay in the lab environment. You may need to sign in again using your Qwiklabs username and password.
