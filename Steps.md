IAM -> Roles -> ec2 -> Administrator

#### jenkins-server :
```
ubuntu 22 t2.2xlarge
no key pair
security -> 8080 , 9000 ,22(here for our convinent but in video not used) (jen and sonar)
30 gb storage

advanced details -> 
IAM instance profile -> Adminitrator access (role -> ec2 -> Administraor)

disbaled ssh access

USER DATA (jdk , jenkins , docker , terraform, aws cli , sonarqube conatiner , trivy) :
(https://github.com/Shabbirsyed05/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Jenkins-Server-TF/tools-install.sh)
connect by session-> manager
```
```
sudo su ubuntu
cd
ls 
sudo htop

java --version
docker version
docker ps
terraform --version
aws s3 help
dcoker ps
trivy --help

ip : 8080
systemctl status jenkins.service (copy password)
-> install suggested plugins

Manage Jenkins -> Plugins 
aws credentials , pipeline: AWS Steps
plugins -> terraform
plugins -> pipeline:stage view

Manage Jenkins -> Credentials -> System -> Global Cred -> 
Kind (Aws Credentials) ,ID (aws-creds) , access key , secret key

IAM -> user -> admin -> access key

plugins -> terraform

ec2 -> whereis terraform

configure terraform
Manage jenkins -> tools-> terraform =>
name (terraform) , install directory (/usr/bin/terraform) (from above)

plugins -> pipeline:stage view

Create a bucket with name "shabbir-bucket-for-devecops2"
Create Dynamo table with name Lock-Files , primary key (LockID) 

new job(infra) -> pipeline -> pipeline script (from git copy)
https://github.com/Shabbirsyed05/EKS-Terraform-GitHub-Actions/blob/master/Jenkinsfile

In 1st build -> it will just plan . In 2nd build , we need to select apply in parameters for getting it created.

As EKS cluster is in private vpc . we are using jump server to access eks cluster.

DEV VPC will be created
```
##### Jump server :
```
ubuntu 22  -> t2.medium
no key pair
vpc -> DEV (newly created one)
subnet -> public

30 gb storage

Advanced details ->  IAM instance profile -> Admin-access(it can use aws resources)

UserData (aws cli, kubectl (to get kubeconfig of kubernetes cluster), helm , eksctl (to create service accoints)):
```
```
sudo su ubuntu
cd
ls
aws --version
kubectl version
eksctl
helm
helm repo list
jenkins --verion

aws -> eks -> kubernetes version(1.29)
```
#### jenkins-server
```
aws configure

aws eks update-kubeconfig --name dev-medium-eks-cluster --region us-east-1
kubectl (not install as of now) (install it , if not installed)
sudo apt update
kubectl
kubectl get nodes (it will give error . As it is only accessible from jump server)
```
##### jump server :
```
aws configure
kubectl get nodes (error)
aws eks update-kubeconfig --name dev-medium-eks-cluster --region us-east-1
kubectl get nodes 
kubectl get all
```
###### Loadbalancer and ARGO cd now:

Download the policy for the LoadBalancer prerequisite:
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
Create the IAM policy using the below command
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```

```
cat im_policy.json
aws -> awsLoad (gui) AmazonEKSLoadBalancingPolicy()

in simple terms what we are trying to do right now is uh eks cluster is a kubernetes cluster right and within the kubernetes Clusters we have pods running
for example you have the Ingress controller running within the kubernetes cluster now how will a pod inside your eks cluster that is the Ingress controller pod create a load balancer
that is another AWS resource so for that what we need to do is we need tie up
the service account of the Pod with a im user or a im role in this case we will
use the IM role and using the policy we will grant permissions to the kubernetes
Pod so that it can using that service account create a load balancer so
basically we are tying up the concept of kubernetes with the IM am exactly
```
Create the IAM policy using the below command:
```
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```
Create OIDC Provider: (opt) (as it is configtured in github)
```
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=Three-Tier-K8s-EKS-Cluster --approve
```
Create a Service Account by using below command and replace your account ID with your one
```
eksctl create iamserviceaccount --cluster=dev-medium-eks-cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your_account_id>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1
```
```
kubectl get sa -n kube-system (if cant find loadbalancer delete it from cloudformation)
cloudformation -> stack -> loadbalancer -> delete
run the service account command
kubectl get sa -n kube-system 
```
Run the below command to deploy the AWS Load Balancer Controller :
```
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

