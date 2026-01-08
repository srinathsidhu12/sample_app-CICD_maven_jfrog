pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "srinathsidhu12/java_spring_boot_sample_app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        K8S_DEPLOYMENT_NAME = "springboot-demo"
        JFROG_USER  = credentials('Jfrog_credentials').username
        JFROG_TOKEN = credentials('Jfrog_credentials').password
    }
    tools {
       jdk 'JDK-21'
       maven 'Maven-3.9.11'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/srinathsidhu12/sample_app-CICD_maven_jfrog.git'	
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Maven Deploy') {
            steps {
                 withMaven(mavenSettingsConfig: 'jfrog-maven-settings') {                          //jfrog-maven-settings = Managed Maven settings.xml
                  sh 'mvn deploy -DskipTests'
                 }
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
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                     sed -i "s|image:.*|image: ${DOCKER_HUB_REPO}:${IMAGE_TAG}|" ./k8s/k8s-deployment.yaml
                     kubectl apply -f ./k8s/k8s-deployment.yaml
                     kubectl apply -f ./k8s/k8s-service.yaml
                     kubectl rollout status deployment/${K8S_DEPLOYMENT_NAME}
                    """
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

