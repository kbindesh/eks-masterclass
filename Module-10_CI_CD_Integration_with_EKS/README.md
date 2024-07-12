# CI/CD Pipeline for Amazon EKS Cluster using AWS DevOps services

## Step-01: Introduction to DevOps

- Version Control System (VCS)
- Continuous Integration (CI)
- Build Artifacts
- Continous Delivery (CD)

## Step-02: Introduction to AWS DevOps services stack

- AWS CodeCommit
- AWS CodeBuild
- AWS CodePipeline

## Step-03: Create an Amazon EKS cluster

Kindly refer this link for step-by-step process for setting-up Amazon EKS cluster: </br></br>
https://github.com/kbindesh/eks-masterclass/blob/main/Module-02_Setting_up_EKS_Cluster/01-Create_and_Configure_EKS_Cluster.md
</br>[Press ctrl + click to open the link in a new tab]

## Step-04: Create an Amazon ECR Repository

- Sign-in to AWS Management console >> **Services** >> **Elastic Container Registry**
- From the left-side panel, under **Private registry** section, click **Respositories** >> **Create Repository**
- **Visibility Settings**: Private
- **Repository name**: eks-devops-nginx
- **Tag Immutability**: Enable
- **Scan On Push**: Enable
- Click on **Create Repository**

  ```
  # Sample ECR Repository URI will look like this
  154511248558.dkr.ecr.us-east-1.amazonaws.com/eks-devops-nginx
  ```

- Verify the pushed code on from AWS Management console.

## Step-05: Create CodeCommit Repository and check-in the code

### Step-5.1: Create an AWS CodeCommit Repository

- Navigate to **Services** >> **Developer Tools** >> **CodeCommit**
- From the left-side panel, under **Source** section >> Select **Repositories**.
- Click on **Create Repository** button.
- **Repository name**: eks-devops-nginx
- **Description**: This repos has bindesh's demo k8s app components.
- Leave rest of the setting to defaults >> **Create**.

### Step-5.2: Create an IAM user for CodeCommit Authentication

- Navigate to **IAM** service >> **Users** >> Click on **Create user**
- **User name**: codecommituser
- **Provide user access to the AWS Management Console**: Unchecked
- **Permission option**: Attach policies directly
- **Permissions policies**: AdministratorAccess
- Review the entered details and click **Create** button.

### Step-5.3: Develop the application and manifests

- Launch the Visual Studio Code (or any other IDE) and open the
- Clone the CodeCommit repository on your local system by running following command:

  ```
  git clone <codecommit_git_url_here>
  ```

- Get inside the cloned repo directory/folder:

  ```
  cd <your_cloned_git_repo_name>
  ```

- Now, develop the following application files:

  1. Dockerfile
  2. Buildspec.yml
  3. Kubernetes Manifests
  4. Application Code
  5. Dependencies (optional)

- You may clone this github repo to get all the above listed files.

### Step-5.4: Commit the changes locally

- Commit the changes in your local system by running following commands:

  ```
  # Stage the changes
  git add .

  # Check the local git repo status
  git status

  # Commit the changes
  git commit -m "App Version 1.0"
  ```

### Step-5.5: Push the changes to remote repository (GitHub)

- Push the code to remote CodeCommit repository:

  ```
  # Rename the branch to main
  git branch -M main

  # Push the changes to your CodeCommit repo
  git push -u origin main

  # To check the remote origin URL
  git remote -v
  ```

## Step-06: Create STS Assume IAM Role for CodeBuild to interact with AWS EKS

- In this lab, we will use AWS CodeBuild to deploy changes to our Kubernetes cluster (EKS).
- This requires an AWS IAM role capable of interacting with the EKS cluster.
- Here, we will create an IAM role and add an inline policy **EKS:Describe** that we will use in the CodeBuild stage to interact with the EKS cluster via kubectl.

### Step-6.1: Connect to your AWS Account using AWS CLI

- Pre-requisite: AWS CLI
- Launch Command Prompt/PowerShell/Terminal on your system >> Run following command to connect to your AWS Account:

  ```
  aws configure

  # Enter the requested details like access keys, secret keys, default region and format

  AWS Access Key ID : ****************YLCE
  AWS Secret Access Key: ****************P8pf
  Default region name: us-east-1
  Default output format: json
  ```

