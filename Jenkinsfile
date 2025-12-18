pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "srinathsidhu12/java_spring_boot_sample_app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_DEPLOYMENT_NAME = "springboot-demo"
    }
    tools {
       jdk 'JDK-21'
       maven 'Maven-3.9.11'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/srinathsidhu12/Jave_spring_boot_sample_app.git'	
            }
        }
        stage('Check Java') {
            steps {
               sh 'java -version'
               sh 'javac -version'
               sh 'echo $JAVA_HOME'
            }  
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                  docker build -t ${DOCKER_HUB_REPO}:${IMAGE_TAG} .
                """
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-id',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                      docker push ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                  withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                     // Update the image in the Kubernetes manifest
                    sh """
                     #use token-based authentication
                     git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/srinathsidhu12/Java_spring_boot_sample_app.git
                    
                     sed -i "s|image:.*|image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}|" ./k8s/k8s-deployment.yaml
                     git add ./k8s/k8s-deployment.yaml
                     git commit -m "Update deployment image to ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
                     git push origin master
                    """
                  }
                    // Apply the updated manifest to the cluster
                  withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                     kubectl apply -f ./k8s/k8s-deployment.yaml
                     kubectl apply -f ./k8s/k8s-service.yaml
                     kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME}
                    """
                  }
             }
         }   
       } 
    }

    post {
        success {
            echo " Successfully deployed image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}

