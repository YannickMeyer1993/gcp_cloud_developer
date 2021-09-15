It is best practice to run the Cloud Logging agent on all your VM instances.
Auf VM via SSH
Run the Monitoring agent install script command in the SSH terminal of your VM instance to install the Cloud Monitoring agent.

`curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh sudo bash add-monitoring-agent-repo.sh
sudo apt-get update
sudo apt-get install stackdriver-agent`

Run the Logging agent install script command in the SSH terminal of your VM instance to install the Cloud Logging agent

`curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh sudo bash add-logging-agent-repo.sh
sudo apt-get update
sudo apt-get install google-fluentd`

Create an uptime check
In the Cloud Console, click Navigation menu > Monitoring
In the Cloud Console, in the left menu, click Uptime checks, and then click Create Uptime Check.
Title: Lamp Uptime Check, then click Next.
Protocol: HTTP
Resource Type: Instance
Applies to: Single, lamp-1-vm
Path: leave at default
Check Frequency: 1 min

Create an alerting policy
In the left menu, click Alerting, and then click Create Policy.
Click Add Condition.
Set the following in the panel that opens, leave all other fields at the default value.
Target: Start typing "VM" in the resource type and metric field, and then select:
Resource Type: VM Instance (gce_instance)
Metric: Type "network", and then select Network traffic (gce_instance+1). Be sure to choose the Network traffic resource with agent.googleapis.com/interface/traffic

Configuration
Condition: is above
Threshold: 500
For: 1 minute

Click on drop down arrow next to Notification Channels, then click on Manage Notification Channels.
Scroll down the page and click on ADD NEW for Email.
In Create Email Channel dialog box, enter your personal email address in the Email Address field and a Display name.
Click on Notification Channels again, then click on the Refresh icon to get the display name you mentioned in the previous step.

Create a dashboard and chart
In the left menu select Dashboards, and then Create Dashboard.
Name the dashboard Cloud Monitoring LAMP Qwik Start Dashboard.
Add the first chart
Click Line option in Chart library.
Name the chart title CPU Load.
Set the Resource type to VM Instance.
Set the Metric CPU load (1m). Refresh the tab to view the graph.
Add the second chart
Click + Add Chart and select Line option in Chart library.
Name this chart Received Packets.
Set the resource type to VM Instance.
Set the Metric Received packets (gce_instance). Refresh the tab to view the graph.
Leave the other fields at their default values. You see the chart data.


View your logs
Cloud Monitoring and Cloud Logging are closely integrated. Check out the logs for your lab.
Select Navigation menu > Logging > Logs Explorer.
Select the logs you want to see, in this case, you select the logs for the lamp-1-vm instance you created at the start of this lab:
Click on Resource.
Select VM Instance > lamp-1-vm in the Resource drop-down menu.
Click Add.
Leave the other fields with their default values.
Click the Stream logs.
