`gcloud config set compute/zone us-east1-d`

`git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git`

`cd continuous-deployment-on-kubernetes`

`gcloud container clusters create jenkins-cd \
--num-nodes 2 \
--machine-type n1-standard-2 \
--scopes https://www.googleapis.com/auth/projecthosting,cloud-platform`

`gcloud container clusters get-credentials jenkins-cd`

`helm repo add stable https://charts.helm.sh/stable`

`helm repo update`

Use the Helm CLI to deploy the chart with your configuration set.

`helm install cd stable/jenkins -f jenkins/values.yaml --version 1.2.2 –wait`

Run the following command to setup port forwarding to the Jenkins UI from the Cloud Shell

`export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &`

Now, check that the Jenkins Service was created properly:

`kubectl get svc`

We are using the Kubernetes Plugin so that our builder nodes will be automatically launched as necessary when the Jenkins master requests them. Upon completion of their work, they will automatically be turned down and their resources added back to the clusters resource pool.

Notice that this service exposes ports 8080 and 50000 for any pods that match the selector. This will expose the Jenkins web UI and builder/agent registration ports within the Kubernetes cluster. Additionally, the jenkins-ui service is exposed using a ClusterIP so that it is not accessible from outside the cluster.

The Jenkins chart will automatically create an admin password for you. To retrieve it, run:

`printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`

To get to the Jenkins user interface, click on the Web Preview button in cloud shell, then click Preview on port 8080:

You should now be able to log in with username admin and your auto-generated password.
You now have Jenkins set up in your Kubernetes cluster!
