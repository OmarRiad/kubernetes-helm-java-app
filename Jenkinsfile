#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        DOCKER_REPO_SERVER = '662930028566.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
        IMAGE_NAME = "1.0-${BUILD_NUMBER}"
        CLUSTER_REGION = "us-east-1"
    }
    stages {
        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh './gradlew clean build'
                }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]){
                        sh "docker build -t ${DOCKER_REPO}:${IMAGE_NAME} ."
                        sh "aws ecr get-login-password --region ${CLUSTER_REGION} | docker login --username AWS --password-stdin ${DOCKER_REPO_SERVER}"
                        sh "docker push ${DOCKER_REPO}:${IMAGE_NAME}"
                    }
                }
            }
        }
        stage('deploy') {
            environment{
                APP_NAME = 'java-app'
                APP_NAMESPACE = 'my-app'
                DB_USER  = credentials('db_user')
                DB_PASS  = credentials('db_pass')
                DB_NAME  = credentials('db_name')
                DB_ROOT  = credentials('db_root_pass')
            }
            steps {
                script {
                    echo 'Preparing values file...'
                    sh """
                    envsubst < ./HelmChart/values-override.yaml > ./HelmChart/rendered-values.yaml
                    """
                   echo 'Deploying with Helm...'
                    sh """
                    /var/jenkins_home/bin/helm upgrade --install my-app ./HelmChart -f ./HelmChart/rendered-values.yaml
                    """
                }
            }
        }
    }
}
