pipeline {
  agent { kubernetes {
    inheritFrom 'Main'
    yaml'''
apiVersion: v1
kind: Pod
metadata:
  name: ansible-terraform-pod
spec:
  containers:
    - name: ansible-terraform-container
      image: matrek/ta:latest  # Replace with the name of your Docker image
  imagePullSecrets:
    - name: myregistrykey
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
        script {sh '''
        git clone https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/MostafaT-soli/terraform_GKE.git
        '''
        }

      }
    }
    stage("build environemnt on GKE") {
      steps { 
        script {
           withCredentials([
           file(credentialsId: 'terrafrom-file', variable: 'terrafromfile'),
           file(credentialsId: 'ssh-privet-key', variable: 'sshprivetkey'),
           file(credentialsId: 'ssh-pub', variable: 'sshpub')])
           {
          container('ansible-terraform-container') {
          sh '''
          mkdir ./key
          cat  $terrafromfile >  ./key/crested-acrobat-430808-n2-ccb8bff2b333.json
          echo "====================="
          cat $sshpub > ./key/id_rsa.pub
          terrafrom init
          terrafrom plan
          terrafrom apply -auto-approve
          '''
         }
        }
      }
    }
    
}
}
}