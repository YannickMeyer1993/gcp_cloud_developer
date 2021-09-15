Erstelle VM mit Namen blue, unter Networking, Network tags kommt „web-server“.
Networks use network tags to identify which VM instances are subject to certain firewall rules and network routes. Later in this lab, you create a firewall rule to allow HTTP access for VM instances with the web-server tag. Alternatively, you could check the Allow HTTP traffic checkbox, which would tag this instance as http-server and create the tagged firewall rule for tcp:80 for you.

Erstelle VM mit Namen green
Auf blue mit SSH nginx installieren:
`sudo apt-get install nginx-light -y`
Open the welcome page in the nano editor:

`sudo nano /var/www/html/index.nginx-debian.html`

Editieren.

Analog für green

Create the tagged firewall rule

Über Cloud Console firewall regel „allow-http-web-server” erstellen.


|Property		       |Value (type value or select option as specified)
| ---|---|
|Name			       |allow-http-web-server|
|Network		       |default|
|Targets			   |Specified target tags|
|Target tags		   |web-server|
|Source filter		   |IP Ranges|
|Source IP ranges	   |0.0.0.0/0|
|Protocols and ports   |Specified protocols and ports, and then check tcp,type: 80; and check Other protocols, type: icmp.|

Test vm erstellen:

`gcloud compute instances create test-vm --machine-type=f1-micro --subnet=default --zone=us-central1-a
`

Auf blue’s externe kann zugegriffen werden, auf green’s externe nicht.

Explore the Network and Security Admin roles
Cloud IAM lets you authorize who can act on specific resources, giving you full control and visibility to manage cloud resources centrally. The following roles are used in conjunction with single-project networking to independently control administrative access to each VPC Network:

Network Admin:
* Permissions to create, modify, and delete networking resources, except for firewall rules and SSL certificates. (but listing works)

Security Admin:
* Permissions to create, modify, and delete firewall rules and SSL certificates.

Explore these roles by applying them to a service account, which is a special Google account that belongs to your VM instance, instead of to an individual end user. Rather than creating a new user, you will authorize test-vm to use the service account to demonstrate the permissions of the Network Admin and Security Admin roles.

Verify current permissions
Currently, test-vm uses the Compute Engine default service account, which is enabled on all instances created by Cloud Shell command-line and the Cloud Console.

Try to list or delete the available firewall rules from test-vm.

Über SSH in test-vm: Try to list the available firewall rules:

`gcloud compute firewall-rules list`

Try to delete the allow-http-web-server firewall rule:

`gcloud compute firewall-rules delete allow-http-web-server`

The Compute Engine default service account does not have the right permissions to allow you to list or delete firewall rules. The same applies to other users who do not have the right roles.

Create a service account
Create a service account and apply the Network Admin role.

In the Console, navigate to Navigation menu (mainmenu.png) > IAM & admin > Service Accounts.

Notice the Compute Engine default service account.

Click Create service account.

Set the Service account name to Network-admin and click CREATE.

For Select a role, select Compute Engine > Compute Network Admin and click CONTINUE then click DONE.

After creating service account 'Network-admin', click on three dots at the right corner and click Create Key in the dropdown, then Create to download your JSON output.

Click Close.

A JSON key file download to your local computer. Find this key file, you will upload it in to the VM in a later step.

Rename the JSON key file on your local machine to credentials.json

To upload credentials.json through the SSH VM terminal, click on the gear icon in the upper-right corner, and then click Upload file.

Select credentials.json and upload it.

Click Close in the File Transfer window.

Authorize the VM with the credentials you just uploaded:

`gcloud auth activate-service-account --key-file credentials.json`

In the Console, navigate to Navigation menu (mainmenu.png) > IAM & admin > IAM.

Find the Network-admin account. Focus on the Name column to identify this account.

Click on the pencil icon for the Network-admin account.

Change Role to Compute Engine > Compute Security Admin.

Click Save.

Return to the SSH terminal of the test-vm instance.


