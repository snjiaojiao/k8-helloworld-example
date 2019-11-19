# k8-helloworld-example
What is covered in the example?
1. use kops to create a kubernetes cluster on AWS
2. install istion on k8 cluster
3. Deploy nginx application on the cluster
4. Configure Istio so the application is accessible externally 

Prerequisite
1. AWS account
2. kubectl is installed
3. kops is installed
4. Docker is installed
5. helm is installed

Part1: Create a k8 cluster using KOPS
1. Create a route53 domain for your cluster 
Since kops us DNS for discovery, you need a vaild DNS for reach kubernetes API server.
You can create route53 from AWS console or via awscli command
Note: Please create a private hosted-zone if you don't have a registered domain. You will need a VPC to create a private hosted-zone

Step2: Create an S3 bucket to store you clusters state

aws s3 mb s3://<name>

You can
export KOPS_STATE_STORE=s3://<name>

Step3: Build your cluster configuration

kops create cluster --zones=<zone> <cluster-name>

This will create a defualt clouster configuration for you, you can modify it if you want.

Step4: Create the clouster on AWS
   Before update the clouster we need to create secret for ssh into the master and node servers:
   kops create secret --name <clustername> sshpublickey admin -i ~/.ssh/<pub_key> --state=s3://<s3_bucketname>
   Setup the cluster:
   kops update cluster <cluster-name> --yes
For more reference please got to: https://kubernetes.io/docs/setup/production-environment/tools/kops/

Part2: install istio to your cluster (there are multiple ways to install istio,i am using helm template here)
1. Download istio release https://istio.io/docs/setup/getting-started/#download
   curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.2.9 sh -
   Note: I am using istio-1.2.9 here as i have found the latest version 1.4 doesn't work with kubernetes version v1.11.10 
2. Create istio-system namespace
   kubectl create namespace istio-system
3. Install all the Istio Custom Resource Definitions (CRDs) using kubectl apply:
   helm template istio-init install/kubernetes/helm/istio-init  --namespace istio-system | kubectl apply -f -
4. Wait for all Istio CRDs to be created:
   kubectl -n istio-system wait --for=condition=complete job --all
5. Select a configuration profle (I am using demo profile here)
   helm template istio install/kubernetes/helm/istio --namespace istio-system | kubectl apply -f -
Note: helm command here are for helm3, helm2 command is slightly different
Reference: https://istio.io/docs/setup/install/helm/#option-1-install-with-helm-via-helm-template

Part3: Build Docker image and deploy v1 and v2 to kubernete cluster

Step1: Build docker image v1 and v2
    in v1, the index.html has the content "I am on Server B"
    in v2, the index.html has the content "I am on Server C"
    To build docker version 1:
    docker build -t <name>:v1 . 
    To build docker version 2:
    Update the index.html content and build the docker image again
    docker build -t <name>:v2 .
Step2: Push image to AWS ECR
  Please create a repository on AWS ECR first
  Follow the steps here: https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html
Step3: Deploy v1 and v2 application to the namespace dev
  1. create namespace and enable istio
     kubectl apply -f namespaces.yaml
  2. add label to two nodes (since we want two version run on different node, we need to add labels to the node)
     kubectl label nodes <node1> servergroup=group_b
     kubectl label nodes <node2> servergroup=group_c
  3. deploy v1 use helm template
     helm template v1 helloworld --namespace dev -f namespace-overrides/v1.yaml | kubectl apply -n dev -f -
  4. deploy v2 use helm template 
     helm template v2 helloworld --namespace dev -f namespace-overrides/v2.yaml | kubectl apply -n dev -f -
  Note: the difference between v1 and v2 are:
     1. imageTag (Docker image for the POD deployment)
     2. version (the POD label)
     3. serverGroup (for node selector)
  
     nodeSelector:
        servergroup: {{ .Values.serverGroup }}
  if you run kubectl get pods -n dev, you should see two pods are running now
  5. Create a clusterIp servcie
     kubectl apply -f service.yaml -n dev
 
 Part4: Istio configuration (using istio default controller here: ingressgateway
 1. Create a gateway:
    kubectl apply -f gateway.yaml -f istio-system
 2. Create virtual service: (loadbalancer is round robin by default)
    kubectl apply -f virtualservice.yaml -f istio-system

        
  


