

// WINDOWS INSTRUCTIONS
    
    choco install terraform --y
    // refresh ps console
    terraform version

    choco install python --y
    // refresh ps console

    pip3 install awscli --user

    pip3 install --upgrade pip

    pip3 install ansible --user

    mkdir deploy_iac_tf_ansible
    cd .\deploy_iac_tf_ansible\

    wget https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_version/ansible.cfg -OutFile ansible.cfg

    // add C:\Users\Devin\AppData\Roaming\Python\Python312\Scripts to PATH

    aws --version 

    ansible --version  // failing here, Ansible is not supported for Control nodes on Windows. 


## SETTING UP TERRAFORM

    // UBUNTU INSTRUCTIONS
        // from https://developer.hashicorp.com/terraform/install
            wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
            echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
            sudo apt update && sudo apt install terraform

    terraform version

## SETTING UP AWS CLI AND ANSIBLE

    sudu apt-get install python3-pip
    pip3 install awscli  --user
    sudu pip3 install --upgrade pip
    pip3 install ansible --user
    mkdir deploy_iac_tf_ansible
    cd deploy_iac_tf_ansible
    wget https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_version/ansible.cfg

    aws --verson // didn't work
    ansible --version // didn't work

    export PATH="/home/dlambert/.local/bin:$PATH"

    aws --version
    ansible --version

    Create Root Access Key
        Access key: AKIAYS2NRV6UME5PTIKX
        Secret access key: <redacted, see secrets file>

## AWS IAM PERMISSIONS FOR TERRAFORM 

aws configure
    Default-Region-Name: us-east-1
    output format: json

Create Policy - terraformUserPolicy
    based on json provided in resource of course

Create User - terraformUser
    Access key: AKIAYS2NRV6UL6WJIOJE
    Secret access key: <redacted, see secrets file>

Create Role - EC2TFRole
    Applied terraformUserPolicy to EC2


# Set up VSCode to connect to Ubuntu via WSL 
    realized I need to setup vscode so that I can better manipulate files in ubuntu

    // Source: https://code.visualstudio.com/docs/remote/wsl-tutorial


    cant connect to ubuntu via vscode

    // Source: https://code.visualstudio.com/docs/remote/troubleshooting#_wsl-tips

    sudo apt-get update && sudo apt-get install wget ca-certificates

    realized I was connecting to docker, not ubuntu, needed to specify which distro in prompt, connected now.

## PERSISTING TERRAFORM STATE TO S3 BACKEND

    aws s3api create-bucket --bucket terraformstatebucket52890
        logged into aws, can see s3
    code backend.tf
        added configuration code from video
    terraform init
    terraform fmt


## SETTING UP MULTIPLE AWS PROVIDERS IN TERRAFORM

    code variables.tf    
        adding variables code from video
    terraform fmt
    code providers.tf
        adding providers code from video
    


## Setting up Git and reconciling windows and ubuntu folders

    had to setup new personal access token in github 



## NETWORK SETUP PART 1: DEPLOYING VPCs, INTERNET GWs, AND SUBNETS

    code networks.tf
        adding network code from video
    terraform fmt
    terraform validate
    terraform plan
    terraform apply // holding on this for now

### Checking out costs
    Found this plugin - https://registry.terraform.io/modules/terraform-aws-modules/pricing/aws/latest
    Which led me to this online tool - https://terraform-cost-estimation.com/

    I was able to use this command, to upload json from terraform plan 
    `terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json`


## NETWORK SETUP PART 2: DEPLOYING MULTI-REGION VPC Peering

    updating network.tf
        adding network code from video for peering
            intellisense is showing error for two "route" nodes in one of the aws_route_tables
        terraform fmt
            still complaining, realized route takes [] instead of {}, so flattend the second route into first one as array, fmt passing now.
        terraform validate
            complaining about other aws_route_table with single route, wrapping in [].
            Rerun fmt and validate, found a typo with _ instead of a -.
            Rerun fmt, complaining about missing attributes in my route elements, doing some research. 
                fixed first issue by correcting syntax for single route. 
                fixed other issue by adding dafault attributes, running fmt and validate are good now. 

        terraform plan
        terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json
        copying plan.tfplan into https://terraform-cost-estimation.com/
            estimate is $0

        terraform apply
        
