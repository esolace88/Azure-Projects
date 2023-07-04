# Kubernetes CI/CD Projects

Welcome to my Kubernetes CI/CD Project! This project's focus is to build a full CI/CD environment leveraging Jenkins and Kubernetes to automate deployments. 

## Getting Started

To get started, you will need a GitHub account, Docker Account, and Azure account. In this demo, we will be rapidly deploying applications via Jenkins, Docker, and Kubernetes. 

**Note:** I've while this project was created in an Azure environment. It can easily be replicated in AWS, simply update the cloud-init files to conform to AWS standards. 

-------------

## INFO 
The environment utilized in this project is as follows: 

	-- Servers Type: Ubuntu 20.04 Focal Fossa LTS
	-- K8s Network Engine: Calico

	-- Source Control Management: GitHub
	-- Build Automation: Gradle
	-- Continuous Integration (CI): Jenkins w/ Webhooks
	-- Continous Delivery (CD): Jekins /w Pipelines
	-- Containers: Docker
	-- Orchestration: Kubernetes

The additional options that I will be covering are the following:

	-- Self-Healing
	-- Auto Scaling

## Step by Step 

**Instance Creation**

1. Login into Azure
2. Open Azure Terminal
3. Create Jenkins and Kube init files
4. Create a "Resource-Group" Variable
      ```bash
           $RG = az group list --query "[0].name" -o tsv
      ```
5. Create 3 VM instances, 1-Jenkins Server & 2-Kube Servers (Control & Node). **Note** In the cmd below ensure you replace the fields with your password and file name. 
      ```bash
           az vm create --resource-group $RG --name [VM-Jenkins|VM-Control|VM-Node] `
           --image Canonical:0001-com-ubuntu-server-focal:20_04-lts:latest `
           --size Standard_D2s_v3 --admin-username cloud-user `
           --admin-password <enterpass> --public-ip-sku Standard `
           --custom-data <enter-init-filename>
      ```
6. Open Ports for Jenkins Server && Node Server
      ```bash
       az vm open-port --port 8080 --priority 900 --resource-group $RG --name VM-Jenkins | VM-Node
      ```
7. List IP & Wait patiently (5-10min) for instance to deploy && Check /var/log/cloud-init-output.log deployment status. 
	```bash
	az vm list-ip-addresses -o table
	```

**Kubernetes Configuration**

**On All Server ENABLE the Services**
```bash
	sudo systemctl enable [jenkins | docker]
```

**On Control Server**
1. Add a Current User to the Docker group or Create a Deployment User and restart Docker service
	```bash
	sudo usermod -aG docker $USER
	sudo systemctl restart docker
	```
2. On the server designated as Control, create a kube-config.yml file. **Note Delete kube-config.yml file once applied**
      ```bash
    	apiVersion: kubeadm.k8s.io/v1beta3
		kind: ClusterConfiguration
		networking:
  		  podSubnet: "10.244.0.0/24"
		kubernetesVersion: "v1.24.0"
		apiServer:
  		  extraArgs:
    	    service-node-port-range: 8000-31274
      ```
3. Initiate the Kubeadm
	```bash
	sudo kubeadm init --config kube-config.yml
	```
4. Make Kube Working Directory
	```bash
	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config
	```
5. Apply Network Add-On (Calico is used in)
	```bash
	kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
	```
6. Print Kubeadm Token to Join Node, Copy CMD and Paste into Node:
	```bash
	kubeadm token create --print-join-command
	```
7. Before leaving run the following cmd to watch for results
	```bash
	kubectl get nodes -w
	```
8. After running the cmd on the Node, return to the control server and wait for the node to show ready. 

**On Node Server**

1. Paste the Kubeadm Join cmd with Sudo
	```bash
	sudo kubeadm join....
	```

**Jenkins Configuration**

1. Add Jenkins user to Docker group and restart both Docker and Jenkins
	```bash
	sudo usermod -aG docker jenkins
	sudo systemctl restart docker && sudo systemctl restart jenkins
	```
2. Open a Browser to the server's public IP on port 8080, ex (http:public.ip:8080)
3. Navigate to the initial password location and paste it into the browser
	```bash
	sudo cat /var/lib/jenkins/secrets/initialAdminPassword
	```
3. Select "Install suggested plugins"
4. Create the Admin User for the Jenkins Server
5. Install Docker Pipeline Plug-in
	
	Manage Jenkins > Plugins > Available Plugins > Docker Pipeline > Install without restart

**Under the Jenkins System Page, Manage Jenkins > System**

