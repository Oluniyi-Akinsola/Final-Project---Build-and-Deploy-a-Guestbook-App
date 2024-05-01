# Final-Project---Build-and-Deploy-a-Guestbook-App
Final project to build and deploy a simple guestbook application consisting of a web front end which will have a text input where you can enter any text and submit.

# Project Overview
In this final project, you will build and deploy a simple guestbook application. The application consists of a web front end which will have a text input where you can enter any text and submit. For all of these we will create Kubernetes Deployments and Pods. Then we will apply Horizontal Pod Scaling to the Guestbook application and finally work on Rolling Updates and Rollbacks.

Guestbook is a simple web application that we will build and deploy with Docker and Kubernetes. The application consists of a web front end which will have a text input where you can enter any text and submit. For all of these we will create Kubernetes Deployments and Pods. Then we will apply Horizontal Pod Scaling to the Guestbook application and finally work on Rolling Updates and Rollbacks.

# Solution
Clone the git repository that contains the artifacts needed for this lab
[ ! -d 'guestbook' ] && git clone https://github.com/ibm-developer-skills-network/guestbook

# Build the guestbook app
To begin, we will build and deploy the web front end for the guestbook app
Complete the Dockerfile with the necessary Docker commands to build and push your image
# Dockerfile
 FROM golang:1.15 as builder
 RUN go get github.com/codegangsta/negroni
 RUN go get github.com/gorilla/mux github.com/xyproto/simpleredis
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
