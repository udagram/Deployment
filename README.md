# Udagram

Udagram is a simple cloud application developed alongside the Udacity Cloud Engineering Nanodegree. It allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice.

## Getting started

### Prerequisites
The following tools need to be installed on your machine:

- [Docker](https://www.docker.com/products/docker-desktop)
- [AWS CLI](https://aws.amazon.com/cli/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [KubeOne](https://github.com/kubermatic/kubeone)

Furthermore, you need to have:
- an [Amazon Web Services](https://console.aws.amazon.com) account
- a [DockerHub](https://hub.docker.com/) account

### Create an S3 bucket

The application uses an S3 bucket to store the images so an AWS S3 Bucket needs to be created

#### Permissions

Save the following policy in the Bucket policy editor:

```JSON
{
 "Version": "2012-10-17",
 "Id": "Policy1565786082197",
 "Statement": [
 {
 "Sid": "Stmt1565786073670",
 "Effect": "Allow",
 "Principal": {
 "AWS": "__YOUR_USER_ARN__"
 },
 "Action": [
 "s3:GetObject",
 "s3:PutObject"
 ],
 "Resource": "__YOUR_BUCKET_ARN__/*"
 }
 ]
}
```
Modify the variables `__YOUR_USER_ARN__` and `__YOUR_BUCKET_ARN__` by your own data.

#### CORS configuration

Save the following configuration in the CORS configuration Editor:

```XML
<?xml version="1.0" encoding="UTF-8"?>
 <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
 <CORSRule>
 <AllowedOrigin>*</AllowedOrigin>
 <AllowedMethod>GET</AllowedMethod>
 <AllowedMethod>POST</AllowedMethod>
 <AllowedMethod>DELETE</AllowedMethod>
 <AllowedMethod>PUT</AllowedMethod>
 <MaxAgeSeconds>3000</MaxAgeSeconds>
 <AllowedHeader>Authorization</AllowedHeader>
 <AllowedHeader>Content-Type</AllowedHeader>
 </CORSRule>
</CORSConfiguration>
```

### Create a PostgreSQL Instance

The application is using `PostgreSQL` database to store the feed data.

Create a PostgresSQL instance via Amazon RDS.

## Build Docker images

You need to have a built and published docker images to the following services:
- [users-api](https://github.com/udagram/users-api)
- [feed-api](https://github.com/udagram/feed-api)
- [forntend](https://github.com/udagram/frontend)
- [reverseproxy](https://github.com/udagram/reverseproxy)

You can use the already built images [here](https://hub.docker.com/u/omarradwan213). 

Alternatively, you can build your own images. To do so, you have to clone each service locally and run:
```
docker build -t <your-DockerHub-username/name-of-the-service> .
```
Then:
```
docker push <your-DockerHub-username/name-of-the-service>
```

## Deploy on AWS

- Launch a new EC2 instance on your AWS account `t2.micro` is ok.
- Access the EC2 instance and Clone this repository into it.
```
git clone git@github.com:udagram/Deployment.git
```

### Create a Kubernetes cluster

Please refer to [this](https://github.com/kubermatic/kubeone/blob/master/docs/quickstart-aws.md) step by step guide to install kubernetes on AWS cluster using KubeOne.

### Deploy the application services

Deploy and start the services on Kubernetes by executing:

```
kubectl apply -f /services/.
```

### Set your environment variables

You need to set you own environment variables in the files of the `/config` directory

```
POSTGRESS_USERNAME=__YOUR_MASTER_USERNAME__
POSTGRESS_PASSWORD=__YOUR_MASTER_PASSWORD__
POSTGRESS_DB=__YOUR_INITIAL_DATABASE_NAME__
POSTGRESS_HOST=__YOUR_AMAZON_RDS_DB_HOST__
JWT_SECRET=__YOUR_JWT_SECRET__
AWS_BUCKET=__YOUR_AWS_BUCKET_NAME__
AWS_REGION=__YOUR_AWS_BUCKET_REGION__
AWS_PROFILE=__YOUR_AWS_PROFILE__
AWS_CREDENTIALS=`cat ~/.aws/credentials | base64`
APP_URL=http://__YOUR_FRONTEND_SERVICE_URL__:8100
```

Replace the values by your data. `__YOUR_FRONTEND_SERVICE_URL__` can be retrieved using the command:

```
kubectl get svc
```

Add the reverseproxy URL to the file `src/environments/environment.prod.ts` in the [forntend](https://github.com/udagram/frontend) repository then build and push the new image.

You can also retrieve the reverse proxy URL by running

```
kubectl get svc
```

### Deploy the Kubernetes pods

If you chose to build your own docker images, then for each file in the `/deployments` directory, replace the image name by your own Docker Hub name. Example:

```YAML
containers:
 - image: __YOUR_DOCKERHUB_NAME__/feed-api
```

Deploy the Kubernetes pods by running

```
kbuectl apply -f /deployments/.
```