6. Input VM-Control Server Public IP and Jenkins "Name" variable
*	Manage Jenkins > System > Global Properties > Environment variables 
*	--Name: prod_ip
*	--Value: public IP

7. Add GitHub Server & Secret Key, Add and Select Newly created Key
*	Manage Jenkins > System > GitHub Server
*	Name: GitHub
*	Credentials: Add > Jenkins 
*	Kind: Secret Text
*	Secret: Paste GitHub Token
*	ID: github_key
*	Description: Enter a Description
**Ensure you Select "Manage hooks"**

	To Create a Secret Key (Personal Access Tokens)
*	Github > Account "Settings" > Developer Settings > Personal Access Tokens > Generate New Token (Classic)
*	-- Name the Token
*	-- Select "admin:repo_hook"
*	-- Generate token


**Under the Jenkins Credentials Page, Manage Jenkins > Credentials**

8. Create Credentials for your Control Server and Docker Account
*	Manage Jenkins > Credentials > (global) > Add Credentials
*	-- Kind: Username with password
*	-- Username: enter a username
*	-- Password: enter the password
*	-- ID: webserver_login | docker_hub_login

**Deployment Testing**
**Under the Jenkins Item Page, Dashboard > New Item**

9. Create Jenkins Job
*	Dashboard > New Item 
*	-- Enter an Item Name
*	-- Select Multibranch Pipeline
*	Job Configuration
*	-- Display Name: Enter Name
*	-- Branch Source: GitHub
*	-- Repository HTTPS: Paste HTTPS Repo URL
*	-- Save

10. Run the Job
*	Scroll to the bottom of the page, in the Build Executor Status:
*	-- Select ">> master"
*	-- Wait and Watch the progress of your initial job running

11. Once Job successfully deploys, go to the Control Server to run
	```bash
	kubectl get pods
	```
12. Then open a browser to Node Public IP

**Congradulaiton** 
	If you were able to complete the steps above then you've successfully completed a CI/CD pipeline. With the configurations above anytime commits are published to your GitHub Repo - Jenkins will automate the deployment. 

Please Continue if you would like to implement further options to improve this CI/CD deployment. Note: below shows how to complete and test said option. To fully incorporate it into your CI/CD deployment ensure your docker and yml files are configured accordingly. 

**Features and Options Check**

1. Self-Healing Kubernetes Pods
   
**Manual Testing**
*	-- On the Control Server:
		```bash
		kubectl get pods
		```	
		```bash
		kubectl exec <Enter Pod Name> -- pkill
		```
*	-- If you run-run "kubectl get pods" you should see the restart count go up. This is due to Kubernetes' self-healing option.

  
**Liveness Probs**: Allows Kubernetes to fix pod before serious issues arise
*	-- First Manually download the Repo onto your Control Server
*	-- Then run: 
		```bash
		kubectl apply -f self-healing-option.yml
		```
*	-- Wait for Deployment to complete 
		```bash
		kubectl get pods -w
		```
*	-- Test the Liveness Prob
*	-- Open Browser to Node Public IP on Port 8080 / break
		```bash
		http://PublicIP:8080/break
		```
*	-- Wait a few seconds then delete "/break"
*	-- Then go back to the Control Server and run:  
		```bash
		kubectl get pods -w
		```
*	-- You should see the pods have Auto-Restarted

2. Auto Scaling (Horizantol)

	a. On the Control Server, Copy the metric yml file:
		```bash
		curl -LO https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
		```


	b. Modify the component and add hostNetwork: true & kubelet-insecure-tls, **Under Spec**

![alt text](https://github.com/esolace88/Azure-Projects/blob/main/Kubernetes-CICD/img/edits.png)
		```bash
		kubectl apply -f components.yaml
		```

	c. Wait for the new service to run
		```bash
		kubectl get pods -n kube-system -w
		```

	d. Apply Auto-Scaling Pods from Copied Repo
		```bash
		kubectl apply -f autoscaling-option.yml
		```

	e. Test Scaling, Must open two ssh sessions to run cmd and see results. 
*	-- Session 1:
*	-- Run The following:
			```bash
			kubectl run -i --tty load-generator --image=busybox /bin/sh
			```
	**Note Enter you private IP of the Node**
			```bash
			while true; do wget -q -O- http://<kubernetes node private ip>:8080/generate-cpu-load; done
			```
*	-- Session 2: 
*	-- Run The following: 
			```bash
			kubectl get hpa -w
			```
**After a while you should see the number of replicas increase. Once you terminate the busy box in session 1, Kubernetes will reduce the number of replicas.**