### Step-6.2: Create required IAM components (IAM Role, Policies)

```
# Export your Account ID
export ACCOUNT_ID=<YOUR_AWS_ACCOUNT_ID_HERE>

# Set Trust Policy
TRUST="{ \"Version\": \"2012-10-17\", \"Statement\": [ { \"Effect\": \"Allow\", \"Principal\": { \"AWS\": \"arn:aws:iam::${ACCOUNT_ID}:root\" }, \"Action\": \"sts:AssumeRole\" } ] }"

# Verify inside Trust policy, your account id got replacd
echo $TRUST

# Create IAM Role for CodeBuild to Interact with EKS service
aws iam create-role --role-name EksCodeBuildKubectlRole --assume-role-policy-document "$TRUST" --output text --query 'Role.Arn'

# Define Inline Policy with eks Describe permission in a file iam-eks-describe-policy
echo '{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow", "Action": "eks:Describe*", "Resource": "*" } ] }' > /tmp/iam-eks-describe-policy

# Associate Inline Policy to our newly created IAM Role
aws iam put-role-policy --role-name EksCodeBuildKubectlRole --policy-name eks-describe --policy-document file:///tmp/iam-eks-describe-policy
```

- Verify the above create role and policy from AWS Management Console >> IAM

## Step-07: Update EKS Cluster aws-auth ConfigMap with a custom IAM Role

- In this step, we will assign an IAM role to the `aws-auth ConfigMap` for the EKS cluster.
- Amazon EKS `aws-auth` configMap will allow `kubectl` in the CodeBuild stage of the pipeline to interact with the EKS cluster via IAM role that we created on **Step-5.2**.

  ```
  # Verify the present aws-auth configmap (before change)
  kubectl get configmap aws-auth -o yaml -n kube-system

  # Export your AWS Account ID
  export ACCOUNT_ID=154511248558

  # Store IAM role name in a variable (i.e ROLE)
  ROLE=" - rolearn: arn:aws:iam::$ACCOUNT_ID:role/EksCodeBuildKubectlRole\n username: build\n groups:\n - system:masters"

  # Get current aws-auth configMap data and attach new role info to it
  kubectl get -n kube-system configmap/aws-auth -o yaml | awk "/mapRoles: \|/{print;print \"$ROLE\";next}1" > /tmp/aws-auth-patch.yml

  # Patch the aws-auth configmap with new role
  kubectl patch configmap/aws-auth -n kube-system --patch "$(cat /tmp/aws-auth-patch.yml)"

  # Verify the aws-auth configmap after updating the IAM role
  kubectl get configmap aws-auth -o yaml -n kube-system
  ```

## Step-08: Configure the Build process

### Step-8.1: Review the `buildspec.yml` file

### Step-8.2: Create a new CodeBuild Project-

- Sign-in to **AWS Management console** >> **Services** >> **Developer Tools** >> **CodeBuild**.
- From the left-side panel, select **Build Projects** >> click **Create Project** button.
- **Project name**: <name_of_the_build_project>>
- **Source 1 - Primary**: AWS CodeCommit
- **Provisioning model**: On-Demand
- **Environment image**: Managed Image
- **Compute**: EC2
- **Operating System**: Amazon Linux
- **Runtime**: Standard
- **Image**: aws/codebuild/amazonlinux2-x86_64-standard:5.0
- **Image Version**: Always use the latest version for this runtime
- **Service role**: New Service Role
- **Role name**: <assign_new_name_or_leave_to_default>
- **Additional Configurations**
  - **Environment variables**
    - **REPOSITORY_URI** = 154511248558.dkr.ecr.us-east-1.amazonaws.com/eks-devops-nginx
    - **EKS_KUBECTL_ROLE_ARN** = arn:aws:iam::154511248558:role/EksCodeBuildKubectlRole
    - **EKS_CLUSTER_NAME** = <eks_cluster_name>

### Step-8.3: Trigger the build manually (to test)

## Step-09: Create and Configure AWS CodePipeline

### Step-9.1: Create a new AWS CodePipeline

