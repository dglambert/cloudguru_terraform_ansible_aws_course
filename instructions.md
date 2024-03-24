# Intro

    Following cloud guru course - Deploying to AWS with Terraform and Ansible 
        - https://www.pluralsight.com/cloud-guru/courses/deploying-to-aws-with-terraform-and-ansible

    You can find my notes below. 

    AWS free tiers - https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all



## SETTING UP TERRAFORM

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
        
## NETWORK SETUP PART 3: DEPLOYING SECURITY GROUPS

    code security_groups.tf
        adding security groups code from video
    terraform fmt
    terraform validate
    terraform plan
    terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json
        copying plan.tfplan into https://terraform-cost-estimation.com/
            estimate is $0
    terraform apply --auto-approve

    ### fixing typo west-2 to west-1
        corrected description
        terraform fmt
        terraform validate
        terraform plan
        terraform apply --auto-approve
        validated in AWS, description is now correct          
    
    ## fixing typo oregon to ncalifornia 
        updated both definition and references in network.tf of vpc_master_oregon to vpc_master_ncalifornia
        terraform fmt
        terraform validate
        terraform plan
        terraform apply --auto-approve
            unsuccessul after several minutes
            removing remainning resource references to oregon
            terraform fmt
            terraform validate
            terraform plan
            terraform apply --auto-approve
            giving up, going forward with terraform destroy
    
    terraform destroy
    check aws for resources, see them removed

    terraform plan
    terraform apply
    check aws see all the resource created
    terraform destroy 


## APP VM DEPLOYMENT PART 1: USING DATA SOURCE (SSM PARAMETER STORE) TO FETCH AMI IDs

    code instances.tf
        copy code in from video

    terraform init
    terraform fmt
    terraform validate
    terraform plan
    terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json
        copying plan.tfplan into https://terraform-cost-estimation.com/
            estimate is $0
    
    ### how come video has 18 resource in plan, but I have 16
        x  aws_internet_gateway.igw: 
        x  aws_internet_gateway.igw_ncalifornia: 

        x  aws_main_route_table_association.set-master-default-rt-aws_main_route_table_association: 
        x  aws_main_route_table_association.set-worker-default-rt-ssoc: 

        x  aws_route_table.internet_route: 
        x  aws_route_table.internet_route_ncalifornia: 

        x  aws_security_group.lb-sg: 
        x  aws_security_group.jenkins-sg-ncalifornia: 
        x  aws_security_group.jenkins-sg: 

        x  aws_subnet.subnet_1: 
        x  aws_subnet.subnet_2: 
        x  aws_subnet.subnet_1_ncalifornia: 

        x  aws_vpc.vpc_master: 
        x  aws_vpc.vpc_master_ncalifornia: 

        x  aws_vpc_peering_connection.useast1-uswest2: 
        x  aws_vpc_peering_connection_accepter.accept_peering: 


        MISSING aws_route_table_association.internet_association
        MISSING aws_route_table_association.internet_association_ncalifornia

        I reviewed videos, but can't find where this should be coming from. Moving on for now.


    terraform apply

    // using following command to see actual value used from parameters

    aws s3 cp s3://terraformstatebucket52890/terraformstatefile .

    terraform destroy
    


## APP VM DEPLOYMENT PART 2: DEPLOYING KEY PAIRS FOR APP NODES

    ssh-keygen -t rsa
        Your identification has been saved in /home/dlambert/.ssh/id_rsa
        Your public key has been saved in /home/dlambert/.ssh/id_rsa.pub

    open instances.tf file
        copy code in from video

    terraform fmt
    terraform validate
    terraform plan
    terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json
        copying plan.tfplan into https://terraform-cost-estimation.com/
            estimate is $0
    terraform apply


