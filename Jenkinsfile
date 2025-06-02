pipeline {
    agent { label 'docker-host' }

    environment {
        IMAGE_NAME = "veerannadoc/argocd:${BUILD_NUMBER}"  // Dynamic Docker image tag
        GIT_USER_NAME = "veerdevops-git"
        GIT_REPO_NAME = "p4-deploy-on-k8s-with-gitops"
    }

    stages {
        stage('🔧 Checkout Code') {
            steps {
                echo '🔧 Checkout stage started'
                git url: "https://github.com/${env.GIT_USER_NAME}/${env.GIT_REPO_NAME}.git", branch: 'main'
                echo '✅ Checkout complete'
            }
        }

        stage('📦 Build Project') {
            steps {
                echo '📦 Build stage started'
                sh 'mvn clean package'
                echo '✅ Build complete'
            }
        }

        stage('🧪 Run Tests') {
            steps {
                echo '🧪 Test stage started'
                sh 'mvn test'
                echo '✅ Tests passed'
            }
        }

        stage('🐳 Build Docker Image') {
            steps {
                script {
                    echo '🐳 Docker build stage started'
                    def artifactName = sh(script: "ls target/*.jar | head -1", returnStdout: true).trim()
                    echo "📦 Using artifact: ${artifactName}"
                    sh "docker build --build-arg artifact=${artifactName} -t ${IMAGE_NAME} ."
                    echo "✅ Docker image built with tag ${IMAGE_NAME}"
                }
            }
        }

        stage('☁️ Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    echo '☁️ Docker push stage started'
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh "docker push ${IMAGE_NAME}"
                    sh 'docker logout'
                    echo "✅ Docker image pushed: ${IMAGE_NAME}"
                }
            }
        }

        stage('✏️ Update Deployment File') {
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    echo '✏️ Updating deployment manifest with new image tag'
                    sh '''
                        rm -rf deployment_repo
                        git clone https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git deployment_repo
                        cd deployment_repo
                        git config user.email "example.com"
                        git config user.name "${GIT_USER_NAME}"
                        sed -i "s|veerannadoc/argocd:.*|${IMAGE_NAME}|g" manifest_file/deployment.yml
                        git add manifest_file/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git
                    '''
                    echo '✅ Deployment manifest updated and pushed to GitHub'
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed!'
        }
        success {
            echo '🚀 Pipeline completed successfully!'
        }
    }
}
