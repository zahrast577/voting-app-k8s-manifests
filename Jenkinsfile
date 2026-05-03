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
                sh '''#!/bin/bash
                    /usr/bin/docker build -t zahrast577/vote:${BUILD_NUMBER} ./vote
                    /usr/bin/docker build -t zahrast577/result:${BUILD_NUMBER} ./result
                    /usr/bin/docker build -t zahrast577/worker:${BUILD_NUMBER} ./worker
                '''
            }
        }
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''#!/bin/bash
                        echo $DOCKER_PASS | /usr/bin/docker login -u $DOCKER_USER --password-stdin
                        /usr/bin/docker push zahrast577/vote:${BUILD_NUMBER}
                        /usr/bin/docker push zahrast577/result:${BUILD_NUMBER}
                        /usr/bin/docker push zahrast577/worker:${BUILD_NUMBER}
                        /usr/bin/docker tag zahrast577/vote:${BUILD_NUMBER} zahrast577/vote:latest
                        /usr/bin/docker tag zahrast577/result:${BUILD_NUMBER} zahrast577/result:latest
                        /usr/bin/docker tag zahrast577/worker:${BUILD_NUMBER} zahrast577/worker:latest
                        /usr/bin/docker push zahrast577/vote:latest
                        /usr/bin/docker push zahrast577/result:latest
                        /usr/bin/docker push zahrast577/worker:latest
                    '''
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
                    sh '''#!/bin/bash
                        git config --global user.email "jenkins@ci.com"
                        git config --global user.name "Jenkins CI"
                        sed -i "s|image: zahrast577/vote:.*|image: zahrast577/vote:${BUILD_NUMBER}|g" vote/deployment.yaml
                        sed -i "s|image: zahrast577/result:.*|image: zahrast577/result:${BUILD_NUMBER}|g" result/deployment.yaml
                        sed -i "s|image: zahrast577/worker:.*|image: zahrast577/worker:${BUILD_NUMBER}|g" worker/deployment.yaml
                        git add .
                        git commit -m "CI: update images to build ${BUILD_NUMBER} [skip ci]"
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/zahrast577/voting-app-k8s-manifests.git main
                    '''
                }
            }
        }
    }
    post {
        success { echo 'Pipeline reussi !' }
        failure { echo 'Pipeline echoue.' }
        always  { sh '#!/bin/bash\n/usr/bin/docker logout || true' }
    }
}