## APP VM DEPLOYMENT PART 3: DEPLOYING JENKINS MASTER AND WORKER INSTANCES


    code instances.tf
        copy code in from video to existing file 
    code variables.tf
        copy code in from video to existing file
    code outputs.tf
        copy code in from video to new file



    fixing typo in set-master-default-rt-assoc & set-worker-default-rt-assoc

    terraform destroy
        ran into issues, had to do a git stash, then terraform destroy, as the new resources were confusing the destroy command
    terraform fmt
    terraform validate
    terraform plan
        error in jenkins-worker-ncalifornia
            was missing 'id', this was same error when using destroy, I probably could have fixed it and destroy may have worked without having to do the git stash
        2nd error in jenkins-worker-ncalifornia
            was missing 'value' this time
        rerun terraform plan, no issues

    terraform plan -out=plan.tfplan && terraform show -json plan.tfplan > plan.json
        copying plan.tfplan into https://terraform-cost-estimation.com/
            Total estimated costs: USD 0.02 per hour, or USD 16.42 per month.


    time terraform apply 
        error in jenkins-worker-ncalifornia, security group and subnet belong to different networks
            looks like I missed defining vpc_id in jenkins-sg-ncalifornia

            terraform fmt
            terraform validate
            time terraform apply
            no issues

    Jenkins-Main-Node-Public-IP = "<redacted to secrets file>"
    Jenkins-Worker-Public-IPs = {
    "i-0f099cec5d6ccbbdb" = "<redacted to secrets file>"
    }

    ssh ec2-user@<redacted to secrets file>
    succesfully connected to main node
    exit

    ssh ec2-user@<redacted to secrets file>
    succesfully connect to worker node
    exit

    tried to get windows terminal ssh setup, but gave up for now.


