pipeline {
    agent any
    environment {
        DOCKERHUB_USER = 'zahrast577'
        GITHUB_USER    = 'zahrast577'
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Recuperation du code source...'
                checkout scm
            }
        }
        stage('Build Images') {
            steps {
                echo 'Build des images Docker...'
                sh "docker build -t ${DOCKERHUB_USER}/vote:${BUILD_NUMBER} ./vote"
                sh "docker build -t ${DOCKERHUB_USER}/result:${BUILD_NUMBER} ./result"
                sh "docker build -t ${DOCKERHUB_USER}/worker:${BUILD_NUMBER} ./worker"
            }
        }
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${DOCKERHUB_USER}/vote:${BUILD_NUMBER}"
                    sh "docker push ${DOCKERHUB_USER}/result:${BUILD_NUMBER}"
                    sh "docker push ${DOCKERHUB_USER}/worker:${BUILD_NUMBER}"
                    sh "docker tag ${DOCKERHUB_USER}/vote:${BUILD_NUMBER} ${DOCKERHUB_USER}/vote:latest"
                    sh "docker tag ${DOCKERHUB_USER}/result:${BUILD_NUMBER} ${DOCKERHUB_USER}/result:latest"
                    sh "docker tag ${DOCKERHUB_USER}/worker:${BUILD_NUMBER} ${DOCKERHUB_USER}/worker:latest"
                    sh "docker push ${DOCKERHUB_USER}/vote:latest"
                    sh "docker push ${DOCKERHUB_USER}/result:latest"
                    sh "docker push ${DOCKERHUB_USER}/worker:latest"
                }
            }
        }
        stage('Update K8s Manifests') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-credentials',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh 'git config --global user.email "jenkins@ci.com"'
                    sh 'git config --global user.name "Jenkins CI"'
                    sh "sed -i 's|image: ${DOCKERHUB_USER}/vote:.*|image: ${DOCKERHUB_USER}/vote:${BUILD_NUMBER}|g' vote/deployment.yaml"
                    sh "sed -i 's|image: ${DOCKERHUB_USER}/result:.*|image: ${DOCKERHUB_USER}/result:${BUILD_NUMBER}|g' result/deployment.yaml"
                    sh "sed -i 's|image: ${DOCKERHUB_USER}/worker:.*|image: ${DOCKERHUB_USER}/worker:${BUILD_NUMBER}|g' worker/deployment.yaml"
                    sh 'git add .'
                    sh "git commit -m 'CI: update images to build ${BUILD_NUMBER} [skip ci]'"
                    sh "git push https://\${GIT_USER}:\${GIT_PASS}@github.com/${GITHUB_USER}/voting-app-k8s-manifests.git main"
                }
            }
        }
    }
    post {
        success { echo 'Pipeline reussi !' }
        failure { echo 'Pipeline echoue.' }
        always  { sh 'docker logout || true' }
    }
}