`gcloud config set project $(gcloud projects list --format='value(PROJECT_ID)' --filter='qwiklabs-gcp')
`
`mkdir gsp761 && cd $_
`
`nano go.mod
module github.com/ymotongpoo/pet-theory
go 1.13`

nano main.go

`package main`
`import (
"fmt"
"log"
"net/http"
"os"
)`

`func main() {
port := os.Getenv("PORT")
if port == "" {
port = "8080"
}
http.HandleFunc("/v1/", func(w http.ResponseWriter, r *http.Request) {
fmt.Fprintf(w, "{status: 'running'}")
})
log.Println("Pets REST API listening on port", port)
if err := http.ListenAndServe(":"+port, nil); err != nil {
log.Fatalf("Error launching Pets REST API server: %v", err)
}
}`

nano Dockerfile (standard template)

`FROM gcr.io/distroless/base-debian10
WORKDIR /usr/src/app
COPY server .
CMD [ "/usr/src/app/server" ]`

The file server is the execution binary built from main.go.

`go build -o server`

Deploy your simple REST API by running:

`gcloud builds submit \
--tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1`

Once the container has been built, deploy it:

`gcloud beta run deploy rest-api \
--image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.1 \
--platform managed \
--region us-central1 \
--allow-unauthenticated`

Firebase  Native  nam5  Create Database

`gsutil cp -r gs://spls/gsp645/2019-10-06T20:10:37_43617 gs://$GOOGLE_CLOUD_PROJECT-customer
gcloud beta firestore import gs://$GOOGLE_CLOUD_PROJECT-customer/2019-10-06T20:10:37_43617/`

Rebuild the source code:

`go build -o server`

`gcloud builds submit \
--tag gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2`

`gcloud beta run deploy rest-api \
--image gcr.io/$GOOGLE_CLOUD_PROJECT/rest-api:0.2 \
--platform managed \
--region us-central1 \
--allow-unauthenticated`