- Navigate to **AWS Management Console** >> **Services** >> **CodePipeline** -> **Create Pipeline**

- **Pipeline Settings**
  - **Pipeline Name**: eks-devops-pipe
  - **Pipeline type**: Queued (Pipeline type V2 required)
  - **Service Role**: New Service Role (leave to defaults)
  - **Role Name**: Auto-populated
  - Rest all leave to defaults >> click **Next**
- **Add source stage**
  - **Source Provider**: AWS CodeCommit
  - **Repository Name**: eks-devops-nginx
  - **Branch Name**: master
  - **Change Detection Options**: AWS CloudWatch Events (recommended)
  - **Output artifact format**: CodePipeline default
- **Add Build stage**
  - **Build Provider**: AWS CodeBuild
  - **Region**: US East (N. Virginia)
  - **Project Name**: <select_the_build_project_name>
  - **Build type**: Single build
  - **Logs**
    - Group Name: eks-deveops-cb-pipe
    - Stream Name: <cloudwatch_stream_name>
  - Click **Next**
- **Add Deploy stage**
  - Click on **Skip Deploy Stage**
- **Review**
  - Review and click on **Create Pipeline**

## Step-10: Updae CodeBuild IAM Role to have ECR full access

- Update the CodeBuild Role to have access to ECR to upload images built by codeBuild.
  - **Role Name**: codebuild-eks-devops-cb-for-pipe-service-role
  - **Policy Name**: AmazonEC2ContainerRegistryFullAccess

## Step-11: Trigger the AWS CodePipeline

- Switch to your local system >> Project repo.
- Make some changes to any file >> Stage & Commit the changes >> Push the change to CodeCommit repo.

  ```
  git add .
  git status
  git commit -am "V2 Deployment"
  git push -u origin master
  ```

## Step-12: Verify the Docker image upload in Amazon ECR

- Verify CodeBuild Logs
- A new image should be uploaded to ECR, verify the ECR with new docker image tag date time.
- Build will fail again at post build stage at STS Assume role section. Let's fix that in next step.

## Step-13: Update CodeBuild Role to have access to STS Assume Role (created in step#6.2)

- Build will fail due to CodeBuild's insufficient access permissions to EKS cluster. In other words, CodeBuild do not have access to perform updates in EKS Cluster.
- Create STS Assume Policy and assign to CodeBuild role `codebuild-eks-devops-cb-for-pipe-service-role`

### Step-13.1: Create STS Assume Role Policy

- Go to Services IAM -> Policies -> Create Policy
- In **Visual Editor Tab**
- **Service**: STS
- **Actions**: Under Write - Select `AssumeRole`
- **Resources**: Specific

  - Add ARN
  - **Specify ARN for Role**: arn:aws:iam::154511248558:role/EksCodeBuildKubectlRole [Refer Step#6.2]
  - Click **Add**

- Click on **Review Policy**
- **Name**: eks-codebuild-sts-assume-role
- **Description**: CodeBuild to interact with EKS cluster to perform changes
- Click on **Create Policy**

### Step-13.2: Associate Policy to CodeBuild Role

- **Role Name**: codebuild-eks-devops-cb-for-pipe-service-role
- **Policy to be associated**: `eks-codebuild-sts-assume-role`

## Step-14: Trigger the AWS CodePipeline

- Make changes to index.html (Update as V3)
- Commit the changes to local git repository and push to codeCommit Repository
- Monitor the codePipeline
- Test by accessing the static html page

```
git status
git commit -am "V3 Deployment"
git push
```

- Verify CodeBuild Logs

## Step-15: Clean up the resources

- Delete All kubernetes Objects in EKS Cluster

  ```
  kubectl delete -f kube-manifests/
  ```

- Delete Pipeline
- Delete CodeBuild Project
- Delete CodeCommit Repository
- Delete Roles and Policies created

## References

- https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html
- https://github.com/aws/aws-codebuild-docker-images/blob/master/al2/x86_64/standard/3.0/Dockerfile
- https://docs.aws.amazon.com/cli/latest/reference/sts/assume-role.html
- https://docs.aws.amazon.com/IAM/latest/UserGuide/troubleshoot_roles.html
