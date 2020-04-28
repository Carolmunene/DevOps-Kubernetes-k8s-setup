Setup Kubernetes (K8s) Cluster on AWS

1. Create Ubuntu EC2 instance
2. Install AWSCLI

	curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
 	apt install unzip python
	 unzip awscli-bundle.zip
 	#sudo apt-get install unzip - if you dont have unzip in your system
 	./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws

3  Install kubectl

	curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage        .googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	sudo mv ./kubectl /usr/local/bin/kubectl
4.  Create an IAM user/role with Route53, EC2, IAM and S3 full access

5.  Attach IAM role to ubuntu server
    Note: If you create IAM user with programmatic access then provide Access keys.
    aws configure
6.  Install kops on ubuntu instance:

    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep     tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64 
7.  Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
8.  create an S3 bucket
    aws s3 mb s3://dev.k8s.valaxy.in
9.  Expose environment variable:
    export KOPS_STATE_STORE=s3://dev.k8s.valaxy.in
10. Create sshkeys before creating cluster
    ssh-keygen
11. Create kubernetes cluster definitions on S3 bucket
    kops create cluster --cloud=aws --zones=ap-southeast-1b --name=dev.k8s.valaxy.in --dns-zone=valaxy.in --dns private
12. Create kubernetes cluser
    kops update cluster dev.k8s.valaxy.in --yes
13. Validate your cluster
    kops validate cluster
14. To list nodes
    kubectl get nodes

-------------------------------------------------------------------------------------------------
Deploying Nginx container on Kubernetes

1. Deploying Nginx Container
	kubectl run sample-nginx --image=nginx --replicas=2 --port=80
	kubectl get pods
	kubectl get deployments

2. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them:
	kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
	kubectl get services -o wide
3. To delete cluster
	kops delete cluster dev.k8s.valaxy.in --yes
	
	
	
ANOTHER VERSION:
Installing Kubernetes on AWS using Kops
1. Launch Linux EC2 instance in AWS, we will use this EC2 instance to launch
our Kubernetes cluster by installing Kops on it. I will launch this instance in Ohio region but you can pick any region of your choice

2. Create and attach IAM role to EC2 Instance.
Kops would need permissions following permissions in order to operate:
	S3
	EC2
	VPC
	Route53
	Autoscaling
	etc..
	
Create a Role for the above specified permissions and then attach it to the running EC2 instance
3. Install Kops on EC2
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
4. Install kubectl
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
5. Create S3 bucket in AWS
S3 bucket is used by kubernetes to persist cluster state and so, create any S3 bucket of your choice

simplilearn.k8s

6. Create private hosted zone in AWS Route53
Head over to aws Route53 and create a private hosted zone
Choose a name of your choice - masterbarbercosmo.com
Choose type as private hosted zone for VPC
Select default vpc in the region you are setting up your cluster
7 Configure environment variables.
Open .bashrc file

	vi ~/.bashrc
Add following content into .bashrc, you can choose any arbitary name for cluster and make sure buck name matches the one you created in previous step.

export KOPS_CLUSTER_NAME=masterbarbercosmo.com
export KOPS_STATE_STORE=s3://masterbarbercosmo.com.k8s
Then running command to reflect variables added to .bashrc

	source ~/.bashrc
8. Create ssh key pair
This keypair is used for ssh into kubernetes cluster

ssh-keygen
9. Create a Kubernetes cluster definition.
kops create cluster \
--state=${KOPS_STATE_STORE} \
--node-count=2 \
--master-size=t2.micro \
--node-size=t2.micro \
--zones=us-east-2a,us-east-2b \
--name=${KOPS_CLUSTER_NAME} \
--dns private \
--master-count 1
10. Create kubernetes cluster
kops update cluster --yes
Above command may take some time to create the required infrastructure resources on AWS. Execute the validate command to check its status and wait until the cluster becomes ready

kops validate cluster
11. To connect to the master
ssh admin@api.masterbarbercosmo.com
