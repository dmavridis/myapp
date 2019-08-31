# Capstone Project 

## Scope of the Project

The purpose of this project is to develop a simple application applying the principles of CI/CD and GitOps, using Kubernetes. 

Due to small experience with DevOps, I decided to use a simple application, that includes an `index.html` file, which is deployed to a Docker image of an Nginx server, as shown in the following pipeline. 

![pipeline](images/pipeline.png)

## Approach

My initial approach was to use Jenkins as it was described in the previous project. I was able to successfully generate and deploy the docker image to **Amazon ECR** as shown in the following part of the Jenkinsfile.

```
pipeline {
    agent any
    stages {
        stage('Lint HTML'){
            steps{
                sh 'tidy -q -e *.html'
            }
        }
        stage('Upload Docker to ECR') {
            steps {
                sh 'echo "Building docker image "'
                script{
                    docker.build('dimi-nginx')
                    docker.withRegistry('https://929444784092.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:7d072b26-ca5f-4a96-93bc-885bab7f5b00') {
                        docker.image('dimi-nginx').push('latest')
                    }
                }
            }
        }
        stage('Remove current cluster from EKS'){
            steps{
                withKubeConfig([credentialsId: 'AKIA5QZZZ47OO6PXKRWA', serverUrl:' https://9705639DB1B9054B46FD3B8A1FCA64EB.yl4.us-east-1.eks.amazonaws.com']){
                    sh 'kubectl delete deployment.apps/dimicloud'
                }
            }
        }  
    }
```

This approach would work but I has issues to deploy to **Amazon EKS** from Jenkins, as there were issues with the plugins, that I could not overcome. 

Therefore I decided to switch to **Jenkins-X** that has native support for Kubernetes. 

I followed the tutorial: <https://aws.amazon.com/blogs/opensource/continuous-delivery-eks-jenkins-x/>

### Steps

Initially I creates an EC2 instance that runs the JenkinsX master and I used the following commands to create the cluster.

```
eksctl create cluster \
--name dimicloud \
--version 1.13 \
--nodegroup-name standard-workers \
--node-type m5.large \
--nodes 2 \
--nodes-min 1 \
--nodes-max 3 \
--node-ami auto
```

This script is using the `eksctl` command that automatically creates the VPC and the nodegroups using Cloudformation. Two nodes are up and running as shown in the screenshot.

![nodes](images\nodes.png)

Also, running jx command the following is generate:

```
ubuntu@ip-172-31-56-251:~$ jx get eks
NAME            REGION
dimicloud       us-east-1
```



## Creating a project

Jenkins-X comes with project templates. Since the application I wanted to use is similar to one of the templates that creates a docker image running PHP, I created that template and modified the docker image accordingly. Following this process, the code scaffolding is implemented, as shown in the GitHub <https://github.com/dmavridis/myapp>

The command I used was:

```
jx create quickstart
```

The docker file is generating and uploading the container to AWS ECR. It is then deployed to the cluster.

The initial webpage looks like the screenshot. 

![version1](images/version1.png)



Making a small change in the html file, after a few minutes a new version is up as shown in the following screenshots:



![1566341023889](images/steps.png)

![version2](images/version2.png)