## CONFIGURING TERRAFORM PROVISIONERS FOR CONFIG MANAGEMENT VIA ANSIBLE

    mkdir ansible_templates
    cd ansible_templates
    code jenkins-master-sample.yml
        copy code in from video to new file

    code jenkins-worker-sample.yml
        copy code in from video to new file

    cd ..
    code ansible.cfg
        copy code in from video to existing file

    cd ansible_templates
    mkdir inventory_aws
    cd inventory_aws
    get yml url from resource section in course - https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/inventory_aws/tf_aws_ec2.yml

    wget -c https://raw.githubusercontent.com/linuxacademy/content-deploying-to-aws-ansible-terraform/master/aws_la_cloudplayground_multiple_workers_version/ansible_templates/inventory_aws/tf_aws_ec2.yml

    code tf_aws_ec2.tf
        open downloaded file
        updating supported regions from west-2 to west-1

    pip3 install boto3 --user

        got an error but I think its just a warning, moving on for now

            Installing collected packages: botocore, boto3
                Attempting uninstall: botocore
                    Found existing installation: botocore 1.34.64
                    Uninstalling botocore-1.34.64:
                    Successfully uninstalled botocore-1.34.64
                ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
                awscli 1.32.64 requires botocore==1.34.64, but you have botocore 1.34.67 which is incompatible.
                Successfully installed boto3-1.34.67 botocore-1.34.67

        cd ..
        cd ..

        code instances.tf
            open existing file and copy code in from video

        terraform fmt
        terraform validate
        terraform plan
        terraform apply
            no changes, I think I need to destroy 
        
        terraform destroy 
        terraform apply

        copy new public ips to secrets file

            Jenkins-Main-Node-Public-IP = "<redacted to secrets file>"
            Jenkins-Worker-Public-IPs = {
            "i-06ee1cbcb37712ad3" = "<redacted to secrets file>"
            }

        ssh ec2-user@<redacted to secrets file>
        succesfully connect to worker node
        jq command not working
        exit

        ssh ec2-user@<redacted to secrets file>
        succesfully connect to master node
        git command not working
        exit

        found issue i think, accidentally used .yaml instead of .yml in ansible.cfg
            terraform destroy
            terraform apply
                still referencing .yaml also saw an error couldn't ssh to worker node, ignoring for now. trying again
                    terraform destroy
                    terraform fmt
                    terraform validate
                    terraform plan
                    terraform apply

        yaml issue seems to be resolve now, as I think master was successfully provisioned.
            testing ssh into master 

            ssh ec2-user@<redacted to secrets file>
            succesfully connect to master node
            git command IS working
            exit

        trying to enable trace logging 
            export TF_LOG="TRACE"
            
            terraform destroy 
                seeing trace logs now
            terraform apply 

            turning on log file, as the terminal is getting truncated to quickly due to the size
                export TF_LOG_PATH="tmp/terraform.log"

                echo "export TF_LOG_PATH=./logs/terraform.log" >> .bashrc

            going through and replacing all ncalifornia refrences to oregon and west-1 to west-2
                terraform fmt
                terraform validate
                terraform plan
                export TF_LOG="TRACE"
                terraform apply --auto-approve
                    no error messages, but could not match host pattern on worker node, so provisioning was skipped


            unset TF_LOG
            adding log_path = ansible.log to ansible.cfg

            adding -v ansible-playbook command to ec2 jenkins-worker-oregon provisioner

            reinstalling boto3, not sure if the error I got earlier could be causing issue with awscli? 
                --- https://stackoverflow.com/questions/51911075/how-to-check-awscli-and-compatible-botocore-package-is-installed
                --- https://www.activestate.com/resources/quick-reads/how-to-list-installed-python-packages/
                --- https://stackoverflow.com/questions/5226311/installing-specific-package-version-with-pip
                pip3 install boto3==1.34.64 --user --force-reinstall
                    complaining about botocore
                        upgrading awscli, uninstalling boto3 and botocore and reinstalling
                            pip3 install awscli --upgrade --user
                                # removed v 1.32.64, installed 1.32.69
                            pip3 uninstall botocore
                            pip3 uninstall boto3

                            pip3 install botocore --user
                                installed 1.34.69
                            pip3 install boto3 --user
                                installed 1.34.69
            
            terraform destroy
            terraform plan
            clearing ansible.log and terraform log 
            export TF_LOG="DEBUG"
            terraform apply --auto-approve


            works locally
            ansible-playbook --extra-vars 'passed_in_hosts=tag_Name_jenkins_master_tf' ansible_templates/jenkins-master-sample.yml

            doesnt work locally
            ansible-playbook -vvv --extra-vars 'passed_in_hosts=tag_Name_jenkins_worker_tf_1' ansible_templates/jenkins-worker-sample.yml


            https://developer.hashicorp.com/terraform/language/resources/provisioners/local-exec
            realized explicit call to ansible-playbook in provisioner block, going to focus on this call. 
            may try adding following block to provisioner block to see if they execute, I suspect terraform/provisioner block is not the problem though 
                echo profile: ${var.profile}
                echo region: ${var.region-worker}
                echo instance-ids" ${self.id}
                echo tags.Name: ${self.tags.Name}



            after tooling around for a bit, I realized that terraform provisioner was probably not the problem, but the call the ansible-playbook, so I pulled it out and ran it from terminal locally. 
            I am able to run master playbook, but worker fails, so this indicates an issue with ansible.
            upon further investigation, I am investigating the Ansible AWS Dynamic Inventory Plugin
                - https://docs.ansible.com/ansible/latest/collections/amazon/aws/docsite/aws_ec2_guide.html
                - https://github.com/ansible/terraform-provider-ansible/issues/94

                ansible-inventory -i ansible_templates/inventory_aws/tf_aws_ec2.yml  --graph 
                    I can see when running this command, the corret ip is found when using tag tag_Name_jenkins_worker_tf_1



                I also noticed when logging into aws, that the worker tag was set to jenkins_worker_tf, i manually added _1 and then after a few minutes I am no longer having the issue of host name not found. 
                A new issue has appeared which is:     "msg": "Failed to connect to the host via ssh: dlambert@54.213.120.67: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).",

                adding -vvv to ansible-playbook command
                https://stackoverflow.com/questions/68199480/ssh-permission-denied-for-ec2-using-ansible
                ansible-playbook -vvv --extra-vars 'passed_in_hosts=tag_Name_jenkins_worker_tf_1' ansible_templates/jenkins-worker-sample.yml


                Noticed in the logs for worker 'ESTABLISH SSH CONNECTION FOR USER: None', but when looking at logs for master I see 'ESTABLISH SSH CONNECTION FOR USER: ec2-user'

                    adding -e "ansible_user=ec2-user" fixed it.
                        https://stackoverflow.com/questions/35024576/establish-ssh-connection-for-user-none-when-specify-a-single-host-on-the-comman
                        ansible-playbook -vvv --extra-vars 'passed_in_hosts=tag_Name_jenkins_worker_tf_1' ansible_templates/jenkins-worker-sample.yml -e "ansible_user=ec2-user" 
                
                    Going to hold off on this till I fix the missing _tf_1 issue from worker instances

                terraform destroy --auto-approve
                terraform fmt
                terraform validate
                terraform plan
                clear logs
                terraform apply --auto-approve

                it worked this time, not sure how, but tagging is correct, and worker ansible playbook executed succesfully


                    ssh ec2-user@<redacted to secrets file>
                    succesfully connect to worker node
                    jq command working
                    exit

                    ssh ec2-user@<redacted to secrets file>
                    succesfully connect to master node
                    git command working
                    exit

                export TF_LOG="INFO"


