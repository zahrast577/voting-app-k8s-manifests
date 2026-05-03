pipeline {
    agent any
    environment {
        DOCKERHUB_USER = 'zahrast577'
        GITHUB_USER    = 'zahrast577'
        DOCKER_HOST = 'unix:///var/run/docker.sock'
        PATH = "/usr/bin:/usr/local/bin:/bin:/usr/sbin:/sbin:${env.PATH}"
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
                sh """
                    export PATH=/usr/bin:/usr/local/bin:\$PATH
                    /usr/bin/docker build -t ${DOCKERHUB_USER}/vote:${BUILD_NUMBER} ./vote
                    /usr/bin/docker build -t ${DOCKERHUB_USER}/result:${BUILD_NUMBER} ./result
                    /usr/bin/docker build -t ${DOCKERHUB_USER}/worker:${BUILD_NUMBER} ./worker
                """
            }
        }
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        export PATH=/usr/bin:/usr/local/bin:\$PATH
                        echo \$DOCKER_PASS | /usr/bin/docker login -u \$DOCKER_USER --password-stdin
                        /usr/bin/docker push ${DOCKERHUB_USER}/vote:${BUILD_NUMBER}
                        /usr/bin/docker push ${DOCKERHUB_USER}/result:${BUILD_NUMBER}
                        /usr/bin/docker push ${DOCKERHUB_USER}/worker:${BUILD_NUMBER}
                        /usr/bin/docker tag ${DOCKERHUB_USER}/vote:${BUILD_NUMBER} ${DOCKERHUB_USER}/vote:latest
                        /usr/bin/docker tag ${DOCKERHUB_USER}/result:${BUILD_NUMBER} ${DOCKERHUB_USER}/result:latest
                        /usr/bin/docker tag ${DOCKERHUB_USER}/worker:${BUILD_NUMBER} ${DOCKERHUB_USER}/worker:latest
                        /usr/bin/docker push ${DOCKERHUB_USER}/vote:latest
                        /usr/bin/docker push ${DOCKERHUB_USER}/result:latest
                        /usr/bin/docker push ${DOCKERHUB_USER}/worker:latest
                    """
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
                    sh """
                        export PATH=/usr/bin:/usr/local/bin:\$PATH
                        git config --global user.email "jenkins@ci.com"
                        git config --global user.name "Jenkins CI"
                        sed -i 's|image: ${DOCKERHUB_USER}/vote:.*|image: ${DOCKERHUB_USER}/vote:${BUILD_NUMBER}|g' vote/deployment.yaml
                        sed -i 's|image: ${DOCKERHUB_USER}/result:.*|image: ${DOCKERHUB_USER}/result:${BUILD_NUMBER}|g' result/deployment.yaml
                        sed -i 's|image: ${DOCKERHUB_USER}/worker:.*|image: ${DOCKERHUB_USER}/worker:${BUILD_NUMBER}|g' worker/deployment.yaml
                        git add .
                        git commit -m "CI: update images to build ${BUILD_NUMBER} [skip ci]"
                        git push https://\${GIT_USER}:\${GIT_PASS}@github.com/${GITHUB_USER}/voting-app-k8s-manifests.git main
                    """
                }
            }
        }
    }
    post {
        success { echo 'Pipeline reussi !' }
        failure { echo 'Pipeline echoue.' }
        always {
            sh """
                export PATH=/usr/bin:/usr/local/bin:\$PATH
                /usr/bin/docker logout || true
            """
        }
    }
}