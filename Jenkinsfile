// prerequisites: a nodejs app must be deployed inside a kubernetes cluster
//                repo must include the Dockerfile
//                docker pipeline must be installed on Jenkins
//                docker credentials must be created on Jenkins
// TODO: look for all instances of [] and replace all instances of 
//       '[VARIABLE]' with actual values 
//        e.g https://github.com/techiemad2/events-app-api-server.git might become https://github.com/MyName/external.git
// Look for the following variables in this file and replace them:
//      docker  //id of your global docker credentials in Jenkins https://www.google.com/search?q=add+docker+credentials+Jenkins&oq=add+docker+credentials+Jenkins
//      https://github.com/techiemad2/events-app-api-server.git
//      techiemad2
//      api-server-image
//      cnd-events-cluster 
//      us-east-1  
//      user19
//      demo-api - Kubernetes deployment name- This is found in the kubernetes deployment.yaml
//      demo-api - Name of the container to be replaced - in the template/spec section of the deployment.yaml


pipeline {
    agent any 
   environment {
        registryCredential = 'docker'
        imageName = 'techiemad2/api-server-image'
        dockerImage = ''
        }
    stages {
        stage('Run the tests') {
             agent {
                docker { 
                    image 'node:14-alpine'
                    args '-e HOME=/tmp -e NPM_CONFIG_PREFIX=/tmp/.npm'
                    reuseNode true
                }
            }
            steps {
                echo 'Retrieving source from github' 
                git branch: 'main',
                    url: 'https://github.com/techiemad2/events-app-api-server.git'
                echo 'Did we get the source?' 
                sh 'ls -a'
                echo 'install dependencies' 
                sh 'npm install'
                echo 'Run tests'
                sh 'npm test'
                echo 'Tests passed on to build and deploy Docker container'
            }
        }
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build imageName
                }
            }
            }
            stage('Deploy Image') {
            steps{
                script {
                docker.withRegistry( '', registryCredential ) {
                    dockerImage.push("$BUILD_NUMBER")
                }
                }
            }
        }     
         stage('get k8s credentials') {
             agent {
                docker { 
                    image 'coursedemos/awscli-kubectl:v1.0'
                    reuseNode true
                    }
                    }
            steps {
                sh "rm -rf ${WORKSPACE}/.kube/"
                sh "mkdir ${WORKSPACE}/.kube/"
                echo 'Get cluster credentials'
                sh 'aws eks --region us-east-1 update-kubeconfig --kubeconfig ${WORKSPACE}/.kube/config --name cnd-events-cluster'
            }
        }     
         stage('update k8s') {
             agent {
                docker { 
                    image 'coursedemos/awscli-kubectl:v1.0'
                    reuseNode true
                        }
                    }
            steps {
                echo 'Set the image'
                     sh "kubectl -n user19 set image deployment/demo-api demo-api=${env.imageName}:${env.BUILD_NUMBER} --kubeconfig=${WORKSPACE}/.kube/config"
            }
        }     
        stage('Remove local docker image') {
            steps{
                sh "docker rmi $imageName:latest"
                sh "docker rmi $imageName:$BUILD_NUMBER"
            }
        }
    }
}