## LOAD BALANCING: CREATING AN ALB AND ROUTING TRAFFIC TO EC2 APP NODE

    code alb.tf
        copy in code from video to new file

    code variables.tf
        add webserver-port variable

    code security_groups.tf
        update hardcoded webserver port to use variable

    code alb.tf
        copy in code from video to existing file

    code outputs.tf
        add new output for lb dns name

    code ansible_templates/jenkins-master-sample.tf
        replacing git with install httpd/Apache, and start Apache.

    terraform fmt
    terraform validate
    terraform plan
    terraform apply

        copy and paste dns from output into browser
            502 bad gateway, I didn't anticipate this to work because the video had a destroy infrastructure to start, and I don't think our ansible-playbook changes trigger changes in terraform. 

        terraform destroy
            i'm going to wait 5 minutes before applying again, I think some of my issue could be caused by back to back deployments
        
        terraform apply

            copy and pase dns from output into browser
                works this time



## PUTTING IT ALL BEHIND DNS: SETTING UP HTTPS AND ROUTE 53 RECORD

    purchased grocerysage.com on aws route53

    aws route53 list-hosted-zones

        copy name => grocerysage.com.

    code variables.tf
        add new variable dns-name with copied value from above

    code dns.tf
        copy in code from video to new file

    code acm.tf
        copy in code from video to new file

    code dns.tf
        add cert validation
    
    code acm.tf
        add cert

    code alb.tf
        enable redirect on port 80 listener
        add https listener

    code dns.tf
        add route53 record

    code outputs.tf
        add url 


    terraform fmt
    terraform validate
    terraform plan
    terraform apply
        failed

    terraform destroy
    terraform apply

    copy url output and paste into browser
        its working 

    terraform destroy --auto-approve

    
## TERRAFORM OUTPUTS AND TERRAFORM GRAPH

    terraform apply --auto-approve


    terraform output

    cat outputs.tf
        grabbing url

    terraform output url

    terraform state list

    terraform console
        aws_instance.jenkins-master
        aws_instance.jenkins-master.private_ip

        join(".", ["terraform", "acg"])

    terraform graph > tf.dot
    code tf.dot

    sudo apt install graphviz

    cat tf.dot | dot -T png -o tf.png

    code tf.png 



## ANSIBLE PLAYBOOKS AND SYNTAX CHECKING

    cd ansible_templates
    code sample.yml
        copy in code from video

    fixing vscode error - "Ansible-lint is not available. Kindly check the path or disable validation using ansible-lint"
        ansible-lint --version
            not recognzed 
        pip3 install ansible-lint
        
        ansible-lint --version
            working now
        reload vs code, no errors. 

    ansible-playbook --syntax-check sample.yml

    echo $? 
        checks status code of last command - 0 
    
    code sample.yml
        make a typo
    
    ansible-playbook --syntax-check sample.yml
        throws error

    echo $?
        returns 4


    
## BUILDING PLAYBOOK FOR JENKINS MASTER SETUP


    cd ansible_templates

    code install_jenkins_msater.yml
        copy in code from video
    
     ansible-playbook --syntax-check -e "passed_in_hosts=localhost" install_jenkins_master.yml

     echo $?
        returns 0



## BUILDING PLAYBOOK FOR JENKINS WORKER SETUP

    cd ansible_templates

    code install_jenkins_worker.yml
        copy in code from video 

     ansible-playbook --syntax-check -e "passed_in_hosts=localhost" install_jenkins_worker.yml
    
     echo $?
        returns 0



## BUILDING JINJA2 TEMPLATES FOR ANSIBLE 

    cd ansible_templates

    code node.j2 
        copy in code from resources tab in cloudguru

    code cred-privkey.j2
        copy in code from resources tab in cloudguru

    
    



















