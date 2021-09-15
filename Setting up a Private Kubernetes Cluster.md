Creating a private Cluster
When you create a private cluster, you must specify a /28 CIDR range for the VMs that run the Kubernetes master components and you need to enable IP aliases.

Next you'll create a cluster named private-cluster, and specify a CIDR range of 172.16.0.16/28 for the masters. When you enable IP aliases, you let Kubernetes Engine automatically create a subnetwork for you.

Run the following to create the cluster:

`gcloud beta container clusters create private-cluster \
--enable-private-nodes \
--master-ipv4-cidr 172.16.0.16/28 \
--enable-ip-alias \
--create-subnetwork ""`

List the subnets in the default network:

`gcloud compute networks subnets list --network default`

In the output, find the name of the subnetwork that was automatically created for your cluster. For example, gke-private-cluster-subnet-xxxxxxxx. Save the name of the cluster, you'll use it in the next step.

now get information about the automatically created subnet:

`gcloud compute networks subnets describe [SUBNET_NAME] –region us-central1`

The output shows you the primary address range with the name of your GKE private cluster and the secondary ranges. In the output you can see that one secondary range is for pods and the other secondary range is for services. Notice that privateIPGoogleAccess is set to true. This enables your cluster hosts, which have only private IP addresses, to communicate with Google APIs and services.

At this point, the only IP addresses that have access to the master are the addresses in these ranges:
•	The primary range of your subnetwork. This is the range used for nodes.
•	The secondary range of your subnetwork that is used for pods.

To provide additional access to the master, you must authorize selected address ranges.

Create a source instance which you'll use to check the connectivity to Kubernetes clusters:

gcloud compute instances create source-instance --zone us-central1-a --scopes 'https://www.googleapis.com/auth/cloud-platform'

Erhalte externe IP:

`gcloud compute instances describe source-instance --zone us-central1-a | grep natIP
`
Copy the <nat_IP> address and save it to use in later steps.

Run the following to Authorize your external address range, replacing [MY_EXTERNAL_RANGE] with the CIDR range of the external addresses from the previous output (your CIDR range is natIP/32). With CIDR range as natIP/32, we are allowlisting one specific IP address:

`gcloud container clusters update private-cluster \
--enable-master-authorized-networks \
--master-authorized-networks [MY_EXTERNAL_RANGE]`

In a production environment replace [MY_EXTERNAL_RANGE] with your network external address CIDR range.

Now that you have access to the master from a range of external addresses, you'll install kubectl so you can use it to get information about your cluster. For example, you can use kubectl to verify that your nodes do not have external IP addresses.

SSH into source-instance with:

`gcloud compute ssh source-instance --zone us-central1-a
`
In SSH shell install kubectl component of Cloud-SDK

`gcloud components install kubectl`
oder
`sudo apt-get install kubectl`

Configure access to the Kubernetes cluster from SSH shell with:

`gcloud container clusters get-credentials private-cluster --zone us-central1-a
`
Verify that your cluster nodes do not have external IP addresses:

`kubectl get nodes --output yaml | grep -A4 addresses
`
Here is another command you can use to verify that your nodes do not have external IP addresses:

`kubectl get nodes --output wide
`
„exit“ the SSH session

Delete the Kubernetes cluster:


`gcloud container clusters delete private-cluster --zone us-central1-a
`
Creating a private cluster that uses a custom subnetwork

In the previous section Kubernetes Engine automatically created a subnetwork for you. In this section, you'll create your own custom subnetwork, and then create a private cluster. Your subnetwork has a primary address range and two secondary address ranges.

Create a subnetwork and secondary ranges:

`gcloud compute networks subnets create my-subnet \
--network default \
--range 10.0.4.0/22 \
--enable-private-ip-google-access \
--region us-central1 \
--secondary-range my-svc-range=10.0.32.0/20,my-pod-range=10.4.0.0/14`

Create a private cluster that uses your subnetwork:

`gcloud beta container clusters create private-cluster2 \
--enable-private-nodes \
--enable-ip-alias \
--master-ipv4-cidr 172.16.0.32/28 \
--subnetwork my-subnet \
--services-secondary-range-name my-svc-range \
--cluster-secondary-range-name my-pod-range`

MY_EXTERNAL_RANGE von oben

`gcloud container clusters update private-cluster2 \
--enable-master-authorized-networks \
--master-authorized-networks [MY_EXTERNAL_RANGE]`

SSH auf source-instance:

`gcloud compute ssh source-instance --zone us-central1-a`

Configure access to the Kubernetes cluster from SSH shell with:

`gcloud container clusters get-credentials private-cluster2 --zone us-central1-a`

Verifizieren, dass die Knoten des Clusters keine externe IP haben (wie oben)

At this point, the only IP addresses that have access to the master are the addresses in these ranges:
•	The primary range of your subnetwork. This is the range used for nodes.
•	The secondary range of your subnetwork that is used for pods.