kubectl get deployment -n kube-system aws-load-balancer-controller
```
```
If the pods are getting Error or CrashLoopBackOff, then use the below command:
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-west-1 --set vpcId=<vpc#> -n kube-system
```
##### ARGOCD:
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml

kubectl get all -n argocd

kubectl get svc -n argocd
```
```
aws -> loadbalancer -> no loadbalancer

kubectl edit svc argocd-server -n argocd
change type to LoadBalancer from ClusterIP

aws -> loadbalancer -> can be seen

copy DNS Name by opening the LoadBalancer and paste in browser (ARCD ui )
```
```
loadbalancer is is actually created by the cloud control manager component of eks cluster , 
it is not created by the load balancer controller that we have installed right because Aman has changed
the service type to load balancer who is receiving that request that request is received by the cloud control manager
component of eks
```
```
kubectl get secrets -n argocd
kubectl edit secret argocd-initial-admin-secret -n argocd (copy password)
echo password | base64 --decode
username -> admin
password -> from above
```
#### jenkins server :
```
docker ps
curl ifconfig.me (copy ip)
paste it in browser ip:9000
admin , admin
```
```
Sonarqube -> Administrator -> security -> users -> update token -> generate token -> copy the token

Sonarqube -> Administrator -> webhook -> name (jenkins) , url (http://sonar_ip:8080/sonarqube-webhook/)
frontend -> react , backend -> nodejs

Sonarqube -> Projects -> manually -> display name (frontend) , branch (main) -> locally
existing token -> (earlier copied one) , run analysis (other) , os (linux) -> copy it (later for frontend)

Sonarqube -> Projects -> manually -> display name (backend) , branch (main) -> locally
existing token -> (earlier copied one) , run analysis (other) , os (linux) -> copy it (later for Backend)

Manage jenkins -> Credentials -> System -> Global credentials -> kind (secret text) ->
secret (sonar-token copied) , id (sonar-token)

Manage jenkins -> Credentials -> System -> Global credentials -> kind (secret text) ->
secret (aws_account_id_copy) , id (ACCCOUNT_ID)
```
ECR registory :
```
ECR registory -> create repository -> private , frontend
ECR registory -> create repository -> private , backentend

Manage jenkins -> Credentials -> System -> Global credentials -> kind (secret text) ->
secret (frontend) , id (ECR_REPO1)

Manage jenkins -> Credentials -> System -> Global credentials -> kind (secret text) ->
secret (backend) , id (ECR_REPO2)

Manage jenkins -> Credentials -> System -> Global credentials -> kind (username and password) ->
username(shabbirsyed05) , password (token) , ID (GITHUB-APP)

Manage jenkins -> Credentials -> System -> Global credentials -> kind (secret text) ->
secret (github_token) , ID (github)
```
plugins :
```
Docker , Docker Pipeline , Docker commons , Docker API
NOdeJS
OWASP Dependency-Check
Sonarqube Scanner
```
```
Manage Jenkins -> Tools ->
Nodejs => Name (nodejs) , Install automatically (Install from nodejs.org)
SonarQube Scanner => Name (soanr-scanner) , Install automatically (Install from Maven Central)
Dependency-Check(for owasp) => Name (DP-Check) , Install automatically (Install from github.com)
Docker => Name (docker) , Install automatically (Install from docker.com)
```
```
we havent configured webhook from jenkins to sonarqube (here 1st code analysis and 2nd stage quality gates)
Manage Jenkins -> System -> SonarQube Installation => Name (sonar-server), Server URL : http://jenkins_ip:9000
server auth : sonar-token
```

Pipeline :

Job (three-tier-frontent) -> pipeline -> pipeline (github -> jenkins pipeline -> frontend)
#### In code change the project_name from 3tier-frontend to frontend and commnet owasp by //
##### remove jdk from tools
build

##### Change the image name in the ecr repository in frontend and backend deployment (image: 727646485718 to current name)
https://github.com/Shabbirsyed05/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Kubernetes-Manifests-file/Frontend/deployment.yaml

