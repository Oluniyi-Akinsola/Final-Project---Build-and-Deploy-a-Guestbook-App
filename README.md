# Final-Project---Build-and-Deploy-a-Guestbook-App
Final project to build and deploy a simple guestbook application consisting of a web front end which will have a text input where you can enter any text and submit.

# Project Overview
In this final project, you will build and deploy a simple guestbook application. The application consists of a web front end which will have a text input where you can enter any text and submit. For all of these we will create Kubernetes Deployments and Pods. Then we will apply Horizontal Pod Scaling to the Guestbook application and finally work on Rolling Updates and Rollbacks.

Guestbook is a simple web application that we will build and deploy with Docker and Kubernetes. The application consists of a web front end which will have a text input where you can enter any text and submit. For all of these we will create Kubernetes Deployments and Pods. Then we will apply Horizontal Pod Scaling to the Guestbook application and finally work on Rolling Updates and Rollbacks.

# Solution
 1. Clone the git repository that contains the artifacts needed for this lab

```
[ ! -d 'guestbook' ] && git clone https://github.com/ibm-developer-skills-network/guestbook
```
2. Change to the `guestbook` directory for this project

```
cd guestbook/v1/guestbook
```
# Build the guestbook app
To begin, we will build and deploy the web front end for the guestbook app. Complete the Dockerfile with the necessary Docker commands to build and push your image
The path to this file is `guestbook/v1/guestbook/Dockerfile`.

# Dockerfile
3. Create a Dockerfile to build and push the image
 ```
 FROM golang:1.15 as builder
 RUN go get github.com/codegangsta/negroni
 RUN go get github.com/gorilla/mux github.com/xyproto/simpleredis/v2
 COPY main.go .
 RUN go build main.go

 FROM ubuntu:18.04

 COPY --from=builder /go//main /app/guestbook
 COPY public/index.html /app/public/index.html
 COPY public/script.js /app/public/script.js
 COPY public/style.css /app/public/style.css
 COPY public/jquery.min.js /app/public/jquery.min.js

 WORKDIR /app
 CMD ["./guestbook"]
 EXPOSE 3000
 ```
4. Export your name as an environment variable so it can be used in sebsequent commands.
   
```
export MY_NAMESPACE=sn-labs-$USERNAME
```
5. Build the guestbook app using the Docker Build command.

```
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1
```
6. Push the image to IBM Cloud Container Registry
```
 ibmcloud cr login
 ibmcloud cr region-set us-south
 docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```
7. Verify that the image was pushed successfully
```
ibmcloud cr images
```
8. Open the `deployment.yml` file in the `v1/guestbook` directory & view the code for the deployment of the application
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  replicas: 1
  selector:
    matchLabels:
      app: guestbook
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - image: us.icr.io/<your sn labs namespace>/guestbook:v1
        imagePullPolicy: Always
        name: guestbook
        ports:
        - containerPort: 3000
          name: http
        resources:
          limits:
            cpu: 50m
          requests:
            cpu: 20m
```
9. Apply the deployment using
```
kubectl apply -f deployment.yml
```
10. Open a New Terminal and enter the below command to view your application
```
kubectl port-forward deployment.apps/guestbook 3000:3000
```

11. Launch your application on port 3000 and you should be able to see your running application

# Autoscale the Guestbook application using Horizontal Pod Autoscaler

12. Autoscale the Guestbook deployment using `kubectl autoscale deployment`
```
kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10
```
You can check the current status of the newly-made HorizontalPodAutoscaler, by running:
```
kubectl get hpa guestbook
```
13. Open another new terminal and enter the below command to generate load on the app to observe the autoscaling (Please ensure your port-forward command is running
```
kubectl run -i --tty load-generator --rm --image=busybox:1.36.0 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- <your app URL>; done"
```
14. Continue further commands in the 1st terminal. Run the below command to observe the replicas increase in accordance with the autoscaling
```
kubectl get hpa guestbook --watch
```
Run the above command again after 5-10 minutes and you will see an increase in the number of replicas which shows that your application has been autoscaled.

Run the below command to observe the details of the horizontal pod autoscaler:
```
kubectl get hpa guestbook
```
Please close the other terminals where load generator and port-forward commands are running

# Perform Rolling Updates and Rollbacks on the Guestbook application
Please update the title and header in index.html to any other suitable title and header like <Your name> Guestbook - v2 & Guestbook - v2.
Run the below command to build and push your updated app image:
```
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```
Update the values of the CPU in the deployment.yml to cpu: 5m and cpu: 2m
Apply the changes to the deployment.yml file
```
kubectl apply -f deployment.yml
```
Open a new terminal and run the port-forward command again to start the app
```
kubectl port-forward deployment.apps/guestbook 3000:3000
```
Launch your application on port 3000
Run the below command to see the history of deployment rollouts:
```
kubectl rollout history deployment/guestbook
```
Run the below command to see the details of Revision of the deployment rollout:
```
kubectl rollout history deployments guestbook --revision=2
```
Run the below command to get the replica sets and observe the deployment which is being used now
```
kubectl get rs
```
Run the below command to undo the deploymnent and set it to Revision 1
```
kubectl rollout undo deployment/guestbook --to-revision=1
```
Run the below command to get the replica sets after the Rollout has been undone
```
kubectl get rs
```





