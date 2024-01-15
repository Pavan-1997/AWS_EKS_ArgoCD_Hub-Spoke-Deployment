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

7. Set the SVC as NodePort
```
kubectl edit svc argocd-server -n argocd
```
Change type: ClusterIP -> type: NodePort


8. Verify the NodePort
```
kubectl get svc -n argocd
```

9. Pick an Publice IP followed by port number assigned (30879)

Before doing the above open the Security Group for Hub Instances (Any instace that your accessing must be permitted to access the above port) the above port 


10. Login to ArgoCD
```
kubectl get secret -n argocd
```
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

13. Login to ArgoCD CLI
```
argocd login 54.193.31.170:30879
```

14. Now add cluster in ArgoCD CLI
```
argocd cluster add iam-root-account@spoke-cluster-1.us-west-1.eksctl.io --server 54.193.31.170:30879
```
```
argocd cluster add iam-root-account@spoke-cluster-2.us-west-1.eksctl.io --server 54.193.31.170:30879
```

15. Go to the ArgoCD UI

	Click on Create Application
	
	Give name
	
	Project Name - default
	
	Sync Policy - Automatic
	
	Repository URL - https://github.com/sqlonlinux97/Guest-Book-Pavan
	
	Path - Give path if the YML file are in a folder in Repo or give " . "  
	
	Destination - Cluster - URL - Add the Spoke2 cluster
	
	Namespace - default 
	
	Click on Create 
	
	Similarly Create Application for Spoke2
	
	Auto sync happens every 3 minutes


16. Switch context to Spoke1
```
kubectl config use-context iam-root-account@spoke-cluster-1.us-west-1.eksctl.io
```

17. Now go to the GitHub 

	Try to edit the configmap.yml 
	
	Changing the parameter to ui_properties_file_name: "pavan-interface.properties"
	

18. Now go to the ArgoCD UI 

	Sync all the Apps 
	
	You can check the configmap in UI and also in terminal
	
	You can similarly check for another App created on ArgoCD for the another cluster
