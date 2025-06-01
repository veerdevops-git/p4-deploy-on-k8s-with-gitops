pipeline {
    agent { label 'docker-host' }  // Use Jenkins agent/node labeled 'docker-host'

    stages {
        stage('🔧 Checkout Code') {
            steps {
                echo '🔧 Checkout stage started'  // Log the start of checkout
                git url: 'https://github.com/veerdevops-git/p4-deploy-on-k8s-with-gitops.git', branch: 'main'  // Clone GitHub repo from 'main' branch
                echo '✅ Checkout complete'  // Log successful checkout
            }
        }

        stage('📦 Build Project') {
            steps {
                echo '📦 Build stage started'  // Log the start of build
                sh 'mvn clean package'  // Use Maven to clean and build the project, generates JAR file
                echo '✅ Build complete'  // Log build success
            }
        }

        stage('🧪 Run Tests') {
            steps {
                echo '🧪 Test stage started'  // Log start of test phase
                sh 'mvn test'  // Run Maven unit tests
                echo '✅ Tests passed'  // Log test success
            }
        }

        stage('🐳 Build Docker Image') {
            steps {
                script {
                    echo '🐳 Docker build stage started'  // Log start of Docker build
                    def artifactName = sh(script: "ls target/*.jar | head -1", returnStdout: true).trim()  // Get path to first JAR in target/
                    echo "📦 Using artifact: ${artifactName}"  // Log the artifact being used
                    sh "docker build --build-arg artifact=${artifactName} -t veerannadoc/argocd:6 ."  // Build Docker image with artifact, tag it
                    echo '✅ Docker image built'  // Log Docker build success
                }
            }
        }

        stage('☁️ Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    echo '☁️ Docker push stage started'  // Log push start
                    sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'  // Login to DockerHub securely
                    sh 'docker push veerannadoc/argocd:6'  // Push the built Docker image to DockerHub
                    sh 'docker logout'  // Logout from DockerHub
                    echo '✅ Docker image pushed to DockerHub'  // Log success
                }
            }
        }

        stage('✏️ Update Deployment File') {
            environment {
                GIT_USER_NAME = "veerdevops-git"  // GitHub username
                GIT_REPO_NAME = "p4-deploy-on-k8s-with-gitops"  // Deployment GitHub repo name
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    echo '✏️ Updating deployment manifest with new image tag'  // Log manifest update start
                    sh '''
                        if [ -d deployment_repo ]; then
                            rm -rf deployment_repo
                        fi

                        git clone https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git deployment_repo

                        cd deployment_repo

                        git config user.email "example.com"
                        git config user.name "${GIT_USER_NAME}"

                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" manifest_file/deployment.yml

                        git add manifest_file/deployment.yml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"

                        git push origin main
                    '''
                    echo '✅ Deployment manifest updated and pushed to GitHub'  // Log success
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Pipeline failed!'  // Log failure if any stage fails
        }
        success {
            echo '🚀 Pipeline completed successfully!'  // Log success if all stages pass
        }
    }
}
