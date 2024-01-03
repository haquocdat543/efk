# efk
This is a demonstration of monitoring on EKS cluster using EFK stack
## Prerequisites
* [git](https://git-scm.com/downloads)
* [awscli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
* [config-profile](https://docs.aws.amazon.com/cli/latest/reference/configure/)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
# Start
## 1. Clone repo
```
git clone https://github.com/haquocdat543/aws-cicd.git
cd aws-cicd
```
## 2. Initiaize
### 1. k8s cluster
```
aws cloudformation deploy --stack-name k8s --template-file k8s.yaml --capabilities CAPABILITY_IAM
```
### 2. pipeline
```
aws cloudformation deploy --stack-name pipeline --template-file pipeline.yaml
```
## 4. Get output
```
aws cloudformation describe-stacks --query Stacks[].Outputs[*].[OutputKey,OutputValue] --output text
```
## 3. Install Role and Addon
### 1. OIDC
Replace `$region` to your custom
```
eksctl utils associate-iam-oidc-provider --cluster=EKSCluster --region $region --approve
```
### 2. Create EBS CNI Role
Replace `$cluster` to your custom
```
eksctl create iamserviceaccount --name ebs-csi-controller-sa --namespace kube-system --cluster EKSCluster --role-name AmazonEKS_EBS_CSI_DriverRole --role-only --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy --approve 
```
### 3. Add addons
Replace and `YOUR_AWS_ACCOUNT` to your custom
```
aws eks create-addon --cluster-name EKSCluster --addon-name aws-ebs-csi-driver --service-account-role-arn arn:aws:iam::YOUR_AWS_ACCOUNT_ID:role/AmazonEKS_EBS_CSI_DriverRole
```
## 4. Deploy k8s resources
### 1. Namespace
```
kubectl apply -f namespace.yaml
```
### 2. Elasticseach
service:
```
kubectl apply -f elasticsearch-svc.yaml
```
statefulset
```
kubectl apply -f elasticsearch.yaml
```

### 3. Kibana
service:
```
kubectl apply -f kibana-svc.yaml
```
statefulset
```
kubectl apply -f kibana.yaml
```
### 4. Fluentd
```
kubectl apply -f fluentd.yaml
```
### 5. Access Kibana
```
kubectl config set-context --current --namespace kube-logging
kubectl get svc
```
copy kibana loadbalancer dns and paste it in your browser: `<dns>:5601`


## 5. Destroy
### 1. Delete k8s resources
```
kubectl delete namespace kube-logging
```
### 2. Delete cloudformation stack
```
aws cloudformation delete-stack --stack-name k8s
```
```
aws cloudformation delete-stack --stack-name eksctl-EKSCluster-addon-iamserviceaccount-kube-system-ebs-csi-controller-sa
```
