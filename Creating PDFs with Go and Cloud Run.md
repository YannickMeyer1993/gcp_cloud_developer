Create a new service account to trigger the Cloud Run services:
gcloud iam service-accounts create pubsub-cloud-run-invoker --display-name "PubSub Cloud Run Invoker"

Give the service account permission to invoke the PDF converter service:

`gcloud run services add-iam-policy-binding pdf-converter \
--member=serviceAccount:pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
--role=roles/run.invoker \
--region us-central1 \
--platform managed`

Find your project number by running this command:

`PROJECT_NUMBER=$(gcloud projects list \
--format="value(PROJECT_NUMBER)" \
--filter="$GOOGLE_CLOUD_PROJECT")`

Enable your project to create Cloud Pub/Sub authentication tokens:

`gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
--member=serviceAccount:service-$PROJECT_NUMBER@gcp-sa-pubsub.iam.gserviceaccount.com \
--role=roles/iam.serviceAccountTokenCreator`

`curl -X GET $SERVICE_URL`

The anonymous GET request will result in an error message "Your client does not have permission to get URL". This is good; you don't want the service to be callable by anonymous users.

Now try invoking the service as an authorized user:

`curl -X GET -H "Authorization: Bearer $(gcloud auth print-identity-token)" $SERVICE_URL
`
Create a Pub/Sub subscription so that the PDF converter will be run whenever a message is published to the topic new-doc:

`gcloud pubsub subscriptions create pdf-conv-sub \
--topic new-doc \
--push-endpoint=$SERVICE_URL \
--push-auth-service-account=pubsub-cloud-run-invoker@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
`

