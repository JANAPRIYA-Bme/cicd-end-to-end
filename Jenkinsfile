pipeline {

    agent any

    environment {
        IMAGE_NAME = "cicd-e2e"
        DOCKER_REGISTRY = "Manijana123"     // Your Docker Hub username
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670',
                    url: 'https://github.com/JANAPRIYA-Bme/cicd-end-to-end.git'
            }
        }

        stage('Build Docker') {
            steps {
                sh '''
                echo "Building Docker Image"
                docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push the artifacts') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}

                    docker logout
                    '''
                }
            }
        }

        stage('Checkout K8S manifest SCM') {
            steps {
                dir('k8s-manifests') {
                    git branch: 'main',
                        credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670',
                        url: 'https://github.com/JANAPRIYA-Bme/cicd-demo-manifests-repo.git'
                }
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                dir('k8s-manifests') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'f87a34a8-0e09-45e7-b9cf-6dc68feac670',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {
                        sh '''
                        cat deploy.yaml

                        sed -i "s/[0-9]\\+/${BUILD_NUMBER}/g" deploy.yaml

                        cat deploy.yaml

                        git config user.name "Jenkins"
                        git config user.email "jenkins@example.com"

                        git add deploy.yaml
                        git commit -m "Updated deploy.yaml | Build #${BUILD_NUMBER}" || echo "No changes to commit"

                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/JANAPRIYA-Bme/cicd-demo-manifests-repo.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
