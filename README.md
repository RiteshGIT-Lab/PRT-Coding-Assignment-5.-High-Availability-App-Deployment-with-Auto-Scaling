# PRT-Coding-Assignment-5.-High-Availability-App-Deployment-with-Auto-Scaling
Deployed an auto-scaling app on EKS using HPA and Cluster Autoscale

## terraform-autoscaler-project

High Availability App Deployment with Auto Scaling Scenario: Deploy an auto-scaling app on EKS using HPA and Cluster Autoscaler. Requirements: - Provision EKS with auto-scaling enabled. - Containerize and deploy the app with resource limits. - Configure HPA and Cluster Autoscaler. Deliverables: - Terraform config, Kubernetes HPA YAML. - Dockerfile and Jenkins pipeline. Evaluation Criteria: - Scalability and resource efficiency. - Pipeline reliability. you are an advanced devops engineer. kindly implement this project step by step also write code files for each step so that i can copy
Yes. This is a complete real-world DevOps project and is commonly asked in interviews. Since there are many files (Terraform, Dockerfile, Kubernetes manifests, Jenkinsfile, IAM policies, Cluster Autoscaler, metrics-server, HPA, etc.), the implementation is too large to fit accurately into a single response.
To keep everything correct and copy-paste ready, I recommend building it as a complete project in sequential parts, where every file is fully written and tested conceptually.
Project Architecture
GitHub
   │
   ▼
Jenkins Pipeline
   │
   ├── SonarQube
   ├── Trivy Scan
   ├── Docker Build
   ├── Push Image to ECR
   ▼
Amazon EKS
   │
   ├── Deployment
   ├── Service
   ├── HPA
   ├── Metrics Server
   ├── Cluster Autoscaler
   ▼
EC2 Worker Nodes
      ▲
      │
Terraform provisions everything
________________________________________
We will implement in 15 Parts
Part 1
Project Structure

eks-autoscaling-project

│
├── terraform
│     ├── provider.tf
│     ├── versions.tf
│     ├── variables.tf
│     ├── outputs.tf
│     ├── vpc.tf
│     ├── eks.tf
│     ├── iam.tf
│     ├── nodegroup.tf
│
├── app
│     ├── Dockerfile
│     ├── package.json
│     ├── server.js
│
├── kubernetes
│     ├── namespace.yaml
│     ├── deployment.yaml
│     ├── service.yaml
│     ├── hpa.yaml
│     ├── ingress.yaml
│
├── autoscaler
│     ├── cluster-autoscaler.yaml
│
├── metrics-server
│     ├── metrics-server.yaml
│
├── jenkins
│     ├── Jenkinsfile
│
└── README.md
________________________________________
Part 2
Terraform
•	VPC 
•	Subnets 
•	Internet Gateway 
•	NAT Gateway 
•	Route Tables 
•	IAM 
•	EKS 
•	Managed Node Group 
•	Auto Scaling enabled 


<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/d7e3b3e0-00c9-4964-a512-b1884ad827b1" />

Dockerize Application
•	Dockerfile 
•	.dockerignore 

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/0e4eacca-9d26-40bb-9c35-f79ec05088be" />


Push Image to Amazon ECR
Building the image
docker build -t eks-ha-app:v1 .

login into ecr repo
aws ecr get-login-password \
--region ap-south-1 \
| docker login \
--username AWS \
--password-stdin \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com

Building the docker image
cd app
docker build -t eks-ha-app .

tag the image
docker tag eks-ha-app:latest \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/eks-ha-app-app:latest

Push the image
docker push \
<ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/eks-ha-app-app:latest

Verify
aws ecr describe-images \
--repository-name eks-ha-app-app

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/b2c1f50f-9442-42c9-86eb-dd31914871a9" />

Kubernetes
•	Namespace 
•	Deployment 
•	Service 
•	Resource Limits 

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/0a74a7cb-cc4b-4d3f-a88e-67b7dc9fa33c" />

Metrics Server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl get deployment metrics-server -n kube-system
kubectl top nodes

kubectl top pods -n autoscaling-app

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/383dad9a-0d8e-45b5-9c5a-9b30af51433e" />

Horizontal Pod Autoscaler
Created hpa.yaml
kubectl apply -f kubernetes/hpa.yaml

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/8aaa2724-7f3e-43ce-8ec8-32e096f9eda5" />

Cluster Autoscaler

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/7010b50e-53dd-4e3d-9996-14dc2ce9a4b0" />

Part 6.4 - Generate CPU Load
Run a temporary pod to create load:
kubectl run load-generator \
  --rm -it \
  --image=busybox \
  -- /bin/sh
Inside the pod:
while true; do
    wget -q -O- http://nginx-service.autoscaling-app.svc.cluster.local
done
Alternatively, if you want to generate CPU-intensive work:
while true; do
    dd if=/dev/zero of=/dev/null
