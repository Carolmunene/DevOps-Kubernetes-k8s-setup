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
