Überblick
1.	Install/configure an advanced deployment using Deployment Manager sample emplates.
2.	Enable Cloud monitoring.
3.	Configure Cloud Monitoring Uptime Checks and notifications.
4.	Configure a Cloud Monitoring dashboard with two charts, one showing CPU usage and the other ingress traffic.
5.	Perform a load test and simulate a service outage.

Cloud Monitoring collects metrics, events, and metadata from Google Cloud Platform, Amazon Web Services, hosted uptime probes, application instrumentation, and a variety of common application components including Cassandra, Nginx, Apache Web Server, Elasticsearch, and many others.

deployment manager samples
`git clone https://github.com/GoogleCloudPlatform/deploymentmanager-samples.git
`
Beispielanpassungen:
Zone, in der deploy werden soll
Maximales Scaling Limit des Frontend

Enter this command to name the application advanced-configuration and pass Deployment Manager the configuration file (nodejs.yaml).
`gcloud deployment-manager deployments create advanced-configuration --config nodejs.yaml
`
Find the global load balancer forwarding rule IP address

Since the IP address was established dynamically when the Deployment Manager implemented the global forwarding rule (specified in the template), you'll need to find that address to test the application.

`gcloud compute forwarding-rules list
`
Unter Navigation menu > Monitoring ein uptime Check einrichten, Protocol TCP und Port 8080.
Unter Monitoring > Alerting > +Create Policy Emailbenachrichtung für CPU Usage/Utilization (Threshold type 20, FOR 1 minute), Email als Notification. Dash Board einrichten
