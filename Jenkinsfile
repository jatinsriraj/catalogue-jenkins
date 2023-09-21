pipeline {
  agent any
  parameters{
        string(name: 'VERSION', description: 'Enter the APP VERSION')
    }
environment{
        AWS_ACCOUNT_ID="936411044994"
        REGION="ap-south-1"
        REPO_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/catalogue"
    }
  stages {
    stage('Clone') {
        steps{
        script{
                    echo "Clone started"
                    gitInfo = checkout scm
                            
        }
      }
    }

    stage('Docker build'){
            steps{
                script{                  
                        sh """
                         docker build -t catalogue:${VERSION} .
                        """
                }
            }
        }

    stage('Image push to ECR'){
            steps{
                script{
                    withAWS(credentials: 'aws-auth', region: "${REGION}") {
                        sh """
                            aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 936411044994.dkr.ecr.ap-south-1.amazonaws.com
                        docker build -t cata-1 .
                        docker tag cata-1:latest 936411044994.dkr.ecr.ap-south-1.amazonaws.com/cata-1:latest
                        docker push 936411044994.dkr.ecr.ap-south-1.amazonaws.com/cata-1:latest
                        """
                    }
                }
            }
        }
        
    stage('Image push to Docker Hub'){
            steps{
                script{
                        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDENTIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    
                        sh """
                        docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                        docker tag catalogue:${VERSION}  techworldwithsiva/catalogue:${VERSION}
                        docker push techworldwithsiva/catalogue:${VERSION}
                        """
                    
                }
                }
            }
        }
    stage('Deployment'){
            steps{
                script{
                    withAWS(credentials: 'aws-auth', region: "${REGION}") {
                        sh """
                        aws eks update-kubeconfig --region ${REGION} --name eks-cluster
                        cd helm
                        helm install catalogue . --set deployment.imageVersion=${VERSION}
                        """
                    }
                }
            }
        }
  

}
}
