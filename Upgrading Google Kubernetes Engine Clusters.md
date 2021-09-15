Task 1. Deploy a GKE cluster
In this task, you use Google Cloud Console to deploy a GKE cluster running a Kubernetes version that is not the most recent release. You will upgrade this cluster to a more recent release in a later task.

In the Google Cloud Console, on the Navigation menu ( 9a951fa6d60a98a5.png), click Kubernetes Engine > Clusters.
Click Create cluster.
Click the Static version drop down menu in Master version to expand the list of available Kubernetes versions.


Select the lowest version number in the list. This might already be selected as the default.


Leave all of the other settings at the defaults and click Create to begin creating a GKE cluster.
Note: You need to wait a few minutes for the cluster deployment to complete.

Click Check my progress to verify the objective.
Deploy GKE cluster

Task 2: Upgrade your GKE cluster
In this task, you upgrade your GKE cluster and all of the nodes.

Upgrade your GKE cluster
In the Google Cloud Console, on the Navigation menu ( 9a951fa6d60a98a5.png), in the Kubernetes Engine section, click Clusters.
Click the name of your cluster to open its properties.
To the right of the master version, click the Upgrade available link to start the upgrade wizard.
cluster_details.png

Select the most recent (highest) build available and then click Change.

Note:

Version numbers are presented in the following format major.minor.patch.For Example in the version 1.14.10, 1.14.10 is the major version, 1.14.10 is the minor version, and 1.14.10 is the patch version.

If the version you are upgrading to is more than 1 minor version away from the current version. You may have to do this step in stages.

For Example: I will upgrade from 1.14.10 to 1.15.11 first, then I will upgrade from 1.15.11 to my most recent version (1.16.9).

Note: You need to wait 2 or 3 minutes for the master upgrade to complete.
You can refresh the page to check the status of the upgrade.

When the master upgrade has completed, the master version number changes to the version that you selected in the upgrade wizard.

Upgrade the node pool in your cluster
You must now upgrade the nodes of your cluster to the same version as the master.

Refresh your web browser to display the prompt, and then click Upgrade oldest node pool.

In the upgrade wizard window, choose the most recent version (at the top of the list), and then click Change to continue with the upgrade.

Because this process must upgrade all nodes in your cluster, it might take several minutes to complete.

You can check the status by refreshing the web browser. A progress bar appears.

When the node pool upgrade process finishes, your cluster upgrade is complete.

When all of the nodes have been upgraded the new version is reflected in the node pool details pane.
