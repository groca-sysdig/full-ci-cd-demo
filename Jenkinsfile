pipeline {
    agent any
    environment {
        DOCKER_IMAGE_NAME = "grocamador/cicd-demo"
        DOCKERHUB_CREDENTIALS= credentials('dockerhubcredentials')
        }
    
stages {

/*        stage('Test') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
               archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
*/


    stage('Build Docker Image') {
            when {
                branch 'master'
            }
        steps {
                echo 'Building docker image'
 //               sh "aws eks --region us-east-1 update-kubeconfig --name groca-eks"
                sh "kubectl config get-contexts"
                sh "whoami"
                sh "kubectl get pods -A"
                sh "docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ."
            }
        }

      stage('Scanning Image with Sysdig') {
        steps {
            
            sh "echo grocamador/demo-scan:${env.BUILD_NUMBER} > sysdig_secure_images"
            sricpt {
            try {
            sysdig engineCredentialsId: 'sysdig-secure-api-credentials', name: 'sysdig_secure_images', inlineScanning: true
            }
            catch (Exception e) {
                            input "Sysdig Vulnerability scanner showed some security issues, Are you sure you want to continue?"  
                        }
                    }
        }
       } 

    stage('Push Docker Image') {
        when {
            branch 'master'
        }
        steps {

                echo "Login in docker registry"
                sh "docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW"                		
	            echo 'Login Completed' 
                echo "Pushing docker image with current build tag"
                sh " docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                echo 'Pushing docker image with tag latest'
                sh "docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest"
                sh "docker push ${DOCKER_IMAGE_NAME}:latest"
            }
        }
            


    stage('Deploy to stage') {
        when {
            branch 'master'
        }
            steps {
            echo "Getting existing pods"
            sh "kubectl get pods -A"
            echo "Getting nodes"
            sh "kubectl get nodes"
            sh ("""     
                  kubectl delete -f train-schedule-kube-stage.yml
                  kubectl apply -f train-schedule-kube-stage.yml
                """)
 
            }
        }
        
       stage("Deploy to Production"){
        when {
                branch 'master'
            }
             steps {              
                input 'Deploy to Production?'
                milestone(1)   
              sh ("""                
                  echo \$KUBECONFIG
                  kubectl delete -f train-schedule-kube.yml
                  kubectl apply -f train-schedule-kube.yml
                """)
                
             }
         }
    }
}
