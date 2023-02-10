## Simple Kubernetes Cluster (SKS)

This repo contains a Makefile for building a very minimal Kubernetes cluster in AWS.

### Requirements

* GNU Make version ```4.3```
* AWS Command Line Interface version: ```2.9.16```
* KOPS version: ```1.25.3```
* Kubernetes version: 
    * Client Version: ```v1.26.1```
    * Kustomize Version: ```v4.5.7```

SKS assumes that you own a domain in Route53 to use as the name of your Kubernetes cluster. Make sure your domain is registered (through Route53) before you continue:

```dig ns <your-domain-name>.com```
    
### Linux Installing Requirements

1. Make should come installed if you're using linux. You can check using:
    
    ```make -version```
   
2. Install AWS CLI
    
    ```curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"```
    
    ```unzip awscliv2.zip```
    
    ```sudo ./aws/install```
   
3. Configure AWS CLI
    
    ```aws configure```
    
    Enter CLI credentials for a user with sufficient permissions. 
    
    Your user will need at least the permission to create a KOPS IAM user in your account and provision the following priviledges for them:
    
   * ```arn:aws:iam::aws:policy/AmazonEC2FullAccess```
   * ```arn:aws:iam::aws:policy/AmazonRoute53FullAccess```
   * ```arn:aws:iam::aws:policy/AmazonS3FullAccess```
   * ```arn:aws:iam::aws:policy/IAMFullAccess```
   * ```arn:aws:iam::aws:policy/AmazonVPCFullAccess```
   * ```arn:aws:iam::aws:policy/AmazonSQSFullAccess```
   * ```arn:aws:iam::aws:policy/AmazonEventBridgeFullAccess```

    You can also just attach the ```arn:aws:iam::aws:policy/AdministratorAccess``` policy to your user account, but this isn't recommended for security reasons.
   
4. Install KOPS
    
    ```curl -Lo kops https://github.com/kubernetes/kops/releases/download/v1.25.3/kops-linux-amd64```
    
    ```chmod +x kops```
    
    ```sudo mv kops /usr/local/bin/kops```

5. Install Kubectl
    
    ```curl -LO https://dl.k8s.io/release/v1.26.0/bin/linux/amd64/kubectl```
    
    ```sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl```

### Configuring SKS

All configuration settings (and the only settings you need to change to run SKS) are in the .config file. To configure, do the following:

1. Set the variables for the AWS user and group which SKS will create:
    
    ```KOPS_GROUP_NAME=<some-arbitrary-name>```
    
    ```KOPS_USER_NAME=<some-arbitrary-name>```

2. Set the variable name for the AWS bucket which SKS will create to hold the state of your KOPS deployment:
    
    ```AWS_KOPS_STATE_BUCKET=<some-arbitrary-name>```

3. Set the variable name for the name you'd like for your Kubernetes cluster. It must match the domain you own in Route53; otherwise you'll get an error.
        
    ```export NAME=k8s.<your-domain-name>.com```

### Using SKS

* Start cluster:

    ```make -k create```

* Verify cluster is running:

    ```make health-check```

        Note: It may take some time for your cluster to get up-and-running. It is not abnormal for 20 minutes to pass after cluster creation for the health-checks to run without error.

* Destroy cluster: 
  
    ```make -k destroy```

### Security Note

SKS stores access keys for the users it creates in a local ```.env``` file. The ```make -k destroy``` command will remove the file; the command ```make remove-local-env``` will also remove it. However, be cautious not to share the ```.env``` file with anyone (ex. by accidentally commiting to git etc.). Anyone with the keys will be able to use your AWS account via the CLI.