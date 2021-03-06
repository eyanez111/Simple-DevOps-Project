# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```

1. Install kubectl on ubuntu instance
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access

1. Attach IAM role to ubuntu instance
   ```sh
   # Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough
   aws configure
    ```

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   ```sh
   Routeh53 --> hosted zones --> created hosted zone  
   Domain Name: mbaclassesonline.com
   Type: Private hosted zone for Amzon VPC (public if you are using your own domain)
   ```

1. create an S3 bucket
   ```sh
    aws s3 mb s3://demo.k8.mbaclassesonline.com
   ```
1. Expose environment variable:
   ```sh
    export KOPS_STATE_STORE=s3://demo.k8.mbaclassesonline.com
   ```

1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
   kops create cluster --cloud=aws --zones=us-east-2a --name=demo.k8.mbaclassesonline.com --dns-zone=mbaclassesonline.com --dns private --ssh-public-key
    ```

1. Create kubernetes cluser
    ```sh
    kops update cluster demo.k8.mbaclassesonline.com --yes
    ```


1. Validate your cluster
     ```sh
      kops validate cluster
    ```

1. To list nodes
   ```sh
   kubectl get nodes
   ```

1. To edit cluster nodes
   ```sh
   kops edit ig --name=demo.k8.mbaclassesonline.com nodes
   ```
   
1. To edit master node
   ```sh
    kops edit ig --name=demo.k8.mbaclassesonline.com master-us-east-1d
   ```
   
1. TO roll an update in the cluster when is already running
   ```sh
    kops update cluster --name demo.k8.mbaclassesonline.com --yes
    kops rolling-update cluster
    kops rolling-update cluster --cloudonly --yes
   ```

1. login to master node 
    ```sh
     ssh -i ~/.ssh/id_rsa admin@api.demo.k8.mbaclassesonline.com
    ```   
 
 1. If ssh key was not added during build
    ```sh
    kops delete secret --name demo.k8.mbaclassesonline.co sshpublickey admin
    kops create secret --name demo.k8.mbaclassesonline.co sshpublickey admin -i ~/.ssh/newkey.pub
    kops update cluster --yes to reconfigure the auto-scaling groups
    kops rolling-update cluster --name demo.k8.mbaclassesonline.co --yes to immediately roll all the machines so they have the new key (optional)

1. To delete cluster
    ```sh
     kops delete cluster demo.k8.mbaclassesonline.com --yes
    ```
   
#### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl run sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl run simple-devops-project --image=yankils/simple-devops-image --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```
