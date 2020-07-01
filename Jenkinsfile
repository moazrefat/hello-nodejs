pipeline {
    agent any
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("moazrefat/hello:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }   
        stage("update image") {
            steps {  
                 sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' k8s/envs/*/values.yaml" 
         }      
        }   
        
        stage('Deploy to kubernetes cluster on Dev environment') {
            steps {          
                 withKubeConfig([credentialsId: 'kubernetes-secret-id']) {
                    sh '''
                        helm install --generate-name  --debug ./k8s --set environment=dev -f k8s/envs/dev/values.yaml
                '''
            }
        }

        }
        stage('Deploy to kubernetes cluster on Prod environment') {
            steps {  
                script {
                    env.RELEASE = input message: 'User input required', ok: 'Release!',
                            parameters: [choice(name: 'RELEASE', choices: 'release\nstop', description: 'Do you want to proceed releasing on prodcution ?')]
                    echo "${env.RELEASE}"
                    if (env.RELEASE =="release"){        
                        withKubeConfig([credentialsId: 'kubernetes-secret-id']) {
                            sh '''
                                helm install --generate-name  --debug ./k8s --set environment=prod -f k8s/envs/prod/values.yaml
                            '''
                              }
                         }
                      }

                  }
      }    
    }
}