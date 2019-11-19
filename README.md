# k8-helloworld-example
What is covered in the example?
1. use kops to create a kubernetes cluster on AWS
2. install istion on k8 cluster
3. Deploy nginx application on the cluster
4. Canary deployment use istio

Prerequisite
1. AWS account
2. kubectl is installed
3. kops is installed
4. Docker is installed

Step1: Create a k8 cluster using KOPS
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
kops update cluster <cluster-name> --yest

For more reference please got to: https://kubernetes.io/docs/setup/production-environment/tools/kops/

Step