https://github.com/Shabbirsyed05/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Kubernetes-Manifests-file/Backend/deployment.yaml
```
sonarqube -> frontend -> check here all

pipeline : 
Job (three-tier-backend) -> pipeline -> pipeline (github -> jenkins pipeline -> frontend)
# In code change the project_name from three-tier-backend to backend and commnet owasp by //
# remove jdk from tools
build (if failing with no add and commit. Build it again)

pipeline : 
Job (three-tier-frontend) -> pipeline -> pipeline (github -> jenkins pipeline -> frontend)
# In code change the project_name from three-tier-frontend to backend and commnet owasp by //
# remove jdk from tools
build (if failing with no add and commit. Build it again)

after build check -> github -> kubernetes-manifest -> deployment.yaml (commit will be just ago)
```

#### argo CD :
```
Argo Cd -> Settings -> Repository -> connect using ssh -> 
type (git) , project (default) , URL (git url) -> connect

on kubernetes cluster -> 1st databse then backend then frontend
ArgoCD -> Application -> create Application =>
name (three-tier-database) , project Name (default) , sync (automatic), self heal (tick)
Source => URL (from dropdown) , Path (Kubernetes-Manifests-file/Database) (from github)
Destination => Url (from dropdown) , Namespace (three-tier)
```

jumpserver :
```
sudo su ubuntu
cd
kubectl create ns three-tier

Now database is deployed by ArgoCD
We have deployed it manually for database. For frontend and backend we have used ci/cd pipeline
```
jumpserver :
```
kubectl get all -n three-tier
kubectl get pv -n three-tier
```
```
ArgoCD -> Application -> create Application =>
name (three-tier-backend), project Name (default) , sync (automatic), self heal (tick)
Source => URL (from dropdown) , Path (Kubernetes-Manifests-file/Backend) (from github)
Destination => Url (from dropdown) , Namespace (three-tier)

ArgoCD -> Application -> create Application =>
name (three-tier-frontend), project Name (default) ,sync (automatic), self heal (tick)
Source => URL (from dropdown) , Path (Kubernetes-Manifests-file/Frontend) (from github)
Destination => Url (from dropdown) , Namespace (three-tier)

ArgoCD -> Application -> create Application =>
name (three-tier-ingress), project Name (default) ,sync (automatic), self heal (tick)
Source => URL (from dropdown) , Path (Kubernetes-Manifests-file/) (from github)
Destination => Url (from dropdown) , Namespace (three-tier)
```
```
AWS -> Load Balancer -> k8s loadbalancer will be created -> copy the dns and paste it browser

Route 53 -> hosted zones -> Create Record =>
record name ()
record type -> A
Route traffic to => Alias to application and Classic Load Balancer , 
Region -> USEast(N. Virginia)
Load Balancer -> from dropdown

try to acess amanpahtakdevops.study
```
Jump server :
```
kubectl get all -n three-tier
kubectl get ing -n three-tier

amanpahtakdevops.study (chrome)
amanpahtakdevops.study/api/tasks

add 2 or 3 tasks
```
Jump server :

Add the prometheus repo by using the below command
```
helm repo add stable https://charts.helm.sh/stable
```
Install the Prometheus

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
```
```
kubectl get all
kubectl get pvc
kubectl get deploy
kubectl get svc

kubectl edit svc prometheus-server (change CLusterIP to LoadBalancer)
kubectl get svc prometheus-server (external IP u will get. it will take sometime for getting accessed. it will be created by ccm)

aws -> load balancer -> copy DNS Name (just now created one) . paste in chrome (not accessible) (takes time-> prometheus dashboard)

check for grafana(its already installed above with prometheus) :
kubectl get svc 
kubectl get all

kubectl edit svc grafana (change CLusterIP to LoadBalancer)

aws-> LoadBalancer -> copy DNSName (new one) . Open in chrome (takes time)
kubectl get pods
kubectl get secrets
kubectl edit secrets grafana (copy admin password)
echo admin_password | base64 --decode

admin, password from above
```
```
Grafana => connections -> Add new connection -> Data Sources -> URL (prometheus_DNS_Name or (prometheus_IP:9000))

Grafana -> Dashboards -> 6417(Kubernetes Cluster) -> Load 
Grafana -> Dashboards -> 17375 (pod mem and cpu usage)-> Load
Grafana -> Dashboards
```
