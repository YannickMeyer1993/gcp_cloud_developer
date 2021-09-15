`git clone https://github.com/GoogleCloudPlatform/training-data-analyst
`
cd ~/training-data-analyst/courses/developingapps/java/cloudstorage/start

`. prepare_environment.sh
`
This script file:

* Creates an App Engine application
* Exports an environment variable GCLOUD_PROJECT
* Runs mvn clean install
* Prints out the Project ID.

mvn spring-boot:run

To create a Cloud Storage bucket named <Project ID>-media, execute the following command:

`gsutil mb gs://$DEVSHELL_PROJECT_ID-media
`
You can create a bucket using the gsutil mb command, passing through the name of the bucket as gs://BUCKET_NAME You can use $DEVSHELL_PROJECT_ID as the bucket name prefix followed by -media

`import com.google.cloud.storage.*;`

`@Value("${google.storage.bucket}")
private String bucketName;`

`BlobInfo blobInfo = storage.create(
BlobInfo.newBuilder(bucketName, fileName)
.setContentType(file.getContentType())
.setAcl(new ArrayList<>(
Arrays.asList(Acl.of(Acl.User.ofAllUsers(),
Acl.Role.READER))))
.build(),
file.getInputStream());`
