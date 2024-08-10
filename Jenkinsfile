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
        script {
           container('jnlp'){
            sh '''
              git clone https://${GIT_CREDENTIALS_USR}:${GIT_CREDENTIALS_PSW}@github.com/MostafaT-soli/terraform_GKE.git
              '''}
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
          container('ansible-terraform-container') {dir('./terraform_GKE'){
            def terraformOutput
            try{
              sh '''
                mkdir ./key
                cat  $terrafromfile >  ./key/crested-acrobat-430808-n2-ccb8bff2b333.json
                echo "====================="
                cat $sshpub > ./key/id_rsa.pub
                terraform init
                
                '''
              terraformOutput = sh script: 'terraform import  -input=false google_compute_instance.default projects/crested-acrobat-430808-n2/zones/us-west1-a/instances/example-instance 2>&1' ,  returnStatus: true, returnStdout: true
              echo "Terraform output: ${terraformOutput}"
                // terraform import  -input=false google_compute_instance.default projects/crested-acrobat-430808-n2/zones/us-west1-a/instances/example-instance
                // terraform plan
                // terraform apply -auto-approve
          }
            catch (Exception e) {
                if (e.getMessage().contains("Error: Cannot import non-existent remote object")) {
                          
                            echo "Caught a specific error message: " + e.getMessage()
                            echo "This is Normal"
                            // Continue the pipeline
                        } else {
                            // Handle any other exceptions
                            echo "----------------------------------"
                            echo "Terraform output: ${terraformOutput}"
                            echo "An exception occurred while changing the directory: " + e.getMessage()
                            error "Pipeline failed due to an exception"
                        }
                    } 
            finally {
                // Perform cleanup or finalization steps
                echo "Finally block executed"
            }
          }
         }
        }
      }
    }
    
}
}
}