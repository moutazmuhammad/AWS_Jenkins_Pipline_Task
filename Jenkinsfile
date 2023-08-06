pipeline {
  agent any
  
    tools {
       terraform 'terraform'
    }

    stages {
        stage('terraform init'){
          steps{
             withAWS(credentials: 'awsCredential') {
                sh 'terraform -chdir=./terraform init'
                }
          }
        }

        stage('terraform plan'){
          steps{
             withAWS(credentials: 'awsCredential') {
               sh 'terraform -chdir=./terraform plan --var-file dev.tfvars'
             }
          }
        }

        stage('terraform build and using outputs'){
          steps{
             withAWS(credentials: 'awsCredential') {
                sh 'terraform -chdir=./terraform apply --auto-approve --var-file dev.tfvars '
                }
          }
        }

        stage('Configer IPs of public and private ec2 for inventory on master node'){
          steps{
             withAWS(credentials: 'awsCredential') {
sh 'chmod 400 ~/.ssh/privatekey.pem'

sh """
echo "Host *
    Port 22
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null
    ServerAliveInterval 60
    ServerAliveCountMax 30
Host bastion
    HostName `terraform -chdir=./terraform output -raw Public_Instace_IP`
    User ubuntu
    IdentityFile ~/.ssh/privatekey.pem
" > ~/.ssh/config 
"""

                sh 'chmod +x edit_inventory.sh'
                sh './edit_inventory.sh'
            }
          }
        }

        stage('Run Ansible'){
          steps{
                    sh ' ansible-playbook -i /var/jenkins_home/workspace/terraform_apply/ansible/inventory /var/jenkins_home/workspace/terraform_apply/ansible/playbook.yaml'
          }
        }

    }
    
}


