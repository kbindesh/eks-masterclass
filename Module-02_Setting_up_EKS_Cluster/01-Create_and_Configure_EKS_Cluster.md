# Seting up Elastic Kubernetes Service (EKS) Cluster

In this section, we will learn how to setup an Amazon EKS cluster, a Production-grade Kubernetes distribution.

- **Step-01: Install AWS CLI**

- **Step-02: Install kubectl CLI**

- **Step-03: Install eksctl CLI**

- **Step-04: Create and configure EKS Cluster**

- **Step-05: Access the EKS cluster**

## Step-01: Set up the `AWS CLI`

- To provision resources in AWS from the command line, you need to obtain an AWS access key ID and secret key to use in the command line.
- Then you need to configure these credentials in the AWS CLI.

### 01. Download & Install AWS CLI

- For **AWS CLI install or update**, refer to this link: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

### 02. Create an access key

- Sign into the AWS Management Console.
- Navigate to **IAM** service >> **Create new User with just programmatic access (CLI)** >> Assign **AdministratorAccess** Policy.
- Choose the name of the user whose access keys you want to manage, and then choose the **Security credentials** tab.
- In the **Access keys** section, say **Create access key**
- Choose **Download .csv file** with keys.

### 03. Configure the AWS CLI

- After installing the AWS CLI, do the following steps to configure it.
- On command prompt/terminal, enter the following command:

```
aws configure
```

- Enter the requested details:
  - **AWS Access Key ID** [None]: <iam_user_access_key>
  - **AWS Secret Access Key** [None]: <iam_user_secret_access_key>
  - **Default region name** [None]: us-east-1
  - **Default output format** [None]: json

## Step-02: Install `kubectl` CLI

- For kubectl installation process, refer to this link:
  https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

- [Direct download link (Kubernetes 1.29) for Windows](https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.0/2024-01-04/bin/windows/amd64/kubectl.exe)

- After you install kubectl, you can verify its version:

  ```
  kubectl version --client
  ```

- When first installing kubectl, it isn't yet configured to communicate with any server.

```
# If you ever need to update the eks cluster config
aws eks update-kubeconfig --region region-code --name my-cluster

```

## Step-03: Install `eksctl` CLI

- For eksctl installation process, refer to this official documentation link: https://eksctl.io/installation/

## Step-04: Create an Amazon EKS cluster & Node groups

### 01. Create EKS cluster using eksctl

```
eksctl create cluster --name=labekscluster --region=us-east-1 --zones=us-east-1a,us-east-1b --without-nodegroup

# Get List of clusters
eksctl get cluster
```

:warning: **The cluster creation process can take around 10 to 15 mins to complete.**

### 02. Create EC2 Keypair for Nodegroup EC2 instances

- Create a new EC2 keypair with name as `eks-nodegrp-keypair`

### 03. Create `IAM Open ID Connect provider for EKS cluster`

- To enable Amazon EKS cluster to use AWS IAM roles for kubernetes service accounts, we must create & associate OIDC identity provider.

```
# Replace the region & cluster name value with specs
eksctl utils associate-iam-oidc-provider --region us-east-1 --cluster labekscluster --approve
```

### 04. Create `Node Group` in public subnets

```
# Create a Node group

eksctl create nodegroup --cluster=labekscluster --region=us-east-1 --name=eksdemo1-ng-public1 --node-type=t3.small --nodes=2 --nodes-min=2 --nodes-max=4 --node-volume-size=20 --ssh-access --ssh-public-key=binWinWebServerKey --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access
```

## Step-05: Verify EKS Cluster, Worker nodes and other Key Resources

### 01. Verify NodeGroup subnets | Make sure EC2 instances are in public subnet

- Go to **Services** -> **EKS** -> **labekscluster** -> **Networking**
- Click on any listed Subnets >> Click on **Route Table** Tab.
- You should see an internet route via Internet Gateway (0.0.0.0/0 -> igw-xxxxxx)

### 02. Verify EKS Cluster and NodeGroup from AWS management console

- Navigate to **Services** -> **Elastic Kubernetes Service** -> **<eks_cluster_name>**

### 03. Fetch EKS cluster details

```
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<clusterName>

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# The kubectl context should be automatically changed to new cluster
kubectl config view --minify
```

### 04. Verify Worker Node IAM Role and list of IAM Policies

- Navigate to **EC2** service >> **Instances** >> You will get to see your worker nodes
- Select any of the worker node >> **Actions** >> **Security** >> **Modify IAM Role** >> You will get to see a new role got created and assigned to your worker node (EC2 Instance).

### 05. Verify Security Group associated woth Worker Nodes (EC2 instances)

### 06. Verify Amazon CloudFormation Stacks

## Step-06: Connect to EKS cluster's worker node (EC2 instance)

## Step-07: Update the Worker node's Security group to allow ingress traffic

## References

- https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html
- https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html
- https://docs.aws.amazon.com/eks/latest/userguide/create-service-account-iam-policy-and-role.html
