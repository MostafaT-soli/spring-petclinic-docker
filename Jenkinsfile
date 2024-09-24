pipeline {
  agent { kubernetes {
    inheritFrom 'Main'
    yaml'''
apiVersion: v1
kind: Pod
metadata:
  name: ansible-terraform-pod
spec:
  imagePullSecrets:
    - name: myregistrykey
  containers:
    - name: ansible-terraform-container
      image: matrek/ta:latest  # Replace with the name of your Docker image
      command: ["/bin/bash", "-c"]
      args: ["sleep 1d"]  # Command to sleep for a day
    - name: java
      image: eclipse-temurin:21-jdk-jammy  # Replace with the name of your Docker image
      command: ["/bin/bash", "-c"]
      args: ["sleep 1d"]  # Command to sleep for a day
    '''
  } 
  }
  environment {
        GIT_CREDENTIALS = credentials('Github-PAT')
    }
  stages {
    stage("Git the package") {
      steps {
        script {
           container('jnlp'){
            sh '''
              git clone https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/MostafaT-soli/terraform_GKE.git
              git clone https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/MostafaT-soli/Ansible_session.git
              '''}
        }

      }
    }
    // stage("build the package") {
    //   steps {
    //     script {
    //        container('java') {
    //           sh './mvnw package -DskipTests '
    //         }  
    //     }

    //   }
    // }
    
    stage("build environemnt on GKE") {
      steps { 
        script {
           withCredentials([
            file(credentialsId: 'terrafrom-file', variable: 'terrafromfile'),
            file(credentialsId: 'ssh-privet-key', variable: 'sshprivetkey'),
            file(credentialsId: 'ssh-pub', variable: 'sshpub')])
            {
            container('ansible-terraform-container') {
              dir('./terraform_GKE') {
                def terraformOutput
                sh '''
                  mkdir ./key
                  cat  $terrafromfile >  ./key/crested-acrobat-430808-n2-ccb8bff2b333.json
                  echo "====================="
                  cat $sshpub > ./key/id_rsa.pub
                  terraform init
                  '''
                terraformOutput1 = sh script: 'terraform import  -input=false google_compute_instance.default projects/crested-acrobat-430808-n2/zones/us-west1-a/instances/example-instance 2>&1' ,  returnStatus: true, returnStdout: true
                terraformOutput2 = sh script:'terraform import  -input=false google_compute_firewall.allow-http  projects/crested-acrobat-430808-n2/global/firewalls/allow-http'  ,  returnStatus: true, returnStdout: true             
                echo "Terraform output: ${terraformOutput}"
                sh 'terraform plan'
                sh 'terraform apply -auto-approve'

                if (terraformOutput1 != 0 ) {
                  echo "There is no VM here lets create it "
                  sh 'terraform plan'
                  sh 'terraform apply -auto-approve'
                }
                def VM_IP = sh(script: 'terraform output -raw VM_IP', returnStdout: true)
                echo "$VM_IP"
                env.VM_IP = VM_IP
              }
              dir('./Ansible_session') {
                sh '''
                echo ${VM_IP} >> hosts.ini
                cat $sshprivetkey > key1.pam
                chmod 600 ./key1.pam
                export ANSIBLE_HOST_KEY_CHECKING=False
                ansible-playbook -i hosts.ini --private-key ./key1.pam -u tarekm_mvpengineer install_nginx.yml
                '''
              }  
            }
            
            }
         }
        }
      }
    }
    
}