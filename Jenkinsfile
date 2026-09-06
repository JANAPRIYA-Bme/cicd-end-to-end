pipeline {

    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    credentialsId: 'github-creds',
                    url: 'https://github.com/JANAPRIYA-Bme/cicd-end-to-end',
                    branch: 'main'
                )
            }
        }

        stage('Build Docker') {
            steps {
                script {
                    sh '''
                        echo 'Build Docker Image'
                        docker build -t manijana123/cicd-e2e:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Push the artifacts') {
            steps {
                script {
                    echo "Push to Docker Hub"

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            usernameVariable: 'DOCKER_USERNAME',
                            passwordVariable: 'DOCKER_PASSWORD'
                        )
                    ]) {
                        sh '''
                            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                            docker push manijana123/cicd-e2e:${BUILD_NUMBER}
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Checkout K8S manifest SCM') {
            steps {
                git(
                    credentialsId: 'github-creds',
                    url: 'https://github.com/JANAPRIYA-Bme/cicd-demo-manifests-repo.git',
                    branch: 'main'
                )
            }
        }

        stage('Update K8S manifest & push to Repo') {
            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            usernameVariable: 'GIT_USERNAME',
                            passwordVariable: 'GIT_PASSWORD'
                        )
                    ]) {

                        sh '''
                            echo "Before update:"
                            cat deploy.yaml

                            sed -i "s/32/${BUILD_NUMBER}/g" deploy.yaml

                            echo "After update:"
                            cat deploy.yaml

                            git config user.name "$GIT_USERNAME"
                            git config user.email "$GIT_USERNAME@users.noreply.github.com"

                            git add deploy.yaml
                            git commit -m "Updated deploy yaml | Jenkins Pipeline" || true

                            git push https://$GIT_USERNAME:$GIT_PASSWORD@github.com/JANAPRIYA-Bme/cicd-demo-manifests-repo.git HEAD:main
                        '''
                    }
                }
            }
        }
    }
}
