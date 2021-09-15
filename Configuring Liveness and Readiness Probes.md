`gcloud container clusters get-credentials $my_cluster --zone $my_zone`

`git clone https://github.com/GoogleCloudPlatform/training-data-analyst`

`ln -s ~/training-data-analyst/courses/ak8s/v1.1 ~/ak8s`

`cd ~/ak8s/Probes/`

A Pod definition file called exec-liveness.yaml has been provided for you that defines a simple container called liveness running Busybox and a liveness probe that uses the cat command against the file /tmp/healthy within the container to test for liveness every 5 seconds. The startup script for the liveness container creates the /tmp/healthy on startup and then deletes it 30 seconds later to simulate an outage that the Liveness probe can detect.
```yaml
apiVersion: v1
kind: Pod
metadata:
labels:
test: liveness
name: liveness-exec
spec:
containers:
- name: liveness
  image: k8s.gcr.io/busybox
  args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
      livenessProbe:
      exec:
      command:
        - cat
        - /tmp/healthy
          initialDelaySeconds: 5
          periodSeconds: 5
```

`kubectl create -f exec-liveness.yaml`

Within 30 seconds, view the Pod events:

`kubectl describe pod liveness-exec`

After 35 seconds, view the Pod events again:

`kubectl describe pod liveness-exec`

veness probe..
Container will be killed and recreated.
Warning  Unhealthy      5s ... Liveness probe failed:
cat: can't open '/tmp/healthy': No such file or directory

Wait another 60 seconds, and verify that the Container has been restarted:

`kubectl get pod liveness-exec`

The output shows that RESTARTS has been incremented in response to the failure detected by the liveness probe:

`kubectl delete pod liveness-exec`

Configure a Deployment with liveness and readiness probes
Readiness probes are configured similarly to liveness probes. The only difference in configuration is that you use the readinessProbe field instead of the livenessProbe field. Readiness probes control whether a specific container is considered ready, and this is used by services to decide when a container can have traffic sent to it.

In this task a Deployment definition file called readiness-deployment.yaml has been provided for you that defines a deployment called readiness-deployment with three Pods running containers with the Hello-World demo application.

Each container has a readiness probe defined that uses the cat command against the file /tmp/healthy within the container to test for readiness every 5 seconds.

Each container also has a liveness probe defined that uses the cat command against the same file within the container to test for readiness every 5 seconds but it also has a startup delay of 45 seconds to simulate an application that might have a complex startup process that takes time to stabilize after a container has started.

The startup script for the containers sleeps for 30 seconds, launches the hello-world web application in the background, and then creates the /tmp/healthy file. This keeps each container from passing the readiness test for at least 30 seconds after startup. The pods will then seem to pause for 30 seconds after starting up before they are flagged as ready and therefore before any service will select them as endpoints. The startup script then waits for a random period of time, between 60 and 180 seconds, before deleting the /tmp/healthy file. That will cause both the readiness and liveness probes to fail so the readiness-svc service will remove the endpoint for that container and at more or less the same time the failure of the liveness probe will cause the container to restart.

Once the service has started handling traffic this pattern ensures that the service will forward traffic only to containers that are ready to handle traffic.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
labels:
test: readiness
name: readiness-deployment
spec:
replicas: 3
selector:
matchLabels:
app: readiness-test
template:
metadata:
labels:
app: readiness-test
spec:
containers:
- name: readiness
image: gcr.io/google-samples/hello-app:1.0
ports:
- containerPort: 8080
protocol: TCP
args:
- /bin/sh
- -c
- sleep 30; nohup ./hello-app &2>/dev/null & touch /tmp/healthy; export xx=$((60+$RANDOM % 120)) ; sleep $xx ;  rm -rf /tmp/healthy
livenessProbe:
exec:
command:
- cat
- /tmp/healthy
initialDelaySeconds: 45
timeoutSeconds: 1
periodSeconds: 5
readinessProbe:
exec:
command:
- cat
- /tmp/healthy
initialDelaySeconds: 5
timeoutSeconds: 1
periodSeconds: 5
```

`kubectl create -f readiness-deployment.yaml`

`kubectl describe deployment readiness-deployment`

`kubectl get pods`

After about 30 seconds the startup script will have created the /tmp/healthy file that then allows the next scheduled readiness test to pass and the Pods will be listed as Ready as shown here.

A service definition file called readiness-service.yaml is provided for you that creates a load balancer service definition for a service called readiness-svc. The service is configured to select containers that have the app label readiness-test that will match the containers in the deployment you created in the previous task.

```yaml
apiVersion: v1
kind: Service
metadata:
name: readiness-svc
spec:
type: LoadBalancer
selector:
app: readiness-test
ports:
- protocol: TCP
  port: 80
  targetPort: 8080
```

`kubectl create -f readiness-service.yaml`

Monitor the behavior of the liveness and readiness probes
You can now check how the ready status of the Pods in the deployment corresponds to the endpoints that are actively enabled for your service. As Pods fail the readiness and liveness probe tests they are marked as not ready, their endpoints are removed from the service, and the liveness probe initiates the restart process to recover the failed pod. The restarted pods are not marked as ready immediately and have to wait for the readiness test to pass before the service will add the endpoint back into it's pool.

To check that you can connect to the application first query the external ip-address from the load balancer service details and save it in an environment variable:

`export EXTERNAL_IP=$(kubectl get services readiness-svc -o json | jq -r '.status.loadBalancer.ingress[0].ip')`

`curl $EXTERNAL_IP`

`kubectl get pods`

You should continue to see responses with no failures or timeouts even though individual containers are being restarted regularly due to the failures detected by the liveness probes. If all three containers restart at more or less the same time you might still see a timeout but that should rarely happen.

The combination of the liveness and readiness probes provides a way to ensure that failed systems are restarted safely while the service only forwards traffic on to containers that are known to be able to respond.