done
________________________________________
Part 6.5 - Watch Autoscaling
Monitor the HPA:
kubectl get hpa -n autoscaling-app -w
Monitor the pods:
kubectl get pods -n autoscaling-app -w
Monitor deployment status:
kubectl get deployment -n autoscaling-app -w
Check CPU metrics:
kubectl top pods -n autoscaling-app
________________________________________
Expected Scaling Behavior
Initial state:
Pods : 2
CPU  : 12%
After sustained CPU load:
Pods : 3
CPU  : 65%
As load increases:
Pods : 4
Pods : 5
Pods : 6
Up to:
Pods : 10
When the load stops, Kubernetes will gradually scale the deployment back down to the minimum of 2 pods.
________________________________________
Summary
At this stage, your deployment includes:
•	✅ Namespace 
•	✅ Deployment 
•	✅ Service 
•	✅ Resource Requests and Limits 
•	✅ Metrics Server 
•	✅ Horizontal Pod Autoscaler (HPA) 
•	✅ Automatic scaling based on CPU utilization 
The remaining autoscaling component for your project is Cluster Autoscaler (Part 7), which automatically adds or removes EC2 worker nodes in your EKS cluster when pod scheduling demands exceed current node capacity.
pls write code for part 7
For this project, Part 7 is the Cluster Autoscaler. Unlike the HPA, which scales pods, the Cluster Autoscaler scales the EC2 worker nodes in your EKS cluster.
Project Structure
cluster-autoscaler/
├── iam-policy.json
├── service-account.yaml
└── cluster-autoscaler.yaml
________________________________________
Part 7.1 - Create IAM Policy
cluster-autoscaler/iam-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "autoscaling:DescribeAutoScalingGroups",
        "autoscaling:DescribeAutoScalingInstances",
        "autoscaling:DescribeLaunchConfigurations",
        "autoscaling:DescribeScalingActivities",
        "autoscaling:DescribeTags",
        "autoscaling:SetDesiredCapacity",
        "autoscaling:TerminateInstanceInAutoScalingGroup",
        "ec2:DescribeLaunchTemplateVersions"
      ],
      "Resource": "*"
    }
  ]
}
Create the policy:
aws iam create-policy \
  --policy-name AmazonEKSClusterAutoscalerPolicy \
  --policy-document file://cluster-autoscaler/iam-policy.json
________________________________________
Part 7.2 - Create IAM Service Account
Replace <cluster-name> with your EKS cluster name.
eksctl create iamserviceaccount \
  --cluster <cluster-name> \
  --namespace kube-system \
  --name cluster-autoscaler \
  --attach-policy-arn arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AmazonEKSClusterAutoscalerPolicy \
  --approve
Example:
eksctl create iamserviceaccount \
  --cluster ha-eks-cluster \
  --namespace kube-system \
  --name cluster-autoscaler \
  --attach-policy-arn arn:aws:iam::254292659362:policy/AmazonEKSClusterAutoscalerPolicy \
  --approve
________________________________________
Part 7.3 - Service Account (Optional if created by eksctl)
cluster-autoscaler/service-account.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cluster-autoscaler
  namespace: kube-system
If you created the IAM service account with eksctl, you do not need to apply this manifest.
________________________________________
Part 7.4 - Cluster Autoscaler Deployment
cluster-autoscaler/cluster-autoscaler.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler

spec:
  replicas: 1

  selector:
    matchLabels:
      app: cluster-autoscaler

  template:
    metadata:
      labels:
        app: cluster-autoscaler

      annotations:
        cluster-autoscaler.kubernetes.io/safe-to-evict: "false"

    spec:
      serviceAccountName: cluster-autoscaler

      containers:
      - name: cluster-autoscaler

        image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.30.0

        imagePullPolicy: Always

        command:
        - ./cluster-autoscaler

        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false

        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<cluster-name>

        env:
        - name: AWS_REGION
          value: ap-south-1

        resources:
          requests:
            cpu: 100m
            memory: 300Mi

          limits:
            cpu: 300m
            memory: 600Mi
Replace:
<cluster-name>
with your EKS cluster name.
For example:
ha-eks-cluster

Part 7.5 - Enable Auto Discovery Tags
Tag your Auto Scaling Group so the Cluster Autoscaler can discover it.
aws autoscaling create-or-update-tags \
--tags \
ResourceId=<ASG_NAME>,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/enabled,Value=true,PropagateAtLaunch=true \
ResourceId=<ASG_NAME>,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/<cluster-name>,Value=owned,PropagateAtLaunch=true
Example:
aws autoscaling create-or-update-tags \
--tags \
ResourceId=eks-nodegroup-123456,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/enabled,Value=true,PropagateAtLaunch=true \
ResourceId=eks-nodegroup-123456,ResourceType=auto-scaling-group,Key=k8s.io/cluster-autoscaler/ha-eks-cluster,Value=owned,PropagateAtLaunch=true

Part 7.6 - Deploy the Cluster Autoscaler
kubectl apply -f cluster-autoscaler/cluster-autoscaler.yaml
Verify:
kubectl get deployment -n kube-system cluster-autoscaler
Check the pod:
kubectl get pods -n kube-system | grep cluster-autoscaler
View logs:
kubectl logs -f deployment/cluster-autoscaler -n kube-system
________________________________________
Part 7.7 - Verify Autoscaling
Check nodes:
kubectl get nodes
Watch HPA:
kubectl get hpa -n autoscaling-app -w
Watch nodes:
watch kubectl get nodes


final output:
Generate CPU Load
kubectl run load-generator \ --rm -it \ --image=busybox \ -- /bin/sh

while true; do wget -q -O- http://nginx-service.autoscaling-app.svc.cluster.local done
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/590349da-75db-44a7-8ff4-79ee7fc195ca" />


<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/28d82d25-289d-4fed-83c2-4bbe887f62c4" />

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/52830768-9c9d-40fb-bb7a-50333ba6edf2" />
<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/261ce7aa-9084-4e71-bf5a-4dc094879aca" />

## Nginx runing 

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/483bf469-357a-4d75-9d4f-ab13aad32653" />
