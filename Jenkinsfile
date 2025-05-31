pipeline {
    agent { label 'docker-host' }  // Run pipeline on node labeled 'docker-host'

    stages {
        stage('🔧 Checkout Code') {
            steps {
                echo '🔧 Checkout stage started'
                git url: 'https://github.com/veerdevops-git/p4-deploy-on-k8s-with-gitops.git', branch: 'main'
                // Pull code from GitHub repo main branch
                echo '✅ Checkout complete'
            }
        }

        stage('📦 Build Project') {
            steps {
                echo '📦 Build stage started'
                sh 'mvn clean package'
                // Build the project using Maven, creating a JAR file
                echo '✅ Build complete'
            }
        }

        stage('🧪 Run Tests') {
            steps {
                echo '🧪 Test stage started'
                sh 'mvn test'
                // Run unit tests to verify application correctness
                echo '✅ Tests passed'
            }
        }

        stage('🐳 Build Docker Image') {
            steps {
                script {
                    echo '🐳 Docker build stage started'
                    def artifactName = sh(script: "ls target/*.jar | head -1", returnStdout: true).trim()
                    // Identify the built JAR file for Docker build

                    echo "📦 Using artifact: ${artifactName}"
                    sh "docker build --build-arg artifact=${artifactName} -t veerannadoc/argocd:6 ."
                    // Build Docker image tagged veerannadoc/argocd:6 with the artifact
                    echo '✅ Docker image built'
                }
            }
        }

        stage('☁️ Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    echo '☁️ Docker push stage started'
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
                    // Log in securely to DockerHub using credentials
                    sh 'docker push veerannadoc/argocd:6'
                    // Push built image to DockerHub registry
                    sh 'docker logout'
                    // Log out from DockerHub after push
                    echo '✅ Docker image pushed to DockerHub'
                }
            }
        }

        stage('✏️ Update Deployment File') {
            environment {
                GIT_USER_NAME = "veerdevops-git"
                GIT_REPO_NAME = "p4-deploy-on-k8s-with-gitops"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    echo '✏️ Updating deployment manifest with new image tag'
                    sh '''
                        # Clean clone deployment repo to fresh folder
                        if [ -d deployment_repo ]; then
                          rm -rf deployment_repo
                        fi
                        git clone https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git deployment_repo
                        cd deployment_repo
                        git config user.email "example.com"
                        git config user.name "${GIT_USER_NAME}"
                        # Replace placeholder 'replaceImageTag' with the current build number
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" manifest_file/deployment.yml
                        git add manifest_file/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push origin main
                    '''
                    echo '✅ Deployment manifest updated and pushed to GitHub'
                }
            }
        }

        stage('🚀 ArgoCD Sync') {
            steps {
                echo '🚀 Triggering ArgoCD application sync'
                // Assuming argocd CLI is installed and credentials are configured securely
                sh '''
                    argocd login <ARGOCD_SERVER> --username <ARGOCD_USERNAME> --password <ARGOCD_PASSWORD> --insecure
                    argocd app sync <ARGOCD_APP_NAME>
                    argocd logout
                '''
                echo '✅ ArgoCD sync triggered successfully'
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
