# AWS_EKS_ArgoCD_Hub-Spoke-Deployment 
   
### Pre-Requisite:         
         
### Install AWSCLI, EKSCTL & KUBECTL as below

Install AWS CLI
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

sudo apt install unzip

unzip awscliv2.zip

sudo ./aws/install

aws --version
```


Configure AWS CLI
```
aws configure
```
`Just give Access Key and Secret Key followed by ENTER-ENTER`


Install Kubectl
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.23.6/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

kubectl version
```


Install Eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp 

sudo mv /tmp/eksctl /usr/local/bin

eksctl version
```



1. Create 3 EKS Clusters 
```
eksctl create cluster --name hub-cluster --region us-west-1
```
```
eksctl create cluster --name spoke-cluster-1 --region us-west-1
```
```
eksctl create cluster --name spoke-cluster-2 --region us-west-1
```

2. Switch to right Context point to the Cluster you want to be on (Hub)

	After creating the clusters kubectl will point to the latest cluster created 
	```
	kubectl config get-contexts
	```
	(Retrieve and display a list of available contexts in the current Kubernetes configuration, showing information such as context name, associated cluster, authentication details, and namespace)
	![CONTEXT](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/99c3c08f-d48b-4e9a-af5d-e09d633917b0)

 	```
	kubectl config get-contexts | grep us-west-1
	```
 	(Identifying contexts related to a specific region or environment, such as "us-west-1")
	```
	kubectl config use-context iam-root-account@hub-cluster.us-west-1.eksctl.io
	```
 	(Switches the active context to the one specified, allowing subsequent kubectl commands to operate in the context of the specified Kubernetes cluster and user)
	```
 	kubectl config current-context
	```
 	(identify which Kubernetes cluster and user credentials are currently in use for kubectl command)

	![CURRENT-CONTEXT](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/1e1c88e7-bbdd-433b-b5e9-1d69efdba120)


3. Install  ArgoCD
```
kubectl create namespace argocd
```
```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```


4. Verify the ArgoCD pods
```
C:\Users\pavan\Desktop\YT\EKS-ARGOCD-MUTLI-CLUSTER
```


5. Run ArgoCD in HTTP Mode(Insecure)
```
kubectl get cm -n argocd
```
![CM-ARGOCD](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/4bc4418f-87c6-4b32-be29-7a02e69daf22)
	
```
kubectl edit cm argocd-cmd-params-cm -n argocd
```

Now add the below and save it 

```
data:
	server.insecure: "true"
```
```
kubectl describe deployment/argocd-server -n argocd
```
(You can see all configurations are read from the argocd-cmd-params-cm which is mounted)
```
kubectl edit deployment argocd-server -n argocd
```
(You can search for /insecure)


6. Verfiy the service
```
kubectl get svc -n argocd
```
![SVC](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/49cf75f1-e22f-4bb8-9f6c-11df49eac66f)


7. Set the SVC as NodePort
```
kubectl edit svc argocd-server -n argocd
```
Change type: ClusterIP -> type: NodePort


8. Verify the NodePort
```
kubectl get svc -n argocd
```
![NODEPORT](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/29deb455-5677-4a24-8c35-69837e907de4)


9. Pick an Public IP followed by port number assigned (30879)

Before doing the above open the Security Group for Hub Instances (Any instace that your accessing must be permitted to access the above port) the above port 


10. Login to ArgoCD
```
kubectl get secret -n argocd
```
![SECRET-LIST](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/7e69d979-7426-42ff-9a29-12786fdf32d8)


```
kubectl edit secret argocd-initial-admin-secret -n argocd
```
```
echo "dVFRTWhCeEVPb21qTUV6cQ==" | base64 -d && echo
```
(Gives the password to login)
```
Username: admin
Password: ()
```
![ARGOCD-UI](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/cf0d3104-3fd5-4834-b9ba-e225d142715a)


11. ArgoCD UI doesn't support adding of Clusters but only deleting of clusters


12. Install ArgoCD CLI
```
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
```
```
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```
```
rm argocd-linux-amd64
```
```
argocd version
```
![ARGOCD-CLI-VERSIOM](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/b828f22a-751a-461d-9928-1cd33b7fd910)


13. Login to ArgoCD CLI
```
argocd login 54.193.31.170:30879
```
![ARGOCD-CLI-LOGIN](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/46155f33-0841-4597-ba82-7056ece7d6e2)


14. Now add cluster in ArgoCD CLI

![ARGOCD-SETTTINGS-CLUSTER](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/3da62c4b-119c-441e-ac25-aed3ee72a055)

```
argocd cluster add iam-root-account@spoke-cluster-1.us-west-1.eksctl.io --server 54.193.31.170:30879
```
![SPOKE1-ADD](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/0486581e-d53e-4ed0-ab50-e9ee670cd3f8)

```
argocd cluster add iam-root-account@spoke-cluster-2.us-west-1.eksctl.io --server 54.193.31.170:30879
```
![SPOKE2-ADD](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/a73f281d-de62-4461-98d6-335a460de6cc)

![ARGOCD-SETTTINGS-CLUSTER-3-ADD](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/bd37e555-18e8-4c73-a172-b5a92e9e8e78)


15. Go to the ArgoCD UI

	![ARGOCD-UI-LOGIN](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/e716b292-ab17-42ac-b4dd-f90e7207c39e)

	Click on Create Application
	
	Give name
	
	Project Name - default
	
	Sync Policy - Automatic
	
	Repository URL - https://github.com/sqlonlinux97/Guest-Book-Pavan
	
	Path - Give path if the YML file are in a folder in Repo or give " . "  
	
	Destination - Cluster - URL - Add the Spoke2 cluster
	
	Namespace - default 
	
	Click on Create

	![APP1-ARGOCD](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/4a01968f-50d4-4078-ac4f-533ab6730155)

	Similarly Create Application for Spoke2

 	![APP2-ARGOCD](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/1b66f19e-9e75-412a-ad79-79416e63864f)

	Auto sync happens every 3 minutes


17. Switch context to Spoke1
```
kubectl config use-context iam-root-account@spoke-cluster-1.us-west-1.eksctl.io
```

17. Now go to the GitHub 

	Try to edit the configmap.yml  
	
	Changing the parameter to ui_properties_file_name: "pavan-interface.properties"

	![GIT-TERM-TEST](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/ed3807a9-3cd1-4824-b22a-69c11fa2958b)
	

19. Now go to the ArgoCD UI 

	Sync all the Apps 
	
	You can check the configmap in UI and also in terminal
	
 	![GIT-TEST](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/30f30e17-8bd6-4a07-8e0c-ec49c92af1a7)

	You can similarly check for another App created on ArgoCD for the another cluster

![APP-PODS-SPOKE1](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/b1ddc5ab-64c8-41ff-bb93-74dae5b5237a)

---

### EKS CLUSTER:

![AWS-K8S-CLUSTER](https://github.com/Pavan-1997/AWS_EKS_ArgoCD_Hub-Spoke-Deployment/assets/32020205/a2e4bd7f-8eff-4dfc-961e-240c266a3fda)
